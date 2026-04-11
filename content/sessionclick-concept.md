---
title: "SessionClick — Main Screen Concept"
layout: "single"
url: "/sessionclick-concept/"
summary: "The original UI concept and feature specification for the SessionClick main screen, written before development began."
---

_This is my original concept for the SessionClick main screen — written before any code was produced. The mockup and the feature descriptions are my own work. Some details were later refined through discussion with Claude, but the ideas here are the starting point._

---

## Mockup

![SessionClick main screen mockup](/images/sessionclick-main-screen-mockup.png)

---

## (A) The playlist

- The songs in the current playlist are shown here
- Scrollable up and down with a finger
- Short tap on a list item selects the song as the current
- The current song is highlighted (e.g. by inverting the text and background colours)
- The current one should always be displayed as the second from the top
- Long tap on a list item makes it movable up and down in the list
- Swipe to the left to delete an item
- Swipe to the right to edit the text

## (B) A song record / list item

- Song title (limited to ~40 characters)
- The subtitle can hold any information — song or performance related. For example: "Wait for the singer to announce the song", "Pause after this one", "Count in 2 bars"
- Along with title and subtitle we store the tempo and the date the entry was edited. (The date will not be shown, but may be useful to sort the list of all songs.)

## (C) Volume control

- It is important to have an instant volume control and not have to use the hardware buttons on the device. The volume will stay the same and not be saved for individual songs.

## (D) The Tempo Flashlight

- Like a "big LED", pulsing or blinking along the beat, synchronously to the sound
- The tempo to be shown in the centre of this area
- Even when the metronome is stopped and no sound is playing, just the ring should pulse with low light at the selected BPM — to get a feel for the tempo before starting the player

## (E) +/− Buttons to change the tempo

- Will increment or decrement the current tempo by one
- On long press, the interval changes to 5

## (F) Save tempo button

- Only appears after the tempo was changed by the +/− buttons (E)
- When pressed: stores the new tempo for the current song. When saved, the button disappears.

## (G) Start and Stop buttons

- I would like to have two separate buttons
- When using one button for start and stop, you sometimes accidentally press it twice and the metronome does not start or stop as expected

## (H) "Feel the beat" button

- Another feature that makes this app unique
- As long as you tap and hold this button, the device will vibrate along with the beat
- This should also be possible when the metronome is not playing
- Example use case: when I play in silent environments — e.g. on the piano, accompanying a singer — I often only want a hint of the tempo before I play the intro and do not want the app to make noise or flash a light

---

## Playlists and the pool of all songs

The app should be able to handle an unlimited number of playlists.

Any song in a playlist is also part of the "song pool" — a list of all songs. Each song is a unique entry. If you edit a song in your current playlist it will also change in any other playlist that contains this song.

When I create a new playlist, I want to search and select songs from the list of all songs, or create a new entry.

In my playlist I also want to create "special entries" like "Pause", "Speaker on stage", or "Wait for special guest". These should be special items with only one line of text. They must not be added to the list of all songs.

### Open questions (at time of writing)

- How to add a song to a playlist?
- How do I select another playlist?
- How to delete and create playlists?
