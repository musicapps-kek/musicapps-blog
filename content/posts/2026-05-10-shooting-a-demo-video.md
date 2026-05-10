---
title: "Shooting a demo video the old-fashioned way"
date: 2026-05-10
draft: false
tags: ["marketing", "video", "ai-assisted", "suno", "play-store"]
summary: "I needed a demo video for the Play Store listing. Generative video tools couldn't produce anything usable, so I shot it with two phones and a stand. Suno made the soundtrack. Total takeaway: AI video for specific app footage is still hard, filming a phone screen is harder than it looks, and Suno is fun."
---

SessionClick is on the Play Store, but the listing still has no video. I wanted a one-minute demo of the app being used in something that looks like a real stage situation. Today I tried to make one.

## What didn't work: AI video generation

I started where I usually start — see if a generative tool can do it. I tried Gemini's video generator and OpenAI's image/video tools. I gave them sample images of the app, short reference videos, and detailed instructions: a musician on stage, phone in front, finger tapping the play button, the metronome running.

The results were not usable. When I supplied screenshots of the actual UI as reference, the model would hold the layout for half a second and then drift. Maybe I was prompting wrong, but after enough attempts I decided that for very specific app footage, this isn't there yet.

## What did work: two phones and a stand

So I went "analog". I wrote a script for a one-minute video — by hand, no AI — listing the actions I wanted to show in order: open the app, pick a song, start the click, adjust the BPM, stop it. A few test runs to find a sequence that fits in 60 seconds without feeling rushed.

Filming was the hard part. The setup is one phone in a stand running the app, a second phone on another stand filming it, and a hand entering the frame to operate the screen. Things that ate time:

- Reflections on the screen. Every light in the room ends up visible on the glass at some angle.
- Lighting that's bright enough to see the hand but doesn't blow out the screen.
- Keeping the filming phone's autofocus locked on the screen instead of hunting between the screen and the moving hand.
- Filming a phone from close range while operating it by hand and keeping the framing steady.
- Preventing Moire patterns from the screen's pixel grid, which meant not just filming the screen but also adjusting the angle and distance to find a sweet spot.

I made several takes and went with the one that was "not that bad", but far from perfect. I had to force myself to stop and ship rather than keep reshooting. The result is honestly worse than I'd hoped, but it exists.

## Soundtrack: Suno

The video needed audio because the click in the app is at 122 BPM and silent video with just the click would feel wrong. I used Suno and prompted it for an instrumental pop/rock track at 122 BPM so it locks to the click in the footage.

To use the output commercially I had to subscribe — about €10 for a month. The track came back good enough that the €10 felt fine.

## Putting it together

I cut the audio and video together in DaVinci Resolve and uploaded the result to YouTube as a Short:

{{< youtube id="S0ufrsf-AzY" >}}

The link is going on the Play Store listing tomorrow.

## What I learned

- **Generative video for specific app demos is still hard.** Or I'm prompting it wrong. Either way, the path of least resistance today is not "type a prompt and get a usable demo".
- **Filming a phone screen is its own skill.** Reflections, focus, and steady framing while your own hand is in the shot — none of that is solved by enthusiasm.
- **Suno is fun.** Genuinely. Type a tempo and a vibe, get a track that locks to your video. €10 well spent for one month.
- **Where the AI helped vs. didn't on this post specifically:** writing the script, doing the filming, the colour pass in Resolve, and choosing the take were all me. AI video generation was tried and abandoned. Suno produced the music. Claude is helping me write up the result you're reading now.

Maybe one day I'll take another run at this — possibly just screen-capturing the simulator and overlaying a hand, which would sidestep most of the filming problems. For now, the video is good enough to put on the listing, and "good enough and shipped" beats "perfect and not shipped".

## Today's changelog

- New: demo video for the Play Store listing. Filmed traditionally, soundtrack by Suno, edited in DaVinci Resolve.

---

**Time spent yesterday and today:** ~half a day

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
