# CHILL SLIDE RAIL

Neon-city 2-pipe rail-grind endless runner. Mobile-first portrait. Single-file Three.js.

**Brand:** NEO-FROST (fictional cyberpunk hard-seltzer)
**Status:** v1.0
**See:** [DESIGN.md](DESIGN.md) for full spec.

## Run locally

```bash
cd chill-slide-rail
python3 -m http.server 8000
open http://localhost:8000
```

Or just open `index.html` directly (CDN-only deps).

## Deploy

```bash
npx surge . chill-slide-rail.surge.sh
```

## Controls

- **Mobile:** swipe / tap left or right of screen to switch pipes. Tap center = vertical hop. Swipe down = slide.
- **Desktop:** A/D or ←/→ = switch pipes. Space = hop. S/↓ = slide.

## Pitch deck

`marketing/pitch.html` — open in browser, shows the game inside a live iframe + ROI / deployment plan for "real" brand activation.

## Tech

- Three.js r160 (CDN)
- Vanilla JS, no build step
- WebAudio synth (no audio files)
- Procedural geometry (no model files)
- localStorage persistence
- Optional Firebase leaderboard (NO-OP shim if not configured)
- PWA installable (manifest + service worker)
