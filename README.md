# Series Landing Page — Developer Handoff Guide

Single-page onboarding flow with Rive animation intro followed by an 11-step profile-building card carousel. Fully responsive: mobile (<= 480px) and desktop (> 480px) with separate behaviors.

---

## Quick Start

```bash
node server.js
# → http://localhost:3000
```

Server binds to `0.0.0.0:3000`. All files served statically from project root.

---

## Project Structure

```
├── index.html                  # Single-file app (HTML + CSS + JS, ~3300 lines)
├── server.js                   # Node.js static HTTP server (port 3000)
│
├── series.riv                  # Rive animation — mobile
├── series_desktop.riv          # Rive animation — desktop
│
├── Card_Background.png         # Card texture background
├── Desktop Logo.png            # Top-left logo (desktop only)
├── Back_Desktop.svg            # Back arrow icon (desktop buttons)
├── Next_Desktop.svg            # Next arrow icon (desktop buttons)
├── Next Step.svg               # Next arrow icon (mobile)
├── Photo Upload.svg            # Upload icon on photo card
├── Final SVG.svg               # Final screen asset
│
├── Social Icons/
│   ├── Email.svg
│   ├── instagram.svg
│   ├── Linkedin.svg
│   └── X.svg
│
└── final animation/            # Card deck frames (9 PNGs)
    ├── Frame 241.png
    ├── Frame 243.png
    ├── Frame 245.png
    ├── Frame 246.png
    ├── Frame 247.png
    ├── Frame 248.png
    ├── Frame 249.png
    ├── Frame 250.png
    └── Frame 252.png
```

---

## Responsive System

| | Mobile | Desktop |
|---|---|---|
| **Breakpoint** | <= 480px | > 480px |
| **Rive file** | `series.riv` | `series_desktop.riv` |
| **Rive scale** | Fit width | Fit height × 1.2 |
| **Visible cards** | 3 (center + 1 each side) | 5 (center + 2 each side) |
| **Side card opacity** | 100% | 30% |
| **Arc radius** | 586px | 586 × 1.8 × 1.35 ≈ 1426px |
| **Arc angle** | 36° | 15° |

Page auto-reloads when crossing the 480px threshold. Detection variable: `isDesktop` (set once on load).

**Critical rule**: Mobile CSS/JS is locked. All desktop changes are gated behind `isDesktop` checks in JS and `@media (min-width: 481px)` in CSS.

---

## Card Flow & User Input Reference

The 11-step flow is defined in the `cardFlow` array. Here's every user input, its element ID, and how to collect the data:

### Step 0 — Name
| Field | Selector | Type |
|---|---|---|
| First name | `#name-card input[placeholder="First name"]` | text |
| Last name | `#name-card input[placeholder="Last name"]` | text |
| **Validation** | Both fields non-empty | |

### Step 1 — Birthday
| Field | ID | Type |
|---|---|---|
| Date of birth | `#birthday-input` | date |
| **Validation** | Value not empty | |

### Step 2 — Pronouns (single-select radio)
| Card ID | `#pronouns-card` |
|---|---|
| Options | She/her, He/him, They/them, Other |
| **Selected value** | `document.querySelector('#pronouns-card .card-option-wrapper.selected .card-option-row').dataset.value` |

### Step 3 — Ethnicity (multi-select toggle)
| Card ID | `#ethnicity-card` |
|---|---|
| Scroll container | `#ethnicity-scroll` |
| Options | Hispanic/latino, Middle Eastern, African/African Diaspora, South Asian, East Asian, Southeast Asian, Pacific Islander, Indigenous, White/Caucasian, Other |
| **Selected values** | `document.querySelectorAll('#ethnicity-card .card-option-wrapper.selected .card-option-row')` → read `.dataset.value` |

### Step 4 — Sexuality (multi-select toggle)
| Card ID | `#sexuality-card` |
|---|---|
| Scroll container | `#sexuality-scroll` |
| Options | Heterosexual, Gay, Lesbian, Bisexual, Pansexual, Asexual, Queer, Other |
| **Selected values** | Same pattern as ethnicity with `#sexuality-card` |

### Step 5 — Religion (multi-select toggle)
| Card ID | `#religion-card` |
|---|---|
| Scroll container | `#religion-scroll` |
| Options | Christianity, Islam, Judaism, Hinduism, Buddhism, Sikhism, Bahá'í Faith, Jainism, Shinto, Taoism, Zoroastrianism, Spiritual but not religious, Agnostic, Atheist, Other |
| **Selected values** | Same pattern with `#religion-card` |

### Step 6 — Location
| Field | ID | Type |
|---|---|---|
| Location input | `#location-input` | text with autocomplete |
| Dropdown | `#location-dropdown` | auto-populated |
| **API** | Nominatim OpenStreetMap (`nominatim.openstreetmap.org/search`) |
| **Debounce** | 300ms |

### Step 7 — Job Title
| Field | ID | Type |
|---|---|---|
| Job title | `#job-input` | text |

### Step 8 — Socials (skippable)
| Field | ID | Type | Notes |
|---|---|---|---|
| Email | `#email-input` | email | Required if not skipped |
| LinkedIn | `#linkedin-input` | text | Readonly → opens login URL → manual fallback |
| Instagram | `#instagram-input` | text | Same pattern |
| X (Twitter) | `#x-input` | text | Same pattern |

**Social platform URLs** (opened on field click):
- LinkedIn: `https://www.linkedin.com/login`
- Instagram: `https://www.instagram.com/accounts/login/`
- X: `https://x.com/i/flow/login`

Desktop opens URLs directly via `window.open()`. Mobile shows a confirm dialog first.

### Step 9 — Interests (skippable)
| Element | ID |
|---|---|
| Search input | `#interests-search` |
| Tags container | `#interests-tags` |
| **Selected interests** | JS variable `selectedInterests` (Set) |
| **Interest list** | 56 pre-defined tags; users can also type custom interests |

### Step 10 — Photo Upload
| Element | ID |
|---|---|
| Upload area | `#photo-upload-area` |
| File input | `#photo-file-input` |
| Preview img | `#photo-preview` |
| Upload icon | `#photo-upload-icon` |

---

## Collecting All User Data (Integration Point)

To submit the profile to a backend, gather all values at the end of the flow. Here's a ready-to-use collection snippet:

```javascript
const profileData = {
  firstName:    document.querySelector('#name-card input[placeholder="First name"]').value,
  lastName:     document.querySelector('#name-card input[placeholder="Last name"]').value,
  birthday:     document.getElementById('birthday-input').value,
  pronouns:     document.querySelector('#pronouns-card .selected .card-option-row')?.dataset.value,
  ethnicity:    [...document.querySelectorAll('#ethnicity-card .selected .card-option-row')].map(r => r.dataset.value),
  sexuality:    [...document.querySelectorAll('#sexuality-card .selected .card-option-row')].map(r => r.dataset.value),
  religion:     [...document.querySelectorAll('#religion-card .selected .card-option-row')].map(r => r.dataset.value),
  location:     document.getElementById('location-input').value,
  jobTitle:     document.getElementById('job-input').value,
  email:        document.getElementById('email-input').value,
  linkedin:     document.getElementById('linkedin-input').value,
  instagram:    document.getElementById('instagram-input').value,
  x:            document.getElementById('x-input').value,
  interests:    [...selectedInterests],  // JS Set variable
  photo:        document.getElementById('photo-preview').src  // base64 data URL
};
```

**Best place to hook this**: Inside `playFinalAnimation()` (search for `function playFinalAnimation`) — this runs after the user completes the last card (photo upload).

---

## Buttons & Navigation

### Primary Flow Buttons
| Button | ID | Action |
|---|---|---|
| Create your profile | `#cta-button` | Starts onboarding, triggers Rive state 1 |
| Next Step | `#next-step-button` | Rive state 2 → shows first card |
| Next (per-card) | `#name-card-next` | Advances card if validation passes |
| Back | `#go-back-btn` | Returns to previous card |
| Skip (socials) | `#skip-socials` | Skips socials card |
| Skip (interests) | `#skip-interests` | Skips interests card |

### Header Buttons (Desktop Only)
| Button | ID | Destination |
|---|---|---|
| Logo (top-left) | `#desktop-logo` | `https://series.so` (new tab) |
| Login (top-right) | `#already-account-btn` | `https://series.so` (new tab) |

### Post-Flow Button
| Button | ID | Action |
|---|---|---|
| Return to Series | `#return-to-series` | Opens native SMS (`sms:`) |

**To change any destination URL**: search for the button ID in `index.html` and update the `href` or `window.open()` call.

---

## Progress Ring

| Property | Value |
|---|---|
| Container ID | `#progress-ring` |
| Foreground stroke class | `.ring-fg` |
| Total steps | 11 (`RING_STEPS`) |
| Circumference | 60.319 (`RING_CIRCUMFERENCE`) |
| Update function | `updateProgressRing(step, complete)` |
| Position | Top-center, 24px from top edge (desktop) |

Ring completes and fades out when `playFinalAnimation()` runs.

---

## Key JS Constants (Tweakable)

Search for these variable names in `index.html` to adjust:

### Layout & Motion
| Variable | Value | Description |
|---|---|---|
| `BREAKPOINT` | 480 | Mobile/desktop threshold (px) |
| `ARC_RADIUS` | 586 / 1426 | Card orbit radius (mobile/desktop) |
| `ARC_ANGLE` | 36 / 15 | Degrees between cards (mobile/desktop) |
| `ARC_DURATION` | 640 | Card transition time (ms) |
| `ARC_EASE` | cubic-bezier(0.5, 0, 0, 1) | Transition curve |
| `VISIBLE_RANGE` | 1 / 2 | Side cards shown (mobile/desktop) |
| `SIDE_OPACITY` | 1 / 0.3 | Unfocused card opacity (mobile/desktop) |
| `BTN_BOTTOM` | 40px / 32px | Button distance from bottom (mobile/desktop) |

### Final Deck Animation
| Variable | Value | Description |
|---|---|---|
| `PHASE1_DURATION` | 1200 | Gather phase (ms) |
| `PHASE2_DURATION` | 600 | Fan-out phase (ms) |
| `welcomeAngle` | -10 | Welcome card rest angle (degrees) |
| `SWIPE_THRESHOLD` | 80 | Min drag to dismiss (px) |
| `VELOCITY_THRESHOLD` | 0.5 | Min velocity to dismiss (px/ms) |
| `MAX_ROTATION` | 15 | Max card rotation during drag (degrees) |

---

## Rive Animation

| Property | Mobile | Desktop |
|---|---|---|
| File | `series.riv` | `series_desktop.riv` |
| State machine | `State Machine 1` | `State Machine 1` |
| Input name | `Number 1` | `Number 1` |
| Value 0 | Intro animation | Intro animation |
| Value 1 | "Before You Know" state | "Before You Know" state |
| Value 2 | "Handoff" state | "Handoff" state |
| Fit mode | FitWidth | FitHeight |
| Canvas wrapper | `#rive-wrapper` | `#rive-wrapper` (pointer-events: none) |

---

## Social Confirm Dialog (Mobile Only)

| Element | ID |
|---|---|
| Overlay | `#social-confirm-overlay` |
| Dialog | `#social-confirm-dialog` |
| Open button | `#social-confirm-open` |
| Manual input button | `#social-confirm-manual` |
| Cancel button | `#social-confirm-cancel` |

On desktop, clicking a social field opens the URL directly and converts to manual input on return.

---

## CSS Media Query Reference

- **Mobile styles**: Default (no media query)
- **Desktop styles**: `@media (min-width: 481px) { ... }` — search this in `index.html`
- **Desktop text sizes**: All use `vh` units for proportional scaling
- **Card width**: Mobile uses `vw`-based calc, desktop uses `vh`-based calc

---

## Fonts

- **SF Pro Display** — Card titles, button text
- **SF Pro Regular** — Input fields, option labels, subtitles, skip text, login button
- **SF Pro Medium** — Login button text

Loaded via `@font-face` at top of `<style>` block in `index.html`.
