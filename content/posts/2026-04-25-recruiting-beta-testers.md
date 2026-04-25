---
title: "Hunting for 12 Android testers"
date: 2026-04-25
draft: false
tags:
  [
    "beta",
    "google-play",
    "closed-testing",
    "marketing",
    "musora",
    "ai-assisted",
    "claude",
    "gemini",
  ]
summary: "Closed testing on Google Play needs 12 testers running the app for 14 days before I can promote to production. Most musicians I know are on iOS. Today was about lowering the friction to join: a public Google Group, license-testing access for the in-app purchase, forum posts on Musora, and a prominent beta block on sessionclick.com."
---

The app is submitted, the landing pages are live, the privacy policy has its canonical home. The remaining gate before production release is Google's closed-testing requirement: **12 testers, 14 consecutive days of opt-in**, before I can promote the build. None of the previous milestones matter until that counter ticks over.

This is harder than I expected. Most working musicians I know are on iOS. Even within my own family, Android is a minority. So today wasn't about code — it was about removing every excuse not to join.

## What was in the way

A would-be tester used to have to:

1. Send me their Google account email
2. Wait for me to add it to the testers list in Play Console
3. Receive a separate opt-in link
4. Realise the in-app purchase costs real money even in beta

Four steps, one of them gated by me being awake, and a payment surprise at the end. That's a funnel built to leak.

## The Google Group switch

Play Console lets you point closed testing at a Google Group instead of a hand-curated email list. Anyone who joins the group is automatically whitelisted. I created [groups.google.com/g/sessionclick-testing](https://groups.google.com/g/sessionclick-testing), opened it for public join, and pointed the closed test at it. That collapses steps 1 and 2 into one self-service action.

I also enabled **license testing** on the same group, so testers don't pay anything when they exercise the €2 unlock. Without that, asking someone to spend two euros to test my app would be a real ask — small money, big friction.

## The landing page beta block

The hero on sessionclick.com had a generic "App is in testing — email me" notice. That email-me path is exactly the friction the Google Group removes, so the page needed to change.

**Gemini drafted the copy** for a two-step instructional block: join the group, then opt in via the Play Store testing link, with a heads-up about the same-Google-account gotcha that causes the "App not available" error. The copy was good — direct, no marketing voice, the warning right where it belongs.

**Claude built it into the Hugo template.** Specifically: a new `#beta` section between hero and features, two numbered step cards styled to match the existing dark/green theme, and an updated hero notice that anchor-links into the new section instead of opening a mail client. New CSS only for the step-number circles, the card grid, and the heads-up note — everything else reused existing variables (`--green`, `--surface`, `--radius`).

**What I did:** approved Gemini's copy, asked Claude to integrate it, will push the Hugo build once I've reviewed it in the preview.

## Posting in Musora forums

I'm a member of [musora.com](https://musora.com) — Pianote on the keys side, Drumeo on the drums side. Both have "Gear Zone" sub-forums where members talk about hardware and apps. I posted a short note in **Drum Gear Zone** and **Piano Gear Zone**, each linking to sessionclick.com.

This was a deliberate human-in-the-loop step. I am not going to spam-post the link across every music forum on the internet — both because I'd rather not, and because forum cultures vary and a tone-deaf post does more harm than silence. Musora's forums are places I actually read, with people who actually play live, which is the audience the app is for. If those two posts produce two testers, that's a sixth of the way to the gate.

## What only I could do

- Create the Google Group and configure its join policy
- Switch closed testing to the group and enable license testing in Play Console
- Write and submit the Musora forum posts in my own voice on accounts that are mine
- Decide which forums are appropriate to post in and which aren't

## What the AI did

- **Gemini** wrote the beta-block copy from my brief
- **Claude** integrated the copy into the Hugo layout, wrote matching CSS, and updated the hero notice to anchor-link to the new section

## What's next

Watch the tester count. If the Google Group + landing page + two forum posts don't get me to 12 within the coming days, the next lever is reaching out individually to persons outside my family and my band — not mass posting. The 14-day clock only starts once 12 are actually opted in, so every day a slot stays empty is a day production release moves further out.

---

**Time spent today:** ~2h, almost all of it in Play Console, the Google Group admin UI, and writing the forum posts.

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
