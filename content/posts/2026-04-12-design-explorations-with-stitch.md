---
title: "Day 6: Exploring the UI design with Google Stitch"
date: 2026-04-12
draft: false
tags: ["design", "stitch", "ui", "android"]
summary: "A design session using Google Stitch to explore visual directions for the SessionClick main screen — plus a first proper look at project time and costs."
---

Today I spent some time exploring the UI design for the SessionClick main screen using Google Stitch. I wanted to see how different design elements and layouts would look and feel, and Stitch provided a great way to quickly iterate on ideas.

I tried two approaches: feeding the mockup I created in SnagIt into Stitch, and starting from scratch with just a text prompt describing the desired design.

## First approach: Mockup + Stitch

This is my mockup:

![KEKs mockup](/images/sessionclick-main-screen-mockup.png)

I chose the "Redesign with Nano Banana" style in Stitch. With some basic prompts and 4–5 rounds of iteration, I got this result:

![iOS Stitch mockup V1](/images/stitch_ios_metronome_playlist.png)
![Android Stitch mockup V2](/images/stitch_metronome_playlist-end.png)

I also instructed the model to use a combined play/stop button and move it to the lower right corner, which is a common placement for such controls in mobile apps. During testing the app, I found that having the play button on the left side does not feel natural.

## Second approach: Text prompt + Stitch

I took the UI element description from my [concept document](/concept/) and fed it into Stitch with only a brief introduction but without a mockup. I chose the "Thinking with 3.1 Pro" model for this one.

The first result was "quite interesting," but not really what I had in mind:

![Stitch result V2 - first try](/images/stitch-v2-01.png)

After a few iterations and more specific prompts, I got this:

![Stitch result V2 - improved](/images/stitch-v2-02.png)

This was an interesting look into a possible future, but in the end I will continue with implementation using a "function-first" approach and will not try to replicate the Stitch designs directly. I will use them as a source of inspiration rather than a blueprint.

## Tracking time and costs

The second task today was less creative but just as necessary: getting a clear picture of what this project is actually costing in time and money.

I exported a detailed report from Toggl Track covering all sessions since April 8 and tracked down all the financial information — domain setup fees, monthly hosting, and the Claude Code subscription — in my Obsidian project notes.

The result is a new [Resources](/resources/) page on this blog. It shows:

- **Time invested** — a session-by-session table from Toggl, currently at ~11h 09min total
- **One-time costs** — domain setup fees: €5.97 in total
- **Monthly costs** — domain hosting plus Claude Code: €25.29/month, comfortably under the €40 budget ceiling
- **Tools** — a full list of everything used so far, from Android Studio and Xcode to Obsidian, SnagIt, and the various AI tools

The page will be updated as the project progresses. Having it in one place makes it easier to answer the question I keep getting: "what does something like this actually cost?"

---

**Time spent today:** 1h 30min

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
