# PRD: Kidonk Kidonk Web

**Status:** Draft v1
**Last updated:** September 25, 2026
**Hosting:** Vercel (free tier, static)
**Storage:** Browser only (IndexedDB), no database, no backend

---

## 1. Overview

Kidonk Kidonk is a viral social media trend where someone "deletes" 42 photos from a gallery and the photos disappear one by one, in reverse order, with a rhythmic sound. This project recreates that moment as a small web app that anyone can open from a link, load their own photo, and record the deletion for their own video.

The app is a single static page styled like the macOS Photos app. It needs no sign-up, no server, and no database. Photos never leave the user's device.

## 2. Problem Statement

People who want to join the trend currently have to fill their real phone gallery with 42 copies of a photo and actually delete them, which is tedious and risky (they may delete real photos by accident). There is no simple, safe tool that reproduces the look and rhythm of the trend. A browser-based recreation removes the setup work, protects the user's real gallery, and makes the effect easy to replay and record.

## 3. Goals

1. A first-time user goes from opening the link to watching the full 42-photo deletion in under 30 seconds.
2. The deletion sequence feels close enough to the trend that users are comfortable posting a screen recording of it.
3. The app runs entirely on Vercel's free tier with zero ongoing cost, regardless of traffic.
4. No user photo is ever uploaded to a server.
5. The app works on current mobile Safari (iOS) and Chrome (Android), which is where the trend is recorded.

## 4. Non-Goals

| Out of scope for v1 | Reason |
|---|---|
| User accounts or login | Adds a backend and friction for a one-minute experience. |
| Sharing a link to someone else's photo set | Requires server storage (e.g. Vercel Blob); revisit in v2 if requested. |
| Built-in video recording or export | Native screen recording on iOS and Android already covers this and is higher quality. |
| Deleting photos from the user's real gallery | Browsers cannot do this, and it would be unsafe. The app only simulates deletion. |
| Photo editing, filters, or captions | Not part of the trend; adds scope without improving the core moment. |

## 5. Target Users

**Trend participant:** A casual social media user, mostly on a phone, who wants to make their own Kidonk Kidonk video quickly. Low patience for setup, no technical knowledge.

**Content creator:** Someone who makes several takes, tries different photos, and cares about timing and visual polish because the recording will be posted.

## 6. User Stories

**Trend participant**
- As a trend participant, I want to pick a photo from my gallery so that the grid fills with my own image.
- As a trend participant, I want to take a photo with my camera directly so that I don't have to save it to my gallery first.
- As a trend participant, I want to see a confirmation before deletion so that the moment feels like the real "Delete 42 Photos?" prompt.
- As a trend participant, I want to cancel the confirmation so that I can reposition my screen recording before starting.
- As a trend participant, I want the photos to vanish one by one from last to first with a sound so that the video matches the trend.

**Content creator**
- As a content creator, I want to replay the deletion without re-selecting photos so that I can record multiple takes quickly.
- As a content creator, I want my photos to still be there after I refresh the page so that I don't lose my setup.
- As a content creator, I want to swap to a different photo set so that I can try another version.

**Edge cases**
- As a user who picks fewer than 42 photos, I want my photos repeated to fill all 42 slots so that the grid is always full.
- As a user who picks more than 42 photos, I want only the first 42 used so that the count stays correct.
- As a user who picks a file that is not a readable image, I want a clear message telling me to choose another photo.

## 7. Requirements

### P0: Must-have

**R1. Photo input (gallery and camera)**
Two entry points on the empty state: "Choose from Library" and "Take Photo".
- [ ] "Choose from Library" opens the device picker with `accept="image/*"` and multiple selection.
- [ ] "Take Photo" uses `capture="environment"` and opens the rear camera on mobile. On desktop it falls back to a file picker.
- [ ] Up to 42 images are accepted; extra files are ignored.
- [ ] Unreadable files are skipped; if none are readable, an error message is shown and the app stays in the empty state.

**R2. 42-photo grid**
- [ ] The grid always renders exactly 42 tiles in 4 columns.
- [ ] If fewer than 42 source photos exist, they repeat in order (photo index = slot index mod photo count).
- [ ] All tiles show a selected state (blue checkmark) once loaded.
- [ ] The title bar shows "42 Photos" and the toolbar shows "42 Photos Selected".

**R3. Delete confirmation**
- [ ] Clicking the trash button opens a macOS-style alert: title "Delete 42 Photos?", a short explanation, and two buttons, "Cancel" and "Delete".
- [ ] "Delete" is styled as destructive (red text or red button).
- [ ] "Cancel", the Escape key, or clicking outside the alert closes it with no changes.
- [ ] The trash button is disabled while deletion is running.

**R4. Reverse, one-by-one deletion**
- Given 42 selected tiles, when the user confirms "Delete", then tiles are removed starting from tile 42 down to tile 1, one at a time.
- [ ] Each removal plays a scale-down and fade animation (about 240 ms).
- [ ] Interval between removals starts around 180 ms and accelerates to about 70 ms by the last tile.
- [ ] The photo counter decrements with each removal (42, 41 ... 0).
- [ ] The grid auto-scrolls so the tile being removed is always visible.
- [ ] A short "kidonk" sound plays on each removal, alternating two pitches.
- [ ] On supported Android devices, a 12 ms vibration fires per removal.
- [ ] After the last tile, the empty state shows "No Photos" with "Replay" and "Use Other Photos".

**R5. Local persistence without a database**
- [ ] Before storing, each photo is resized to a maximum of 500 px on the long edge and encoded as JPEG at 0.82 quality.
- [ ] Resized photos are saved to IndexedDB (database `kidonk`, store `photos`, key `set`).
- [ ] On page load, saved photos are restored and the grid renders immediately.
- [ ] "Use Other Photos" clears the stored set.
- [ ] If IndexedDB is unavailable (e.g. some private browsing modes), the app still works for the session and simply does not persist.

**R6. Hosting**
- [ ] The app deploys to Vercel as static files with no build step.
- [ ] No network requests are made after the page loads, apart from optional fonts.

### P1: Nice-to-have

- **Custom sound upload:** let the creator replace the synthesized sound with their own audio clip (stored in IndexedDB).
- **Speed control:** Slow, Normal, Fast presets for the deletion interval.
- **Countdown:** optional 3-second countdown after confirming, so the user can settle their screen recording.
- **Light and dark appearance:** follow `prefers-color-scheme`, with a manual toggle.
- **PWA manifest:** "Add to Home Screen" with an icon and fullscreen display.

### P2: Future considerations

- **Shareable sets:** upload the resized set to Vercel Blob and create a share link. Keep the storage layer behind a small interface now (`save`, `load`, `clear`) so it can be swapped later.
- **Built-in recording:** capture the grid with `MediaRecorder` and export a video.
- **Other trend variants:** configurable photo count (e.g. 10, 42, 100).

## 8. Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Markup, styles, logic | Plain HTML, CSS, and vanilla JavaScript in one `index.html` | No framework or build step needed for a single screen; fastest load on mobile. |
| Photo input | `<input type="file" accept="image/*">`, with `capture="environment"` for camera | Native gallery and camera on iOS and Android with no permissions code. |
| Image resizing | Canvas API (`drawImage` + `toBlob`) | Keeps storage small and rendering smooth. |
| Storage | IndexedDB (stores Blobs directly) | Much larger capacity than localStorage (about 5 MB, text only), no server needed. |
| Sound | Web Audio API (oscillator + gain envelope) | No audio file to host; tiny and instant. `AudioContext` must be created on the "Delete" click to satisfy mobile autoplay rules. |
| Haptics | Vibration API (`navigator.vibrate`) | Supported on Android; silently ignored on iOS. |
| Animation | CSS keyframes, with `prefers-reduced-motion` respected | GPU-friendly and simple. |
| Hosting | Vercel static hosting, deployed from GitHub | Free, HTTPS by default (required for camera input), automatic deploys on push. |
| Analytics (optional) | Vercel Web Analytics free tier | Measures page views and funnel without collecting photos. |

**Suggested file structure**

```
kidonk/
  index.html        # markup, styles, and script
  favicon.png
  README.md
  vercel.json       # optional, only if custom headers are needed
```

If the project grows (P1 and beyond), migrate to Vite with vanilla TypeScript. It still deploys as static output on Vercel.

**App states**

```
empty ──(photos chosen)──> ready ──(trash)──> confirming
  ^                          ^                   │    │
  │                          └────(Cancel)───────┘    │
  │                                               (Delete)
  │                                                   v
  └──(Use Other Photos)── done <──(last tile)── deleting
                           └──(Replay)──> ready
```

## 9. Design Guide: macOS Photos View

The interface imitates a macOS Photos window: calm, neutral, precise. The only strong visual moment is the deletion itself, so everything else stays quiet.

### 9.1 Layout

**Desktop (width ≥ 768 px):** a centered window, max 960 px wide and 640 px tall, on a soft desktop background.

```
┌──────────────────────────────────────────────────────────┐
│ ● ● ●            Recents                          [🗑]    │  title bar + toolbar (52 px)
├─────────────┬────────────────────────────────────────────┤
│ Library     │ Recents                                    │
│  Recents  ◀ │ 42 Photos                                  │
│  Favorites  │ ┌────┬────┬────┬────┐                      │
│             │ │ ✓  │ ✓  │ ✓  │ ✓  │                      │
│ Albums      │ ├────┼────┼────┼────┤                      │
│  Kidonk     │ │ ✓  │ ✓  │ ✓  │ ✓  │   4-column grid      │
│             │ └────┴────┴────┴────┘                      │
│ (220 px)    │                     42 Photos Selected     │
└─────────────┴────────────────────────────────────────────┘
```

**Mobile (width < 768 px):** the window fills the screen. The sidebar is hidden, window corners are square, and the traffic lights stay visible in the title bar so the macOS look survives in a screen recording. Respect safe-area insets at the top and bottom.

### 9.2 Color

Use macOS system colors. Define them as CSS variables with light and dark values.

| Token | Light | Dark | Use |
|---|---|---|---|
| `--window` | `#FFFFFF` | `#1E1E1E` | Main content background |
| `--sidebar` | `#F0F0F2` (with blur) | `#2A2A2C` (with blur) | Sidebar |
| `--titlebar` | `#F6F6F6` | `#2C2C2E` | Title bar and toolbar |
| `--separator` | `#D9D9DC` | `#3A3A3C` | Hairline borders (1 px) |
| `--label` | `#1D1D1F` | `#F5F5F7` | Primary text |
| `--secondary` | `#6E6E73` | `#98989D` | Counts and captions |
| `--accent` | `#007AFF` | `#0A84FF` | Selection, checkmarks, focus rings |
| `--destructive` | `#FF3B30` | `#FF453A` | Delete button and trash hover |
| Traffic lights | `#FF5F57`, `#FEBC2E`, `#28C840` | same | Window controls (decorative) |

### 9.3 Typography

- Font stack: `-apple-system, BlinkMacSystemFont, "SF Pro Text", "Helvetica Neue", "Segoe UI", sans-serif`. This shows real SF Pro on Apple devices and a close match elsewhere.
- Window title: 13 px, semibold, centered.
- Content heading ("Recents"): 22 px, bold.
- Body and sidebar items: 13 px, regular (the macOS standard).
- Counts and captions: 11 to 12 px, `--secondary`, `font-variant-numeric: tabular-nums` so the counter doesn't jitter as it changes.
- Sentence case everywhere except proper names. No all-caps labels.

### 9.4 Components

**Window:** 10 px corner radius, 1 px `--separator` border, shadow `0 20px 60px rgba(0,0,0,.25)`.

**Traffic lights:** three 12 px circles, 8 px apart, 20 px from the left edge. Decorative only.

**Sidebar item:** 28 px tall, 6 px radius. Selected item gets a light accent tint background.

**Photo tile:** square, `object-fit: cover`, 2 px gap between tiles, no radius. Selected state: a 20 px blue circle with a white checkmark at the bottom right, plus a 3 px inset accent border.

**Trash button:** 28 × 28 px toolbar button, SF-style outline trash icon, 6 px radius. Hover shows a subtle gray background; the icon turns red on hover.

**Delete alert (macOS style):**

```
        ┌───────────────────────────────┐
        │            [ app icon ]       │
        │                               │
        │      Delete 42 Photos?        │  13 px bold, centered
        │  These photos will be removed │  11 px, secondary
        │  from this album.             │
        │                               │
        │  [   Cancel   ] [  Delete   ] │  Delete = red
        └───────────────────────────────┘
```

- 260 px wide, 12 px radius, translucent background with backdrop blur, centered on the window.
- Buttons are equal width, 28 px tall, 6 px radius. "Delete" uses `--destructive` background with white text. "Cancel" is the default for Escape; Enter triggers "Delete".
- Appears with a 150 ms scale from 0.95 to 1 and a fade. The rest of the window dims to 30% black.

**Empty state:** centered in the content area. Heading "No Photos" (17 px semibold), one line of helper text, and two buttons: "Choose from Library" (accent, filled) and "Take Photo" (gray, bordered).

### 9.5 Motion

- One signature moment: the reverse deletion. Tiles scale to 0 and fade over 240 ms with a slight overshoot (scale 1.08 at 40%).
- Alert open and close: 150 ms.
- No other decorative animation.
- With `prefers-reduced-motion: reduce`, tiles disappear instantly (sound still plays) and the alert opens without scaling.

### 9.6 Accessibility

- All buttons reachable by keyboard with a visible 2 px `--accent` focus ring.
- The alert uses `role="alertdialog"`, traps focus, and returns focus to the trash button on close.
- Counter updates use `aria-live="polite"`, throttled so screen readers are not flooded during deletion.
- Text contrast meets WCAG AA in both light and dark mode.

## 10. Privacy

- Photos stay on the device in IndexedDB and are never sent to any server.
- State this on the empty screen in one plain sentence: "Your photos stay on this device."
- Clearing site data in the browser, or tapping "Use Other Photos", removes the stored photos.

## 11. Success Metrics

Measured with Vercel Web Analytics custom events (no photo data collected).

**Leading indicators (first 2 weeks)**
- Activation: at least 60% of visitors load a photo (event `photos_loaded`).
- Completion: at least 80% of users who load photos confirm deletion (event `delete_confirmed`).
- Replay rate: at least 30% of completers press "Replay" at least once (signals recording takes).
- Error rate: under 2% of photo selections end in the unreadable-image message.

**Lagging indicators (1 to 2 months)**
- Returning visitors above 15%.
- Hosting cost stays at $0 on the Vercel free tier.

## 12. Open Questions

| Question | Owner | Blocking? |
|---|---|---|
| Do we have rights to use the original trend audio, or must we keep a synthesized sound? | Legal / creator | Yes, before adding any real audio |
| Should the mobile layout keep the macOS window chrome, or switch to an iOS Photos look that feels more native on phones? | Design | No |
| Is 180 ms to 70 ms the right tempo compared with the most popular trend videos? | Design (compare with 3 to 5 reference videos) | No |
| Do we want a visible watermark or URL in the corner for organic growth when videos are posted? | Stakeholder | No |

## 13. Timeline and Phasing

| Phase | Scope | Estimate |
|---|---|---|
| v1 | P0 requirements, macOS design, Vercel deploy | 2 to 3 days |
| v1.1 | P1: countdown, speed control, light/dark toggle, PWA | 2 days |
| v2 | P2: shareable sets via Vercel Blob, optional recording | To be scoped after v1 metrics |

No hard deadline, but trends fade quickly, so shipping v1 fast matters more than polish in P1.