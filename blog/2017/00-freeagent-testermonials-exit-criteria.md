---
:title: "FreeAgent Testermonials: Making user stories valuable with exit criteria"
:created_at: 2017-01-25 12:00:00.000000000 +00:00
:kind: article
:tags:
- Testing
- Exit Criteria
- User Stories
:author: matt
:slug: freeagent-testermonials-exit-criteria
---

<% excerpt do %>
In this post, FreeAgent's resident test engineer explores the use of exit criteria in user stories, the value they provide and how to write them (example included).
<% end %>

# Table of contents

- [Background and prologue](#freeagents-first-test-engineer)
	- [FreeAgent's first test engineer](#freeagents-first-test-engineer)
	- [Testing as an embedded practice](#testing-as-an-embedded-practice)
	- [Testermonials](#testermonials)
- [Exit criteria in user stories](#introducing-exit-criteria)
	- [Introducing exit criteria](#introducing-exit-criteria)
	- [Exit criteria wins](#exit-criteria-wins)
	- [Writing valuable exit criteria](#writing-valuable-exit-criteria)
	- [An example user story with exit criteria](#an-example-user-story-with-exit-criteria)

## FreeAgent’s first test engineer

I joined FreeAgent in June 2016 as their first hire in a dedicated test role. Coming into the organisation as the first test engineer, I was admittedly a bit nervous about what I would encounter in terms of culture, practice and general resistance to test as a dedicated function. Although never personally my experience, I had heard horror stories of testers joining organisations where they encountered minimal test procedures and occasionally experienced downright hostility from developers based on preconceptions of testers being nitpicky ‘gatekeepers’ or ‘guardians of quality’. I also knew that testers were sometimes parachuted into organisations as some kind of testing panacea following the release of a particularly nasty bug into the wild.

Thankfully FreeAgent already had a very decent testing culture in place and there had been no major bug driving the advertisement of a test engineer vacancy. I think the mere fact that this had not been the motivation for hiring me stands as testament to the kind of organisation FreeAgent is: An organisation that values the trust of its customers, and one that is proactively driven to continually improve its processes in order to deliver increasingly valuable and reliable products to them.

## Testing as an embedded practice

So, when I joined the Compliance team here at FreeAgent back in June, I was not compelled to go in with all guns blazing or to be overly critical. Rather, I had the opportunity to learn about and understand the processes and procedures that were already in place. Testing, as an activity, is something that provides the most value when it is embedded throughout the development process _and_ when it is a responsibility shared by the entire team. That is why I was pleased to discover some rather robust practices within my team (and in the engineering organisation at large). These included embedded collaborative practices such as pairing and peer review, regular pre-release sanity checks and regression testing, and a culture of writing automated test specs for every story and fix released to production. That my only initial gripe was the rather entrenched use of the term ‘QA’ — to refer to the pre-release manual testing which occurs before deploying a user story to production — is really saying something.

## Testermonials

In this series of _FreeAgent Testermonials_ — of which this post is the first — I hope to explore some of my experiences in helping to define what the role of test is at FreeAgent, to discuss the evolution of our wider organisational test strategy, and to highlight the tools and techniques we have introduced to embed testing within our entire development process. (There will undoubtedly also be a post explaining my concerns with the term ‘quality assurance’ and why semantics matter.) First up, I examine the value that we have added to user stories through the use of exit criteria.

## Introducing exit criteria

FreeAgent already had rather robust testing practices in place before I started as the first dedicated test engineer here, however there is always room for improvement. Change, however, is difficult to effect and it is a real challenge to start somewhere new and immediately start implementing adjustments to processes and procedures. My game plan has therefore been to incrementally introduce rather small adaptations into the way we work as a team so that these changes feel like natural evolutions, and the (hopefully positive) impacts can be recognised.

One of these things has been the introduction of exit criteria into the user stories we write. These criteria act like mini specifications for each user story and I believe that they help us in several ways.

## Exit criteria wins

* They help everyone on the team to have a shared understanding of the scope of a story.
* They allow us to define when a story is complete and ready to ship.
* They help us to move testing upstream where it can prevent bugs from occurring in the first place.
* They make estimation easier.
* They help reveal technical difficulties and uncover challenges.
* They ensure that we’re not missing any related stories.
* They make sure that a story is not too big, or where it might benefit from being broken up into multiple stories.
* They help uncover the implementation risk of a story and how it might be tested.

## Writing valuable exit criteria

Exit criteria certainly seem helpful, but what makes for a good set of exit criteria and how should they be approached? Below are a few of the ground rules that I have developed while experimenting with exit criteria within my team.

* They should work for your team and everyone should recognise their value.
* They should have cross-functional input into their definition, ideally using the [Three Amigos](https://www.agilealliance.org/glossary/three-amigos/) approach. In our case we try to get developer, tester and product manager input as a minimum.
* Everyone in your team should be able to understand them and ultimately when a story can be considered complete.
* They should not be prescriptive about implementation, but should concentrate on the resulting behaviour (unless the implementation is important).
* They should be unfixed and fluid as understanding of a story grows, but communication about any changes made to them while a story is in progress is key!
* They should be pragmatic and risk-based, while still addressing possible regressions.

## An example user story with exit criteria

If you are anything like me, I know you will appreciate an example of how to approach writing exit criteria for a particular story. As a team we’ve recently been developing a solution for construction industry subcontractors who use our software and are required to comply with HMRC’s [Construction Industry Scheme](https://www.gov.uk/what-is-the-construction-industry-scheme). One of the required user stories for our solution is that when subcontractors invoice their respective contractor, the invoice should indicate to the contractor how much of the invoice to withhold and pay over to HMRC on the subcontractor’s behalf. Below is a user story we wrote for this along with its exit criteria.

> **'As a contractor for a CIS user I can see CIS deductions clearly summarised on CIS invoices I receive'**

> * A **CIS user** is a user in a UK company that has CIS enabled on their account (i.e. the CIS income categories exist and access to ledger 812 is available) and the `construction_industry_scheme` feature switch is enabled.
> * A **non-CIS user** is a user not in a UK company, or is a user in a UK company that does not have CIS enabled on their account (i.e. no CIS income categories exist and access to ledger 812 is unavailable), or when the `construction_industry_scheme` feature switch is not enabled.
> * When a CIS user creates any invoice where they select a CIS deduction rate other than _Gross_, then a line directly below the **GBP Total** (and above the **GBP Due** line) should be displayed showing the total CIS deduction amount and category, e.g. 'CIS Deduction (20%)'. If the CIS income category selected for the line items is _Gross_, or the invoice is not a CIS invoice, then no CIS deduction line should display and no **GBP Due** line should display until the invoice has any payments.
> * Where the CIS deduction line is displayed, the amount displayed with it should reflect the sum of the deduction amounts calculated from every invoice line item marked with the CIS deduction rate.
> * When an invoice with a CIS deduction amount is paid, then an additional line for the payment should be displayed below the **GBP Total** (and CIS deduction amount lines, if applicable), and above the **GBP Due** line, and it should be labelled with a clickable link saying **Payment: dd mmm yy** along with the amount. The link should be to the relevant invoice payment bank transaction. Where part payments have taken place, there should be a payment line as above for each part payment.
> * The **GBP Due** line should reach zero when the sum of any payment (or part payments) and the CIS deduction amount are equal to the amount for **GBP Total**.
> * This display of CIS on invoices should work and display appropriately for all existing themes and their corresponding PDF renderings.
> * The above should not impact existing invoice setup and should not have any impact on invoices for non-CIS users.

Have you used exit criteria before? What were your findings? How do you gain a shared understanding of user stories and a definition of done? We’d love to hear about your experiences in the comments below.

_This blog was [cross-posted on TestChi.li](http://testchi.li/2017/01/making-user-stories-valuable-with-exit-criteria) on 25 January 2017._