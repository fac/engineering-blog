---
:title: Atlas Probes 
:created_at: 2014-01-24 10:48:00.000000000 +00:00
:kind: article
:tags:
- operations
- monitoring
:author: nathan
:slug: atlas-probes
---

<% excerpt do %>
Last Monday evening we received this tweet:

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p><a href="https://twitter.com/freeagent">@freeagent</a> your servers are running really slow tonight, making data input a real drag</p>&mdash; Warwicka (@Warwicka) <a href="https://twitter.com/Warwicka/statuses/422857540515672064">January 13, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Naturally we take anything like this seriously so
we started digging. First stop was [New Relic](https://newrelic.com/) which shows us
average application and browser response times as well as a whole host of other useful
metrics. Everthing looked normal. OK, time to hit the logs in search of any long running
requests.
<% end %>

We store all our logs in [ElasticSearch](http://www.elasticsearch.org/) which
we can search using [Kibana](http://www.elasticsearch.org/overview/kibana/).
This combination has proved invaluable to us and once again it started to point
us in the right direction.

While searching for all long running requests, we noticed a few patterns. Firstly that
the long running requests were only long running on the load balancers and not
the application servers. Secondly that most of these long running requests were
from BT-owned IP addresses.

Then as suddenly as this started, it stopped. This isn't the first time we've
had issues with BT. Guess they had some incident which they resolved.

Along comes Tuesday evening and again:

<blockquote class="twitter-tweet" data-conversation="none" data-cards="hidden" data-partner="tweetdeck"><p><a href="https://twitter.com/freeagent">@freeagent</a> sorry, but again tonight, site is so slow. It&#39;s taking over a minute to load a page. I&#39;m on a very fast connection here as well</p>&mdash; Warwicka (@Warwicka) <a href="https://twitter.com/Warwicka/statuses/423195031303106560">January 14, 2014</a></blockquote>

Two disruptions in two nights, each lasting only a couple of hours. This was
odd. I decided that if this was going to happen again, I wanted to be ready for
it

Lets rewind a few months. A friend of mine told me about the 
[RIPE Atlas](https://atlas.ripe.net/) project and kindly gave me some credits
to try it out. RIPE are trying to build the largest internet measurement network
by giving probes away to people to host on their networks. In return for hosting
a probe you gain credits which can be used to query other probes.

The great thing about these probes is most of them are hosted on residential
internet connections and not in data centres which usually have lots of
bandwidth and redundant links.

I [applied for a probe](https://atlas.ripe.net/get-involved/become-a-host/)
and a few weeks later it arrived.

![Atlas Probes](http://freeagent-engineering-blog.s3.amazonaws.com/atlas-probe.jpg){:.aligncenter}

With users reporting problems which we believed to be network related it was
time to test this out.

I set up 2 measurements:

1. Find 10 UK probes and ping our app every 5min.
2. Find 10 BT probes and ping our app every 5min.

The results were interesting and confirmed exactly what we were seeing. The
results from the UK wide probes showed that only BT was having.

![UK Probes](http://freeagent-engineering-blog.s3.amazonaws.com/atlas-uk-probes.png){:.aligncenter}

Looking at the BT probes we see that nearly all of them have packet loss or
increased ping times at the same time.

![BT Probes](http://freeagent-engineering-blog.s3.amazonaws.com/atlas-bt-probes.png){:.aligncenter}

It's important to remember here that these are hosted on home connections so we
do expect them to have fluctuations every now and again.

As it happens the problem turned out not to be with BT at all but with a company
our [hosting provider](http://www.thebunker.net/) has a
[BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol) peering with. 
BT customers were favouring this route which was
[flapping](https://en.wikipedia.org/wiki/Route_flapping) causing the packet loss
and slowness our customers were seeing.

Of course we used 
[other tools](https://www.linx.net/pubtools/looking-glass.html), which might be
the subject of a future blog post, to find exactly where the problem was but the 
[Atlas project](https://atlas.ripe.net/) provided a lot of useful information.

Thankfully this issue is now resolved and we've been carefully monitoring
it over the last week or so.

If you're interested in this technology, you can help
[@RIPE_Atlas](https://twitter.com/RIPE_Atlas) expand their network by
[hosting a probe](https://atlas.ripe.net/get-involved/become-a-host/) for them.
