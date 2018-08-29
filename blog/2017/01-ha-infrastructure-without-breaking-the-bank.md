---
:title: "Running a high-availability SaaS infrastructure without breaking the bank"
:created_at: 2017-02-06 16:54:00.000000000 +00:00
:kind: article
:tags:
- Cloud
- Hosting
- Infrastructure
:author: olly
:slug: ha-infrastructure-without-breaking-the-bank
---

The cloud is commoditising web application hosting but at FreeAgent we continue to build and manage our own infrastructure, using hand-picked servers, switches and elbow grease. Why we do this is a question I commonly get asked. In this article I'll share our hosting history, how it has evolved over the last ten years, and how we now operate a high-availability SaaS infrastructure on our own hardware without breaking the bank.

## 2006

Cast your mind back to 2006. This was the year that YouTube was acquired by Google for $1.65 billion, the iPhone was an unsubstantiated rumour, [Facebook opened to the world](https://www.facebook.com/notes/facebook/welcome-to-facebook-everyone/2210227130/) (and had a mere 12 million users), [Twitter launched](https://twitter.com/jack/status/20) and Pluto's planetary status was demoted. 2006 was also the year Amazon [launched a beta](https://aws.amazon.com/blogs/aws/amazon_ec2_beta/) for a new product called "Elastic Compute Cloud", or EC2. In my mind it was this event that marked the true beginning of the modern "cloud computing" era.

At the very same time as Jeff Bezos was penning his EC2 blog post, the first FreeAgent code was being developed and soon after a physical server was provisioned in a data centre managed by Rackspace UK. This single server was to be FreeAgent's first home for our first two years of service.

## Keep your eggs in one basket 🐣

Bootstrapping our business meant paying for hosting ourselves, so we were incentivised to keep costs low. The torrent of new customers we'd hoped would arrive post [our Sept 2007 launch](https://www.freeagent.com/central/blast-off/) didn't exactly materialise at a meaningful rate, so there wasn't a driving need to build out our single server infrastructure beyond a single server for a while.

![](/assets/images/2017/01-ha-infrastructure-without-breaking-the-bank/freeagent-system-2007.png)

By 2009 we had a sufficient number of paying customers to generate enough revenue to be able to spend more on infrastructure and address some single points of failure. Despite EC2 becoming ever more popular, I decided to stick with our managed server strategy – I felt a better use of my time during this period was building the features our customers were demanding in the product. The first task was to split out our growing number of asynchronous background jobs to a separate server, and to introduce a dedicated master database server.

As we completed our Series A round in early 2010 we took things a bit further by adding a load balancer, two web servers, and an asynchronous slave database.

![](/assets/images/2017/01-ha-infrastructure-without-breaking-the-bank/freeagent-system-2010.png)

At this point, we had our first disaster recovery (DR) strategy which went something like this:

- Master DB fails
- Ensure slave DB replication has caught up
- Promote slave DB to master

This basic strategy gave us improved resilience on the web tier (load balanced web servers) as well as on the database tier (data asynchronously replicated across two hosts), but we still had all our eggs in one basket with regards to our hosting provider. If Rackspace's LON3 data centre were to go offline for an extended period of time, so would FreeAgent. And we had no plan B.

## Toying with the cloud 🌥

In 2011 Rackspace started offering their own cloud hosting product. In theory this allowed us to start provisioning servers on demand in their cloud using an API, with the advantage that these cloud servers were on the same network as our managed servers, simplifying the networking involved. This solution offered a way for us to become familiar with cloud provisioning as well as experiment with a new DR process. If we could develop a way of provisioning servers in the Rackspace cloud on demand, we could extend this to provision servers on other cloud platforms. This could form the basis of a new, improved DR strategy – spinning up a failover stack on demand in the event of our primary DC going offline.

We pushed forward with a hybrid infrastructure approach for a while, only running our less critical "job servers" on cloud VMs, but it proved to be a frustrating experience. We encountered reliability problems with cloud servers and we lacked the ability to properly drill down into system level details to really understand what was going on when we experienced failures. While we may have had a better experience with more mature cloud products, we didn't have the confidence we wanted from a cloud stack: the reliability we wanted wasn't there. Our theoretical DR process of spinning up a cloud stack on demand was in doubt.

## DR Index 📈

![](/assets/images/2017/01-ha-infrastructure-without-breaking-the-bank/dt000815.gif)
[http://dilbert.com/strip/2000-08-15](http://dilbert.com/strip/2000-08-15)

In light of this experience we took a step back to evaluate our strategy and look at the high-level picture of what our business really wanted in terms of disaster recovery. Our first exercise was to define what we called our **DR Index**, a [DEFCON](https://en.wikipedia.org/wiki/DEFCON)-like table of levels describing the absolute worst case scenario, to our idealistic best case. It looked like this:

| Level | Description | MTTR | Data loss |
| ----- | ----------- | ---- | --------- |
| DR6   | Multiple active DCs with synchronous replication | Seconds | Zero |
| DR5   | Production infrastructure replicated in secondary DC with MySQL synchronous slave | Seconds (DNS TTL) | Zero |
| DR4 | Production infrastructure replicated in secondary DC with MySQL asynchronous slave | 1 - 4 hours | Seconds |
| DR3 | MySQL asynchronous slave in secondary DC. Ability to provision a new application stack in a pre-determined secondary DC | 2 - 4 hours | Seconds |
| DR2 | Ability to provision a full stack in a pre-determined secondary DC. Manual restore of DB from offsite backup | 12 - 24 hours | Worst case, 5-10 minutes |
| DR1 | MySQL data backed up offsite, manual provisioning of replacement servers | Days | Worst case, 5-10 minutes |
| DR0 | Nothing! We are doomed. All data lost forever | ∞ | Unrecoverable |

This was a revealing and humbling exercise to undertake. Fortunately we have never sat at **DR0**. We have always had a paranoid approach to data, and we have taken DB backups and stored them, encrypted, offsite from the outset. However, although we had aspirations for **DR2**, the reality was we actually remained firmly in **DR1** territory. With an MTTR of "days" and with a customer base of over 10,000 – growing by over 1000 a month – something had to be done.

## Thinking big 🌁

By 2012 our Puppet skills were rapidly improving and we had become adept at cloud provisioning. We had provisioned stacks in Rackspace Cloud, Brightbox and EC2. However, the cloud provisioning process for the growing variety of servers our increasingly complex stack required was slow and somewhat cumbersome. We had the option of keeping a cloud stack on at all times, but we were uncomfortable with the ROI in terms of cost and performance. With experienced operations engineers on the team, highly skilled in low-level OS and networking skills, we were starting to get frustrated with managed hosting and the lack of visibility we had into our underlying hypervisors.

We had a demand from the business to get us to **DR4** and an Ops team hungry to flex their skills, so we started work on a brand new co-location hosting strategy.

We started evaluating hosting providers around the London area. We specifically avoided looking at any *central* London DCs because of the huge cost of power in the city. We eventually decided upon [The Bunker](https://www.thebunker.net/) who had two geographically isolated DCs (one in Ash, Kent, the other in Greenham Common, Newbury) and offered a low latency interconnect between the two sites – a key requirement to allow efficient data replication.

Co-locating your own hardware across dual sites might sound like a huge, expensive undertaking but this really wasn't the case. It required a lot of planning, technical nous and investment in time, as well as money, but the cost of a 42U rack in each DC, the interconnect and sufficient bandwidth came in at £1500 a month. Of course we had the additional capital expenditure of 14 Dell R620 hypervisors (7 in each site), which came to around £80,000, but we had budgeted for a 5-year lifetime, and purchased a next-day, on-site warranty through Dell to quickly deal with hardware failure. Spread across 5 years, the ROI was high.

In early 2013 [we made the switch](http://engineering.freeagent.com/2013/02/07/going-underground/) with great success.

## The proof is in the pudding 🍰

We have been operating out of these two DCs now for 4 years. Below is a table of FreeAgent's availability over the past 6 years – you can clearly see a big improvement from 2013 onwards. We've been fortunate to have world class uptime for many years and no small part of this has been down to the reliability of our infrastructure.

| Year | Uptime % | Downtime |
| --- | --- | --- |
| 2011 | 99.933% | 5h 52m |
| 2012 | 99.965% | 3h 4m |
| **2013** | **99.996%** | **21m** |
| 2014 | 99.992% | 42m |
| 2015 | 99.997% | 15m |
| 2016 | 99.984% | 1h 24m[^footnote_1] |

We had successfully achieved **DR4**, we were happy, the business and our investors were happy... but by 2015 there was one thing still troubling us. Despite testing FreeAgent in staging out of both data centres, we had never actually performed a real DR test in production. We were pretty confident that should we *have* to do a test that it would be ok, but until we had *proved* that it works, [we could never be 100% sure](https://mobile.twitter.com/garybernhardt/status/826657568437129216).

After a not-insignificant amount of planning, in March 2015 we embarked on our journey into the unknown, with confidence high but with an equally large amount of trepidation:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">We will be performing a full disaster recovery test between 2000-2359 GMT on Friday.<br>More details on our status blog: <a href="http://t.co/XwjQAQAoxw">http://t.co/XwjQAQAoxw</a></p>&mdash; FreeAgent Operations (@FreeAgentOps) <a href="https://twitter.com/FreeAgentOps/status/578206200782843904">March 18, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

As luck – or, as I prefer to think, good planning – would have it, it worked!

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">All done for the night now. Here is the obligatory graph showing the traffic moving between data centres :) <a href="http://t.co/9RbBfNaURo">pic.twitter.com/9RbBfNaURo</a></p>&mdash; FreeAgent Operations (@FreeAgentOps) <a href="https://twitter.com/FreeAgentOps/status/579061343527018496">March 20, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Total failover from one DC to another in production with virtually zero downtime (see above uptime table). Our team was incredibly proud of what we'd achieved, and rightly so.

We have continued to run these DR failover tests in production every quarter since, and will continue to do so in the future.

## But what about the cloud? ⛅

We're extremely happy with our current strategy. We're running our own private cloud, we have a solid, well-tested DR process, and by running [SmartOS](https://www.joyent.com/smartos) as our hypervisor, we're able to squeeze the most out of the compute resource across our DCs. The business is happy, not only because of the DR success, but also because we've managed to keep the cost of our production infrastructure to below 2% of revenue. It's difficult to compare this with what running FreeAgent on a 100% cloud stack would cost since we would take different architectural approaches in each situation. However, what's clear is that our infrastructure spend is very low by modern SaaS standards and we've achieved it without cutting corners. We also managed to do all this with only 3 full-time operations staff, who are responsible for many more things than managing the bare metal, which for the main part largely takes care of itself.

Despite being all-in with co-located bare metal, we're still big fans of the cloud. AWS S3 has been a key component in our architecture since the early days for storing customer receipts, amongst other things, and we store terabytes of data on there. We use Route53, we use EC2 for a variety of internal prototyping of new apps and services (although, we are now starting to replace this with [Triton](https://www.joyent.com/triton)) and we are experimenting with AWS Redshift as part of our ongoing data warehousing strategy.


## The future 🤖

Although we're happy with our current infrastructure, we know we can do better. From a DR point of view, we want to reach the pinnacle of **DR6**, which involves running multiple active data centres in an always-on configuration. This will remove the need for manual 'DR flips', and will allow us to survive a complete data centre outage without manual intervention.

This is a significant and ambitious next step, but it's one we hope to realise in 2018.

<span style="font-size: 4em">🚀</span>

[^footnote_1]: The eagle-eyed among you may have spotted we failed to achieve 'four nines' availability in 2016. This was in large part down to the [Dyn DNS cyberattack](https://en.wikipedia.org/wiki/2016_Dyn_cyberattack) which affected us in October 2016.
