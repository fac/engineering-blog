---
:title: FreeAgent's AV Adventure Continues - 12 Months On
:created_at: 2016-01-19 12:00:00.000000000 +00:00
:kind: article
:tags:
- AV
:author: dave
:slug: freeagent-av-adventure-continues
---

It has been exactly a year since my [first blog post](http://engineering.freeagent.com/2015/01/19/freeagent-av-adventure) about FreeAgent’s AV Adventure. A lot has changed since then so I’d love to bring you up to speed.
For those who haven’t read the original blog post, I’d strongly encourage you to; it outlines our business requirements and gives some background on our earlier AV experiments.

When you last saw our boardroom, it was equipped with four desktop boundary microphones, a handheld radio mic and an accompanying wireless lapel.
All of the above fed into a 12-channel mixing desk which, when correctly configured, allowed our remote staff to join boardroom meetings and listen in on wider team gatherings.

Back in September we moved into a new office, allowing us to redesign our boardroom from the ground up. This was a great opportunity to design the space with AV in mind, investing in the correct technology so that remote and office-based staff could have a seamless experience from the start.

One of our biggest advances, at least from an audio perspective, is the replacement of our desktop boundary microphones with four [Polycom HDX](http://www.polycom.co.uk/products-services/hd-telepresence-video-conferencing/realpresence-accessories/realpresence-accessories-ceiling-microphone-array.html) hanging mics. These discreet little devices look like golf balls hanging from the ceiling, though each compact unit houses four highly sensitive digital microphones, arranged in a circle to give a 360° of audio pickup.

![Kitchen Polycom HDX](/assets/images/2016/01-av-adventure-continues/Polycom_HDX.jpg){:.aligncenter}

Two of our hanging mics are suspended within the boardroom, giving 8 digital mics to play with, while the remaining units hang within our dining area. The dining microphones are usually muted, only enabled during all-hands meetings when our boardroom opens and the team spills across the dining area floor.

All four of our hanging microphones daisy chain together using CAT6 network cables, leading back to a [Polycom C16](http://www.polycom.co.uk/products-services/voice/conferencing-solutions/installed-audio/soundstructure-c-series.html) controller. This clever little device takes the digital signals from the hanging microphones and converts them to analogue XLR, which can be fed into our faithful [mixing desk](http://www.music-group.com/Categories/Behringer/Mixers/Small-Format-Mixers/X2442USB/p/P0A0M).

![You Have The Conn](/assets/images/2016/01-av-adventure-continues/The_Conn.jpg){:.aligncenter}

The value of the C16 controller goes far beyond its ability to convert digital to analogue - it can be considered as a Sound Engineer in a box, performing all sorts of processing to the digital signals prior to output via XLR. For example, by monitoring the input volume from each individual microphone, the C16 can identify and isolate a single speaker within the room, increasing the sensitivity of the surrounding mics while simultaneously lowering the volume of microphones further afield.

Using the Polycom SoundStructure Studio software, we’ve boosted the mics orientated towards the (more distant) breakfast bar, where people often congregate and pose questions during team meets. Furthermore, we’ve logically isolated the boardroom and kitchen microphones after discovering that loud noises, such as clattering plates in the kitchen, were interfering with mic levels in the boardroom; less than ideal for lunchtime meetings.

So far we’ve spent a lot of time discussing the Audio (A) element of our AV setup, but what about the Video (V) elements, too? Our new boardroom sports 4 x wall-mounted, 1080p [Logitech C920](http://www.logitech.com/en-gb/product/hd-pro-webcam-c920) webcams. These cameras feed back (via active USB 2 extension cable) to a Mac Mini that we affectionately refer to as “The Cameraman”. This computer runs some clever software called [ManyCam](https://manycam.com) which, as the name implies, allows you to take video from multiple camera inputs and to seamlessly switch between them using ManyCam’s controller interface. ManyCam allows each camera to be zoomed, artificially panned/tilted and transitioned to/from via a range of effects. Furthermore, we can overlay “lower-third” graphics to the master output, handy for introducing speakers and displaying messages to remote viewers.

![The ManyCam Interface](/assets/images/2016/01-av-adventure-continues/ManyCam.png){:.aligncenter}

Our final major investment was in an [8X8 Matrix Switch](http://www.cypeurope.com/store/store/app/product/PU-8H8HBTE). This clever device allows us to route up to 8 different HDMI inputs to 8 different output devices, such as wall-mounted TVs and/or video projectors. By adjusting the routing on the matrix switch, we can display presentation slides on both our boardroom projector and wall-mounted television in the kitchen. During another meeting, the same screens can be used to show participants in a Skype conversation or even the displays from office visitors (who simply plug their devices into floor boxes in the boardroom).

The best feature of the Matrix Switch is that it’s constantly listening for commands on the network, allowing office staff to toggle between different routing presets using [DemoPad](http://demopad.com) software installed on our iPad. When a meeting has finished, anyone can safely power-down the TVs and projector by simply selecting the “All Off” preset on the DemoPad homescreen.

![Our Completed Boardroom](/assets/images/2016/01-av-adventure-continues/Boardroom.jpg){:.aligncenter}

We’re very proud of our new boardroom and will happily bend the ear of anyone who’ll listen. Even with all of this new technology, AV is a constantly moving target and we’re still working hard to improve the quality, usability and reliability of our setup. Keep an eye out for future updates about our configuration and please do share your experiences, positive or negative, when it comes to AV hardware and software solutions.
