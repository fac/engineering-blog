---
:title: Hack Week 2.0 round-up
:created_at: 2012-08-20 12:57:25.000000000 +01:00
:kind: article
:tags:
- General
- Work life
:author: olly
:slug: hack-week-2-0-round-up
---
Wow, what a absolute blast. Hack Week 2.0 has now been and gone, we’ve
had a weekend to relax and this week we can take the time to look back
and reflect on what we achieved. We had just over four days to get our
projects polished (Friday afternoon was set aside for demoing our work
to the company) so it was a challenge, but one everyone on the team rose
to and the results were simply outstanding.

Here’s a run down of what we
made. I’d love to get your feedback so please add your comments or send
us a [Tweet](https://twitter.com/freeagent)!

FieldAgent
----------

An official “mobile version” of FreeAgent is a request we hear a lot, so
a few of the team wanted to see what we could come up with in a week.
Codename: FieldAgent was our attempt to create mobile optimised views of
the mother app, accessible from your smartphone. A week isn’t long but
we managed:

-   A re-imagined Overview screen highlighting important business health
    indicators
-   A contacts list, ordered by who owes you the most, and the ability
    to call/email/stalk them easily
-   A list of all outstanding or open invoices

The team was delighted with the results so we hope to keep the momentum
going and begin rolling this out in the future. 
![Mobile login](http://freeagent-engineering-blog.s3.amazonaws.com/mobile.png){:.aligncenter}
*Team: John, Ed B, Anup, Roan, Robbie, Tane*

Kiseru
------

Moving data between our agile web apps (Trello, Pivotal Tracker, and
YouTrack) is time-consuming and boring - sometimes it seems the tools
that should make us faster end up slowing us down. Kiseru makes working
with web apps as powerful as working with Unix tools - you can simply
grab data from one app, transform it, and pipe it into another app.

You can read more about the project on Github, where we've open-sourced the
code. We'll hopefully be adding to this over time, but feel free to fork
the repo and send us a pull request!

[https://github.com/fac/kiseru](https://github.com/fac/kiseru)

*Team: Ben B, Aaron*

Adding Two Factor Authentication to FreeAgent
---------------------------------------------

Two Factor Authentication is an extra layer of security for online
accounts. It’s the idea that by adding another ‘thing’ to the
authentication process, you can be more certain that the intended person
is logging in and make it a bit more difficult for naughty people to
break into your account. 

This is widely used in the world of online
banking, where the two things (or factors) combined are something you
know, your password and something you have, usually a wee gadget of some
kind that generates a number (or token) which when used together allow
you access to your account. Some online banks also have you generate a
new token to confirm sensitive opperations, like transferring money. 

For
my Hack Week project, I added this to FreeAgent. This serves two
purposes, it makes our login process a bit more secure and also makes
confirming destructive actions (like “Delete My Data”) a lot safer.
Integration was done with [Authy](http://www.authy.com) and [Google
Authenticator](http://en.wikipedia.org/wiki/Google_Authenticator), can
be enabled on a per-user basis and just needs the user to have a mobile
phone. 

I’ll be continuing work on this and investigating dedicated
hardware devices like the [Yubikey](http://www.yubico.com/personal-use)
and the [Gemalto Ezio](http://onlinenoram.gemalto.com). 
![2-factor Authentication](http://freeagent-engineering-blog.s3.amazonaws.com/2fa.png){:.aligncenter}

*Team: Ryan*

Graphite, Geckos and Yaks
-------------------------

FreeAgent of course collects a vast array of metrics using various
different tools like [Ganglia](http://ganglia.sourceforge.net), [New
Relic](http://newrelic.com) as well as pushing some applications metrics
to [Geckoboard](http://www.geckoboard.com) which we have displayed
around the office. 

While all of this is very useful we thought that it
would be great to centralise as much as possible for analysis and have a
way of easily pushing choice things out to Geckoboards. 

Most of the
metrics are time series data and
[Graphite](http://graphite.wikidot.com/) was a good fit providing both
storage and a powerful graph composer. 

By the end we had some collection
agents for getting metrics out of Ganglia, New Relic, and an app to take
data from Graphite and feed it into Geckoboard.

![Graphite](http://freeagent-engineering-blog.s3.amazonaws.com/graphite.png){:.aligncenter}

We of course started pushing metrics out from our
[Campfire](http://campfirenow.com/) bot as well so we can now see how
many images of Yaks people request.

![Yak](http://freeagent-engineering-blog.s3.amazonaws.com/yak.png){:.aligncenter}

*Team: Nathan, Michael*

Error Reporting
---------------

Even FreeAgent isn’t immune from errors, but when they happen we need to
capture as much information as possible about them in order to diagnose
and fix the problem. We’ve been using [Airbrake](https://airbrake.io) to
track this, but it lacks some features that would make our life as
developers easier - primarily the ability to search our error logs. 

A bit of research revealed an open-source clone -
[Errbit](https://github.com/errbit/errbit) - which while still lacking
the features we wanted would allow us to add them and release our
additions back to the community. Search was added using
[ElasticSearch](http://www.elasticsearch.org/), and we also added the
ability to send our error reports asynchronously over
[AMQP](http://www.amqp.org/). The final bit of polish was given by
reskinning Errbit using the [Twitter Bootstrap
framework](http://twitter.github.com/bootstrap) to allow us to make the
app mobile friendly and keep a consistent UI with our other internal
tools.

![Errbit](http://freeagent-engineering-blog.s3.amazonaws.com/errbit.png){:.aligncenter}

*Team: Paul, Tane*

FreeAgent Personal
------------------

A popular request we hear from our customers is for a personal version
of FreeAgent, to allow them to better manage their personal finances as
well as their business ones. This has never been a priority at FreeAgent
since we have so much we want to achieve with our core business. That
said, some of us on the team are pretty disappointed with the personal
finance products on offer in the UK, so we decided to write our own. A
pretty tall order you might think, but given that FreeAgent already has
a lot of suitable technology built in (bank statement processing, for
instance), we decided to repurpose this and built the app from scratch.
We're calling it **Shoestring**.

In only four days we managed to produce
a working MVP. Not just a prototype, but a fully working app with
authentication, statement upload and fancy reconciliation. We’re not
sure where we’ll take this next, but many of the team have expressed an
interest in ‘dogfooding’ it already, so you never know! ![Shoestring
1](http://freeagent-engineering-blog.s3.amazonaws.com/shoestring.png){:.aligncenter}
![Shoestring
2](http://freeagent-engineering-blog.s3.amazonaws.com/shoestring2.png){:.aligncenter}

*Team: Olly, Mihai, Graeme B, Chris*

A content-first website experiment
----------------------------------

We created a new IA for the FreeAgent website supported by a set of new
content guides and constraints. Using this we created a new set of
content (written + video) and used this to make a mobile first prototype
for the site.

*Team: Ben H, Danae*

Profiling
---------

For my Hack Week project, I had the chance to look into profiling some
of our Ruby code. The rest of the world probably knows this already, but
I found [ruby-prof](http://ruby-prof.rubyforge.org/) to be exceptionally
useful.

It’s not all in the tools though; the use cases and the layout
of the data obviously had a huge impact on the results. So I spent the
first half of the week examining the distribution of real data so that I
could set up realistic performance scenarios for the best and worst
cases.

ruby-prof is really easy to use:

    RubyProf.start 

    # Do work 
    result = RubyProf.stop 

The results can be printed in different formats. I used the MultiPrinter
which outputs call stack and call graph HTML reports amongst others.

    printer = RubyProf::MultiPrinter.new(result) 
    printer.print(:path => OUTPUT_DIR, :profile => profile_name) 

The call graph simply lists the methods called and the times spent in
each method.

The call stack report shows this information visually and I
spent most of the week looking at this. It colours the output,
immediately drawing your attention to the areas that are taking up the
most time. For each method, we can see the percentage of the overall
runtime, the percentage of its parent’s runtime as well as the number of
times it was called. The call count actually became quite important as I
found that several methods were being called far more times than I would
have expected for the input data, leading me to branches of code that
could be avoided entirely under certain conditions.

ruby-prof also
outputs a grind report. This can be opened with kcachegrind (I used the
Qt version qcachegrind on the Mac). This is similar to the flat call
graph report but is much quicker to explore. Through the interface, you
can easily drill down to different methods, seeing its callers and
callees.

Although I had preconceived notions of where the performance
bottlenecks were, I learnt the following which surprised me:

1.  Serialised ActiveRecord columns were being deserialised while an
    object was being saved, even when the serialised column was not
    accessed or updated. In fact, when I update a single attribute on a
    model instance, the serialised column is included in the SQL, even
    when its value had not changed.
2.  I was doing alot of unnecessary BigDecimal addition that was eating
    up runtime, that I wouldn’t have seen without profiling
3.  My assumptions about the code branches that were taken were
    inaccurate, again made obvious through profiling

*Team: TJ*

API 2.0 Ruby client library
---------------------------

In April we released our shiny new [API 2.0](http://dev.freeagent.com)
and since then we’ve been excited to see developers bashing away
creating awesome applications. Although we have had an overwhelming
response to API 2.0, we weren’t entirely satisfied. We wanted to make it
even easier for developers. Scratch that; we wanted to make it simple
for anyone to talk to us. So I decided to set myself a monstrous goal;
write an official Ruby Client Library for our API 2.0 in a week. Ready,
set, go!

So how did I do? Even after losing a day to other development
commitments, I far exceed my expectations and managed to finish an alpha
version of the library. Huzzah!

So what does this library have? Well the
library wraps the API into a lovely little package providing rails-esque
querying which makes it trivial to talk to FreeAgent. Let me emphasise
that for you, it is really simple. Don’t be believe me? Well see for
yourself. 

Want all your draft invoices? Simple:

    Invoice.draft 

Want to find all your contacts with open invoices?

    Contact.open_clients 

Want to find your recurring expenses? Come on you're not even making
this hard anymore:

    Expense.recurring 

So when can I have it? Soon, we promise! We plan to open source an alpha
version very soon once further testing has been done. As always we hope
to get feedback on it and we’d love to see contributions from the
community.

*Team: Murray*

TimeAgent Round-Up
------------------

Although it is possible to submit timeslips on FreeAgent, it isn’t
possible to time your work and automatically create timeslips. There are
a number of third-party apps which will do this, but they are often
abandoned. TimeAgent was a hack week project to develop our own time
tracking app.

At FreeAgent we use OS X so the TimeAgent prototype was
developed using
[Cocoa](https://developer.apple.com/technologies/mac/cocoa.html) and the
[FreeAgent API](http://dev.freeagent.com). This means that when we’re
ready, we can release TimeAgent on the App Store, making it really easy
to download and install. As it stands at the end of hack week, TimeAgent
allows you to fetch your projects, contacts and tasks from FreeAgent and
then create timers locally without an internet connection, allowing you
to work from anywhere! 

Looking to the future, we’d like to add the
ability to create projects, contacts and tasks locally so you can use
TimeAgent without a FreeAgent account. If you did then create a
FreeAgent account, we could automatically sync everything you’ve
created, allowing you to easily turn your tracked time into invoices.

*Team: James*

FRS-or-not?
-----------

For the past week we have been working on a Flat Rate Scheme calculator
for our lovely Facebook users. We have been thinking about doing this
for a while, and Hack Week seemed like the perfect opportunity to start
working on such an app.

Even though the functionality of the calculator
itself is quite simple and we thought we would be able to make the app
available by the end of the Hack Week, we had to spend a lot of time
thinking about the best way to present the form and the result to the
users. The result was a working demo that - with a little polishing -
will be soon ready to go live on [our Facebook
page](http://facebook.com/freeagentapp). 

It was great fun working on
something a bit different, and we are already looking forward to the
next Hack Week!
![FRS-or-not](http://freeagent-engineering-blog.s3.amazonaws.com/frs.png){:.aligncenter}

*Team: Ioan, Josh*
