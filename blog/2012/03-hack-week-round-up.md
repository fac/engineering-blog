---
:title: Hack Week round up
:created_at: 2012-01-23 17:08:49.000000000 +00:00
:kind: article
:tags:
- General
:author: olly
:slug: hack-week-round-up
---
<% excerpt do %>
Hack Week has been and gone and I’ve finally got around to collating
feedback from the team. To give you better insight into what everyone
worked on, and the outcome of their efforts, each team has written about
the projects they took on and what they achieved.
<% end %>

Test Suite Speed \#1
--------------------

Ben:

> I investigated the effects of garbage collection on our test suite
> speed. I tried turning off GC entirely (our tests run in parallel in
> child processes that exit before they use up too much memory) and
> deferring GC runs to every few seconds (using code from
> [http://37signals.com/svn/posts/2742-the-road-to-faster-tests](http://37signals.com/svn/posts/2742-the-road-to-faster-tests)).
> I discovered that turning deferring GC runs was the best strategy - it
> reduced our test suite time by about 20%!

> During hack week, I had a ton of fun and worked with team members with
> whom I don’t normally get an opportunity to collaborate. Plus we have
> faster tests, which is a huge benefit for the whole team. I’m looking
> forward to the next one!

App-wide Search
---------------

Mihai:

> As we are about to move to
> [Elasticsearch](http://www.elasticsearch.org) for indexing our logs,
> my Hack Week idea was to experiment with building an app-wide search
> function. It is just a prototype but it enables users to search across
> Contacts, Projects and Expenses and can easily be extended.
> Elasticsearch is accessed from Rails using the [Tire
> gem](https://github.com/karmi/tire). Instead of using Tire's
> `after_save` callback to keep the index up to date, Elasticsearch has
> the concept of
> [rivers](http://www.elasticsearch.org/guide/reference/river/) which
> pulls new data. Every update triggers an AMQP message using
> [Bunny](https://github.com/ruby-amqp/bunny) which is then picked up by
> [Elasticsearch RabbitMQ
> river](https://github.com/elasticsearch/elasticsearch-river-rabbitmq/blob/master/README.md).

> It was an exciting idea and I really enjoyed the hack week and had the
> opportunity of experimenting with new pieces of infrastructure which
> we hope to use soon.

API Client
----------

Graeme B:

> Murray and I, with design input from Tane, built CashAgent: a simple
> mobile cashflow forecasting app using the new version 2 of the
> FreeAgent API. We developed the server side of the app in Ruby using
> [Sinatra](http://www.sinatrarb.com/), and the UI of the app in
> Javascript using [Ember.js](http://emberjs.com/).

> Our goals:
>
> -   It was Murray and Tane’s first week at FreeAgent and we wanted
>     them to jump straight into building apps.
> -   Give the new version of the API a good work out, especially our
>     new OAuth 2.0 authentication system.
> -   Try out the Ember.js framework (which is great by the way!)

> We had loads of fun building the app and are looking forward to
> releasing API v2 and the next Hack Week.

Revisiting HTTP load balancing
------------------------------

Thomas:

> Bugged by a number of shortcomings in the “traditional” approach to
> scaling via HTTP load-balancing, I spent the time prototyping an
> approach to this problem based on an idea that has been rattling
> around in my head for some time. Rather than configuring the address
> of each of our app-servers in a front-end load-balancer and having
> this load-balancer “push” traffic to the servers, I inserted a Message
> Queueing server ([RabbitMQ](http://www.rabbitmq.com/)) into the mix,
> writing a small server to “publish” HTTP requests onto a queue, and
> letting our app workers subscribe to this queue to do the work for
> each request.

> By the end of the week, I had built a relatively robust prototype
> which we’ve used in a testing environment internally, which has
> demonstrated that it’s both fast and scalable enough, and also
> simplified the configuration and maintenance of our infrastructure.

> Which is nice.

> Blog posts and open-sourcing hopefully to follow.

BigDecimal / Ruby 1.9.3
-----------------------

Graeme M:

> We started out the Hack Week by looking at the performance of Ruby’s
> BigDecimal, which we use extensively, based on my gut feeling that it
> was slow, and my secret desire to mess around with C extensions.
> However, after a spot of performance testing, we discovered that it
> wasn’t that slow, and it definitely wasn’t a bottleneck.

> So we switched tack and worked on upgrading FreeAgent to Ruby 1.9.3
> (we’re on 1.9.2 right now). This upgrade, whilst still not complete,
> will decrease our test suite run time as well as greatly improving
> Rails boot time. We hope to move the app fully onto 1.9.3 in the near
> future.

Test Hygiene
------------

JB:

> We test everything at FreeAgent, before, during and after development.
> This means we have a huge suite of tests which we run any time we make
> a change. It also means that suite takes a long time to run. Any
> developer will tell you that Test Driven Development requires fast
> turnaround on your tests. Waiting ten seconds to find out if you've
> broken anything can be deemed too long. The full suite of unit tests
> in FreeAgent takes several minutes.

> Instead of starting from the premise of making tests "faster", we
> thought we'd start by making them "better". Any fool can speed up
> tests by reducing the number or scope of them. Our goal: get faster
> while actually increasing coverage.

> Result? We won, spectacularly. Reviewing our tests in a concerted
> effort revealed a number of anti-patterns, chiefly:

> -   hitting the database when we didn't need to
> -   testing things more than once, in more than one place
> -   trusting our factory-girl factories

> the first two were easily spotted and dealt with, and led to a massive
> speed up. The third was more subtle. Something as innocent-looking as:

>     Factory(:bill)

> was causing a huge overhead. Why? Because a bill needs a contact to be
> valid, and a contact needs a company to be valid, and a company needs
> a bank account to be valid, and all of these objects were being
> created and destroyed every time we needed a bill. Replacing with:

>     Bill.new

> made all of that go away. When all you want to do is check that bill
> correctly decides when it's overdue, you don't care about the rest of
> the object, and you certainly don't need the overhead of going to the
> database to create a load of relationships you aren't testing. Our
> factories had grown, but our tests hadn't evolved with them.

> Obvious stuff, but sometimes you need to take that step back and ask
> "what am I trying to do" instead of "how has this been done before".

> Especially when it takes you from 150 seconds to four.

It goes without saying that Hack Week was an enormous success. Everyone
enjoyed it (although on reflection, some wished they had picked
something a bit more ‘exciting’!) and it has definitely had a positive
impact on the team and the way we’ll approach things going forward
(tests in particular). We’re also really excited about driving some of
the concepts through to production, such as the new load-balancing
solution. And of course we’ll be blogging more about the technologies as
they progress (and are hopefully open sourced!).

Watch this space.
