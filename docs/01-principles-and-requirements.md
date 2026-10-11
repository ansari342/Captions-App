# AyahFlow Mobile — Principles & Requirements

Status: Draft v0.1 · Phase 1 of 3 (Principles & Requirements → Design → Code)

## 0. What we're porting

[ayahflow.ai](https://ayahflow.ai) is an AI caption generator for Quran recitation videos:

1. User uploads a recitation (audio or video).
2. AI detects the surah and ayah range.
3. Arabic text (Uthmani script, KFGQPC Hafs font) is synced word-by-word to the audio, with translation in up to 16 languages.
4. A captioned video is rendered for TikTok / Instagram / YouTube in multiple aspect ratios.
5. Free to try; paid tiers (site metadata indicates 4 offers, $0–$99; exact plans TBC).

The mobile app should be the *fastest path from "I just recorded a recitation" to "captioned video posted"*. Phones are where these videos are recorded and published, so mobile is the natural home for this product, not a cut-down version of it.

> Note: the site could not be fetched directly from the build environment; details above come from search snippets of ayahflow.ai and must be verified against the live site / backend.

---

## 1. Principles

1. **Reverence first.** The Quran's text is sacred. Never mangle Arabic: correct Uthmani glyphs, harakat, ayah markers, RTL shaping. No ads, sounds, emojis, or gimmicky animations over ayahs. Text accuracy beats speed or flashiness.
2. **Phone-native, not a shrunken website.** Built around the camera roll, share sheet, and vertical video. One-handed use, thumb-reachable primary actions.
3. **Three taps to a captioned video.** Pick a clip → confirm the detected surah/ayahs → export. Everything else (styles, languages, fine-tuning) is optional and progressive.
4. **AI suggests, the human confirms.** Detection results are always shown and easy to correct (surah, ayah range, word timing). Never silently publish AI output.
5. **Preview what you'll get.** Live on-device preview of captions over the video. The export must match the preview.
6. **Resilient to real-world conditions.** Large video files, flaky mobile networks, app backgrounding: uploads resume, jobs continue server-side, users get notified when the video is ready.
7. **Respect users' content and privacy.** Upload only what's needed, offer to delete originals after processing, and never use recitations for training without explicit opt-in.
8. **Inclusive and global.** Full RTL support for Arabic/Urdu/Farsi UIs. 16 translation languages. Accessible typography, Dynamic Type, screen readers, sufficient contrast.
9. **One backend, many clients.** Reuse the existing AyahFlow detection/alignment/render pipeline and accounts. The app is a client, not a fork of the product.

---

## 2. Users & key jobs

| Persona | Job to be done |
|---|---|
| **Reciter / Qari** | "I recorded myself. Make it shareable with proper captions, fast." |
| **Islamic content creator / page admin** | "Batch-produce consistent, branded reels from famous reciters' audio." |
| **Masjid / institute media team** | "Turn Taraweeh or Jumu'ah clips into posts with translations for our community." |
| **Learner** | "Save clips with word-by-word captions to revise and share with family." |

---

## 3. Functional requirements

Priority: **P0** = MVP, **P1** = fast follow, **P2** = later.

### 3.1 Onboarding & account
- P0 Sign in with Apple, Google, email (same accounts as the web app).
- P0 Free-trial path: try one video before forcing sign-up (if the backend allows it).
- P0 Default translation language and UI language picker (auto-detected from the device).
- P1 Sync history and projects with the web app.

### 3.2 Import
- P0 Pick video or audio from the photo library / files.
- P0 Record audio or video in-app.
- P1 Import from a link (YouTube/TikTok URL), if the web app supports it.
- P1 Trim the clip before upload, to cut upload size and cost.
- P1 Share-sheet extension ("Share to AyahFlow" from Photos, TikTok, WhatsApp).

### 3.3 AI detection & alignment
- P0 Upload with progress, background/resumable upload.
- P0 Show the detected surah and ayah range with a confidence indicator. User confirms or edits.
- P0 Word-by-word synced Arabic, using the backend alignment.
- P1 Manual timing correction: tap a word and nudge it earlier or later.
- P2 Reciter detection and labelling.

### 3.4 Caption editor & styling
- P0 Live preview: video plus animated captions (Arabic, optional translation, optional transliteration).
- P0 Choose translation language (the 16 supported languages).
- P0 Aspect ratio: 9:16, 1:1, 4:5, 16:9.
- P0 A few curated style presets (font size, position, highlight colour, background dim/blur).
- P1 Backgrounds for audio-only input: solid, gradient, nature/masjid stock, or user image.
- P1 Branding: watermark or logo, channel handle (paid tiers).
- P2 Custom templates saved to account.

### 3.5 Render & export
- P0 Server-side render job. The user can leave the app and gets a push notification when the video is ready.
- P0 Save to camera roll, plus native share sheet with quick targets for TikTok, Instagram Reels, YouTube Shorts, and WhatsApp.
- P0 Export quality matches the plan (e.g. watermark or 720p on free).
- P1 Caption-file export (SRT/VTT) for editing in other tools.

### 3.6 Library
- P0 Project list with status: uploading, processing, ready, failed (with retry).
- P1 Duplicate a project to try a new style or language.
- P1 Delete originals and outputs.

### 3.7 Monetisation
- P0 Show remaining credits/quota.
- P0 Upgrade through native in-app purchase (StoreKit / Play Billing), required by store rules for digital goods. Entitlements map to the web plans.
- P0 Restore purchases.

---

## 4. Non-functional requirements

| Area | Requirement |
|---|---|
| Platforms | iOS 16+ and Android 9+ (API 28+), phones first, tablet-tolerant |
| Performance | Cold start under 2 s. Preview plays at 60 fps on mid-range devices. Upload uses chunked/resumable transfer. |
| Reliability | Uploads and jobs survive app kill or backgrounding. Idempotent job creation. Clear error states with retry. |
| Arabic rendering | Bundle the KFGQPC Hafs Uthmani font. Text comes from a verified Quran text source (e.g. Tanzil / Quran.com API), never OCR or AI-generated text. |
| Accessibility | WCAG 2.2 AA contrast, Dynamic Type, VoiceOver/TalkBack labels, reduced-motion respect |
| i18n | UI strings externalised. RTL layout mirroring. Translations for at least English, Arabic, Urdu, Indonesian, Turkish, French at launch. |
| Privacy & security | HTTPS only. Tokens in Keychain/Keystore. Signed upload URLs. Data deletion from inside the app (App Store requirement). Privacy nutrition labels. |
| Analytics | Privacy-respecting funnel events (import → detect → export → share), with no content captured |
| Store compliance | Apple IAP for upgrades. Account deletion in-app. Sign in with Apple if other social logins exist. |

---

## 5. Proposed technical direction (to confirm in Design)

- **Client:** React Native + Expo (TypeScript). One codebase for iOS and Android, OTA updates, and a mature ecosystem for camera, media library, IAP, and push.
  - Video preview: `expo-video` with a caption overlay timed to the alignment JSON.
  - Uploads: background, resumable (tus or S3 multipart via presigned URLs).
  - Payments: RevenueCat, to map store purchases to web plans.
- **Backend:** reuse the AyahFlow API for auth, upload, detection, alignment, render jobs, and plans. Mobile needs:
  - Job status via webhook → push (APNs/FCM)
  - Presigned/resumable uploads
  - A receipt-validation endpoint (or RevenueCat webhook) to grant entitlements
- **Rendering stays server-side.** It keeps output identical to the web app and keeps phones cool. On-device preview only.

---

## 6. Out of scope (v1)
- On-device AI detection or rendering
- Full timeline video editor (multi-clip, transitions)
- Social feed or community features
- Desktop/tablet-optimised layouts

---

## 7. Success metrics
- Time from import to exported video: median under 3 minutes for a 60 s clip
- Detection accepted without edits in at least 90% of projects
- Import → share conversion
- Free → paid conversion, D7/D30 retention
- Crash-free sessions of at least 99.5%

---

## 8. Open questions (need answers before Design)
1. **Ownership and API access:** do we own ayahflow.ai's backend? Is there an API the app can call, or do we need to build or mock one?
2. **Plans:** what are the exact tiers, limits, and prices so they can be mirrored as IAP products?
3. **Which 16 languages**, and is transliteration supported?
4. **Brand assets:** logo, colours, fonts from the website.
5. **Platform priority:** iOS and Android simultaneously, or iOS first?
6. Is link import (YouTube/TikTok URL) part of the web product today?
