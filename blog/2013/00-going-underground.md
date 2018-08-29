---
:title: Going Underground
:created_at: 2013-02-07 16:05:52.000000000 +00:00
:kind: article
:tags:
- Operations
- Platform
:author: olly
:slug: going-underground
---
Back in the hazy days of last summer we kicked off a project to improve
the infrastructure behind FreeAgent, to prepare ourselves for an order
of magnitude (or two) of very high growth in the coming years, as well
as greatly bolstering our
[DR](http://en.wikipedia.org/wiki/Disaster_recovery) capablility.
Deciding on a hosting strategy is not simple. Unlike [when we first
started FreeAgent](http://www.freeagent.com/central/announce-launch),
today there are a multitude of options: [full](http://aws.amazon.com/)
[cloud](http://www.rackspace.com/cloud/)
[hosting](http://brightbox.com/), fully-managed dedicated hosting,
private clouds, hybrid clouds, managed co-lo, full co-lo. ![Data
Floor](http://freeagent-engineering-blog.s3.amazonaws.com/bunker1-small.jpg "Data Floor"){:.alignleft}
For the past two years we’ve been running a [hybrid
cloud](http://www.rackspace.com/cloud/private/managed_virtualization/)
at Rackspace, UK. This was a mix of managed, dedicated hardware (for us,
[Dell R710s](http://www.dell.com/uk/business/p/poweredge-r710/pd)) –
some servers running on bare metal, some virtualised with VMWare – as
well as on-demand VMs in the Rackspace Cloud. This has worked pretty
well for us. Performance, network and physical device reliability have
been extremely good. The cloud side however, whilst reasonably flexible,
hasn’t met our expectations in terms of performance and reliability.
Furthermore, our in-house operations experience has increased
dramatically, and with this so have our demands and expectations of
control at every level of the stack. ![Blast
Door](http://freeagent-engineering-blog.s3.amazonaws.com/bunker3-small.jpg "Blast Door"){:.alignright}

One critical focus for a financial service such as FreeAgent is security
and resilience. We need the physical security that comes with a [Tier 3
data
centre](http://en.wikipedia.org/wiki/Data_center#Data_center_tiers), we
need to run a VPN to provide network security, and we need
fault-tolerance at every level from the web stack (load-balanced app
servers), to the database tier (clustered MySQL instances, backups) to
multiple peering arrangements at the data centre (DC). At every level we
have worked to remove single points of failure, but ultimately the DC is
itself a single point of failure (admittedly with a far higher
[MTBF](http://en.wikipedia.org/wiki/Mean_time_between_failures)). With
this in mind, we decided to go for DC-level redundancy and operate
FreeAgent out of two sites. 

We looked at a number of options with the
following requirements:

-   Multiple geographically isolated sites
-   At least Tier 3 facilities (power, cooling, etc)
-   Low-latency, dedicated interconnect between sites
-   Outside London (so we're not constrained by power in the future)
-   Military-spec
    [EMI](http://en.wikipedia.org/wiki/Electromagnetic_pulse) protection
    and TEMPEST RFI shielding[*](#footnote)
-   Ability to withstand a 22 kiloton thermonuclear
    bomb[*](#footnote)

![Servers](http://freeagent-engineering-blog.s3.amazonaws.com/bunker2-small.jpg "Servers"){:.alignleft}
Our research (and strong recommendations from existing customers) led us
to our chosen hosting provider, [The Bunker](http://www.thebunker.net/).
Without further ado we acquired two 42U racks, one in each site
(Greenham Common and Ash, Kent) and started [filling them with the
required hardware](/2012/05/21/bunking-off/). What followed was several
months of infrastructure enhancements, network reconfigurations, a lot
of testing (we’ve been running our heavily-used integration environments
out of the Bunker sites for months), as well as providing ongoing
support for our existing infrastructure. Finally, after an epic amount
of work, our Ops team made the final switch over to our new home on
Tuesday this week.

*cue applause*

Moving an extremely popular web app between data centres is an enormous
challenge. Our integration, staging and production stacks now consist of
over 180 virtual machines across 11 VLANs, all of which needed to be
migrated from one physical data centre to two new ones. It wouldn’t have
been wrong to schedule a significant, several-hour outage for FreeAgent
in order to do this, but doing so would significantly inconvenience our
customers, not to mention drastically affect our historically high
uptime stats of which we’re immensely proud! So, we cooked up some
special magic sauce and performed the move in an incredible [**9
minutes**](http://stats.pingdom.com/6q51tweg53gh/29405/2013/02). In
reality, 8 minutes and 55 seconds of this time was spent double-,
triple- and quadruple-checking everything had switched without issue,
which indeed it had. Either way this was an astonishing achievement by
our Ops team and a great live test of our DR procedure.

As FreeAgent hums along happily in her new home, we’re now busy
monitoring, measuring, tweaking and optimising. The first stage was to
make the move with the existing architecture. Check. The next stage is
to build on this, improving performance, building tools and automating
everything we touch.

Even though we’re now 30 metres underground, the
future is extremely bright. 

{:#footnote}
\* *Optional, but obviously desirable*

p.s. If FreeAgent sounds like your kind of company, I should point out that
[we're hiring!](http://www.freeagent.com/company/jobs). We're looking
for talented web and operations engineers of all levels, from graduates
to team leaders. [Get in touch](mailto:jobs@freeagent.com) if you're
interested!
