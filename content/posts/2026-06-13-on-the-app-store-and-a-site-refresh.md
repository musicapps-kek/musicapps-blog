---
title: "On the App Store — and a landing page that finally looks the part"
date: 2026-06-13
draft: false
tags: ["ios", "appstore", "landing-page", "hugo", "css", "design", "privacy", "ai-assisted", "claude"]
summary: "Yesterday's resubmission got approved — SessionClick is live on the App Store, so it's now a real dual-platform app. That triggered an overdue pass on sessionclick.com: an App Store badge, a privacy policy that admits iOS exists, and a redesign from dim-and-dated to bright-and-modern."
---

Yesterday's post ended with "fingers crossed the second review actually presses Play." It did. SessionClick is **live on the App Store**.

That sentence took two months and a false rejection to earn, so let me just sit with it: the metronome now ships on both Android and iOS, from one Kotlin Multiplatform codebase. The thing I sketched as "Android first, then iOS" back in April is actually a dual-platform product now.

The celebration lasted about a minute before I noticed the landing page still said **"iOS coming soon."**

## The site had quietly fallen behind

sessionclick.com was built around the Play Store launch and never updated since. So the launch handed me a punch-list:

- The hero advertised one store. There was no App Store badge, and no App Store link to put on it.
- The privacy policy was written as if the app only existed on Android.
- And — saying the quiet part out loud — the whole thing looked dim and a bit dated. Dark grey background, one shade of green, system font. Fine for a beta. Not for "we're on both stores now."

I worked through it with Claude Code in one session. A few parts were more interesting than "change the text."

### Finding the App Store URL without digging through App Store Connect

I didn't have the public store link handy. Rather than click around in App Store Connect, the fastest route is Apple's own lookup API, keyed by bundle ID:

```
https://itunes.apple.com/lookup?bundleId=eu.musicapps.sessionclick
```

That returns the canonical `apps.apple.com/app/id…` URL straight away. Into `hugo.toml` it went, next to the Play Store URL.

### Two store badges that refused to line up

This one was a genuinely satisfying little bug. On a phone, the badges stack vertically — and the Google Play badge sat noticeably indented compared to the App Store one. The left edges didn't match.

The cause: Apple's official badge SVG is edge-to-edge, but Google's official PNG has a **uniform 41px of transparent padding baked into the image** (a 646×250 file wrapping a 564×168 button). So when the two sat left-aligned in a column, Apple's hugged the margin and Google's floated inward by its invisible border.

The fix was to trim the transparent border off the Google asset so both are edge-to-edge, then size them to the same height in CSS. After that they line up on their own, stacked or side by side. The badge artwork itself is untouched — I only removed empty pixels.

### A privacy policy that admits iOS exists

The app collects nothing and sends nothing off-device, on either platform, so the spirit of the policy didn't change. But the details were Android-only and now read as inaccurate. I split the permissions section into an Android table (foreground service, notifications, vibrate) and an iOS one (background audio, haptics), and rewrote the in-app-purchase paragraph to name **both** Google Play and the Apple App Store as the payment processors, each with their own privacy policy linked. The German *Datenschutzerklärung* got the smaller version of the same treatment.

Worth noting because Apple *requires* a privacy policy URL on the listing, and it's almost certainly pointing here — so "here" had better be true.

### From dim to bright

The redesign was the fun part. I flipped the whole thing to a light theme: white surfaces, near-black text, a soft green-tinted glow in the hero instead of the flat grey. Cards got real depth — soft layered shadows and a small green accent bar over each feature, with a gentle lift on hover — instead of thin dark borders. Dark-mode visitors still get a dark theme, but a refined one via `prefers-color-scheme`, not the old murk.

The one decision I want to flag is the font. A modern sans goes a long way, and the obvious move is a Google Font. But I've been careful to claim on the *Datenschutz* page that the site makes **no third-party requests** — no Google Analytics, no embedded fonts phoning home, nothing that ships your IP to a CDN. Loading Inter from `fonts.googleapis.com` would quietly make that claim false. So I **self-hosted** Inter instead: one ~47 KB variable `woff2` in `static/fonts`, referenced from a local `@font-face`. Modern typography, zero new external requests, privacy promise intact.

### Showing the iPhone, and a second way to the stores

Two finishing touches. First, the page was wall-to-wall Android screenshots — a bit odd now that the badges send iPhone users to a differently-styled app. I added an "Also on iPhone" section with a collage of iOS screens (two portrait, two landscape, light and dark), displayed full-width like the other shots.

Second, the store badges only lived in the hero. If you read all the way down to the pricing, you'd have to scroll back up to act on it. So there's now a "Get SessionClick" call-to-action at the bottom with both badges repeated — the moment someone's convinced, the store is one tap away.

## What shipped

The site is committed and published. SessionClick is on both stores, and sessionclick.com finally looks like it belongs to an app that's actually out.

After two months of "soon," it's a strange and good feeling to delete the word.

---

**Time spent today:** ~1.5h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
