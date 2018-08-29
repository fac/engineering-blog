---
:title: Hack Days Feb 2015
:created_at: 2015-02-27 13:48:00.000000000 +00:00
:kind: article
:tags:
- hack-days
:author: olly
:slug: hack-days
---

We've been running Hack Days at FreeAgent for a few years now. Twice a year everyone in the company takes a two-day break from their normal work, small project teams are formed and magic happens. Yesterday afternoon, after two days of hard work, the team got together and watched 21 (!) demos. That's far too many to write about in detail, so here are a few highlights. Let us know what you think!

## Piggybot

![PiggyBot](/assets/images/2015/01-hack-days/piggybot.png)

A year ago one of our Hack Days teams got together to write an app aimed at helping small children manage their pocket money and savings. The outcome was Piggybot (internal codename: WeeAgent - arf, arf) and it was really fun. As with many hacks, the project was put aside and we didn't do anything further with it. This year a new team got together to apply a bit of polish to the app, make it mobile friendly, rewrite the front-end code in [React.js](http://facebook.github.io/react/) and launch it to the world. You can check out PiggyBot at [www.piggybot.com](http://www.piggybot.com).

If you'd like to see how we implemented it, you can [take a look at the code on Github](https://github.com/fac/piggybot). If you fancy forking the repo and sending us some pull requests, that would be fantastic - there's a lot that still needs some polish! 

You can [follow PiggyBot on Twitter](https://twitter.com/mypiggybot).


## Product Timeline

It’s crazy to think how dramatically FreeAgent has changed since we launched over 8 years ago. We’ve added tons of new features, redesigned (and redesigned again!) practically every part of the app and tirelessly slogged away to improve the technical architecture and infrastructure of the FreeAgent you know and love today. We’ve come a long way, baby.

For this project we wanted to put together a timeline of those product changes, great and small, that have happened over the years, to serve as a comprehensive document of the evolution of FreeAgent and a useful way to keep users informed of new updates in the future.

![FreeAgent Timeline](/assets/images/2015/01-hack-days/timeline.png)

There’s still a little bit more polishing to do, but you’ll see this appear on our website soon, we hope!

## "I'm on Skype" light

Headphones are a part of everyday life here at FreeAgent: we listen to music while we work, we tune in to webinars and we watch the occasional funny cat video (who doesn't?). We also use headphones when we're on Skype calls with clients and partner organisations, but sometimes it's hard for our colleagues to tell that we're actually on a call!

One team decided to develop a solution to this problem for Hack Days: a USB light that turns on automatically when a Skype call starts, letting colleagues know that the user is having a conversation and shouldn't be disturbed. A number of different prototypes were built using [Blinksticks](http://www.blinkstick.com/) together with some [Adafruit Neopixels](https://www.adafruit.com/category/168). Then using the Skype Desktop API it was possible to change the colour of the lights depending on the call status.

![I'm on Skype!](/assets/images/2015/01-hack-days/on-skype.jpg)

We think the "I'm on Skype" light is pretty awesome - check out the team's product launch video to find out more!

<div id="wistia_of398jo3xj" class="wistia_embed" style="width:500px;height:281px;margin-bottom:30px;"><div itemprop="video" itemscope itemtype="http://schema.org/VideoObject"><meta itemprop="name" content="OnSkype Light" /><meta itemprop="duration" content="PT57S" /><meta itemprop="thumbnailUrl" content="https://embed-ssl.wistia.com/deliveries/e61f26b31a7b370612f5d7a6fdb73e265ad9157e.bin" /><meta itemprop="contentURL" content="https://embed-ssl.wistia.com/deliveries/a4e66e5359896be3c53ae93745be8352a08079eb.bin" /><meta itemprop="embedURL" content="https://embed-ssl.wistia.com/flash/embed_player_v2.0.swf?2013-10-04&autoPlay=false&banner=false&controlsVisibleOnLoad=true&customColor=7b796a&endVideoBehavior=default&fullscreenDisabled=true&hdUrl%5B2pass%5D=true&hdUrl%5Bext%5D=flv&hdUrl%5Bheight%5D=720&hdUrl%5Bsize%5D=19040093&hdUrl%5Btype%5D=hdflv&hdUrl%5Burl%5D=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2F448ffe3b2e291346fbeda631f8bf24f9aa53dcc4.bin&hdUrl%5Bwidth%5D=1280&mediaDuration=57.628&playButtonVisible=true&showPlayButton=true&showPlaybar=true&showVolume=true&stillUrl=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2Fe61f26b31a7b370612f5d7a6fdb73e265ad9157e.bin%3Fimage_crop_resized%3D500x281&unbufferedSeek=false&videoUrl=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2Fa4e66e5359896be3c53ae93745be8352a08079eb.bin" /><meta itemprop="uploadDate" content="2015-02-26T15:31:28Z" /><object id="wistia_of398jo3xj_seo" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" style="display:block;height:281px;position:relative;width:500px;"><param name="movie" value="https://embed-ssl.wistia.com/flash/embed_player_v2.0.swf?2013-10-04"></param><param name="allowfullscreen" value="true"></param><param name="bgcolor" value="#000000"></param><param name="wmode" value="opaque"></param><param name="flashvars" value="autoPlay=false&banner=false&controlsVisibleOnLoad=true&customColor=7b796a&endVideoBehavior=default&fullscreenDisabled=true&hdUrl%5B2pass%5D=true&hdUrl%5Bext%5D=flv&hdUrl%5Bheight%5D=720&hdUrl%5Bsize%5D=19040093&hdUrl%5Btype%5D=hdflv&hdUrl%5Burl%5D=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2F448ffe3b2e291346fbeda631f8bf24f9aa53dcc4.bin&hdUrl%5Bwidth%5D=1280&mediaDuration=57.628&playButtonVisible=true&showPlayButton=true&showPlaybar=true&showVolume=true&stillUrl=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2Fe61f26b31a7b370612f5d7a6fdb73e265ad9157e.bin%3Fimage_crop_resized%3D500x281&unbufferedSeek=false&videoUrl=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2Fa4e66e5359896be3c53ae93745be8352a08079eb.bin"></param><embed src="https://embed-ssl.wistia.com/flash/embed_player_v2.0.swf?2013-10-04" allowfullscreen="true" bgcolor=#000000 flashvars="autoPlay=false&banner=false&controlsVisibleOnLoad=true&customColor=7b796a&endVideoBehavior=default&fullscreenDisabled=true&hdUrl%5B2pass%5D=true&hdUrl%5Bext%5D=flv&hdUrl%5Bheight%5D=720&hdUrl%5Bsize%5D=19040093&hdUrl%5Btype%5D=hdflv&hdUrl%5Burl%5D=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2F448ffe3b2e291346fbeda631f8bf24f9aa53dcc4.bin&hdUrl%5Bwidth%5D=1280&mediaDuration=57.628&playButtonVisible=true&showPlayButton=true&showPlaybar=true&showVolume=true&stillUrl=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2Fe61f26b31a7b370612f5d7a6fdb73e265ad9157e.bin%3Fimage_crop_resized%3D500x281&unbufferedSeek=false&videoUrl=https%3A%2F%2Fembed-ssl.wistia.com%2Fdeliveries%2Fa4e66e5359896be3c53ae93745be8352a08079eb.bin" name="wistia_of398jo3xj_html" style="display:block;height:100%;position:relative;width:100%;" type="application/x-shockwave-flash" wmode="opaque"></embed></object><noscript itemprop="description">OnSkype Light</noscript></div></div>
<script charset="ISO-8859-1" src="//fast.wistia.com/assets/external/E-v1.js"></script>
<script>
wistiaEmbed = Wistia.embed("of398jo3xj", {
  videoFoam: true
});
</script>
<script charset="ISO-8859-1" src="//fast.wistia.com/embed/medias/of398jo3xj/metadata.js"></script>

## FreeAgent UI Pattern Library

Anyone who has tried to maintain a UI style guide over long periods of time will know of the pain in keeping it in sync with your application. A UI style guide is often the first thing that gets swept under the rug. Before you know it, inconsistencies start to creep in and you're drowning in dropdown menus all built in subtly different ways. What a nightmare. The reason this typically happens is that UI style guides are often just simple static HTML. However, wouldn't it be dandy if you could have a [living breathing style guide](http://ianfeather.co.uk/a-maintainable-style-guide/)? One where changes in your styles would automatically update your style guide and applications in one full swoop? That would be great.

So thats what we did. We wanted to experiment whether we could extract the core styles to an [asset gem](http://zurb.com/article/814/yetify-your-rails-new-foundation-gem-and-) that would automatically be pulled into the FreeAgent application and build out a separate style guide application. We wanted to find out how coupled our styles were. Would our existing workflow just work? Would less engineering-minded designers understand how to update our core styles?

These are all questions we wanted to answer and are still answering. What? You didn't expect us to spill all the beans just yet, did you? Don't worry though, because we'll follow up soon with a more detailed post on our findings.

![Pattern Library](/assets/images/2015/01-hack-days/pattern-library.gif)

## Code Camp

Last year during our Hack Days, we ran an internal code camp to give a taster session of Ruby to those who had never written code before, or those who wanted a programming refresher. This year, there was lots of interest in going over the basics of HTML and CSS. These basic building blocks of the web are fun to get started with, and are broadly useful for all FreeAgents, from support to communications to business development.

There [are](http://railsgirls.com/materials) [a](http://docs.railsbridge.org/docs/) [number](https://www.girldevelopit.com/materials) [of](http://www.codecademy.com) [great tutorials](https://www.khanacademy.org/hourofcode) [and initatives](http://hourofcode.com/us/resources) that have been developed over the last few years, as part of the general trend of trying to increase the diversity of the tech workforce. Since we were able to run the camp in a workshop style, we decided to use the [introductory HTML tutorials](https://www.girldevelopit.com/materials/html-intro) from Girl Develop It. With these presentations as a starting point one mentor helped five intrepid students built their own static websites in one day, complete with CSS designs, images, and font-size jumping hyperlinks.

The second day saw the same group design and build an internal fire safety website, with an alarm button, Google Maps links and a navigation side page. If you want to run your own code camp, you should! Teaching is a great way to learn new things, and learning, it turns out, is lots of fun.

![Code Camp](/assets/images/2015/01-hack-days/code-camp.png)


## Droborg

[Droborg](https://github.com/afcapel/droborg) is our replacement for the good old [Jenkins](http://jenkins-ci.org). We've been using Jenkins for years, but over the time we've found it hard to use and manage automatically with tools like Puppet. Our new replacement is a very simple Rails app, but is integrated with Github and allows us to distribute our builds between multiple worker nodes. In the near future we also want it to handle our deployments.

![Droborg](/assets/images/2015/01-hack-days/droborg.png)


## Can I Claim It?

FreeAgent has published a ton of great information for small businesses, so in an effort to make this more widely available to people, one of our teams put together a mobile app version of our allowable expenses guide.

As we're already working hard on an official FreeAgent mobile app we had some code and UI elements that we could reuse. Add to this a few more custom React components and the content of our guides (plus a bit more metadata to allow us to index the content in Elasticsearch) and we managed to build a pretty much complete (albeit simple) iOS app in 2 days.

We need to do some more testing and sand down the rough edges, but you can expect to see this hitting the iTunes store sometime soon. Oh, and it turned out that building an Android version was really easy too, so it should be in the Google Play store around the same time.

![Can I Claim It?](/assets/images/2015/01-hack-days/can-i-claim-it.png)


## Bank Feed Quiz

Last hack days a couple of us got together to make a little app to produce 'decision trees' to better present some of the infographics we use on the blog. We ended up with a good proof of concept, but didn’t take it any further.

This time around we found a new use case and decided to revive the **Decision-Tree-Maker-3000** to help customers diagnose their bank feed problems without having to call the support team.

We rewrote the UI in React which was a dream to work with - even for someone with decidedly rusty Javascript skills. We added some cool new features like emailing support with your list of answers so you don’t get asked the same questions over and over again, and hiding all but the most recent question to make the page cleaner. The main achievement though was to make it embeddable in the Knowledge Base - a key thing we missed first time around. Look out for it coming to a KB article near you soon!

![Bank Feed Quiz](/assets/images/2015/01-hack-days/bank-feed-quiz.png)
