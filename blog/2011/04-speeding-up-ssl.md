---
:title: Speeding up SSL
:created_at: 2011-11-29 10:47:35.000000000 +00:00
:kind: article
:tags:
- Platform
- puppet
- ssl
- tls
:author: thomas
:slug: speeding-up-ssl
---
SSL is great; widely supported, easy to set-up, relatively cheap these
days and (relatively) secure. We've required it from our early days and
it hasn't caused us too many issues other than needing us to renew our
SSL certificates from time to time and requiring a few more IP addresses
than we otherwise would have
needed<sup>[1](http://en.wikipedia.org/wiki/Secure_Sockets_Layer#Support_for_name-based_virtual_servers)</sup>.

That said, I recently visited Portland to attend
[PuppetConf](http://puppetconf.com/) (all about
[Puppet](http://projects.puppetlabs.com/projects/puppet), a
configuration management technology that we're using, blog post to
follow) and when I tried to access FreeAgent from the West Coast I had
experience, first-hand, of one of SSLs major drawbacks - namely the
effect of latency.

SSL, or
[TLS](http://en.wikipedia.org/wiki/Transport_Layer_Security) to use it's
more up-to-date name, effectively wraps a normal HTTP connection,
transparently encrypting data as it is transmitted between the web
browser and the server. To establish this secure channel, the client and
server must first exchange certain pieces of information in a phase
known as the "handshake". This negotiation typically comprises of:
<sup>(see [wikipedia](http://en.wikipedia.org/wiki/Secure_Sockets_Layer#Simple_TLS_handshake) for more detailed information)</sup>

1.  The client opens a standard TCP connection to a port appropriate for
    the wrapped application protocol (by default 443 for HTTPS).
2.  The client sends a "ClientHello" message specifying its support for
    protocol versions and encryption algorithms.
3.  The server responds with a "ServerHello" message, agreeing a
    specific protocol version and algorithm to use. It also sends a
    certificate (which is part of a [remarkably clever
    mechanism](http://en.wikipedia.org/wiki/X.509) allowing you to trust
    a previously unknown remote server), and a "ServerHelloDone"
    indicating the server is happy with a given set of parameters.
4.  The client then sends a "ClientKeyExchange" message which, using
    asymmetric cryptography and the trusted server identity from the
    certificate, shares a value crucial to establish a "shared secret"
    which can be used to encrypt all further communication. A
    "ChangeCipherSpec" message is also sent by the client, to mark that
    all subsequent communication is encrypted, and a "Finished" message
    is sent which can be used by the server to work out if the
    negotiation was successful.
5.  The server then uses the clients "Finished" message to perform
    checks and responds with its own "ChangeCipherSpec" and encrypted
    "Finished" messages.
6.  The client receives the server's "Finished" message and verifies it,
    at which point the server and client have enough information to
    transparently encrypt all further (HTTP) data.

Looking at this exchange, it requires two full round-trips from client
to server and back to complete, whilst the peer simply waits, before
HTTP can take over. Bear in mind all this takes place over a TCP
connection, so this is in addition to the usual TCP SYN/ACK dance that
must also happen for the connection to exist. 

Since we're a Scottish
company and our product is currently geared towards a UK audience, our
servers are based in the UK. For an average user hooked up via ADSL,
even with a relatively poor 50-60ms round-trip time, the time taken for
this SSL handshake, 100ms in this case, pales into insignificance
compared to the time our servers spend crunching numbers to handle the
request. And that is a poor link. My home ADSL2+ line, for example,
actually takes 18ms for a round trip, so this handshake just isn't a
problem. 

However, the further you travel from the UK, the more this
picture changes. When a customer in the US will have to wait, on a good
day, 120ms for a packet to get to our servers and back, these small but
necessary exchanges begin to add up. And it turns out we actually have a
sizeable international customer base using our Universal product. Travel
out to Japan and you find the back-forth trip of a message will take a
good 260ms. Also consider that, due to the number of links these packets
are hopping across to reach their destination, this latency can vary
much more wildly (generally increasing!). All things considered, it
really was a surprise that we still have paying customers using the site
in Australia.

So, the fix!
------------

The trivial fix for this is to simply move the server (geographically
and logically) closer to the client, thereby reducing the
round-trip-time, and speeding up the handshake. It's not, however, going
to be *that* simple. In my ideal world, we'd be shipping users' traffic
to a set of servers geographically close to them; splitting and moving
our databases around to accomplish this. We're, sadly, not quite there
in terms of international demand to be able to prioritise the work of
sharding our database and managing the overheads of multiple,
geographically separate clusters. Not to mention overcoming potential
policy issues with shipping and storing customer's data on international
servers. Not yet, anyway.

So, we can't move the app servers or the
database. The next logical conclusion is to move the machines which are
actively handling the server side of the SSL handshake. 

Our infrastructure is composed of multiple application servers handling the
requests (we're using [unicorn](http://unicorn.bogomips.org/), by the
way), with traffic distributed to these using load-balancer software
([nginx](http://nginx.com), in our case). Since both the app servers and
load-balancers reside securely within our production data-center, the
SSL encryption and decryption takes place on the load-balancers and
unencrypted HTTP is used between internal servers. Since we use Puppet
to automate our configuration, it's now straightforward to create and
manage a new remote load-balancer, closer to some of our international
clients, which takes care of the SSL termination and uses HTTP (without
the SSL/TLS overhead) across the latent link to communicate with our
production machines. 

Great, but I'm sure you're politely coughing, ready
to interject with a suggestion that sending unencrypted HTTP requests
and responses half way around the world might not be the best idea. Pfft
to that, I say. 

Actually, you'd be right. So the next step is to encrypt
this traffic. It would be silly to use HTTPS on a per-connection basis,
incurring the handshake penalty for each request again. So instead I'm
using the excellent [OpenVPN](http://openvpn.org/) software to establish
a single long-lived TLS tunnel between the remote load-balancer and our
UK servers. This is ideal as the handshake happens once, it's only
renegotiated every hour and is persistent - able to carry multiple HTTP
requests securely, without the penalty of HTTPS negotiation for each
request. 

So I spent a couple of hours throwing together a proof of
concept, just to see what the potential improvement may be.

Ok, ok. The numbers...
----------------------

To play around with this, I configured an [Amazon
EC2](http://aws.amazon.com/ec2/) instance in their Japan region,
configured to proxy traffic, as described above, to our UK
load-balancers. 

To get a "finger-in-wind" idea of the improvement, I'm
using the
[apachebench](http://httpd.apache.org/docs/2.0/programs/ab.html) utility
on another EC2 instance. To get a feel for the latency involved here, I
ping'ed our UK datacenter from the EC2 instance:

    [thomas@ap-lb1.production:~]$ ping -c3 94.236.51.2
    PING 94.236.51.2 (94.236.51.2) 56(84) bytes of data.
    64 bytes from 94.236.51.2: icmp_seq=1 ttl=44 time=268 ms
    64 bytes from 94.236.51.2: icmp_seq=2 ttl=44 time=259 ms
    64 bytes from 94.236.51.2: icmp_seq=3 ttl=44 time=267 ms

    --- 94.236.51.2 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 1998ms
    rtt min/avg/max/mdev = 259.616/265.232/268.772/4.037 ms

So we're seeing a round-trip time of ~265ms, as expected. The next step
is to do a "baseline" HTTP request from a UK server to the UK app - to
see the time spent on the server actually rendering the page. The page
I'm using, incidentally, is our login page as it doesn't require any
pesky session cookies, and is relatively lightweight.

    ab -n 50 https://tdhtest.freeagentcentral.com/login
                  min  mean[+/-sd] median   max
    Connect:       14   14   0.4     14      16
    Processing:    12   18  22.1     16     171
    Waiting:       11   18  22.0     15     170
    Total:         26   32  22.1     30     185

So, the best-case is roughly 30ms. Let's see how the app currently
performs for international users, by requesting the page from our remote
EC2 instance, via the UK load-balancers:

    ab -n 50 https://tdhtest.freeagentcentral.com/login
    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:      985 1043  28.0   1042    1095
    Processing:   260  279  26.2    276     453
    Waiting:      259  278  26.1    275     452
    Total:       1246 1322  43.3   1318    1501

Woah! 1.3 seconds to receive a login page. Now, let's try with going
through the EC2 load-balancer, with traffic tunnelled back to the UK:

    ab -n 50 https://tdhtest.freeagent.com/login
    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:       16   16   0.8     16      18
    Processing:   507  540  12.5    539     567
    Waiting:      507  540  12.4    539     567
    Total:        523  557  12.4    556     583

557ms - not bad! Less than half the original time, which I'd call quite
an improvement. Just to get a slightly different take, I used a remote
browser timing tool, [loads.in](http://loads.in), to time the page load
from these locations, from the request sent to the page rendered in the
browser. For completeness, the tests were using Safari 4:

-   Baseline reading was **1.9 seconds**
-   A browser in Japan hitting our UK load-balancers took **9.4
    seconds**
-   A browser in Japan hitting our remote load-balancers took **6.7
    seconds**

Whilst, this is a less marked improvement, it's worth considering that
we load our static assets; images, css files, javascripts, animated
gifs, flash videos, background sounds, etc. from a separate domain - and
this *won't* have been routed via the remote load-balancer, so will be
having quite a detrimental impact on the overall page load-time.

I'll re-try the test, some time, without this offloading, to see what the
actual improvement could be.

Now what?
---------

So we can now route specific customers' traffic through this
load-balancer, but what we'd ideally want to do is select which
load-balancer to use based on the origin of the traffic. To do this
"properly" would require us to have DNS servers configured to return
records based on the source IP of the requests. This is more work than I
care to undertake for a few hours messing with SSL, but thankfully
Dynect (our DNS provider) is able to take care of this for us. They have
multiple [anycast'ed
DNS](http://en.wikipedia.org/wiki/Anycast#Domain_Name_System) servers
and offer a [traffic management
system](http://dyn.com/dns/dynect-managed-dns/traffic-management/) which
is perfect.

We've not yet enabled this, as it's little more than a
proof-of-concept, but if you're an international FreeAgent user and
would like to try it out, please [get in
touch](mailto:sslbeta@freeagent.com).

Anything else?
--------------

Since we're now managing both ends of the crazy long link, a world of
tweaks and optimisation becomes possible. For starters, the two things
I'm currently playing with:

-   Endless tweaks to the TCP congestion control and windowing
    algorithms. These regulate the maximum amount of un-acknowledged
    data that can be sent between the client and server which, as long
    as packets aren't dropped, reduces the time either party has to
    spend waiting for data acknowledgements. Google do crazy things with
    this to make their homepage super fast, [check it
    out](http://research.google.com/pubs/pub36640.html).
-   TCP parameters are "tuned" as connections transfer more data, so
    having many short-lived HTTP connections - i.e. one per request -
    kills any benefit, as well as each HTTP connection having an
    associated TCP connection setup cost (of one round-trip). I'm
    currently playing around with multiplexing all HTTP requests down a
    single persistent TCP connection established between the remote and
    local load-balancers, which overcomes the connection set-up cost,
    and will allow this connection's parameters to be tuned as time goes
    on.

I'll see how I get on, and perhaps even post a follow up blog post if
things go well.

Wow. You made it to the bottom. In that case, I should
probably mention to you that [we're
hiring](http://www.freeagent.com/company/jobs/senior-platform-engineer)!
