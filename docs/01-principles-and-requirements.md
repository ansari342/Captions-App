# Quran Captions App: Principles & Requirements

Status: Draft v0.2 · Phase 1 of 3 (Principles & Requirements → Design → Code)

## 0. Product summary

This is a brand-new, free iOS app. It turns a Quran recitation clip into a beautifully captioned video, ready to post to TikTok, Instagram Reels and YouTube Shorts.

1. The user picks or records a recitation (video or audio).
2. The app works out which surah and ayahs are recited.
3. It syncs the Arabic text word by word to the recitation and adds an English translation.
4. It exports a video with captions that look excellent and fade between ayahs smoothly.

The idea is inspired by ayahflow.ai, but this is our own product. It has its own name, brand, design and technology, and copies none of that site's code, copy or visuals.

**The north star is caption quality.** The typography, timing and transitions must look good enough that people share the video because of how it looks. Every other decision serves that.

### Decisions from review (v0.1 → v0.2)

| Question | Decision |
|---|---|
| Backend / API | Nothing exists, so we build our own pipeline |
| Pricing | Free. No accounts, paywalls or in-app purchases in v1 |
| Languages | Arabic captions plus English translation only |
| Brand | A new name, logo, colours and fonts, created in Design |
| Platform | iOS first |

---

## 1. Principles

1. **Captions are the product.** Spend design and engineering effort on caption typography, timing and motion first. Accounts, settings and the rest come second.
2. **Reverence and correctness.** Quran text comes only from a verified source, never from the speech-recognition output itself. Uthmani script, harakat, ayah-end markers and RTL shaping must always be correct. Nothing gimmicky goes over the ayahs: no bouncing, emoji or sound effects.
3. **Motion is calm.** Transitions should breathe: soft fades, gentle easing, no hard cuts, nothing that competes with the recitation. They follow the reciter's rhythm, not a fixed clock.
4. **The preview is the export.** The same rendering engine drives the live preview and the final video, so the result matches the preview frame for frame.
5. **Three steps to a finished video.** Pick a clip, confirm the ayahs, export. Style choices are optional and come with tasteful defaults.
6. **AI suggests, the user confirms.** Always show the detected surah and ayah range, and make it easy to correct.
7. **Private and free to run.** Process on the device wherever possible. That keeps recitations on the phone, keeps the app free without server costs, and lets it work offline.
8. **Feels at home on iOS.** Follow native iOS patterns: the photo picker, the share sheet, haptics, Dynamic Type and VoiceOver.

---

## 2. Users & key jobs

| Persona | Job to be done |
|---|---|
| **Reciter** | "I recorded myself. Make it look beautiful and shareable, fast." |
| **Islamic content creator** | "Produce consistent, high-quality reels from recitations I post regularly." |
| **Masjid media volunteer** | "Turn a Taraweeh clip into a post with English meaning for the community." |
| **Learner** | "Keep clips with synced text to revise and share with family." |

---

## 3. Caption quality requirements (the core)

### 3.1 Typography
- Arabic is set in a high-quality Uthmani Quran font (candidates: KFGQPC Uthmanic Hafs, Amiri Quran; licensing confirmed in Design), with correct shaping, ligatures and full harakat.
- Ayah-end markers (۝ with Arabic-Indic numerals) are rendered properly.
- English translation sits below the Arabic in a clean sans or serif. It is visually secondary through size, weight and opacity.
- Text never touches the frame edges. Layout respects platform safe zones, so the TikTok, Reels and Shorts UI never covers captions.
- Captions stay legible on any footage: an adaptive scrim, soft shadow or blur behind the text, never a heavy box by default.
- Text renders sharply at export resolution (1080×1920 and above). The video is never upscaled, and text is never blurred by compression.

### 3.2 Segmentation
- Long ayahs are split into readable chunks at natural breaks. Waqf (pause) marks come first, then word boundaries, with a maximum of about 2 lines of Arabic on screen.
- English chunks are split to follow the Arabic chunk on screen, so meaning stays aligned.
- Repeated passages, which reciters often repeat, are handled correctly: the chunk appears again rather than skipping ahead.

### 3.3 Timing
- Word-level timestamps come from aligning the audio to the verified text.
- Each chunk appears slightly *before* its first word is recited (lead-in of about 150–250 ms), so viewers read along rather than chase the audio.
- Each chunk stays up through pauses and breaths, then leaves just before the next chunk arrives. There are no blank flashes between consecutive chunks.
- The user can nudge a chunk's start and end in a simple editor (P1), with frame-accurate scrubbing.

### 3.4 Motion & transitions
- The default transition is a soft crossfade with a slight rise and blur-to-sharp settle on entry, and a fade with a slight blur on exit. Durations are about 300–500 ms with ease-in-out curves.
- The outgoing and incoming chunks overlap gracefully. They never stack illegibly and never pop.
- Word-by-word emphasis is optional: the word being recited gently brightens or tints while the rest stays dimmer. It moves smoothly, not as a jumpy karaoke highlight.
- Translation fades in a beat after the Arabic, about 100 ms, for a layered, calm feel.
- At least 3 or 4 curated caption styles ship in v1 (e.g. "Serene", "Classic", "Minimal", "Bold"). Each is a full set of typography plus motion, not just a colour swap.
- Respect Reduce Motion in the app UI. Exported videos always use the chosen style.
- Preview and export run at 60 fps or the source frame rate, with no dropped frames during transitions.

---

## 4. Functional requirements

Priority: **P0** = MVP, **P1** = fast follow, **P2** = later.

### 4.1 Import
- P0 Pick a video or audio file from Photos or Files.
- P0 Trim to the section to caption. The target is clips of up to 3 minutes, which covers short-form social.
- P1 Record audio or video in-app.
- P1 "Share to [App]" share extension from Photos and other apps.

### 4.2 Detection & alignment
- P0 Identify the surah and the start and end ayah from the recitation.
- P0 Show the result with a confidence indicator. The user can confirm, or pick the surah and ayah range manually.
- P0 Word-level alignment of the verified text to the audio.
- P0 Show clear progress while processing on the device.
- P1 Handle recitations that repeat ayahs, or skip around, within the clip.

### 4.3 Editor
- P0 Live preview of the video with captions.
- P0 Choose a caption style preset.
- P0 Toggle the English translation on or off.
- P0 Choose an aspect ratio: 9:16 (default), 4:5, 1:1, 16:9, with crop and reposition of the source video.
- P0 Background for audio-only input: tasteful gradients or blurred imagery that come with the app.
- P1 Adjust caption position, size and word-by-word highlighting.
- P1 Timing editor: nudge chunk boundaries.
- P1 Optional small watermark or handle text set by the user (their own, not ours).

### 4.4 Export & share
- P0 Render to MP4 (H.264/HEVC, 1080p) with the original audio.
- P0 Save to Photos, and open the iOS share sheet (TikTok, Instagram, YouTube, WhatsApp).
- P1 Export a caption file (SRT) alongside the video.

### 4.5 Library
- P0 Local project list with thumbnail and surah/ayah label. Reopen a project to re-edit or re-export.
- P0 Delete a project.

### Not in v1
- Accounts or sign-in, payments, cloud sync
- Translation languages other than English
- Android (planned after iOS)
- Multi-clip timeline editing, social feed

---

## 5. Non-functional requirements

| Area | Requirement |
|---|---|
| Platform | iOS 17+, iPhone first. A recent iPhone handles on-device AI; older devices get graceful fallbacks. |
| Performance | Detection plus alignment finishes in 30 s or less for a 60 s clip on iPhone 13 or later. Export takes no more than real time. The editor preview runs at 60 fps. |
| Text source | Quran text from Tanzil (Uthmani, verified), bundled and checksummed. English translation from a source licensed for our use, chosen in Design. |
| Offline | Full flow works offline after install, apart from any optional downloadable models |
| App size | Under 200 MB including the speech model. Larger models download optionally. |
| Accessibility | Dynamic Type, VoiceOver labels, WCAG AA contrast in the app UI, Reduce Motion support |
| Privacy | No recitation leaves the device in v1. No analytics that capture content. Accurate App Store privacy label. |
| Reliability | Projects autosave. Export survives backgrounding, using a background task with resume if killed. |

---

## 6. Proposed technical direction (to confirm in Design)

**Native iOS: Swift + SwiftUI.** With caption rendering as the core of the product, native gives us the best text and motion control. It also lets one rendering path serve both preview and export.

- **Caption engine:** Core Text / TextKit for Arabic shaping, with a custom animation timeline. The same layer tree drives the preview (AVPlayer plus a synchronised layer) and the export (`AVVideoCompositionCoreAnimationTool` or a Metal compositor). This satisfies the "preview is the export" principle.
- **Speech recognition:** on-device Whisper via WhisperKit (Core ML), using a Quran-tuned Whisper model if its licence allows, otherwise multilingual Whisper. It produces rough Arabic text with timestamps.
- **Identification:** fuzzy-match the normalised recognised text (harakat stripped) against the bundled Quran corpus to find the surah and ayah range.
- **Alignment:** align the *verified* ayah words to the recognised words and timestamps, then refine word boundaries (forced alignment, e.g. CTC). Displayed text always comes from the corpus.
- **Storage:** SwiftData for projects. Media goes in the app container.
- **Backend:** none for v1. If on-device accuracy or speed falls short on older phones, a small optional server-side alignment service is the fallback.

> This replaces the v0.1 direction (React Native plus the existing AyahFlow backend). That direction no longer applies: there is no backend to reuse, the app is iOS-first, and caption quality is the top priority.

---

## 7. Success metrics
- Detection is accepted without manual edits in at least 90% of projects.
- A 60 s clip goes from import to exported video in a median of under 2 minutes.
- At least 60% of exported videos are shared or saved through the share sheet.
- Qualitative caption quality: a blind panel prefers our captions over common alternatives.
- At least 99.5% of sessions are crash-free.

---

## 8. Risks & open items for Design
1. **Alignment accuracy:** tajweed elongation (madd), repetitions and varied recitation styles make word timing hard. We should prototype detection and alignment early, on 20–30 real clips.
2. **Licensing:** we need to confirm the Quran font licence (KFGQPC / Amiri), the English translation (public domain vs. licensed, e.g. Saheeh International requires permission) and the speech-model licence.
3. **Brand:** we need a new app name, logo, colour palette and type system.
4. **Device floor:** pick the oldest supported iPhone, based on on-device model speed.
