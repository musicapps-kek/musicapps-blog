---
title: "Day 13: UI polish, app icon, billing live, and Play Store listing"
date: 2026-04-20
draft: false
tags:
  [
    "android",
    "ui",
    "jetpack-compose",
    "google-play",
    "google-play-billing",
    "kotlin-multiplatform",
    "app-icon",
    "ai-assisted",
  ]
summary: "A full day of UI polish and Play Store prep. Nine UI issues fixed via Claude-prepared Gemini prompts, new app icon designed in Affinity Designer, in-app billing tested end-to-end, and the store listing copy and screenshots completed. Tomorrow: feature graphic, tablet screenshots, and a permission demo video."
---

Today split into two parallel tracks: tidying up the UI before wider testing, and pushing the Play Store listing toward completion.

## UI polish

Nine UI issues had accumulated from the first round of real-device testing on a Samsung S21. I wrote them down in German as I noticed them, then worked through them in a session with Claude.

The workflow is the same as always: I describe what I want, Claude reads the relevant source files and prepares a Gemini prompt with exact file names, line numbers, and the proposed code. I paste that prompt into Gemini in Android Studio. Gemini implements. I test on device.

The changes, roughly in order:

**Middle circle — smaller with a thicker border.** The BPM circle was 210dp with an 8dp border. Claude changed it to 190dp with a 12dp border. Slightly more compact, the border reads better at a glance.

**+/− buttons — outlined instead of filled.** The BPM increment/decrement buttons were solid filled buttons — the same visual weight as the Play button. That made them compete for attention they don't deserve. Claude changed `Button` to `OutlinedButton` in the `BpmButton` composable. The buttons now have a border only, which visually recedes relative to Play.

**"New Playlist" text wrapping on S21.** The button text in the playlist switcher was wrapping to two lines on a compact screen. Claude added `maxLines = 1` and `TextOverflow.Ellipsis` to the button label.

**New Song editor: smaller BPM field + Tap Tempo button.** The BPM input was full-width, which felt wasteful. Claude redesigned that section of `SongEditorSheet` as a Row: a compact 110dp BPM field on the left, and a Tap Tempo outlined button taking the remaining width. The tap tempo button records up to the last 8 taps, averages the intervals, and updates the BPM field in real time.

**TopAppBar logo — replace placeholder.** The top-left logo was a temporary `Surface + MusicNote icon`. Two steps to replace it. First, I designed the actual app icon in Affinity Designer and exported it as a flat PNG — more on that below. Second, Claude wrote the Gemini prompt to replace the Surface/Icon block with an `Image(painter = painterResource(R.drawable.ic_logo), modifier = Modifier.size(40.dp).clip(CircleShape))`. Both portrait and landscape top bars were updated.

One wrong turn here: the first attempt used `R.mipmap.ic_launcher` instead of a dedicated drawable. That crashed immediately — `painterResource()` in Compose cannot load adaptive icons, which is what the mipmap points to on Android 8+. The fix is to keep a separate flat PNG drawable for in-app use. Claude diagnosed the crash and explained the distinction; I imported the PNG directly into `res/drawable/` via Finder rather than through Android Studio's Image Asset tool, which processed it as a monochrome icon and made it invisible.

The remaining four issues were also done, some by directly prompting Gemini in Android Studio: no "Add" button in Song Library when opened from the overflow menu, title wrapping in list items on S21, landscape layout cut-off on S21, and a color inconsistency between the Play and Touch buttons.

## App icon

The placeholder icon from Icon Kitchen served its purpose through internal testing. For Day 13, I wanted something more intentional.

I designed the icon in Affinity Designer: a circular badge with the SessionClick color palette, the "SC" monogram, and a concentric ring that echoes the BPM circle in the app. No AI tools this time — the icon design was entirely manual. Affinity Designer exported the PNGs at the required mipmap densities; I dropped them into the `mipmap-*/` folders directly.

For the in-app TopAppBar logo, I exported a separate 192×192 flat PNG (no adaptive layers) and placed it at `res/drawable/ic_logo.png`. This is the file the `Image` composable now references.

Later in the process, I replaced the "SC" monogram with "123" to make the metronome function more immediately recognizable in the small icon size. The "123" is a common visual shorthand for tempo.

## R8 and download size

The first release build weighed in at ~50 MB — too large for a metronome. The cause: `isMinifyEnabled = false`, meaning R8 was off and the full `compose.icons.extended` library (~4,000 icons) was bundled untouched. Enabling R8 and adding `isShrinkResources = true` brought it down to a reasonable size. R8 handles Compose automatically; the `proguard-rules.pro` file is empty.

## In-app billing tested end-to-end

The billing code has been in place since Day 12 but couldn't be tested without a Play Store account and an active product. Today both were available.

The steps to make it work:

1. In Play Console: Monetize → In-app products → Create product with ID `premium_unlock`, set price (€2), status Active.
2. In Play Console: Settings → License Testing → add personal Google account. License testers complete the purchase flow with a fake card and are never charged.
3. Install from the internal test track (not via USB — must come from the Play Store for billing to work).
4. Trigger the upsell dialog by hitting the free tier limit, tap "Unlock Premium", complete the fake purchase.

It worked. The billing sheet appeared, the fake purchase went through, and `isUnlocked` flipped to `true`. The upsell dialog no longer appears on subsequent launches.

One thing to know: in-app products sometimes take a few hours after first setup before they appear in the billing sheet. If `startPurchase()` does nothing, waiting and retrying is the fix.

## Play Store listing

**Store description.** Claude wrote the English listing copy based on the app's actual differentiators: simple timing (no time signatures or subdivisions), easy setlist management, and three feedback modes — audio, visual pulse, and tactile. I reviewed and approved without changes.

- Title: _SessionClick: Stage Metronome_
- Short description: _The stage metronome for live bands. Setlists, visual pulse, tactile beat._
- Full description: covers the three feedback modes, setlist workflow, tap tempo, and the freemium model (30 songs, 3 playlists free; €2 one-time unlock).

**Screenshots.** Nine screenshots were taken during earlier sessions and stored in the Obsidian vault. Claude analyzed each image, renamed the files from their timestamp names to descriptive slugs (`01-dark-main-screen-stopped.png` through `09-dark-playlist-switcher.png`), and wrote a `screenshots.md` with the theme, orientation, screen, state, and a short description for each.

**Unexpected requirements.** Two requirements appeared that weren't anticipated:

_Tablet screenshots._ Play Console requires tablet screenshots if the app is distributed to tablets. The plan: use the Android Studio emulator with a Pixel Tablet AVD. The landscape layout already exists, so it should look reasonable on a 10" screen without additional work.

_Permission demo video._ Because the app uses `FOREGROUND_SERVICE_MEDIA_PLAYBACK`, Play Console requires a video demonstrating the legitimate use. The permission keeps the metronome running when the screen locks. The required video: open app, select a song, press Play, background the app, show the persistent notification in the notification shade with audio still running, return to app, stop. 30–60 seconds, uploaded as an unlisted YouTube video.

## What's left for tomorrow

- Feature graphic (1024×500 px banner) in Affinity Designer
- Tablet screenshots via Android Studio emulator
- Permission demo video (screen recording on device, upload to YouTube)
- Content rating questionnaire in Play Console
- Submit for review

---

**Time spent today:** ~3h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
