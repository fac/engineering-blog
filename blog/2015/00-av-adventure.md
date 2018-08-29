---
:title: FreeAgent's AV Adventure
:created_at: 2015-01-19 15:00:00.000000000 +00:00
:kind: article
:tags:
- AV
:author: dave
:slug: freeagent-av-adventure
---

As the FreeAgent team continues to grow, we’re finding that more of our staff are based remotely, either working from home or visiting our customers on the road. As such, we’ve been working hard to lessen the divide between office staff and our teams out in the wild. It’s super-important that everyone feels connected and that we can continue to communicate effectively, as if we were all in the same room.

Being a tech company, we’ve naturally leveraged technology to solve some of our problems. Our friendly ‘DevBot’ (a robot who lives in our team chat rooms) emails the entire team whenever customer-facing changes are deployed. Our developers use the brilliant [Discourse](http://www.discourse.org) for discussing all things technical whilst the wider team use [Basecamp](https://basecamp.com) to keep in sync.

This is all well and good, but let’s be honest, sometimes you just can’t beat a face-to-face chat. With this in mind, we set about building a soup-to-nuts audio-visual (AV) solution; a system that allows staff to communicate seamlessly, regardless of the miles that divide them.

![The FreeAgent Team](/assets/images/2015/00-av-adventure/team.jpg){:.aligncenter}

## Echoes Of The Past

To understand how we reached our solution, it’s worth rolling back the clock and looking at our earlier attempts.

When I first joined FreeAgent back in 2012, our development team shared a single morning stand-up, announcing their plans for the day ahead. As is [Scrum](http://en.wikipedia.org/wiki/Scrum_(software_development)) tradition, the team gathered in a circle and projected their voices to a stool-mounted boundary mic in the middle of the room. This worked, but only to a point - the quieter members of our team were hard to hear and people often forgot that the mic was there at all.

To make the mic more noticeable we invested in a handheld radio microphone and handed it around the circle as each person spoke. This significantly improved the audio quality for the person speaking but rendered the rest of the team mute; handheld microphones are designed to amplify only their user. We had simply traded one problem for another.

![Wireless Radio Mics](/assets/images/2015/00-av-adventure/wireless_mics.jpg){:.aligncenter}

Microphone problems weren’t isolated to our morning stand-ups, they were a recurring theme in our board room meetings too. The FreeAgent board room is a versatile space that hosts weekly team meetings (~10 people) as well as monthly company catch-ups (~50 people). To accommodate for different group sizes, the room has glass panels that slide to open or close the space. As you can imagine, this is a nightmare for audio; not only are you significantly altering the acoustics of the room, you’re also changing the available floor space where microphones can be deployed.

![The Board Room Opens Up!](/assets/images/2015/00-av-adventure/open_board_room.jpg){:.aligncenter}

## How Can We Make This Better?

From our experiments with morning stand-ups, it was clear we needed a combination of ambient microphones (to capture conversation) and precision microphones (for amplifying a single presenter). All of these microphones could then be mixed down to a single signal that's broadcast to remote staff, allowing them to hear voices from anywhere in the room.
After some reading around the subject, we invested in 4 table-mounted [boundary mics](http://www.thomann.de/gb/superlux_e303b.htm?sid=3b246085552d859c6d0f3b9d8c997cfa) (for ambient coverage) and a [wireless lapel mic](http://www.thomann.de/gb/the_tbone_tws_one_a_lapel.htm) (for precise amplification). These microphones all feed into a 12 channel (24 input) [mixing desk](http://www.amazon.co.uk/Behringer-X2442USB-Xenyx-Input-Mixer/dp/B0037036QA) that connects to our Mac Mini via USB.

![Behringer Mixing Desk](/assets/images/2015/00-av-adventure/mixer.jpg){:.aligncenter}

The boundary mics were chosen due to their sensible price-point and their cardioid pickup pattern. This means that they detect sound in front of them and to their sides, but not directly behind them. This ‘blackspot’ is actually advantageous as it allows us to more accurately isolate the different sections of the board room and to mix their levels independently.

![Table Mounted Boundary Mic](/assets/images/2015/00-av-adventure/boundary_mic.jpg){:.aligncenter}

We chose the Behringer mixing desk due to it’s immediate compatibility with our Apple Mac Mini (it just appears as a USB soundcard) and for the equaliser and compressor controls available on each channel. We anticipated that due to the awkward acoustics of the board room, we’d need to fine-tune each microphone for a clear signal and reduced background noise. Finally, the 7 unused channels would leave us some headroom, should we decide to add extra microphones at a later date.

After some (actually, a lot) of testing and experimentation, we were happy with the audio quality being captured within the board room itself. Now we needed to tackle the problem where, for larger team meetings, the panels are opened and the team spill out across the reception.

Inspired by BBC Question Time, we invested in a large [boom mic](http://www.amazon.co.uk/TIMETOP-Professional-Shotgun-Microphone-Camcorder/dp/B00BWIG3BO) that could be hoisted above a team member whenever they wished to ask a question. As exciting as it was, it soon became apparent that long poles, confined spaces and walls of sheet-glass are a bad combination. As such, we reverted to our trusty handheld wireless microphone and a ‘runner’ to deliver the mic to different team members.

![Jess Models Our Handheld Mic](/assets/images/2015/00-av-adventure/handheld.jpg){:.aligncenter}

## Mic-check one, two?

As with any complex system, buying the hardware only solves half the problem. Our first weeks were plagued with teething problems as we battled to tweak volume levels, mic positioning and channel equalisation. One of our biggest problems, however, was not related to remote staff hearing the board room but rather remote staff hearing *themselves*.
When we first configured the AV system, all of our attention was focused on the signals going *in* to the mixing desk; ensuring that remote staff could hear everything that happened in the board room. But what happens when remote-staff wanted to ask a question and audio needs to come back *out*?

Originally, all audio output from our Mac Mini was fed via HDMI into our wall-mounted television. Unfortunately, the newly installed microphones could hear the television and, because they didn’t know any better, did their absolute best to amplify the voices they heard. As such, remote team members would hear an extremely off-putting echo of themselves whenever they spoke.

To mitigate this problem, we needed to make our mixing desk aware of the Mac Mini output so that it could apply echo-cancellation and avoid repeating the signal back to remote staff. Therefore, the audio output from the Mac Mini is now fed directly in to our mixing desk but, rather than being included as part of the ‘master mix’ (that’s sent to remote staff), it’s fed to a separate pair of board room speakers along an auxiliary output.

![SpeakUp In Action](/assets/images/2015/00-av-adventure/speakup.png){:.aligncenter}

Other challenges include a constant battle with our office air-conditioning (it’s quite noisy) and bandwidth fluctuations impacting audio quality.  
Finally, all the microphones in the world are no substitute for a presenter who really knows how to project their voice. As such, a colleague built a hack-day project, [SpeakUp](https://github.com/fac/speakup), that allows remote staff to rate the AV quality in real-time. If the weighted average drops below a certain threshold, an on-screen notification alerts the board room that either a microphone needs to be adjusted or a team member needs to work on their public speaking 😄

## The Road Ahead

I think we’re all in agreement, FreeAgent’s AV is in a much better place compared to where we started. That said, we've still got some fine tuning to do, covering quiet zones with extra mics, for example. We've also got an office move on the horizon so we'll have to unplug everything and start afresh. Who knows, perhaps we'll just replace it all with a [Chromebox](https://www.google.com/chrome/business/solutions/for-meetings.html)...
