# Rattil: Design

Status: Draft v0.1 · Phase 2 of 3 (Principles & Requirements → **Design** → Code)
Builds on: `01-principles-and-requirements.md` (v0.2)
Interactive prototype: `docs/prototype/caption-prototype.html` (open it in a browser)

---

## 1. Brand

### 1.1 Name
**Rattil** (رتِّل): "recite with measured, unhurried recitation."
It comes from *Al-Muzzammil 73:4*: وَرَتِّلِ ٱلْقُرْءَانَ تَرْتِيلًا ("and recite the Qur'an with measured recitation").

The name describes the product's core promise: captions that move at the reciter's measured pace. It is short, says easily in English and Arabic, and is distinct from "Tarteel", the well-known recitation app. We still need trademark and App Store name checks before committing.

Backup names: **Tilawa** (recitation), **Sakinah** (tranquillity), **Ayat Studio**.

### 1.2 Voice
Calm, plain and respectful. Short sentences. No hype, exclamation marks or emoji in the UI.
- Do: "Surah Al-Fatiha · Ayahs 1–7. Is this right?"
- Don't: "🔥 We found your surah!!"

### 1.3 Palette
| Token | Hex | Use |
|---|---|---|
| Night Ink | `#0D1615` | Primary background (the app is dark-first: video apps and night recitation) |
| Deep Teal | `#1F3D3A` | Surfaces, cards, selected states |
| Lamp Gold | `#E2B66A` | The single accent: primary buttons, ayah markers, active word, progress |
| Mist | `#A9BAB4` | Secondary text, icons |
| Bone | `#EEF0EE` | Primary text on dark, and the light-mode background |
| Ember | `#D9734E` | Errors and destructive actions only |

The light theme swaps Night Ink and Bone, and darkens Lamp Gold to `#9A6A22` so it keeps AA contrast.

### 1.4 Typography
| Role | Face | Notes |
|---|---|---|
| Quran text (all caption styles) | **Amiri Quran** (SIL OFL) | Uthmani-style naskh with full harakat. We use one font for all scripture, for consistency and correctness. KFGQPC Hafs is a later option once its licence terms are confirmed. |
| UI display | **Gloock** (OFL) | Wordmark, screen titles. Used sparingly. |
| UI body | **Hanken Grotesk** (OFL) | Everything else. Maps to Dynamic Type sizes. |
| Caption English (style-dependent) | Newsreader / Hanken Grotesk | See §3.4 |

### 1.5 Logo & app icon
- **Mark:** a pointed arch (mihrab) in Bone. Inside it are three rounded horizontal strokes in Lamp Gold, rising in length like lines of text appearing. The mark reads as "scripture, framed and unfolding".
- **App icon:** the mark on a Night Ink → Deep Teal vertical gradient. No text in the icon.
- **Wordmark:** "Rattil" in Gloock. The Arabic رتِّل may sit beside it on splash and about screens.

---

## 2. Experience design

### 2.1 Primary flow
```
Library ──(+)──▶ Import ──▶ Trim ──▶ Detecting ──▶ Confirm ──▶ Editor ──▶ Export ──▶ Share
   ▲                                                  │ (edit)        │
   └──────────────────────────────────────────────────┴── autosaved ──┘
```
The target is three decisions: pick a clip, confirm the ayahs, tap Export. Trim defaults to the whole clip, and the editor opens with the default style already applied.

### 2.2 Screens

**Library (home)**
- Large title "Rattil" with a gold "New video" button (bottom, in thumb reach).
- Grid of projects: 9:16 thumbnail showing a caption frame, with the surah name and ayah range below.
- Empty state: one line ("Pick a recitation from your Photos to begin") plus the button.
- Swipe or long-press a project to Duplicate or Delete (Delete asks for confirmation).

**Import & Trim**
- The system PHPicker (videos and audio), plus "Files" for audio files.
- Trim: filmstrip with drag handles and a waveform underneath. Shows the duration, with a note that the maximum is 3:00.

**Detecting**
- Full-screen calm state: the waveform plays left to right while a gold progress line follows it.
- Status text moves through: "Listening…" → "Finding the surah…" → "Matching each word…".
- Cancel is always available. Typical time is about 10–30 s.

**Confirm**
- Card: the surah name in Arabic and English, the ayah range, and the first ayah's Arabic as a preview.
- Confidence:
  - High: no badge.
  - Medium: "Please check" chip.
  - Low: the manual picker opens straight away.
- Buttons: "Looks right" (primary) and "Change" (opens a surah list with search, then ayah start/end steppers).

**Editor**
- At the top, a live preview at the selected aspect ratio, letterboxed in Night Ink, with play/pause and a scrubber. Tick marks on the scrubber show where each chunk starts.
- Bottom sheet tabs:
  1. **Style:** horizontal cards showing a looping thumbnail of each style.
  2. **Format:** aspect ratio (9:16 / 4:5 / 1:1 / 16:9). Drag in the preview to reposition the crop.
  3. **Text:**
     - Translation on/off
     - Word highlight on/off
     - Caption position (Upper / Centre / Lower)
     - Size (S/M/L)
  4. **Background:** shown only for audio-only clips. Bundled gradients and blurred images.
  5. **Timing:** (P1) a list of chunks. Tap one to nudge its start or end in steps of ±1 frame or ±100 ms.
- "Export" sits in the top-right.

**Export**
- Progress ring shown over the last preview frame, then:
  - "Saved to Photos" confirmation
  - Share sheet
  - Shortcut buttons: TikTok, Instagram, YouTube, WhatsApp
- Export settings: resolution (1080p default, 4K if the source allows), with no watermark.

### 2.3 Interaction details
- **Haptics:** a light tap when snapping trim handles, success feedback when the export finishes.
- **Accessibility:**
  - Every control has a VoiceOver label.
  - The preview exposes the current ayah text as an accessibility value.
  - Dynamic Type applies to the UI (not to the caption render, which is sized relative to the frame).
- **Reduce Motion:** UI transitions become crossfades. Caption styles in exported videos are not affected.
- **Autosave:** after every edit. If a project is interrupted, it reopens in its last state.

---

## 3. Caption system (the core)

### 3.1 Coordinate model
All caption measurements use a unit **U = 1% of min(frame width, 0.75 × frame height)**. The same style therefore renders identically at every resolution, and stays proportionate in square and landscape frames. Sizes below written as "% W" mean U.

### 3.2 Safe zones (9:16)
Captions stay inside a **caption-safe box**:
- 8% W side margins
- the top 12% of height is avoided (status UI)
- the bottom 22% is avoided (platform caption and username overlays)
- the right 14% W is avoided, for TikTok's action rail, *at Lower position only*

Other ratios use 6% W margins and 8% top/bottom insets.

### 3.3 Segmentation (ayah → chunks)
Rules are applied in order:
1. Short ayahs (fit in 2 lines at the style's Arabic size) become one chunk.
2. Longer ayahs split at a **waqf mark** (ۖ ۗ ۚ ۛ ۙ ۘ) nearest the middle, provided both halves fit.
3. If no waqf mark gives two fitting halves, split at the word boundary that best balances line widths. Never split inside a word, or between a word and its waqf mark.
4. Ayah-end marker (circle plus Arabic-Indic numeral) is attached to the last chunk of each ayah.
5. **English:** each chunk carries the English for its words. English is paired at the ayah level and split proportionally at the nearest punctuation or clause boundary. The translation data stores optional manual split points for long ayahs, and these override the automatic split.
6. **Repeated recitation:** if the aligner reports that the reciter went back over words, emit a new chunk instance for the repeat rather than rewinding.

### 3.4 The four styles

Every style uses Amiri Quran for Arabic and keeps the same segmentation and timing model. Styles differ in typography, layout, legibility treatment and motion.

| | **Sakinah** (default) | **Mushaf** | **Hairline** | **Unfold** |
|---|---|---|---|---|
| Mood | Serene, cinematic | Classic, framed | Minimal, documentary | Bold, rhythmic |
| Arabic size | 7.4% W | 6.6% W | 5.2% W | 9.0% W |
| Arabic colour | Bone | Bone, with gold ayah marker | Bone 92% | Bone |
| English | Newsreader italic 3.6% W, 80% opacity | Newsreader 3.3% W, 80% | Hanken Grotesk 3.0% W, 75% | Hanken Grotesk 600 3.4% W, uppercase-free |
| Position default | Centre | Centre | Lower | Centre |
| Legibility | Radial dark scrim behind the block (adaptive, §3.7) plus soft text shadow | Translucent ink panel, 14px radius, background blur | Adaptive bottom gradient plus text shadow | Full-frame dim, 35–62% (adaptive) |
| Entry | Fade, rise 2.2% W → 0, blur 1.4% W → 0, 520 ms | Pure crossfade 420 ms | Fade 260 ms | Each word fades and rises in as it is recited |
| Exit | Fade, blur → 0.8% W, drift up 0.8% W, 380 ms | Crossfade 420 ms | Fade 220 ms | Whole block fades with slight blur, 360 ms |
| Word highlight | Active word 100%, others 62%, glow | Off | Off (optional) | Built in (it reveals words) |
| English delay | +140 ms after the Arabic | +0 | +80 ms | Fades in when the chunk's last word is reached |

**Easing:**
- Entries use `cubic-bezier(0.22, 1, 0.36, 1)` (ease-out, decelerating into rest).
- Exits use `cubic-bezier(0.55, 0, 0.75, 0.25)` (ease-in).
- The word-highlight change uses a 180 ms ease-in-out.

### 3.5 Timing model
Inputs: each word in a chunk has recited times `w.t0`/`w.t1` from alignment.

```
LEAD      = 0.20 s   chunk appears before its first word
HOLD      = 0.80 s   stay after the last word when a long silence follows
GAP_LONG  = 1.50 s   silence longer than this → fade out instead of crossfading
MIN_SHOW  = 1.20 s   minimum fully-visible time
HANDOFF   = 0.18 s   next chunk's entry starts this long after the current exit begins

show(c)  = c.first.t0 − LEAD
hide(c)  = next exists and gap(c, next) < GAP_LONG  →  show(next) − HANDOFF
           otherwise                                →  c.last.t1 + HOLD
hide(c)  = max(hide(c), show(c) + entryDur + MIN_SHOW)   then push show(next) later if needed
```

The exit begins at `hide(c)`. Because the incoming chunk starts its entry `HANDOFF` later, the outgoing text is mostly faded before the new text gains contrast. The two are never stacked at full opacity, and the screen is never blank between consecutive chunks. This handoff is the main thing that makes transitions feel smooth rather than "popping".

Word highlight state is computed from `t` against `w.t0`/`w.t1`, with a 60 ms anticipation so the eye lands on the word as it's heard.

### 3.6 Ayah marker
An end-of-ayah ornament is drawn as a vector: a thin gold circle with an Arabic-Indic numeral. It is not the U+06DD glyph, whose rendering varies by font. It is sized at 0.9× the Arabic x-height and inherits the chunk's opacity and blur.

### 3.7 Adaptive legibility
- About 4× per second, sample the mean luminance of the source frame under the caption block, using a downscaled 32×32 luma read.
- Scrim opacity is `clamp(0.20 + 0.90·(lum − 0.30), 0.20, 0.72)`, smoothed with a 600 ms low-pass so it never flickers. Unfold dims at least 35% and at most 62%. Mushaf raises its panel opacity from 0.50 up to 0.82 instead.
- Text contrast must reach at least 4.5:1 against the composited background. Bright, busy footage (sky, white walls) gets a stronger scrim automatically.

### 3.8 Rendering architecture
One pure function draws a caption frame:

```
CaptionRenderer.render(timeline, style, layout, t, into: CGContext / MTLTexture)
```

- **Text is pre-rasterised once per chunk** (Core Text with Amiri Quran, at 2× target scale) into a texture. Each frame only applies transforms, opacity, blur (Core Image / MPS Gaussian) and per-word highlight masks. Word rects come from Core Text glyph runs. This keeps 60 fps preview and fast export.
- **Preview:** `AVPlayer` plus `AVPlayerItemVideoOutput`. A `CADisplayLink` drives a Metal view that composites video and the caption at `player.currentTime()`.
- **Export:** `AVAssetReader` → a custom `AVVideoCompositing` that calls the *same* `render` at each frame's presentation time → `AVAssetWriter` (HEVC/H.264, 1080×1920 default). Audio is passed through.
- The result: preview and export are the same code path, so they match frame for frame.

---

## 4. Recitation engine (detection & alignment)

```
audio (16 kHz mono)
  └▶ WhisperKit (Core ML, Arabic / Quran-tuned model) ─▶ tokens + word timestamps
        └▶ normalise (strip harakat, unify alef/ya/ta-marbuta forms)
              └▶ match against bundled corpus (n-gram index over 6,236 ayahs)
                    ─▶ surah, ayah start…end, confidence
                          └▶ align corpus words ↔ recognised words (Needleman–Wunsch on normalised forms)
                                ─▶ word timings, interpolated for unmatched words
                                      └▶ (P1) refine boundaries with CTC forced alignment
```

- **Displayed text always comes from the corpus.** Recognised text is used only to locate the passage and its timing.
- **Confidence** comes from match coverage and edit distance. Thresholds are calibrated on the test set.
- **Test set:** 30+ clips covering varied reciters, tempos, repetitions and background noise. Word timings from public reciter datasets (e.g. Quran.com / QuranCDN segment data for well-known reciters) give ground truth for timing accuracy (target: median word-start error under 120 ms).

---

## 5. Data model (SwiftData)

```
Project      id, createdAt, updatedAt, sourceAssetURL, trim(start,end), aspect, styleID,
             options(translation, highlight, position, size), backgroundID?, thumbnailURL
Recitation   projectID, surah, ayahStart, ayahEnd, confidence, words[WordTiming]
WordTiming   surah, ayah, wordIndex, t0, t1, isRepeat
ChunkOverride projectID, chunkIndex, showDelta, hideDelta        (manual timing nudges)
```

Bundled read-only data:
- `quran-uthmani.json` (Tanzil, with checksum)
- `translation-en.json` (with optional split points)
- `surah-meta.json`

---

## 6. Content & licensing decisions
| Item | Decision | Action |
|---|---|---|
| Arabic text | Tanzil Uthmani: verbatim, unmodified, with attribution | Add attribution to the About screen |
| Quran font | Amiri Quran (SIL OFL), bundled | Include the OFL licence text |
| English translation | Preferred: Saheeh International (requires written permission). Fallback: a public-domain translation (Pickthall, 1930). | Request permission now. The prototype uses Pickthall. |
| Speech model | Quran-tuned Whisper if its licence permits commercial use, otherwise multilingual Whisper | Verify the licence before M3 |
| UI fonts | Gloock, Hanken Grotesk, Newsreader (all OFL) | Bundle them |

---

## 7. Build plan for the Code phase
| Milestone | Scope | Why this order |
|---|---|---|
| **M1: Caption engine** | Swift package `CaptionKit`: segmentation, timeline, 4 styles, renderer. Includes a demo app with a manual surah pick and fixture timings. | It is the core of the product and the riskiest *quality* area, so we iterate on it first. |
| **M2: Editor & export** | Import, trim, editor UI, preview, export to Photos and share | Delivers an end-to-end usable app (with manual ayah selection) |
| **M3: Recitation engine** | WhisperKit, matching, alignment, confirm screen | The riskiest *technical* area. It is developed against the test set in parallel with M2. |
| **M4: Polish** | Library, onboarding, accessibility, performance on older devices, TestFlight | Ship |

---

## 8. Open questions
1. Should we go with the name **Rattil**, or one of the backups?
2. Do you approve **Sakinah** as the default style? The prototype lets you compare all four.
3. Should we request Saheeh International permission, or ship v1 with a public-domain translation?
4. What is the minimum iOS device? The proposal is iPhone 12 or later, iOS 17.
