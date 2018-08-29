---
:title: Staying On Track
:created_at: 2013-05-28 15:29:00.000000000 +00:00
:kind: article
:tags:
- ruby-on-rails
:author: olly
:slug: staying-on-track
---

<% excerpt do %>
We were fortunate to start FreeAgent at a point in time when things had just 
taken a turn for the better for web developers. Prior to 2005, I was writing 
web apps in Java using technologies such as [Spring](http://static.springsource
.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html), [Velocity](
http://velocity.apache.org/) and (sorry for swearing) [Struts](http://struts.
apache.org/release/1.3.x/index.html). 

Then along came [Ruby on Rails](http://rubyonrails.org/).
<% end %>

Rails was perfect for our bootstrapped startup. Its conventions allowed us to 
focus on rapidly developing the core functionality and front-end UX of our app 
(yes, with hindsight at the cost of ignoring 'purer' [OOD](http://en.wikipedia.org/wiki/Object-oriented_design) approaches... but 
that's another story) without worrying about configuration. Writing Ruby made 
me happy. The Ruby/Rails community encouraged us to write TDD. I was 
productive. We shipped a complex web app in less than a year.

But as any seasoned Rails developer will tell you, it's not all sweetness and 
light. Especially when it comes to upgrading Rails.

## A brief history of Rails in FreeAgent 

Looking at our git history (which we have maintained since the very first 
commit in 2006) you can see the major Rails upgrades:


    2007-02-14: Updated rails to edge
    2007-03-16: Updated vendor/rails to r6419 (Rails release 1.2.3). Locked vendor/rails.
    2008-01-01: WiP 2.0 Upgrade - Remove vendor/rails - working off the 2.0 gem
    2008-07-07: Monster check-in.  Now running with the Rails 2.1 gem.   You need to 'sudo gem install rails'
    2009-05-19: Updating rails again
    2009-07-24: Used 'rake rails:freeze:edge RELEASE=2.3.3" to vendor Rails 2.3.3.
    2011-07-07: Merge branch 'spike/rails-3.0.9' into master
    2012-06-27: Merge branch 'spike/rails-31' 
    2013-05-28: Merge branch 'rails-3.2'

We started back in 2006/early 2007 on Rails 1.1, quickly moving onto 1.2 then 
going through a major upgrade pretty much every year since (clearly we've also 
improved our git workflow, branch naming and commit messages over the years).

As you can see from the final commit we have finally moved through to **Rails 
3.2**, the latest stable version. 

<img class="center" src="http://freeagent-engineering-blog.s3.amazonaws.com/excite.gif" width="500" height="280" />

## Why bother upgrading?

There are many reasons why you want to keep up with framework releases, 
especially in the Rails world: security, compatibility (namely [Gems](http://
guides.rubygems.org/)), functionality (e.g. [asset pipeline](http://guides.rubyonrails.org/asset_pipeline.html)), team morale (new! 
shiny!!), performance. Some frameworks maintain full backwards-compatibility, 
whilst Rails sacrifices this to stay lean(ish) and opinionated. 
Over the long term this is a good thing but it can make upgrading *really*
painful. Especially as your code base grows as much as ours has.

For the last two years we've been running on Rails 3.0 which itself was a 
pretty major undertaking, especially when you're crazy enough to combine it 
with a move from Ruby 1.8.7 to Ruby 1.9.2 like we did. However in the last few 
months there have been a spate of Rails security patches which resulted in 3.0.
x being excluded from future patches.

Via [an annoucement on the Rails Security group](https://groups.google.com/forum/#!msg/rubyonrails-security/G4TTUDDYbNA/wamRH2Tijz0J):

<pre>
## Security issues:

The current release series and the next most recent one 
will receive patches and new versions in case of a security 
issue. 

Currently included series: 3.2.x, 3.1.x

## Severe security issues:

For severe security issues we will provide new versions as 
above, and also the last major release series will receive 
patches and new versions. The classification of the security 
issue is judged by the core team.

Currently included series: 3.2.x, 3.1.x, 2.3.x
</pre>

We take security seriously so this made us to bite the bullet and 
take the time to upgrade to Rails 3.2.

## What about Rails 3.1?

The above git history clearly shows that we did go live with Rails 3.1 back in 
June 2012 but we saw such bad performance degradation that we pulled it after 
an hour and never returned. 

<p>
<img class="center" src="http://freeagent-engineering-blog.s3.amazonaws.com/rails-3.1-cpu.png" width="500" height="292"/>
</p>

<p>
<img class="center" src="http://freeagent-engineering-blog.s3.amazonaws.com/rails-3.1-campfire.png" width="408" height="77" />
</p>

We didn't want to go through this again.

## How we approached the upgrade this time

This user story explains our approach:

    As a developer
    I want to backport all known changes for Rails 3.1/2 into master
    In order to make the upgrade rather less painful

We tracked each of these backports as we do with all our development work: as 
single feature branches that are branched from master, developed, peer 
reviewed, QA'd then merged into master and deployed into production. 
Here's a sample list:

<table class="basic">
<tr>
  <th width="80%">Trackers</th>
  <th>Status</th>
</tr>
<tr>
  <td>Replace <code>RAILS_ROOT</code> by <code>Rails.root</code></td>
  <td></td>
</tr>
<tr>
  <td>Use callbacks to set defaults instead of overriding initialize method</td>
  <td></td>
</tr>
<tr>
  <td>Rename `open` scope on Bill and Invoice</td>
  <td></td>
</tr>
<tr>
  <td>Change tests to use <code>Rack::UploadedFile</code></td>
  <td></td>
</tr>
<tr>
  <td>Remove deprecated methods</td>
  <td></td>
</tr>
<tr>
  <td>Remove acts_as_list and human_attribute_override plugin</td>
  <td></td>
</tr>
<tr>
  <td>Call constants with explicit scope</td>
  <td></td>
</tr>
<tr>
  <td>Remove rails 2.3 style plugins</td>
  <td></td>
</tr>
<tr>
  <td>Avoid usage of errors.reject! since the method is undefined for 
    <code>ActiveModal::Errors instance</code></td>
  <td></td>
</tr>
<tr>
  <td>Gemify asset_packager plugin</td>
  <td></td>
</tr>
<tr>
  <td>Move calendar_helper to lib</td>
  <td></td>
</tr>
<tr>
  <td>Gemify open_id_authentication plugin</td>
  <td></td>
</tr>
<tr>
  <td>Remove deprecated <code>ActiveSupport::Memoizable</code></td>
  <td></td>
</tr>
<tr>
  <td>Ensure that mailers use 
  <code>mail(:subject => ..., :recipients => ...)</code></td>
  <td></td>
</tr>
<tr>
  <td>Remove deprecated <code>has_key?</code> method</td>
  <td></td>
</tr>
<tr>
  <td>Passing a template handler in the template name is deprecated. 
  You can simply remove the handler name or pass 
  <code>render :handlers => [:erb]</code> instead.</td>
  <td></td>
</tr>
<tr>
  <td><code>ExceptionMiddleware</code> doesn't rescue 404 and 500 errors in Rails 3.2</td>
  <td>done but can't be backported</td>
</tr>
<tr>
  <td>Deprecated <code>ActionController::UnknownAction</code> in favour of 
  <code>AbstractController::ActionNotFound</code></td>
  <td>fixed - not backported</td>
</tr>
</table>


In addition to this, we created a new `rails-3.2` branch on which we applied 
changes that couldn't be backported. We kept this in sync with `master` (the 
mainline branch we deploy from) on a daily basis. This took about three 
weeks to work on and get into a QA-able state. From there on in were we pretty 
much home and dry.

## Canary servers and a big Rails 3.1+ gotcha

Learning from our failed Rails 3.1 release, this time 
around, in fear of epic performance degradation, we wanted to stage the roll 
out of Rails 3.2 using a canary server. This seemed a sensible idea, 
temporarily shifting a percentage (in our case 25%) of traffic to an upgraded 
web server and watching the graphs. What we didn't expect was a bombardment of 
500 errors from the Rails 3.2 server!

Fortunately we had a solid rollback plan which meant within a matter of 
seconds we had removed the canary server from the load balancer pool and 
started to dig into the error logs.

So what was going on? As it turns out in Rails 3.0 the [FlashHash](https://github.com/rails/rails/blob/3-0-stable/actionpack/lib/action_dispatch/middleware/flash.rb#L69) class
was derived from `Hash`. In Rails 3.1 and beyond 
[FlashHash](https://github.com/rails/rails/blob/3-1-stable/actionpack/lib/action_dispatch/middleware/flash.rb#L73) 
is *not* derived from `Hash`! Because the flash is marshalled, sessions 
containing a flash that have been generated on a Rails 3.0 server will 
cause the world to end when read from a 3.2 server and vice versa. 

Thanks Rails. Like I said, upgrades can be *really* painful!

Here's a gist that demonstrates the effect:

<script src="https://gist.github.com/lylo/5663750.js"></script>

There are [possible hacky workarounds](https://github.com/rails/rails/issues/2509#issuecomment-6162069) but this was just too risky. 
Instead we dropped the canary plan and decided to change the session key in 
our 3.2 branch and roll it out to everyone when app traffic was low. The only 
downside here is that everyone would be logged out of their session upon 
deploy. All things considered, this was a small price to pay for a safe 
release.

## The inevitable Rails 3.2 issues

Once we'd released the branch into production late on Sunday evening, we did 
see some performance issues but nothing on the scale of the previous Rails 3.1 
roll out. The first problem we noticed was a drop in the memcached hit rate:

<img class="center" src="http://freeagent-engineering-blog.s3.amazonaws.com/rails-3.2-memcached-hit-ratio.png" width="500" height="221" />

After a fair amount of head-scratching we discovered there had been a 
change to `Rack::Cache` in Rails 3.2 which left it enabled in the production 
environment. The effect of this was a memcached GET call occurring for each 
page request without a corresponding SET call. To rectify this we backported 
[a change from Rails 4](https://github.com/rails/rails/pull/7838/files):

    # Rack::Cache is enabled by default in Rails 3.1+ 
    # (and disabled in Rails 4.0+) and is skewing our 
    # memcache stats. Disabling for now.
    config.action_dispatch.rack_cache = nil

This brought the hit rate back up to 90%+. Party time!

<img class="center" src="http://freeagent-engineering-blog.s3.amazonaws.com/disco.gif" width="450" height="317" />

## The path towards Rails 4 and Ruby 2

Although the dust is only just settling on this upgrade, we were pretty 
late to the Rails 3.2 party. The [Rails 4 release](http://weblog.rubyonrails.org/2013/5/1/Rails-4-0-release-candidate-1/) 
is just around the corner and Ruby 2.0 is already in the wild 
(and [Basecamp is already running on both](https://twitter.com/dhh/status/309744999774420993)), so we plan to start experimenting with this 
combination really soon. 

I'm under no illusion this means more pain and a new chapter in our growing 
Rails War Stories book, but as they say around [these parts](http://en.wikipedia.org/wiki/Scotland),
"naething is got without pains but an ill name."

And we wouldn't want that, aye?
