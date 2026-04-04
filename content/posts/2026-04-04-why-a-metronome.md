---
title: "Day 1: Why I'm building a metronome app (and why it probably won't make money)"
date: 2026-04-04
draft: false
tags: ["planning", "metronome", "AI"]
summary: "Starting a solo app project with AI assistance, a 40 EUR budget, and realistic expectations."
---

I'm starting a side project: building music apps for Android and iOS. The first one will be a metronome. This blog will document the entire process — the decisions, the mistakes, and what AI tools actually contribute versus what they promise.

## Why a metronome?

A few weeks ago, I watched a professional drummer at a gig. He had a metronome app running on his phone, taped to his kit. No fancy time signatures, no accent patterns, no visual conductor — just a steady click and a flash. One beat, one sound, one light.

Most metronome apps don't work that way. They want you to configure 4/4 versus 3/4, set accent patterns, choose different sounds for the downbeat. That's useful for practice, but for a live gig, a drummer just needs a reliable pulse at the right tempo for each song.

That's the app I want to build: dead simple, rock solid timing, with a setlist feature so you can line up your songs and their tempos for a gig.

## Why this will be hard

The app stores are full of metronomes. Many are free. Some are excellent. Nobody is searching for "yet another metronome app" — so the product has to be good enough that people find it through the setlist/gig workflow angle, or it won't be found at all.

I'm a solo developer with about 30-60 minutes per day to spend on this. I'm an engineer, not an app developer by trade. My monthly budget for everything — AI tools, hosting, app store fees — is 40 EUR.

## Why I'm doing it anyway

Three reasons:

First, I use music apps every week and I have genuine opinions about what they get wrong. Cloud dependencies, subscription models, over-designed interfaces — I want an app that runs offline, costs a one-time fee, and respects the platform's design language.

Second, I want to seriously test whether AI tools can make a solo developer competitive. Not "AI wrote my app" competitive, but "AI handled enough of the grunt work that one person could ship something polished." This blog will be honest about where that works and where it doesn't.

Third, I want to find out if it's possible to earn anything at all with a small, focused app in a saturated market. My expectation is that it probably isn't — but I'd like to be proven wrong.

## The rules I'm setting for myself

- **No cloud dependency.** The app runs entirely on the device. No login, no account, no server calls while it's running.
- **No subscriptions.** One-time purchase for the paid version.
- **Native apps.** Kotlin for Android, Swift for iOS. Music apps need precise timing; web-based wrappers won't cut it.
- **Platform-native design.** The Android version should look like an Android app. The iOS version should look like an iOS app. No cross-platform design compromises.
- **Budget ceiling.** 40 EUR/month, all in.

## What's next

Over the next few days, I'll be setting up the project infrastructure: this blog (Hugo on GitHub Pages), email, development environments. Then I'll define the MVP feature set and start on the Android version.

I'll also start looking at the competitive landscape seriously — not just "what's out there" but "what do real musicians actually complain about in existing metronome apps."

If you're reading this later and the blog has more than five posts, it means I stuck with it. That alone would be worth documenting.

---

_This is post #1 of an ongoing development diary. The blog is currently private and intended as project documentation._
