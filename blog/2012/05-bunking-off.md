---
:title: Bunking off
:created_at: 2012-05-21 08:22:28.000000000 +01:00
:kind: article
:tags:
- Operations
:author: olly
:slug: bunking-off
---
![Bunker data
floor](http://freeagent-engineering-blog.s3.amazonaws.com/bunker.jpg)

The FreeAgent Ops team head off on a two-day road trip this morning as
we start the initial phase of our plan to build a new home for
FreeAgent, the app.

Since the company was initially founded we have hosted FreeAgent with
Rackspace UK, on the outskirts of London. Our infrastructure has grown
dramatically over the past 18 months as our customer base has [rapidly
increased](http://www.freeagent.com/central/20001-a-growth-odyssey). The
introduction of [our second generation
API](http://www.freeagent.com/central/api-2-is-here), [bank
feeds](http://www.freeagent.com/central/automated-bank-feeds-from-barclays),
and a need for multiple full-stack integration environments for testing
and QA has dramatically increased the need for more servers. Our
[Puppet](http://en.wikipedia.org/wiki/Puppet_(software)) dashboard
currently shows 66 servers in our cluster, a mixture of virtualised and
bare metal, which is spread between managed hosting and the Rackspace
Cloud.

As the strength and depth of our team has increased, so have our needs
as an Operations team and with it our appetite for moving away from
managed and cloud hosting. Not only this, but our needs for maintaining
businsess continuity in the event of a serious data centre (DC) outage
are increasingly important. We plan for disaster recovery (DR), of
course, but we want the best plan we possible can - which, to us, means
running FreeAgent concurrently in multiple data centres so our service
can continue as normal even in the unlikely event of a full data centre
outage.

Which brings us back to our road trip. Over the next two days we’ll be
fitting out racks in two ultra-secure data centres with our new hosting
partner [The Bunker](http://www.thebunker.net). Once we’re finished,
we’ll have 14 Dell R620 servers with a combined 1.7TB RAM and 224 CPU
cores (a whopping 448 threads) available to us. We’ll be making full use
of Puppet, [SmartOS](http://smartos.org) and the network wizardry of our
Ops team to provide a state-of-the-art hosting setup which will grow
with FreeAgent for the next five years and beyond.

It’s a truly fascinating set up which we’ll be writing more in-depth
articles about in the coming months. Stay tuned.
