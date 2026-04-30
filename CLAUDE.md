# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**Wick Rider** is a single-file HTML web app — half day-trading game, half real-time scalping harness. The visual gimmick: 1-minute candlesticks fill in real-time at adjustable speed (1× = real-time, 60× = full session in 1 minute). Aesthetic is wireframe / Bloomberg-meets-Processing — black background, thin white lines, tiny annotated panels around the edges. Sister project to `../trade_by_wire/`. Backup submission for Cursor Vibe Jam 2026.

**Read `STATE.md` first** for cold-start orientation, decisions log, and the reuse map from TBW.

**Read `TODO.md` second** for the phased build plan with checkboxes.

## Running

```bash
cd C:/Users/lucyl/Desktop/hold/projects/stocks_app/wick_rider
python -m http.server 9001
# open http://localhost:9001
```

Port allocation on this machine (verified 2026-04-30):
- **8088–8095 (entire 808x/809x range) — Plushie Rush** (it binds wide; any port in that band returns the Plush Rush HTML)
- 8089 — Trade By Wire (overrides Plush on this single port when TBW is up)
- **9001 — Wick Rider**

If you ever need another free port, start in 9000+ and curl-check first.

Or once it exists, double-click `wick_launch.bat` (gitignored, lifts the TBW pattern).

## Tests

```bash
node _wick_test.mjs   # mirrors TBW's _tbw_test.mjs pattern; local-only / gitignored
```

## Architecture (planned — see TODO.md for current implementation status)

Single-file HTML (`index.html`), no build step, no dependencies beyond Google Fonts CDN and the Firebase REST endpoint we share with TBW.

```
index.html            (everything — HTML, CSS, JS inline)
├── <head>            mono fonts, title
├── <body>
│   ├── outer frame   thin white border + corner cropmarks
│   ├── top strip     title block, session timer, ticker, P/L summary
│   ├── left strip    order book / position / inventory panels
│   ├── center        canvas — the chart (~70% of viewport area)
│   ├── right strip   time & sales, leaderboard rank, settings
│   └── bottom strip  BUY / SELL buttons (compact)
└── <script>          everything below the line, single inline block
    ├── settings + persistence  (lifted from TBW)
    ├── audio module             (lifted from TBW)
    ├── position model           (lifted from TBW: stacking + avg-entry + cap)
    ├── data layer               (lifted from TBW: Yahoo fetch + bundle fetch + tick synth)
    ├── candle renderer          (NEW — fill/empty animation per bar)
    ├── HUD updaters             (P/L panel, position panel, tape)
    ├── game loop                (60 fps requestAnimationFrame, tick scheduler)
    └── input handlers           (kbd + mobile, settings/menu/leaderboard overlays)

data/
├── bars_<SYM>_<DATE>.json   (optional pre-fetched bundles for offline play)
└── index.json               (catalog)
```

## What's lifted from TBW vs new

**Lifted (do not rewrite — copy from `../trade_by_wire/index.html`):**

| Subsystem | TBW search anchor |
|---|---|
| Settings + persistence | `const settings = (() =>` |
| Audio (envTone, sfxCoin, sfxAttack, hum) | `function envTone` |
| Position model (applyOrder/closePosition/setPosition) | `function applyOrder` |
| Yahoo fetch (custom ticker) | `async function fetchYahooBars` |
| Firebase leaderboard helpers | `async function fbWrite`, `submitToLeaderboard`, `fetchGlobalLeaderboard` |
| Profile + onboarding modal | `function getProfile`, `<div id="ob-overlay">` |
| Insufficient-funds toast | `function showToast` |
| Speed control | `function setSpeedMult`, `function tickMs` |
| Mute toggle + persistence | `function setMuted` |
| Settings overlay (step % + speed) | `<div id="settings-overlay">` |
| Time & sales side panel | `function pushTnsRow`, `function renderTns` |
| MACD bars renderer | `function computeMACD`, `function drawMACD` (optional) |

**New code:**

| Subsystem | Why new |
|---|---|
| Candle renderer with developing-bar animation | Core visual mechanic; doesn't exist in TBW |
| Tick synthesis within bars | Yahoo gives O/H/L/C per minute; we synthesize ~15 intra-bar ticks per bar to animate the developing candle |
| Wireframe panel chrome (left/right/top/bottom strips with tiny labels) | Different aesthetic from TBW's CRT phosphor look |
| Avg-entry horizontal line on chart | Lets the trader see the breakeven line visually |
| Crosshair on hover | Standard chart-reading UX |

## Aesthetic rules

These are guardrails so the wireframe vibe stays consistent.

- **Black background** (`#000` or near-black `#0a0a0a`). No gradients, no scanlines, no flicker. Clean.
- **Lines:** 1–2px max. White or near-white (`#f0f0f0`). Sharp.
- **Color:** monochrome by default. Two accent colors: green (`#5cdb5c`) and red (`#ff5c5c`) used **only** for candle bodies and P/L direction. Nothing else gets color.
- **Type:** monospace. VT323 for big readouts, Share Tech Mono / IBM Plex Mono for body text and labels. Letter-spacing slightly wide for the tech-doc feel.
- **Labels:** tiny mono labels everywhere ("001", "workflow", "tape · 60×", section dividers). Low opacity (~0.5–0.65). They're decoration AND functional.
- **Panels:** thin 1px white borders, transparent fill, internal padding generous.
- **Animation:** smooth `requestAnimationFrame` interpolation. **Lerp** the developing candle each frame so geometry doesn't snap. The user said "smooth slick lightening fast" — that's the goal.
- **No skeuomorphism.** No CRT, no scanlines, no chromatic aberration, no flicker. Wick Rider is *digital*, not retro. (TBW is the retro one.)

## API keys / external services

- **Firebase Realtime DB** — `games-8d8a6` (shared with TBW + Plushie Rush). New entries tag `game: 'wick'`.
- **Yahoo Finance via corsproxy.io** — same pattern as TBW. Free, public, may rate-limit aggressively for heavy use.
- **Alpaca paper trading (LIVE mode, deferred)** — keys stored in `localStorage`, entered via settings overlay.
- **No bundler, no npm, no backend** for the jam build.

## Differences from TBW (don't accidentally re-introduce)

- **No "RECORDING / VIDEO / PILOT / EJECT" terminology** — Wick Rider is straight trading-floor language from the start.
- **No CRT effects** — different aesthetic deliberately.
- **No fixed 60-bar session as the only mode** — Wick Rider is meant to also work as a live harness for hours.
- **No Vibe Jam widget by default** — add the `<script>` tag only if/when submitting.
