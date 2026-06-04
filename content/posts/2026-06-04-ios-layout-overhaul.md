---
title: "iOS layout overhaul: portrait/landscape, controls, playlist"
date: 2026-06-04
draft: false
tags: ["ios", "swiftui", "ui", "layout", "ai-assisted", "claude"]
summary: "A full rework of the iOS main screen: a proper two-panel layout that adapts to portrait and landscape, a redesigned controls section, and a more stage-friendly playlist."
---

Today was purely about the iOS side, and purely about layout. No new features — just making the existing screen feel like something you'd actually want to look at from across a rehearsal room.

## Portrait and landscape layouts

The app was portrait-only and everything was stacked vertically in a single column. That worked well enough on a small screen, but it wasn't deliberate — it was just the default. Today I defined what the layout should actually be.

In portrait, the screen is now split horizontally into two halves: the playlist on top, the metronome controls on the bottom. In landscape, it splits vertically: playlist on the left, controls on the right. SwiftUI's `verticalSizeClass` environment value drives the switch — `.compact` means landscape on iPhone, so the layout responds automatically when you rotate.

Getting there required enabling landscape rotation in the Xcode project settings first. It had been locked to portrait only, which is why rotating the simulator did nothing.

## Controls section

The controls panel was also reorganised. The upper two-thirds now has the flashlight circle in the centre, the BPM +/− and save buttons stacked vertically on the left, and the volume slider rotated to vertical on the right. Both side columns are the same width, so the flashlight stays centred regardless of content.

The lower third has the three action buttons in a horizontal row: Feel the Beat, flash toggle, and play/stop. The flash toggle hides itself while the metronome is playing — one less visual distraction on stage.

The Feel the Beat button is always shown in the accent colour now, not just when pressed. Pressing fills it solid. The play/stop and Feel the Beat buttons are slightly larger than before.

## Playlist

The playlist rows got a few changes aimed at readability from a distance: larger text, reduced side padding so the list runs edge to edge, and uniform row heights so songs with and without subtitles don't shift the layout. Break entries (the "pause" markers between songs) have a distinct grey background and a smaller row height, and tapping them does nothing since they carry no tempo information.

Separator lines between rows are now full-width, rendered at the list infrastructure level rather than as a custom overlay — which turned out to matter once the break row backgrounds were in place.

## What's next

Tomorrow: testing all the existing functions on the actual device, then tackling the remaining missing integrations — StoreKit IAP and anything else that surfaces.

---

**Time spent today:** ~2h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
