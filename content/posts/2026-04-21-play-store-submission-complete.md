---
title: "Day 14: Finishing the Play Store submission"
date: 2026-04-21
draft: false
tags: ["android", "google-play", "affinity-designer", "release", "ai-assisted"]
summary: "The remaining Play Store tasks from yesterday — feature graphic, tablet screenshots, permission demo video, content rating — are done. The app is submitted for review. Reflection on how unexpectedly complex the Play Store process is in 2026."
---

Yesterday's post ended with a list of five things left to do before the app could be submitted. Today was about working through that list.

## Feature graphic

The 1024×500 px feature banner required by Play Console was designed in Affinity Designer. It reuses the SessionClick color palette and the SC monogram from the app icon, with the app name and a short tagline. No AI tools — manual layout work.

## Tablet screenshots

Play Console requires tablet screenshots if the app is distributed to tablets. I used the Android Studio emulator with a Pixel Tablet AVD running in landscape. The landscape layout already exists from earlier work, so no new code was needed. The screenshots were taken from the emulator, renamed to match the phone screenshot naming convention, and uploaded.

## Permission demo video

The `FOREGROUND_SERVICE_MEDIA_PLAYBACK` permission triggers a requirement for a demonstration video. The scenario: open the app, select a song, press Play, lock the screen, show the persistent notification in the notification shade with audio still running, return to the app, stop. Recorded as a screen recording on device and uploaded to my Vimeo Account with access via direct link, and linked in Play Console.

## Content rating questionnaire

Play Console has an interactive content rating questionnaire (IARC). SessionClick: no user-generated content, no social features, no ads, no in-app browser. Result: **Everyone** rating.

## Submission

After completing the checklist, the "Submit for review" button became active. Submitted.

## Reflections on the Play Store process

The last time I published an Android app on Google Play was about ten years ago. The process then was relatively straightforward. In 2026, it is considerably more involved:

**Store assets.** Separate screenshots are required for phone and tablet. A 1024×500 feature graphic is required. Any foreground service permission triggers a video requirement. The IARC content rating questionnaire must be completed. None of this existed at the same level of detail ten years ago.

**Minimum testing requirement.** Before a production release is possible, the app must pass through internal testing with at least 12 testers who have opted in via a special link, and the app must be installed on their devices for a minimum of 14 days. The production release track is locked until this is complete. This is a policy change designed to reduce low-quality and spam submissions — it is reasonable, but it is also a real gate that adds weeks to the timeline.

**Business verification.** Publishing as an organization (rather than an individual) requires a legal entity and a D-U-N-S number before Apple Developer enrollment is possible. On the Google side, the Play Console requires an active developer account before any of the other steps can begin.

None of this is unreasonable. But it adds up. Between Play Console setup, Gewerbeanmeldung, store assets, legal groundwork, and today's submission work, the pre-release overhead has been the most time-consuming part of the project so far — more than any single engineering task.

## What's next

The app is now in Google's review queue. Once it passes:

1. **Recruit 12 testers** — each needs a Google account, must opt in via the internal test link, and must have the app installed for at least 14 days.
2. **After 14 days and 12 testers** — the production release track becomes available.
3. **In parallel** — market analysis (still pending via Perplexity Pro, deadline 2026-05-06), and the Apple Developer track (waiting on D-U-N-S number).

---

**Time spent today:** 1h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
