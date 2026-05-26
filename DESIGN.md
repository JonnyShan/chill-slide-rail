# CHILL SLIDE RAIL — Design Doc

Status: spec v1.0 · 2026-05-23 · author: Claude (Opus 4.7)
Brand: **NEO-FROST** (fictional cyberpunk hard-seltzer)
Mechanic: 2-parallel-pipe rail-grind endless runner

---

## High concept (one-liner)

You're a hooded rooftop runner grinding a neon-lit megacorp pipeline through a rain-soaked cyberpunk skyline. Two pipes run side-by-side. Tap to leap between them. Snag floating Hyo-Ketsu-style NEO-FROST cans, dodge drones / fan-blades / billboard collapses, survive 5 escalating boss gates, post your distance to the leaderboard.

---

## Locked decisions

| Axis | Decision |
|---|---|
| Mode | Solo endless |
| Brand | Fictional — **NEO-FROST** (no real-world IP risk) |
| Platform | Mobile-first portrait 9:16, also playable desktop |
| Visual | Photoreal-ish PBR neon (heavy bloom, wet reflectance, rain) |
| Loop | Endless + 5 boss gates every ~60s |
| Engine | Three.js r160 (CDN) single-file `index.html` |
| Audio | WebAudio procedural synth (no external files) |
| Persistence | localStorage + optional Firebase leaderboard hook |
| Distribution | Static HTML → Surge / GitHub Pages / iframe embed |

---

## Core mechanic

### Pipe rail
- Two parallel cylindrical pipes, radius ~0.5m, gap between centerlines 3.5m
- Pipes scroll toward camera at increasing speed (start 18 m/s → cap 42 m/s)
- Player auto-grinds: feet stuck to top of pipe, lean-into-turns
- Pipes are NOT straight — gentle curves left/right + height undulations (sin-wave field)
- Pipes diverge/converge — sometimes wider gap (jump harder), sometimes near-touching (jump trivial)

### Player state
- `lane`: 0 = left pipe, 1 = right pipe (binary, no center)
- `jumpT`: 0..1 jump progress (parabolic interp between pipe centerlines)
- `airT`: 0..1 vertical hop (independent — tap-up for vertical pickup grab)
- `balance`: -1..1 — slight wobble, recovered automatically; spent fully = fall

### Inputs
- **Mobile portrait:** swipe left = jump to left pipe, swipe right = right pipe, tap = vertical hop, swipe down = slide (duck)
- **Desktop:** A/← = left, D/→ = right, Space = hop, S/↓ = slide
- **Fail-safe:** ANY tap on the left third / right third of screen also moves to that pipe (no-swipe accessibility)

### Fail conditions
- Miss a jump (land between pipes) → ragdoll fall into void
- Hit unblockable obstacle (mech arm, dragon) → death anim
- 3 strikes from minor obstacles (drones, fans) → death (hp = 3, regen 1 every 200m clean)

---

## Run loop

Time / Distance bands (early run pace):

| Phase | t (sec) | Speed | Density | Event |
|---|---|---|---|---|
| Calibration | 0–8 | 18 m/s | Sparse cans only | Tutorial overlay |
| Warmup | 8–55 | 18→24 | Drones, gaps | — |
| BOSS 1 | ~60 | hold | scripted | **Drone Swarm** |
| Push 1 | 75–115 | 24→28 | + fans | Combo multipliers unlock at 100 |
| BOSS 2 | ~120 | hold | scripted | **Holo-Serpent** |
| Push 2 | 135–175 | 28→32 | + billboards | Free can wave |
| BOSS 3 | ~180 | hold | scripted | **Billboard Collapse** |
| Push 3 | 195–235 | 32→36 | + bipedal | Shield drops |
| BOSS 4 | ~240 | hold | scripted | **Holo-Ninja Duel** |
| Push 4 | 255–295 | 36→40 | + everything | — |
| BOSS 5 | ~300 | hold | scripted | **Megacorp Mech** |
| Eternal | 300+ | 40→42 | max | Score chase loop |

---

## Boss gates (5)

Each = 15–20s scripted sequence. Player still moves forward; boss occupies camera + scripts the obstacle pattern.

### 1. **Drone Swarm**
6 quadcopter drones spawn in 3 waves, each wave dictates which pipe is safe (red glow = bad, green = safe). Survive 3 waves.

### 2. **Holo-Serpent**
Translucent neon serpent slithers along/around the pipes. Body coils — player must alternate pipes to avoid head-strike. Tail-whip at end = forced jump.

### 3. **Billboard Collapse**
City billboards crash across the pipes from sides. Some span both pipes (must duck-slide). Some span only one (must switch). Crescendo: massive holo-screen falls — duck under window-gap.

### 4. **Holo-Ninja Duel**
Cyber-ninja teleports between pipes, throws projectile shuriken, leaves afterimages. Mirror-match — whichever pipe ninja is on = forbidden. Boss ends with mutual bow + teleport-out.

### 5. **Megacorp Mech**
Final. Bipedal mech walks behind player. Stomps alternate pipes (both pipes shake), fires plasma trails forcing zigzag, climaxes with grab-attempt — leap timing window 0.3s. Survive 20s → mech crashes into a building, player rides on through fireball.

After boss 5 → "ETERNAL MODE" tag in HUD, score-chase tier with all-obstacle-types-active forever.

---

## Scoring

- 1 point per meter
- Cans: 10 pts + combo (combo resets on damage)
- Combo: every 10 consecutive cans, multiplier +1 (cap x5)
- Boss survive: +500 + boss name plaque
- Personal best stored localStorage, leaderboard hits Firebase if configured

---

## Visual style

Photoreal-ish PBR neon, optimized for 60fps on iPhone 12+ / mid-Android.

### Palette (locked)
- Background sky: `#0a0419` (near-black violet)
- Pipe left: emissive cyan `#00e0ff` core, dark chrome shell
- Pipe right: emissive magenta `#ff2db5` core, dark chrome shell
- Rain: light blue alpha lines
- Can: ice-blue + lemon-yellow accent
- Player suit: matte black + neon-pink X accent (matches mockup hoodie)

### Lighting
- Directional moonlight from above-back (cool blue)
- 4 dynamic point lights tracking player (one per pipe-glow + 2 city-wash)
- Bloom: threshold 0.6, strength 1.4, radius 0.4
- Fog: exp2, density 0.012, color matches sky

### Environment
- Animated city skybox: 2 layers of tall buildings (parallax), procedural window-lights flickering
- Floating billboards along route (Japanese-glyph + NEO-FROST ads)
- Rain particle system (1000 pts, recycled)
- Cloud volume = lower-third dark fog ground

### Performance budget
- Triangles < 60k visible
- Drawcalls < 80
- Instanced mesh for windows + cans + drones
- Texture atlas, KTX2 if loadable
- DPR clamp to 1.5 on mobile

---

## Audio

100% procedural via WebAudio:
- **Music:** synthwave loop — analog square bass (E2 root), saw lead arpeggio (Phrygian), kick (sine 60Hz drop), open hi-hat (white noise + bandpass)
- **SFX:** grind drone (filtered noise), jump whoosh (pitch-down sine 800→200), can clink (FM bell), damage thud (kick + saw scrape), boss stinger (detuned saw chord), death (5th-down glissando)
- Pre-game silence; music ducks under boss stingers

Voice-line shim: TTS hooks present, defaults to silent — generate via ElevenLabs later if wanted.

---

## HUD (portrait 9:16)

```
┌─────────────────────────┐
│ ║ 1280m   x3   ❤❤♡     │  ← top bar: distance, combo, hp
│                         │
│                         │
│                         │
│      [GAME VIEW]        │
│                         │
│                         │
│                         │
│        🥫 47            │  ← bot-left: can count
│                  ⚡500   │  ← bot-right: score
│ ─── BOSS: DRONE SWARM ─ │  ← boss intro overlay
└─────────────────────────┘
```

Boss intro: full-bleed animated card slides in 0.4s, holds 1.2s, slides out. Soft-pause game logic during card.

---

## File structure

```
chill-slide-rail/
├── index.html           ← game (single file, ~3000 lines)
├── DESIGN.md            ← this doc
├── README.md            ← run + deploy
├── manifest.webmanifest ← PWA
├── sw.js                ← service worker (offline)
├── assets/
│   ├── logo.png         ← NEO-FROST logo (Flux gen)
│   ├── can-hero.png     ← can render (Flux gen)
│   ├── og.png           ← share image (Flux gen)
│   ├── favicon.png
│   └── splash.png       ← title splash
└── marketing/
    └── pitch.html       ← pitch proposal for "real" brand deployment
```

---

## Anti-goals

- No real-world brand assets (Kirin, Suntory, Asahi) — fictional NEO-FROST only
- No third-party login / data collection beyond optional Firebase leaderboard
- No "spin to win" gambling mechanic
- No real-money microtransactions
- No external audio file dependencies (procedural only)
- No external 3D model files — procedural geometry only (lighter, deterministic, ships in single HTML)

---

## Success criteria (A-grade gates)

1. ✅ Loads <2s on mid-mobile (3G throttled OK at <4s)
2. ✅ 60fps target on iPhone 12, 45fps floor on iPhone X
3. ✅ Zero console errors in 3 full runs
4. ✅ All 5 bosses encounterable in single run (verified by speed-cheat dev tool)
5. ✅ Win-loss feels fair — death always feels player-fault, not engine-fault
6. ✅ Boss intro overlay readable, dismissable
7. ✅ Touch + keyboard inputs both work
8. ✅ Leaderboard NO-OP shim works without Firebase config
9. ✅ Pitch HTML showcases live iframe + ROI mock
10. ✅ Visual matches reference mockup vibe (neon, rain, hoodie runner)

---

## Out of scope (v2)

- Multiplayer / battle royale (architected but not built)
- Real-time co-op
- Custom character skins
- Cosmetic shop with currency
- In-game purchases
- Quest/achievement system beyond personal best
- ElevenLabs voice acting
- Suno music

These can plug in via documented hook points (`__GAMIFY_BLOCK__` markers in code).
