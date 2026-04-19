---
title: "Day 13: Freemium gate, billing, and pre-beta cleanup"
date: 2026-04-19
draft: false
tags:
  [
    "android",
    "monetization",
    "google-play-billing",
    "kotlin-multiplatform",
    "ai-assisted",
    "refactoring",
  ]
summary: "The freemium gate and Google Play Billing integration landed today, completing Phase 2. Then a pre-beta cleanup pass: jitter readout removed, BPM range constant consolidated, open source licenses screen added, and a privacy policy written and published."
---

Today finished Phase 2 and started clearing the pre-beta checklist.

## Freemium gate

The free tier limits are 30 songs in the pool and 3 playlists. I picked the numbers; Claude recommended storing them as named constants so they're easy to change later.

The architecture follows the project rule: business logic goes in `shared/commonMain`, platform code stays in its module. Claude designed two new files:

- `FreemiumConfig` — just the two constants
- `FreemiumGate` — two pure functions: `canAddSong(count, isUnlocked)` and `canAddPlaylist(count, isUnlocked)`

Both live in `shared/commonMain` and are fully testable without a device. Claude prepared the Gemini prompt; Gemini implemented both files plus eight unit tests covering the at-limit, under-limit, over-limit, and unlocked cases for each gate.

## Google Play Billing

Claude designed a `BillingManager` class for `composeApp/androidMain` — a wrapper around the billing-ktx library that exposes a single `isUnlocked: StateFlow<Boolean>`. On startup it queries existing purchases to restore the unlock state without asking the user to re-buy. If a purchase comes through, it acknowledges it and flips the state. Product ID: `premium_unlock` (one-time, not a subscription).

Gemini implemented `BillingManager`, wired it into `MainActivity`, updated `App.kt` to collect the unlock state, and added upsell dialogs at both gate points — "New Song" creation and "New Playlist" creation. "Add from Library" (which adds an existing pool song to a playlist, not a new song) stays ungated by design.

The billing flow can't be fully tested without a Google Play Developer account, which isn't registered yet. The gate logic and upsell dialog are working on device now; end-to-end purchase testing waits until the Play account is set up.

## Bug: gate was checked too early

After testing, I found the song limit wasn't blocking additions. The playlist limit worked; the song limit didn't.

Claude diagnosed the issue: the gate was only checked in the `onClick` handler (when "New Song" is tapped), not at the actual creation point. The `onSave` callback that calls `createSongAndAdd` had no check, so songs could slip through in edge cases. Claude fixed this directly — no Gemini involved — by adding a definitive gate check inside `onSave`, right before `createSongAndAdd` is called. The `onClick` check remains as a UX optimization (it avoids opening the editor unnecessarily), but the `onSave` check is the one that actually enforces the limit.

## Pre-beta cleanup

Three smaller items from the Phase 3 list:

**Jitter readout removed.** The BPM circle was showing live timing statistics (average and max deviation in milliseconds) below the BPM number — useful during development, not appropriate for a release build. Claude removed the display, the state variables, and the calculation loop directly.

**BPM range constant consolidated.** The valid BPM range `20..300` was hardcoded in three places: `AndroidAudioEngine`, `SongEditorSheet`, and `App.kt`. Claude added `BPM_RANGE` as a companion object constant on the `AudioEngine` interface in `shared/commonMain` and updated all three call sites to reference it.

**Open Source Licenses screen.** Required before any public release. Claude designed the approach using the AboutLibraries Gradle plugin (MikePenz); Gemini added the plugin, dependency, and a `LicensesScreen` composable. It's reachable from the three-dot overflow menu. The plugin auto-generates the license data at build time from the project's dependency graph — no manual maintenance needed.

One more small improvement came from a user observation: the Song Library was only reachable through the "+" → "Add from Library" flow, which implies you're about to add songs. But the library screen is also useful for browsing and deleting from the pool. Claude added a direct "Song Library" entry to the overflow menu alongside Export, Import, and Dark Mode.

## Privacy policy

Google Play requires a privacy policy for all apps, especially those with billing. I asked Claude what the policy actually needs to cover for this specific app — and the answer was reassuringly short. SessionClick has no internet permission, no analytics SDK, no user accounts, and no cloud sync. The only data it handles is your playlists, stored locally on your device. Google Play Billing means Google processes payments; the app never sees payment details.

Claude wrote the policy based on the actual `AndroidManifest.xml` permissions (`FOREGROUND_SERVICE`, `POST_NOTIFICATIONS`, `VIBRATE`) and the FileProvider setup used for JSON export. It covers what data is stored, what permissions are used and why, how billing works, and how to delete your data. It's hosted at [blog.musicapps.eu/privacy](https://blog.musicapps.eu/privacy/) for now — the URL can move to sessionclick.com later without any issue, since Play Console lets you update it any time.

Claude also added a "Privacy Policy" entry to the app's overflow menu that opens the policy in the browser. One less thing to forget.

## Where things stand

Phase 2 is complete. The app has a working freemium gate, billing integration ready for a real product, all the pre-beta code cleanup done, and a privacy policy in place. What remains before beta is not code: app icon, store listing, screenshots, and registering the Google Play Developer account.

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
