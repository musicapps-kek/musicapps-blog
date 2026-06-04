---
title: "Raising the free tier, fixing a BPM glitch, and setting up a new machine"
date: 2026-06-04
draft: false
tags:
  [
    "release",
    "bugfix",
    "freemium",
    "compose",
    "ai-assisted",
    "claude",
  ]
summary: "A small but meaningful update: the free tier now allows 50 songs and 5 playlists. Plus a Compose LaunchedEffect fix for a BPM sync issue when adding the first song to a new playlist. And some notes on getting back up to speed on a new Mac."
---

Today's update is a short one — three changes, all shipped in a single session. But two of them directly affect how new users experience the app, so they were worth prioritising.

## Raising the free limits

The original free tier allowed 30 songs and 3 playlists. That seemed reasonable when I set it in April, but there's a practical problem I hadn't fully thought through: the app ships with three sample playlists. A new user who opens SessionClick for the first time is already at their playlist limit. There's nowhere to go without hitting the paywall.

That's a bad first impression. The point of the free tier is to let musicians actually use the app — fill it with their setlists, feel whether it fits their workflow — before deciding to pay. Hitting a wall before you've even started defeats that.

The new limits are 50 songs and 5 playlists. That's enough room for a working musician to set up a full season of gigs without running into a gate. The paid upgrade is still there for anyone who needs more, but it won't be the first thing a new user encounters.

The change itself was trivial — two constants in `FreemiumConfig.kt` in the shared module. Because `FreemiumGate` reads from those constants, and the paywall dialog already interpolates the values rather than hardcoding them, the whole app updated from that single file.

## Fixing the BPM sync on new playlists

A subtler bug: when you create a new playlist and add a song to it, the metronome's displayed BPM didn't update to that song's tempo. The song was selected — it was the only item in the list — but the player still showed whatever BPM it had before.

The cause was in a `LaunchedEffect` in `App.kt` that's responsible for syncing the selected song's BPM to the player. It was keyed only on `selectedIndex`. When the first song is added to an empty playlist, `selectedIndex` stays at `0` both before and after — the index didn't change, so the effect didn't fire.

The fix was to add the selected song's ID as a second key:

```kotlin
val selectedItemId = (sessionViewModel.displayItems
    .getOrNull(sessionViewModel.selectedIndex) as? DisplayItem.SongView)?.song?.id

LaunchedEffect(sessionViewModel.selectedIndex, selectedItemId) {
    // apply BPM to player
}
```

Now the effect fires whenever either the index *or the item at that index* changes. When the first song lands at index 0, `selectedItemId` flips from `null` to the new song's ID, and the BPM updates immediately. Breaks (non-song entries) produce `null` for `selectedItemId` in both directions, so there are no unintended re-triggers there.

This is a good example of a Compose reactivity issue that's easy to miss: `LaunchedEffect` only knows about changes to its keys, and "the item at this position changed" isn't the same as "the position changed."

## New machine setup

The other thing this session involved was getting back up to speed on a new Mac. The project is on GitHub, so that part was straightforward — clone, open in Android Studio, let Gradle sync, open `iosApp.xcodeproj` in Xcode, build both targets. Both ran in the simulator without issues on the first try.

The one thing worth noting for anyone in the same situation: the Android signing keystore should **not** go into the Git repo, even password-protected. The password is the only thing standing between an attacker and the ability to publish fake updates under your identity — and Git history is forever. The right approach is to keep the keystore outside the project directory and reference it from `local.properties`, which is already in `.gitignore`. For backup, store it somewhere like 1Password or an encrypted disk image in iCloud Drive, not in the repo.

## What's next

The iOS port is still the main thread. StoreKit integration is the remaining piece before TestFlight. That's the next session.

---

**Time spent today:** ~1h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
