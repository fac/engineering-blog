---
:title: External authentication scripts in OpenVPN the right way
:created_at: 2017-05-22 11:55:11.000000000 +00:00
:kind: article
:tags:
- openvpn
:author: caius
---

<% excerpt do %>
OpenVPN is a wonderfully flexible piece of software in anyone's toolkit, but recently we found a sharp edge that wasn't the most obvious thing to work around.
<% end %>

After spinning up a new VPN server we wanted to add username/password authentication against an external source. Looking at the OpenVPN documentation, the `--auth-user-pass-verify <script>` flag provides this functionality. Writing the script for this was easy enough — read the credentials from a temporary file OpenVPN hands us and exit with the appropriate exit code depending on whether authentication should pass or fail.

Shortly after letting users loose on it we noticed that the VPN kept reconnecting for some people, and occasionally packet loss was being observed over the VPN specifically. Looking into it, we discovered that the OpenVPN process was blocked and not forwarding any traffic during the time our authentication script took to execute.

As soon as the script finished, OpenVPN behaved as expected. Given our script was talking to a service on another node, our runtime wasn't super quick so people would generally notice a few seconds of seconds of packet loss every time someone authenticated.

    64 bytes from 192.168.2.1: icmp_seq=24 ttl=251 time=21.535 ms
    # Another user starts authentication to the VPN
    Request timeout for icmp_seq 25
    Request timeout for icmp_seq 26
    Request timeout for icmp_seq 27
    Request timeout for icmp_seq 28
    Request timeout for icmp_seq 29
    # Authentication script completes, and OpenVPN process plays catchup
    64 bytes from 192.168.2.1: icmp_seq=25 ttl=251 time=5673.376 ms
    64 bytes from 192.168.2.1: icmp_seq=26 ttl=251 time=4670.284 ms
    64 bytes from 192.168.2.1: icmp_seq=27 ttl=251 time=3667.352 ms
    64 bytes from 192.168.2.1: icmp_seq=28 ttl=251 time=2662.066 ms
    64 bytes from 192.168.2.1: icmp_seq=29 ttl=251 time=1661.908 ms
    64 bytes from 192.168.2.1: icmp_seq=30 ttl=251 time=660.354 ms
    64 bytes from 192.168.2.1: icmp_seq=31 ttl=251 time=22.217 ms
    64 bytes from 192.168.2.1: icmp_seq=32 ttl=251 time=20.984 ms
    64 bytes from 192.168.2.1: icmp_seq=33 ttl=251 time=21.381 ms

As per normal for an open source project, we promptly grabbed the [OpenVPN source code][ovpn source] and looked to see how/where the auth user pass verify script was called. After digging down through the call tree, we end up in [`openvpn_execve`][execve] which handles calling external commands. This is intended to behave like `system()` and therefore blocks the process waiting for the external command to return. This works quite well when the auth user pass script is doing something quickly on the local node, but quickly becomes a blocker (pun intended) when the network is involved.

[ovpn source]: https://github.com/openvpn/openvpn
[execve]: https://github.com/openvpn/openvpn/tree/v2.3.6/src/openvpn/misc.c#L286-292

Thinking there must be a better solution to this than "suck it up", after some quick searching we turned up [this bug from five years ago][ovpn bug] which mentions a C plugin API. After some more looking, we eventually ran across the [IPTables][iptables] & [Okta][okta] OpenVPN plugins, both of which use the C API from a shared library to interact with external scripts without blocking. With this in mind, we went away and wrote a more generic plugin that just calls an external script, much like `--auth-user-pass-verify` - but importantly does so without blocking the main process whilst the script runs.

[ovpn bug]: https://community.openvpn.net/openvpn/ticket/222
[okta]: https://github.com/okta/okta-openvpn
[iptables]: https://github.com/viyh/openvpn-iptables

The API is documented in [`openvpn-plugin.h`][openvpn-plugin.h], and really only needs three methods implementing to have a working plugin. These are:

[openvpn-plugin.h]: https://github.com/openvpn/openvpn/tree/v2.3.6/include/openvpn-plugin.h

* `openvpn_plugin_open_v3`

  Needs to amend the `type_mask` to tell OpenVPN what API calls the plugin wishes to hook into. In our case we just hook into `OPENVPN_PLUGIN_AUTH_USER_PASS_VERIFY`. Also needs to define `retptr->handle`, which is a struct the plugin can store any context it needs in. (We store the pointer to the OpenVPN logger & path of our external script.)

* `openvpn_plugin_func_v3`

  This is invoked to handle all the API hooks you've asked OpenVPN to let you intercept. In our case this is where we tell OpenVPN we're going to defer authentication handling, and write the authentication result to the `auth_control_file` when we've worked out if the attempt can proceed or not. This is also where we fork our child process to exec the external script without blocking the main process.

* `openvpn_plugin_close_v1`

  Called before OpenVPN exits or reloads the plugin, intended for the plugin to tidyup any resources it may have.

We also defined `openvpn_plugin_min_version_required_v1` to make sure the v3 API was available for the plugin to use. Our generic [auth-script-openvpn][] plugin is available [on github][auth-script-openvpn].

[auth-script-openvpn]: https://github.com/fac/auth-script-openvpn

The script we had needed amending slightly too — instead of signifying success/failure of the authentication attempt via exit codes, we were now using the deferred authentication mechanism. When the script is called, OpenVPN generates a temporary file and passes the path of it along. The script is then expected to write a single character to the file to signify whether the authentication succeeds or fails. (OpenVPN watches the file for changes in the meantime, until `hand-window` seconds passes at which point the authentication attempt is considered a failure.)

As an example, let’s pretend we wanted to authenticate anyone called dave to our VPN, we could have a script like this to do the heavy lifting for us:

    #!/usr/bin/env ruby

    if ENV["username"] =~ /dave/i
      value = "1" # Success
    else
      value = "0" # Failure
    end

    File.open(ENV["auth_control_file"], "w+") do |f|
      f.puts(value)
    end

(For a fuller featured example, including comparing static challenge strings, please see [the example external script in the repo][example external])

[example external]: https://github.com/fac/auth-script-openvpn/blob/master/examples/external_script

Once the plugin is built and you have a working script, the only thing that remains is to hook it all together and make users lives' easier once more. This requires telling OpenVPN where to find the plugin, and telling the plugin where the external script to invoke is:

    plugin /opt/local/lib/openvpn/plugins/auth-script.so /opt/local/etc/openvpn/auth-script

Now this VPN's traffic is no longer interrupted whilst users are authenticating, and everyone is happy.

_\[If you like solving problems like this and you're looking for a new career challenge, we'd love to hear from you! <https://www.freeagent.com/company/careers>\]_
