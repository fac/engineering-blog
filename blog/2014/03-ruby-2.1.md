---
:title: Upgrading to Ruby 2.1 (and other fun with YAML and complex regexes)
:created_at: 2014-05-23 09:11:00.000000000 +00:00
:kind: article
:tags:
- ruby
:author: olly
:slug: upgrading-to-ruby-2.1
---

When we first launched FreeAgent, it ran on Ruby 1.8.6 MRI (and Rails 1.2!). We graduated to
[1.8.7 REE](http://www.rubyenterpriseedition.com) when that became popular, then in the summer of 2011 we upgraded
to Ruby 1.9.3. We've been running on that version (1.9.3-p194 to be specific) ever since. It has served us well,
but performance is not one of Ruby 1.9.3's strong points and we've seen our application server response time gradually
 grow over the years to the point where we really wanted to do something about it.

Just over a year ago, [Ruby 2.0 was released](https://www.ruby-lang.org/en/news/2013/02/24/ruby-2-0-0-p0-is-released/).
This introduced DTrace support and GC optimisations, both of which were of particular interest to us at FreeAgent since
they allow us better insight into what Ruby is spending its time doing. As an added bonus, we also expected an
improvement in performance. Late last year we spiked up a branch of FreeAgent that ran on Ruby 2.0 but before we'd
even started to QA this, [Ruby 2.1 was released on Christmas Day](https://www.ruby-lang.org/en/news/2013/12/25/ruby-2-1-0-is-released/)!

Over the past couple of months we've been keeping our Ruby 2.1 branch up to date with master and, after a rather epic QA process, we finally merged those changes into master earlier this week. For the time being we kept running on Ruby 1.9.3 but yesterday we made the flip and FreeAgent is now running on Ruby 2.1.2 across the board.

In this article I'll share the insight we've gained in terms of performance, and as there always seems to be with Rails
and Ruby upgrades, we came across an awkward gotcha that we can hopefully help you avoid when you upgrade.

## Performance

Here's a picture of how FreeAgent behaves on our application servers during a typical afternoon with a throughput of
~2000 requests/min.

![1.9.3 Average](http://freeagent-engineering-blog.s3.amazonaws.com/ruby-2.1/average-1.9.3.png){:.aligncenter}


As you can see, DB latency is encouragingly low, whilst Ruby is responsible for around 80% of the server-side processing time bringing the average over 300ms – much too high for our liking.

In order to roll out the Ruby 2.1.2 upgrade, we did what we call a **canary deployment**. This is where we deploy a
particular branch to a quarter of our application servers (one out of four, as it is today) so we can monitor its
behaviour while reducing the risk of introducing any customer-facing side effects. The following graph shows the
server-side response time after we did this deploy (indicated by the blue line at 15:57 - complete with odd New Relic
response spike, something we'll save for a future post). Even though Ruby 2.1 is only running on a ¼ of the stack, you
can see a visible decline in response time almost immediately. This was encouraging!

![Canary Ruby 2.1](http://freeagent-engineering-blog.s3.amazonaws.com/ruby-2.1/canary-2.1.png){:.aligncenter}

Delving into more detail, we can clearly see that not only has response time dropped significantly, CPU usage also shows
a dramatic decrease. Memory utilisation, on the other hand, increases (our canary server is the light blue line):

![Canary Ruby 2.1 Detail](http://freeagent-engineering-blog.s3.amazonaws.com/ruby-2.1/canary-2.1-detail.png){:.aligncenter}

Although we expected it, the memory growth was a bit of a concern. While we have a fair amount of headroom on our app
servers, we wanted to make sure this didn't get out of hand.

## Tuning the GC

One of the contributing reasons for our decreased response time running Ruby 2.1.2 is the improved garbage collection
algorithm. Ruby now uses a generational GC algorithm, which allows Ruby to perform quick minor GCs to catch short lived
objects, preventing the need to run a full, CPU intensive GC as often. The problem with this is a web request
often involves lots of “medium” lifetime objects that are only around for one or two minor GC’s. The current algorithm
considers an object old if it has survived one GC. This causes the memory to bloat with old objects that are no longer
needed. The old memory continues to bloat by default until it doubles in size, only then a major GC triggered.

The solution? Trigger a major GC sooner to avoid old objects no longer needed wasting space. We can do this by tweaking
the environment variable `RUBY_GC_HEAP_OLDOBJECT_LIMIT_FACTOR`.

After tweaking this from the default value of 2 to 1.3 memory usage was constrained and settled back down to an
acceptable level.

At this stage we were pretty happy, but we wanted to run on the canary server overnight just to make sure no issues came
about.

## The inevitable issues that came about

We ran into an issue with a Time object with nanosecond precision which we were dumping to YAML for processing by a job
worker.  Normal Dates and Times serialise to and from YAML with identical behaviour on Ruby 1.9.3 and Ruby 2.1.2 but the
following date behaves differently:

<script src="https://gist.github.com/lylo/c8ac0d1b525f325f5800.js"></script>

In Ruby 1.9.3 the nanosecond precision time is serialised to a YAML String type but in Ruby 2.1.2 the time is serialised
to a YAML Time type.  As we were running with canary servers, the jobs were created and consumed by both sets of
servers.  So there were four possible combinations:

<script src="https://gist.github.com/lylo/d91d545fa4df32fbd49f.js"></script>

When jobs created under Ruby 2.1.2 were processed by a server running Ruby 1.9.3, we found that the time was
deserialised as a Time object instead of the String we expected.  We quickly patched the symptom but it took longer to
work out why this was happening.

**So why is that?**

There are a few moving parts involved. Ruby 2.1.2 includes version 2.0.5 of the YAML parser
[Psych](https://github.com/tenderlove/psych), while Ruby 1.9.3 includes version 1.3.2 of Psych. When serialising a set
of objects to YAML, Psych visits each object and internally creates a tree of strings. This tree of strings is then
converted to YAML tokens using the class [Psych::ScalarScanner](https://github.com/tenderlove/psych/blob/b6e8b10c83720fe73ab40784b3b0cf2d166cea70/lib/psych/scalar_scanner.rb)
that internally uses regular expressions to identify YAML types. The regular expression used to identity time objects
changed in the new version of Psych, which meant our nanosecond precision time was now correctly converted to a YAML
`Time` type rather than being left as a string.

When the YAML time type is deserialised in Ruby 1.9.3, it's converted to a Ruby Time object. The same would happen in
Psych 2.0.5 if it were not for a new feature called Safe Load which was introduced after the recent Rails YAML
deserialisation security vulnerabilities. In Psych 2.0.5, times are not a "safe" type and so our YAML Time type is
returned as a String.

## In the wild

Once we'd addressed all the (known!) gotchas and everything had been running stable for a few hours, we rolled 2.1.2 out
to everyone. Finally we get to see the dramatic server-side performance improvement, the following graph comparing
server-side performance today with the same time yesterday and the previous week:

![Ruby 2.1 comparison](http://freeagent-engineering-blog.s3.amazonaws.com/ruby-2.1/ruby-2.1-comparison.png){:.aligncenter}

I'm calling that a big win.

![Dance!](http://freeagent-engineering-blog.s3.amazonaws.com/ruby-2.1/fry-disco.gif){:.aligncenter}
