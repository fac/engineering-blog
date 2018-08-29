---
:title: My Summmer Internship at FreeAgent, 2013
:created_at: 2013-09-12 13:32:00.000000000 +00:00
:kind: article
:tags:
- interns
:author: phil.hale
:slug: my-summer-internship-2013
---

This summer I left the [silver city](https://encrypted.google.com/search?q=about+aberdeen+scotland) to work for [FreeAgent](http://www.freeagent.com), makers of online accounting software in Edinburgh.

I was working with one other intern and our first project was to add [Stripe](https://stripe.com/) integration for paying invoices. There was already a structure for other payment processors so this was easy - just copy the pattern and we’re done! ...Or not. After years of minimal feedback, someone was looking at my code.

## Code Review

It’s the job of the reviewer to [look for mistakes](http://www.explosm.net/comics/2083/), but also to start a discussion about your design decisions. Some feedback I’ve gotten:

* Does that method really belong here?
* You should really be mocking this object rather than `Factory#create`
* That isn’t how you spell String

It passed review, let’s ship it! Not so fast. Unfortunately, people actually use this stuff by clicking on buttons and things. I know, right?

## Quality Assurance

There’s no dedicated QA team at FreeAgent - this responsibility is shared between the development and
the support teams - so it’s up to you document the feature using Trello, then to to find someone to 
run through the expected behaviour and try to break it.

At first this felt like a burden ('[works on my machine!](http://www.codinghorror.com/blog/2007/03/the-works-on-my-machine-certification-program.html)' - Ed.), having to wait for other people before you can get your changes out. After a while though it became obvious that the best way to get someone to help you is to help them first. I got to examine parts of the application I otherwise wouldn’t have even known about, and even found a few patterns which could be used in what I was working on.

## Deploy!

I found someone to deploy, we’re done! Well, not quite. After your changes hit production (usually alongside a few other branches that are also complete), it’s time to nervously watch for spikes in [response time](https://newrelic.com/) and any new [exceptions](https://www.honeybadger.io/) and hope you didn’t screw up.

Now you’re done :)

<hr/>

So thanks to everyone at FreeAgent for an amazing summer, it was a pleasure working with you all.

Despite being interns, every effort was made to make us feel like equal members of the team. Our code was held to the same standards, we had similar responsibilities and we were working on live projects under active use.

And I’m sure if it was one of our birthdays we would have gotten an awesome cake, too!

![Cake](http://freeagent-engineering-blog.s3.amazonaws.com/freeagent_cake.jpg){:.aligncenter}


