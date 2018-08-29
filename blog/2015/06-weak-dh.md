---
:title: Weak DH - Time to Level Up
:created_at: 2015-10-09 09:53:49.000000000 +00:00
:kind: article
:tags:
- SSL
- security
:author: nathan
:slug: weak-dh
---

<% excerpt do %>
SSL vulnerabilities have been big news over the last few years. We've had [Heartbleed](http://heartbleed.com/),
[CSS Injection](http://ccsinjection.lepidum.co.jp/), [POODLE](https://en.wikipedia.org/wiki/POODLE) and
[FREAK](https://freakattack.com/) among others. At FreeAgent we take these vulnerabilities very seriously and
work to mitigate these as fast as possible.

The one we will be looking at today is [Weak Diffie-Hellman and the Logjam Attack](https://weakdh.org/) and some
changes we are going to be making in the coming months.
<% end %>

SSL configuration can be confusing with all the different ciphersuites, TLS versions and various parameters.
Thankfully Mozilla have a great [reference document](https://wiki.mozilla.org/Security/Server_Side_TLS) and a 
[config generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/) which I highly recommend you
check out if you're running SSL on your web servers or load balancers. We use this as a base for our configuration
and review regularly to make sure our configuration is inline with their recommendations.

As a result of doing this we were not vulnerable to the LogJam attack when it was announced as we didn't support 
"[export grade ciphers](https://en.wikipedia.org/wiki/Export_of_cryptography_from_the_United_States)".  However in
light of this attack it is recommended to increase the prime size used for Diffie-Hellman key exchange to at least 2048 bits.



## Diffie-What?
The [Diffie-Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) (or DH for short) is
one of the methods you can use to generate a shared secret between the server and client (or 2 people) in such a way that it
can't be observed by any intermediate 3rd party.

The algorithm was first published in 1976 by Whitfield Diffie and Martin Hellman although it was heavily influenced by Ralph
Merkle, so much so that Hellman suggested it be called "Diffe-Hellman-Merkle key exchange". This was pretty much the start of
[Public Key Cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) as we know it today.

Anyway, enough of the history lesson. Let's figure out what this means for people.


## So what are we doing?
We are increasing the size of the prime used in the DH key exchange from 1024 bits to 2048 bits. This is often known as
"DH params" as well. As a result clients connecting to us will either need to be be able to support this or use cipher suite
that uses a different method of key exchange.

We will be making this change at the end of **February 2016**. People will be notified of the exact date nearer the time.


## Users
For our regular users there should be nothing to worry about. Any browser that
[FreeAgent supports](http://www.freeagent.com/support/kb/getting-started/system-requirements) will work just fine.


## Developers using our API
For anyone using a reasonably modern development stack then you should have no problems at all. The easiest way to check is
to test your app against our sandbox environment which has had this change enabled for a few months now. 

Details of this can be found in our [API documentation](https://dev.freeagent.com/docs/quick_start)

If your app works against the sandbox then you can stop reading here if you want.


## What if it doesn't work?
If you are using **Java 6** or anything built on Java 6 then unfortunately it isn't going to work since it doesn't support any of
the required ciphersuites. Ideally you should look to upgrade to Java 7 or 8. If this is really a problem then please contact
us as soon as possible to discuss options.


## But I'm not using Java 6
Most development stacks use a 3rd party library to handle their cryptography. Common ones are:

* [OpenSSL](https://www.openssl.org/) and derivatives ([LibreSSL](http://www.libressl.org/),
[BoringSSL](https://boringssl.googlesource.com/boringssl/))
* [NSS](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS)
* [GnuTLS](http://www.gnutls.org/)

Any of these should be able to support the change. Just make sure that you are running a reasonably recent version. If you're
using some other library then make sure it supports the ciphersuites and parameters listed
[here](https://wiki.mozilla.org/Security/Server_Side_TLS#Intermediate_compatibility_.28default.29).


## What about Java 7?
Java 7 doesn't support > 1024 bit primes for DH key exchange, which might make you think you’re out of luck. The good news is
that this limitation doesn't actually matter, since it can support ECDHE (Ephemeral Elliptic Curve Diffe-Helleman key
Exchange) instead. The bad news is that this isn't always enabled by default.

You'll know this if you try to connect to our sandbox and get the following error.

    java.lang.RuntimeException: Could not generate DH keypair

There are a few ways to fix this. One is to use the [Bouncy Castle](https://bouncycastle.org/) library or you can use 
Mozilla's [Network Security Services](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS) (NSS).
What follows are instructions for enabling the latter on Linux although the
process should be similar for other operating systems.

Make sure the NSS libraries are installed which are either called `libnss3` (on Debian based distros) or `nss` (on RedHat
based distros)

Edit `${JAVA_HOME}/lib/security/java.security` and find the list of security providers which might look something like this.

    security.provider.1=sun.security.provider.Sun
    security.provider.2=sun.security.rsa.SunRsaSign
    security.provider.3=sun.security.ec.SunEC
    security.provider.4=com.sun.net.ssl.internal.ssl.Provider
    security.provider.5=com.sun.crypto.provider.SunJCE
    security.provider.6=sun.security.jgss.SunProvider
    security.provider.7=com.sun.security.sasl.Provider
    security.provider.8=org.jcp.xml.dsig.internal.dom.XMLDSigRI
    security.provider.9=sun.security.smartcardio.SunPCSC
    # security.provider.10=sun.security.pkcs11.SunPKCS11 ${java.home}/lib/security/nss.cfg

That bottom line needs uncommenting then save the file.

Next make sure `${JAVA_HOME}/lib/security/nss.cfg` contains the following

    name = NSS
    nssDbMode = noDb
    attributes = compatibility

Restart your application and you should be good to go.


## Need more help?
Please stop by our [API support group](https://groups.google.com/forum/#!forum/freeagent_api) or contact our
support team who will be more than happy to help you.

Hopefully we've covered everything. Anyone with an application registered with
us should be receiving an email as well to make sure everyone is aware.
