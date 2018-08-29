---
:title: Hack Week update
:created_at: 2012-01-11 15:50:50.000000000 +00:00
:kind: article
:tags:
- General
:author: olly
:slug: hack-week-update
---
We’re two days into our first [Hack
Week](/2012/01/09/hack-week-initial-commit/) and we’re already seeing
good progress.

Testing is a common theme being worked on by two teams. The FreeAgent
code base is fairly large and is complemented by an even larger
automated test suite, containing unit, functional and integration tests.
This test suite is a massive win for us, enabling developers to
aggressively refactor code and be confident that they haven’t introduced
any unwanted side effects by doing so. The downside to the tests is the
time it takes to run them all, which is currently \~20 minutes. That’s
20 minutes parallelised over 4 hyperthreaded cores on a beefed up i7
iMac.

We have one team looking at reducing the total run time of the test
suite, and also reducing the time it takes to execute a single test
which, due to Ruby 1.9.2 and Rails, can be frustratingly slow due to the
boot-up time, hindering
[TDD](http://en.wikipedia.org/wiki/Test-driven_development) flow.
Another team is looking at removing unused test dependencies and
refactoring test cases by removing scenarios where we’re testing things
too often or sometimes unnecessarily. We’ve already seen one particular
test case run time drop from over one minute down to four seconds!

We have a team developing a handy new web app against our new API
(currently [in beta](https://staging.dev.freeagent.com/docs)), and
another team looking at optimising floating point arithmetic, which we
do a lot of in FreeAgent as you might imagine, and we’re also
experimenting with [elasticsearch](http://www.elasticsearch.org/) as a
foundation for an app-wide search feature for FreeAgent.

Our design and front-end development team are collaborating on a new
prototype area of the app, thinking about the past, present and future
of your business.

Finally, leading on from the work we’ve been doing at [Speeding up
SSL](/2011/11/29/speeding-up-ssl/), we’re prototyping an evented and
queue-based middleware by attempting a novel approach at load balancing
web requests.

Now, back to the hack.
