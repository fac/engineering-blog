---
:title: Switching wings mid-flight
:created_at: 2012-09-04 12:28:41.000000000 +01:00
:kind: article
:tags:
- Ruby on Rails
:author: jonathan
:slug: switching-wings-mid-flight
---
How would you choose to release a completely redesigned version of your
web application? Capistrano? Git hooks? Copying files over SFTP? 

Now imagine your application serves over 21,000 active, multi-user accounts.
It’s a very busy app, and forms the backbone of many of those users’
livelihoods. At that point, the question perhaps becomes less about
“how”, and more about “when”. 1am? And how long is the downtime going to
be? Even if it’s minutes, even if it’s seconds, what if something goes
wrong? 

We did [exactly this recently](http://www.freeagent.com/central/freeagent-redesigned). Our
answer to the “when” was 10am on a busy Wednesday morning. Wednesday the
13th, just to be sure. Our answer to “how” was one line on a production
console:

    new_design.update_attributes(:status => 'on') 

Feature switches rock
=====================

`new_design` is a feature switch. If you haven’t come across feature
switches before, the basic idea is simple. You have a list somewhere of
“switches”, maybe with a “status” (though that could be implied simply
by present/not present), and your application code looks something like
this:


    if has_feature?(:my_cool_new_feature) 
      do(:new_hotness) 
    else 
      do(:old_and_busted) 
    end

where `has_feature?` checks the list. Congratulations: you are now using
feature switches. 

We first used basic one-shot feature switches when we
rolled out our [stock management system](http://www.freeagent.com/central/taking-stock), and then
extracted the logic into a little system we call `Flip`. It’s super
simple, but the simplest tools are usually the most powerful. It turns
out that feature switches have a load of benefits.

Launch
------

First, and most obviously, releasing a new feature no longer requires a
code deployment. Well, it still requires a deployment, but the release
of the feature is no longer *tied* to the deployment of the code. You
can deploy the code, check that the old code all still works as
expected, then throw the switch at your leisure.

    Feature.create(:name => 'new_feature', :status => 'on') 

Boom: your new feature is now deployed.

You also know that your exit
strategy is bulletproof, since your users have been using the fallback
code paths since your deployment. Kill the switch if anything goes
wrong, and it’s all good.

    Feature.find_by_name('new_feature')
           .update_attributes(:status => 'off') 

Phased Deployment
-----------------

Pretty soon, you want to check the feature works in production before
you throw the switch for reals. Yes, you’ve tested it in development, in
an integration environment, in staging. It’s been through peer review
and QA and is perfect. But even so, it would be nice if you could throw
the switch in production *just for you* so you can take a peek behind
the curtain and make sure it’s firing on all cylinders. So our
`has_feature?` helper to takes into account the current user’s company,
and extended the Features to have many switches:


    c = Company.last 
    f = Feature.find_by_name('new_feature') 
    f.update_attributes(:status => 'company') 
    f.feature_switches.create(:switchable => c) 

So now, I can not only release my feature whenever I like, I can release
it to my dev team ahead of a release, in production, and have them
hammer on it till it breaks. Which it doesn’t, of course. Winning!

Getting Involved
----------------

The downside of that, however, is that we all know how good dev teams
are at testing applications in the real world. We test the code, but we
miss really obvious UI issues, or workflows that just don’t make sense.
You know who doesn’t miss those things?

Users.

Feature switches turn out
to be a *great* tool for releasing new work to beta testers. There’s no
testing as thorough as living with new code while working with real,
personal data.

I’m sold
========

So were we. In fact, we were so sold that we decided to use switches as
the backbone of our redesign. We split the redesign project into
manageable chunks (generally one project per “section” of the app), and
wrapped the work involved in each project in its own switch. This let us
have a team that methodically worked its way through the app, section by
section, without forking the codebase. When a section was done, we
pushed the code into production. Development could continue on the
application, as the same underlying code was driving both switched and
non-switched views. 

It also meant that, if something urgent landed, the
team could be pulled *off* the redesign project at the end of a section
without worrying about their work going stale. Once something was in
production, it was done done. 

By having the code in production sooner,
we could enlist a team of intrepid beta testers (you know who you are)
who could hammer on our work far earlier in the process. We also gained
great feedback early enough to either do something about it before
launch, or understand better where we needed to be headed further down
the line. 

Thanks to all of this, throwing that switch mid-week,
mid-morning was a no-brainer. We *knew* everything worked (as well as
anyone can ever know that) and we *knew* we could back out trivially if
everything went wrong. 

SHIP IT.

But what about…?
================

It can’t all be wine and roses though. There are some legitimate
concerns about taking this sort of “all in” approach.

Performance
-----------

At launch, we have a codebase littered with `has_feature?` checks.
Literally, every view in the app checks to see if the redesign is
switched on. Doesn’t this impact on performance? 

It does, but minimally.
Our `Flip` system stores all the switches in
[memcached](http://memcached.org), so certainly no end users will be
able to see any slowdown.

Testing
-------

How do you go about testing this? We chose to brute-force it, by simply
duplicating our view tests, running them once with the switch, once
without. We experimented with only testing the changes, but the problem
with that approach is that you lose coverage. 

One of the advantages we
gained by approaching this project with feature switches was that we
could continue to enhance the existing application while the redesign
was underway: we didn’t have to feature-lock a section while it was
being redesigned. That meant we needed reliable coverage of both
“before” and “after” views, and we needed to know if work on one had
been replicated on the other.

It feels like an ugly solution, but the
reality was that we were really running two separate view-layers, and so
doubling the test coverage makes sense. Besides, we knew we were going
to tidy everything up.

Housekeeping
------------

Speaking of which, we now have the task of going through and pulling out
the switches (and forked tests) so that the app simply has one code path
through the views. This is a bit of a pain, but we’ve decoupled this
completely from the act of deploying code to our users. The extraneous
code isn’t impacting on performance, and because each section is wrapped
in its own separate feature switch, we can tackle the clean up in
manageable chunks.

Conclusion
==========

Feature switches played a key role in allowing us to both develop and
deploy the most complex change to the app we’ve undertaken so far. Just
about everything we now deploy (small or large) gets wrapped in a
switch, as the wins are just too good to ignore. 

Hopefully, we’ll get
our Flip module open-sourced in the future, but the truth is that the
code inside it is simplicity itself. You don’t need some fancy gem: you
just need a model, a helper, and a cache.

Have at it.
