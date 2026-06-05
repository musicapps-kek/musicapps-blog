---
title: "iOS: feature complete and on TestFlight"
date: 2026-06-05
draft: false
tags: ["ios", "swiftui", "storekit", "testflight", "ai-assisted", "claude"]
summary: "A full day of iOS feature work — tap tempo, playlist management, export/import, library editing, StoreKit IAP — ending with a successful TestFlight submission and a real test purchase."
---

Today the iOS app went from "mostly there" to feature complete. By the end of the evening it was on TestFlight, and I had done a real sandbox purchase to confirm the payment flow works end to end.

## What got built

**Tap tempo in the Song Editor.** The BPM field in the song editor now has three ways to set a value: type a number directly, nudge with the stepper, or tap a "Tap Tempo" button. The tap algorithm matches the Android version — up to eight taps, discards anything older than two seconds, averages the intervals. First tap does nothing; from the second tap onward it updates the displayed BPM in real time.

**Playlist management.** The playlist switcher was read-only until today. It now supports creating, renaming, and deleting playlists, all via swipe actions and alerts — the standard iOS interaction model. New playlists appear at the top of the list (sorted by creation date, newest first). Creating a new song in an empty playlist now also applies that song's tempo to the metronome automatically.

**Export and import.** Backup and restore are now wired up. Export uses iOS's native `ShareLink` — tapping it opens the share sheet so you can send the file to Files, AirDrop, email, or anywhere else. Import uses `fileImporter` to pick a `.json` file from Files, with a confirmation dialog before the data is replaced. The file format is identical to Android, so backups are cross-platform.

A small note on the export: my first implementation wrapped `UIActivityViewController` in a SwiftUI `.sheet`. That produced a blank screen because the settings sheet was already a sheet — nested sheet presentation is unreliable on iOS. The fix was to replace the whole thing with `ShareLink`, which is the correct SwiftUI API for this use case and doesn't have the nesting problem.

**Library editing.** The song library now has two modes: the existing "Add from Library" picker, and a new "Edit Library" mode reachable from the main menu. In edit mode you can create pool-only songs (without inserting them into the active playlist), edit existing songs, and delete songs from the pool. Deleting from the pool removes the song from all playlists simultaneously — that logic lives in the shared KMP layer and carries over from Android.

**StoreKit in-app purchase.** The one-time unlock is now implemented using StoreKit 2. On first launch the app checks existing entitlements, so purchases survive reinstalls and device switches. The paywall sheet shows the price live from the App Store (so it localises automatically), lists what you get, and has a Restore Purchase button. The three actions that hit the freemium limit — creating a song, creating a pool song, and creating a playlist — all gate against `FreemiumGate` from the shared KMP module and show the paywall if the limit is reached.

**Smaller fixes.** Version number now shows correctly (the `Info.plist` was missing `CFBundleShortVersionString`). The Open Source Licenses screen lists the two KMP dependencies bundled in the app (Kotlin and kotlinx.serialization). The orientation warning in Xcode is resolved by setting `UIRequiresFullScreen = true` — appropriate for a stage app that you want full-screen.

## TestFlight

Getting to TestFlight involved sorting out a few things that had accumulated since the project was created: the bundle ID had a stale `$(TEAM_ID)` suffix from the KMP project template, the signing team was set to my personal Apple ID rather than the MusicApps developer account, and `MARKETING_VERSION` / `CURRENT_PROJECT_VERSION` weren't set in the build settings so the version showed as "—" in the app.

All of that is fixed. The app is live on TestFlight as version 1.0.0 (build 1). I did a test purchase with a sandbox account — the paywall appeared, the purchase went through, the unlock persisted after restarting the app.

My family will test it over the next day or two.

## What's next

Tomorrow: write the App Store listing and submit for review. The TestFlight build is the same binary that goes to the store, so no re-upload is needed — just screenshots, description, keywords, and the privacy policy URL.

---

**Time spent today:** ~3h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
