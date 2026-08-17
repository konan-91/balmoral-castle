# Balmoral Castle App

A cross-platform mobile tour guide for the Balmoral Castle Estate, built with Flutter. Visitors pick a language, then work through a series of narrated video tours of the estate, with an interactive site map and estate information available throughout.

The app is designed for a rural Highland estate where mobile signal is unreliable, so everything runs entirely offline: all video, audio, imagery and copy ship inside the app bundle and no network request is made at runtime.

## Screens

| Screen | Purpose |
| --- | --- |
| Language selection | Six languages (English, German, French, Dutch, Italian, Spanish), with animated selection cards |
| Tour home | Grid of localised tour thumbnails, plus a navigation drawer for estate information |
| Video player | Custom-built player with auto-hiding controls and a scrub bar |
| Interactive map | Pinch, pan and animated zoom over the estate map |
| Information pages | Markdown-rendered content on visiting, admission, accommodation, dining and copyright |

## The interesting problem: one video, six narrations

Shipping a separate video for every language would have multiplied the bundle size by six. Instead the app ships **one silent video per tour** and **one audio narration per language**, and combines them at playback time.

The catch is that the same script takes a different amount of time to narrate in each language, so a single video cut will not line up with all six audio tracks. The app solves this with a per-language, per-video playback rate:

- A bundled `multipliers.txt` lookup table holds a rate for every language and video pairing, derived from the ratio between the narration length and the video length.
- On opening a tour, the video player's rate is set to that multiplier, stretching or compressing the visuals to match the narration exactly.
- Two `media_kit` players are driven in parallel, one for video and one for audio, started and paused together.
- Audio is treated as the source of truth for the timeline. The scrub bar reads the audio position, and seeking maps the requested audio position proportionally onto the video timeline so the two stay locked regardless of the rate applied.

The result is six fully narrated language tracks at roughly the storage cost of one.

## Technical overview

- **Flutter and Dart**, targeting both iOS and Android from a single codebase.
- **`media_kit`** for video and audio playback, chosen over the stock player for its rate control and stream-based state, which the dual-player synchronisation depends on.
- **`provider`** for application state, keeping the selected language available across the navigation stack without prop drilling.
- **Custom playback controls** built from scratch: fade transitions driven by `AnimationController`, a three-second auto-hide timer that only runs while playing, and tap-to-toggle over the whole video surface.
- **Interactive map** using `InteractiveViewer` with `Matrix4Tween` animations for the zoom buttons, clamped translation so the image cannot be panned off screen, and a reset control.
- **Content as assets, not code.** Information pages are plain Markdown files loaded from the bundle and rendered at runtime, so estate staff copy changes need no code changes and no rebuild logic.
- **Careful lifecycle handling**, with every controller, timer and stream subscription disposed of explicitly, and `mounted` guarded state updates around asynchronous playback initialisation.

## Project layout

```
lib/
  main.dart                      app entry, theme, home grid and navigation drawer
  language_selection_screen.dart animated language picker
  language_provider.dart         selected-language state
  video_player.dart              dual-player video and audio synchronisation, custom controls
  get_multiplier.dart            playback-rate lookup for language and video pairs
  map_page.dart                  interactive estate map
  new_page.dart                  Markdown-driven information pages
assets/
  texts/                         page copy and the playback multiplier table
  videos/ audio/ images/ map/    tour media
```

## Running it

```bash
flutter pub get
flutter run
```

Requires the Flutter SDK with Dart 3.7 or later. Note that the tour media (video, audio and imagery) is estate-owned and is not committed to this repository, so a fresh clone will build and run but the tours themselves will not play.
