# Alexander — The Great
### A ScrollLearn Story

A scroll-driven educational story about Alexander the Great. Built as a mobile-first cinematic experience — scroll to learn instead of doom-scroll.

---

## Running Locally

**Option 1 — Python (no install needed on Mac)**
```bash
cd alexander
python3 -m http.server 8000
```
Then open: [http://localhost:8000](http://localhost:8000)

**Option 2 — Node**
```bash
npx serve .
```
Then open: [http://localhost:3000](http://localhost:3000)

---

## Testing on Mobile (same WiFi as your computer)

```bash
# 1. Start your local server (see above)

# 2. Find your computer's IP address
ipconfig getifaddr en0     # Mac (WiFi)

# 3. On your phone, open Safari or Chrome and visit:
http://YOUR_IP_ADDRESS:8000
# Example: http://192.168.1.45:8000
```

Every time you save `index.html`, refresh on your phone to see changes.

---

## Project Structure

```
alexander/
├── index.html          ← The entire story (HTML + CSS + JS)
├── CLAUDE.md           ← Instructions for Claude Code
├── STORY.md            ← All story content and copy
├── README.md           ← This file
└── assets/
    └── images/         ← Place Midjourney images here when ready
```

---

## Adding Images

When your Midjourney images are ready, drop them in `assets/images/` and update the background CSS in `index.html`:

```css
.s1-bg {
  background-image: url('assets/images/scene1-horse.jpg');
  background-size: cover;
  background-position: center;
}
```

See `CLAUDE.md` for full image naming convention.

---

## Working with Claude Code

This project is set up for Claude Code. Open the project folder in Cursor and Claude Code will automatically read `CLAUDE.md` for full context.

Example prompts to use in Claude Code:
- "Add Scene 3 from STORY.md"
- "Make the word-by-word animation slower"
- "Add the completion screen from STORY.md"
- "Swap in the real image for Scene 1"
- "Make the scene transition a white flash instead of dark"

---

## Current Status

- [x] Opening screen
- [x] Scene 1 — The Boy & The Horse
- [x] Scene 2 — The Crossing
- [ ] Scene 3 — The Undefeated
- [ ] Scene 4 — The Edge of the World
- [ ] Scene 5 — 32 Years Old
- [ ] Completion screen
- [ ] Real images (waiting on Midjourney)
