---
:title: "FreeAgent Testermonials: Getting rid of ‘QA’ and why what we call things matters"
:created_at: 2017-05-29 12:00:00.000000000 +00:00
:kind: article
:tags:
- Testing
:author: matt
:slug: freeagent-testermonials-qa-nae-mair
---

<% excerpt do %>
In this Testermonial post, FreeAgent's resident test engineer describes why we've rebranded 'QA' and why what we call things matters.
<% end %>

As mentioned in my [previous _Testermonial_](http://engineering.freeagent.com/2017/01/25/freeagent-testermonials-exit-criteria/), my only gripe when starting at FreeAgent — and a very minor one at that — was the rather entrenched use of the term **quality assurance** or **QA** in the development and release process to describe the pre-release testing phase which occurs before deploying a user story or bug fix to production.

Last week, in anticipation of the launch of an overall ‘FreeAgent Test Strategy’ in the next week or so (blog post to follow), I have taken the bold step of rebranding this release phase to the more aptly named **pre-production testing** or **PPT** phase. (We could nickname it the ‘powerpoint’ phase, if we really have to). You might be asking yourself why we should even bother changing the name of something that is arguably well understood and defined. It is by no means a pedantic or frivolous move and I would like to take the opportunity below to describe my rationale.

In 2015 I had the good fortune to attend a keynote address by the software testing thought leader, [Michael Bolton](http://www.developsense.com/), at the [Ministry of Testing](https://www.ministryoftesting.com)’s TestBash conference in Brighton. His talk explored the things that people often say in software development environments, what they might have meant to say, and why it all matters. It was something I had been thinking about for a while and his talk really resonated with me and informed a lot of how I use language when I talk about testing and software development.

George Orwell, in his essay _Politics and the English Language_, stated “[Language] becomes ugly and inaccurate because our thoughts are foolish, but the slovenliness of our language makes it easier for us to have foolish thoughts. The point is that the process is reversible.” What I believe Orwell is saying here is that because we use language as scaffolding for both our reasoning and for our understanding, it is of paramount importance, from a psychological perspective, that the language we use accurately describes what we are doing and what we are trying to achieve. Misusing language can indeed foster a culture of ineffectiveness and dysfunction, which was one of Bolton’s main points in his address.

The usage of QA in its current context is problematic for a number of reasons:
‘Quality’ is by its nature subjective; it is a measure of value to some person or people (who matter) and decisions about it are usually emotional and/or political.
Different stakeholders will perceive the same product as having different levels of quality.
At the end of the day we are trying to create value for our customers, but since we are not our customers, we are not in the position to assure that (try as we might).
Ignoring the subjectiveness of ‘quality’ for a minute, can we really assure it? Maybe we could say if you run this exact thing, with this browser, with this much memory, at this time of day it will probably work like this, but are we really accounting for all the variables in play, and does that assurance really tell us the whole story?

What then are we doing and trying to achieve in the release phase we previously called QA? We are performing a combination of verification, validation, and regression testing in order to elicit information that can help inform us and stakeholders about risks and impacts, and whether we have built the intended thing in an acceptable way — and without unexpected consequences — before we release it to our customers. Simply put: We are performing pre-production testing. 

QA _nae mair_! Long live PPT!

_This blog was [cross-posted on TestChi.li](http://testchi.li/2017/05/testermonial-getting-rid-of-qa-and-why-what-we-call-things-matters) on 29 May 2017._