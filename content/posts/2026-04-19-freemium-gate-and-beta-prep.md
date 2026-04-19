---
title: "Day 12: Freemium gate, billing, Play Store registration, and first internal build"
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
    "google-play",
    "app-icon",
    "screenshots",
  ]
summary: "The freemium gate and Google Play Billing integration landed today, completing Phase 2. Then a pre-beta cleanup pass, privacy policy, Google Play Developer account registration (surprisingly complex), demo playlists, screenshots, a first app icon, and the first internal build installed on a real device via the Play Store."
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

## Google Play Developer account registration

Registering for Google Play turned out to be more involved than expected — worth documenting in detail.

The $25 one-time registration fee and identity verification went smoothly. The complication was the package name: `eu.musicapps.sessionclick` uses the reverse-domain convention, and Google now requires you to prove ownership of the underlying domain before accepting it.

The normal path is to verify `musicapps.eu` in Google Search Console (done via a DNS TXT record), then register the package name. But the registration UI immediately showed an error: the feature for Play Store apps is not yet fully rolled out. Google's own message acknowledged they're working on it and will register Play Store app package names automatically "this month."

That left the alternative path: prove ownership by uploading a signed APK. Google shows you a SHA-256 fingerprint and asks you to sign an APK with the corresponding private key and include a unique token file (`adi-registration.properties` in `assets/`). The fingerprint turned out to belong to the Android debug keystore — present on every development machine. After adding the token file, rebuilding, and signing with the debug key, the upload verified successfully. Google confirmed ownership within minutes.

The debug key was only needed for this verification step. All actual release builds use the proper keystore created for the project.

After verification, the app was created in Play Console without issues.

## Demo playlists

Before taking screenshots, two demo playlists were added to `SessionSeed.kt` to replace the old placeholder data. Claude wrote the Gemini prompt; Gemini implemented it.

**Jazz Standards Gig** — six jazz standards with BPM values, three songs with performance notes as subtitles ("Keep it floating, don't rush the bridge", "Tender, leave space after the intro", "Start Latin, switch to swing on solos"), and two break entries ("Short Break", "Break — ~10 min").

**Classical Tempi** — seven entries from Largo to Presto, each with its tempo range in the subtitle ("Very slow, broad (40–60 BPM)") and the canonical BPM in the center. Useful both as a reference and as a demonstration of the subtitle field.

Both playlists replaced all previous seed data entirely.

## Screenshots

Nine screenshots were taken on the Android emulator (Pixel profile) in Android Studio, covering dark theme, light theme, landscape/tablet layout, the playlist switcher, the song library, the overflow menu, and the Classical Tempi playlist. One duplicate landscape shot was removed, leaving eight.

Claude renamed the files descriptively (`01-jazz-playlist-dark-stopped.png` through `08-classical-tempi-playing.png`) and wrote a `captions.md` with a headline caption and a description for each screenshot — ready for use in the Play Store listing and marketing.

## App icon

Getting the app icon turned out to be the most frustrating part of the day. Three tools tried, none worked cleanly:

**Gemini Desktop (Mac app):** File upload was broken entirely. Switched to the browser version, which accepted the visual context document and screenshots. After two refinement rounds Gemini produced a reasonable result — a glowing green ring on dark green with "120" in the center — but any further iteration broke the session and images stopped rendering.

**Adobe Firefly:** Free, reliable upload and download, but the results didn't match the brief well enough to be usable.

**Back to Gemini (browser):** Re-feeding the exact prompt that produced the best result yielded a solid candidate — the right colors, the right feel. Not pixel-perfect, but directionally correct.

**Icon Kitchen:** For the actual production icon, I used [Icon Kitchen](https://icon.kitchen) to assemble a clean adaptive Android icon from the color values. The result: deep forest green background, medium green concentric ring, "SC" in white as a minimal identifier. Not the final icon, but good enough for internal testing and store listing submission. Icon Kitchen exports the correct `mipmap-*` folder structure for direct drop-in to the Android project.

## First internal build on the Play Store

With the store listing partially filled in and the icon in place, the first release AAB was built in Android Studio (Build → Generate Signed Bundle, release variant, production keystore) and uploaded to the Internal Testing track in Play Console.

Two warnings: no testers configured yet (expected), and no ProGuard deobfuscation file (not required for internal testing). Both ignored. After adding a test account as an internal tester, the build was installable from the Play Store on a second device within minutes.

It runs. There are layout adjustments to make, but the core app is live on a real device via the official distribution channel.

## Where things stand

The app is on the Play Store — internal track only, but real. The remaining pre-launch work is:

- Layout fixes identified from the first device test
- Complete the store listing (description text, feature graphic)
- Wire up and test the in-app purchase end-to-end
- Content rating questionnaire in Play Console
- Decide on open/closed beta before production release

---

**Time spent today:** ~4h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
