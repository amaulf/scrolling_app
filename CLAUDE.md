---
description:
alwaysApply: true
---

# CLAUDE.md — ScrollLearn: Alexander the Great

This file is read automatically by Claude Code at the start of every session.
Do not delete it. Update it when the project evolves.

---

## What This Project Is

A scroll-driven educational storytelling app. People scroll to learn instead of doom-scroll. Each story is a cinematic, mobile-first experience where scrolling *drives* the narrative — like controlling a film with your thumb.

Single HTML file prototype. Goal: validate the concept before building a full app.

---

## Current State

- Opening screen + Scene 1 + Scene 2 fully built and working
- Both scenes have real Midjourney images (in `assets/images/`)
- Scenes 3, 4, 5 + Completion screen are written in STORY.md but not yet coded
- Everything lives in `index.html` — HTML + CSS + JS in one file

---

## Tech Stack

- **Pure HTML/CSS/JS** — no framework, no build step, no dependencies
- **No npm, no React, no bundler** — must work by double-clicking
- **Google Fonts** — Cormorant Garamond (headlines/pullquotes) + Inter (body)
- **Vanilla JS scroll engine** — custom, no GSAP, no libraries

---

## Design System

```
Colors:
  --bg:       #080810   (near-black base)
  --accent:   #c9a84c   (warm gold — headlines, progress bar, pullquotes)
  --text:     #f0ede6   (warm off-white for body text)
  --text-dim: #9a9080   (muted warm grey for secondary text)

Fonts:
  Headlines:  Cormorant Garamond, italic, 600 weight, 36px, line-height 1.25
  Body:       Inter, 500 weight, 18px, line-height 1.7
  Pullquotes: Cormorant Garamond, italic, 17px, line-height 1.5, gold
  Labels:     Inter, 500 weight, 10px, wide letter-spacing

Mobile width: 390px max-width on body, centered
```

---

## Scroll Architecture — READ THIS BEFORE TOUCHING LAYOUT

```
<div class="scene-wrapper" style="height:2200px" id="w1">  ← defines scroll room
  <div class="sticky-panel" id="panel1">                    ← 100lvh, stays pinned
    <div class="scene-bg s1-bg scroll-blur" id="bg1"></div> ← image background
    <div class="scene-overlay"></div>                        ← dark overlay 52%
    <div class="scene-gradient"></div>                       ← bottom fade to #080810
    <div class="scene-top">                                  ← scene number top-left
      <div class="scene-top-line" id="line1"></div>
      <div class="scene-num" id="num1">01 / 05</div>
    </div>
    <div class="scene-content">                              ← text, lower third
      <div class="scene-headline" id="headline1"></div>      ← word-by-word via JS
      <div class="scene-body" id="body1">                    ← word-by-word via JS
        <p>...</p><p>...</p>
      </div>
      <div class="scene-pullquote" id="pullquote1"></div>    ← slides in at 91%
    </div>
  </div>
</div>
```

**Key constraints:**
- `height: 100lvh` on `.sticky-panel` — use `lvh` (large viewport, stable). Do NOT use `dvh` or `window.innerHeight`-based variables — they change when iOS URL bar shows/hides causing layout jumps.
- `.scene-content`: `position: absolute; bottom: 0; max-height: calc(100lvh - 90px); overflow: hidden; padding: 0 28px 44px`. The `overflow: hidden` clips at the screen's bottom edge (off-screen, invisible). The `max-height` prevents content from reaching the scene number label (at `top: 52px`).
- **Wrapper height = pacing.** Currently 2200px for a slow, deliberate reading experience.
- **Do not rearrange these elements.** Layer order (bg → overlay → gradient → scene-top → scene-content) is intentional.

---

## Animation System — DO NOT SIMPLIFY

All animations are scroll-driven via `rawProgress = clamp(-rect.top / (wrapH - vh), 0, 1)`.
All animations are **bidirectional** — they reverse when scrolling up.

### 1. Scroll-Driven Image Reveal
Background starts fully black (`brightness(0)`) while the headline builds on darkness.
After headline is done, image rises from black to full brightness.
Driven directly by JS — add class `scroll-blur` to `.scene-bg` (sets `transition: none`).
```
rawProgress 0.00 → 0.50 : pure black (brightness 0) — headline lands on darkness
rawProgress 0.50 → 0.75 : image rises in (brightness 0 → 1), sharp, no blur
rawProgress 0.75+        : fully visible
```
JS: `bg.style.filter = \`brightness(\${imageProgress})\`;`
On scene reset (before enter), restore: `bg.style.filter = 'brightness(0)'`.

### 2. Word-by-Word Headline
Headline words are `<span class="word">` elements injected by JS from the `headlines` object.
Lit up sequentially with `.lit` class (opacity 0→1, blur 3px→0).
```
rawProgress 0.00 → 0.50 : headline words appear left to right
```

### 3. Word-by-Word Body Text
Body `<p>` elements are processed by JS — each word becomes `<span class="body-word">`.
Global index spans across paragraphs so the flow is continuous.
Opacity only (no blur — visually distinct from headline).
```
rawProgress 0.55 → 0.88 : body words appear (starts after headline is done)
```

### 4. Pullquote Slide-In
Gold italic quote slides up with border-left accent.
```
rawProgress > 0.91 : pullquote.classList.add('in') — slides up, border appears
rawProgress < 0.91 : pullquote.classList.remove('in') — reverses
```

### 5. Ken Burns Zoom + Dim
Background continuously zooms and dims as scroll progresses.
```js
bg.style.transform = `scale(${1.0 + rawProgress * 0.18})`;
bg.style.opacity = 1 - rawProgress * 0.12;
```
Do NOT put `transform` in the CSS transition — causes shivering on mobile.
CSS transition on `.scene-bg` should only cover `filter`, and `scroll-blur` removes even that.

### 6. Scene Number Entry
`num` and `line` elements get `.in` class on first enter (CSS transition handles animation).
Managed via `sceneState[i]` — fires once per scene entry, resets when scene exits upward.

---

## JS Architecture

```js
// Scene loop — extend by adding to this array when adding scenes
const sceneState = { 1: false, 2: false };   // add: 3: false, etc.
[1, 2].forEach(i => { ... });                 // add 3, 4, 5 here

// Headlines — add new entry for each scene
const headlines = {
  headline1: "He was 13 years old...",
  headline2: "At 20, he inherited...",
};

// Body word injection — add scene number to this array
[1, 2].forEach(i => { ... });   // add 3, 4, 5 here
```

**Desktop scroll limiter**: caps wheel delta at 72px, lerps at 0.1. Wrapped in `isTouch` check — completely skipped on mobile to avoid fighting native touch scroll.

---

## Adding a New Scene — Follow This Exactly

1. Read STORY.md for content (headline, body, pullquote, any special elements)
2. Copy an existing scene HTML block (e.g. Scene 2)
3. Increment IDs: `w3`, `panel3`, `bg3`, `line3`, `num3`, `headline3`, `body3`, `pullquote3`
4. Update scene number label: `03 / 05`
5. Add background CSS class `.s3-bg` with the correct image or gradient palette
6. Add `scroll-blur` class to `<div class="scene-bg">` if using a real image
7. Add `3: false` to `sceneState`
8. Add headline text to the `headlines` object
9. Add `3` to both `forEach` arrays (body injection + scene loop)
10. Do NOT introduce new animation types — keep the system consistent

---

## Background Palettes (for scenes without images yet)

```css
Scene 3 (Undefeated):   dark red, bronze, smoke      → #2a0a08, #8b4a1a, #1a0a05
Scene 4 (Edge of World): dark teal, deep green, mist  → #062018, #0a3025, #041510
Scene 5 (Death):         deep purple, burgundy, candle → #100818, #2a0820, #080810
```

---

## Images

Images go in `assets/images/`. Naming convention: descriptive Midjourney filename.

To apply an image to a scene:
```css
.s3-bg {
  background-image: url('assets/images/filename.jpg');
  background-size: cover;
  background-position: center 30%; /* adjust vertical focus */
}
```
Always add `scroll-blur` class to the bg div. The overlay and gradient stay on top — essential for text readability.

---

## Mobile Layout — Critical Rules

- **`100lvh` everywhere** — never `dvh`, never `window.innerHeight` as CSS variable. `lvh` is stable; `dvh` changes when iOS URL bar shows/hides causing jumps.
- **`overflow: hidden` on `.sticky-panel`** — clips anything below the screen edge.
- **`max-height: calc(100lvh - 90px)` on `.scene-content`** — prevents text from ever reaching the scene number area (at `top: 52px`). Overflow clips off-screen (invisible).
- **`line-height: 1.7` on body text** — this is the minimum that fits all scene 2 content on modern iPhones (390px × 844px+). Do not increase without checking total content height.
- **No CSS `transform` in transition** — causes image shivering on mobile scroll. Only `filter` in transition (and `scroll-blur` removes even that).
- **Touch scroll untouched** — the desktop wheel limiter is wrapped in `isTouch` and never runs on mobile.

---

## Scene 3 Special Elements (from STORY.md)

Scene 3 has a battle names list between headline and body:
```
Granicus. Issus. Gaugamela. Tyre. The Hydaspes.
```
Style: Cormorant Garamond, 22px, italic, gold (`#c9a84c`), each name on its own line.
Add as a separate `<div class="scene-battles">` element. The word-by-word body injection should skip this element — it should appear as a block (not word-by-word), tied to scroll like the pullquote.

Scene 5 has an isolated line immediately after the headline:
```
Younger than most people reading this.
```
Style: Inter, 18px, italic, weight 300, text-dim color. Add as `<div class="scene-aside">`.

---

## What NOT To Do

- Do not add external JS libraries (no GSAP, no ScrollMagic, no jQuery)
- Do not split into multiple files
- Do not add new animation patterns mid-story — consistency is key
- Do not change the font pairing
- Do not use white or light backgrounds anywhere
- Do not add navigation, menus, or UI chrome
- Do not change max-width from 390px
- Do not use `dvh` or `--vh` CSS variables for panel/content heights
- Do not put `transform` in the CSS `transition` property of `.scene-bg`
- Do not put the pullquote inside `.scene-content` — it must live directly in `.sticky-panel`, absolutely positioned at `bottom: 44px`

---

## How to Run

```bash
python3 -m http.server 8000
# Desktop: http://localhost:8000
# Mobile (same WiFi): http://192.168.1.68:8000
```
