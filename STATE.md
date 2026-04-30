# Wick Rider — Project State

> Day-trading scalper game **and** standalone trading harness. Single-file HTML, lifts infrastructure from the sister project Trade By Wire (`../trade_by_wire/`). Backup submission for Cursor Vibe Jam 2026 (deadline **2026-05-01 13:37 UTC**).
>
> **Working name:** Wick Rider. Easy to rename — change `<title>`, `<h1>`, the README, and the folder later.

---

## 🎯 PICK UP HERE (cold-start pointer)

- **Project folder:** `C:\Users\lucyl\Desktop\hold\projects\stocks_app\wick_rider\`
- **Sister project to lift from:** `C:\Users\lucyl\Desktop\hold\projects\stocks_app\trade_by_wire\index.html` (live, ~2550 lines, pushed to https://github.com/lucyellu/trade-by-wire)
- **Reference for game-mechanic feel:** `C:\Users\lucyl\Desktop\hold\projects\stocks_app\nyan-vita\web-port\` (the user said "I found myself in profit more times than not playing" this — capture that feeling, drop the cat IP).
- **Aesthetic reference:** `C:\Users\lucyl\Desktop\hold\projects\stocks_app\trade_by_wire\inspo.gif` (8.8MB — too large for direct view; see screenshots `C:\Users\lucyl\Pictures\Screenshots\Screenshot (3743).png` and `(3746).png`). Wireframe / generative-coding-studio aesthetic, not CRT phosphor.
- **Today:** 2026-04-29 (Wed). Deadline: Fri 2026-05-01 13:37 UTC. Realistic build window: Thu Apr 30 + Fri morning.
- **Current phase:** PHASE 0 (planning + scaffolding done — see TODO.md). Next is PHASE 1: visual scaffold.

## What this is, in one paragraph

A canvas-rendered trading "harness" where developing 1-minute candlesticks fill in real-time at adjustable speed (default 60×, optional 1× = real-time). The visual language is wireframe / Bloomberg-meets-Processing: black background, thin white lines, multiple small annotated panels around the edges with tiny mono-font labels. The player presses ↑/↓ to BUY/SELL fractions of buying power (the same TBW stacking model: each click = one step at the configured %). Score = realized P/L over a session. Vibe Jam version is replay-against-historical-data with a Firebase leaderboard. Standalone harness version (post-jam) swaps the data source for a live broker bridge (Alpaca paper) so the same UI works for actual scalp trading practice.

## Why we're building it

- **For the user:** TBW is a game-shaped game. Wick Rider is a tool-shaped game — the kind of thing you might leave open during market hours. The "developing candle" visualization is missing from real trading platforms (most show only completed bars or raw tick lines), and seeing time pressure within the bar is a real edge for scalpers.
- **For the jam:** different aesthetic + different mechanic = a second submission slot is wasted otherwise. TBW stays the primary jam pick; Wick Rider is the backup.

## Decisions locked in

| Decision | Rationale |
|---|---|
| Working name: **Wick Rider** | User's own first suggestion; captures the candle mechanic; reads as either game or tool. Easy to rename later. |
| Single-file HTML, no build step | Matches TBW pattern. Deploys to Netlify drop-and-go. |
| Lift TBW infrastructure verbatim | Profile, Firebase leaderboard, audio module, settings, position-stacking model, Yahoo fetch, mute/speed controls. ~70% of plumbing is already done elsewhere. |
| Procedural canvas rendering, no sprite sheets | Matches the wireframe-generative aesthetic of `inspo.gif`. Player + trail + candles all drawn each frame from primitives. |
| Position model: TBW stacking (avg-entry + cap) | Already implemented + tested (60 unit tests). Each click = one step at `settings.stepPct`. |
| Game mode = jam scope; Live mode = post-jam stretch | Live broker integration is risky in 1.5 days. Ship the game first; live mode can land in a v1.1 after Friday. |
| Default speed = 60× (1 sec/bar) | Matches TBW; matches the "60x sped up" feel the user described. Slider 1×–60× adjustable. |
| Color palette: monochrome + 2 accents | Black bg, grayscale wireframe, green/red only on candle bodies + P/L. Inspo gif is fully monochrome but trader expects bull/bear color. |

## Decisions still open

- **Final name** — Wick Rider works; Morning Bell more professional; ask before publishing.
- **Game-over condition** — TBW used "session complete" (60 bars) and "trader exited" (manual). Wick Rider could add a max-drawdown stop ("you blew up your account") for tension. TBD.
- **Score formula on leaderboard** — raw $ P/L, %, vs-benchmark, or blended (TBW uses $ P/L). Likely same as TBW for cross-comparability since they share Firebase namespace.
- **Candle interval** — 1-minute matches Yahoo Finance fetch resolution. 5-second or 15-second would feel snappier but requires synthetic interpolation.

## Reuse map from TBW

| Need | Source in TBW (`../trade_by_wire/index.html`) | Notes |
|---|---|---|
| Profile + onboarding modal | search `'NEW TRADER'` | Just rebrand to new name |
| Firebase REST helpers (`fbWrite`, `fbRead`, `submitToLeaderboard`, `fetchGlobalLeaderboard`) | search `FIREBASE_DB` | Same `games-8d8a6` project, **add `game:'wick'` tag** to entries so Wick Rider doesn't pollute TBW's board |
| Audio module (envTone, sfxCoin, sfxAttack, sfxStartHum, etc.) | search `function envTone` | Drop in wholesale; replace ATC chatter |
| Position model (`applyOrder`, `setPosition`, `closePosition`, `state.netShares`/`avgEntry`) | search `function applyOrder` | Drop in wholesale, change names if rebranding |
| Settings overlay + persistence (`settings.stepPct`, `settings.speedMult`, `tickMs()`) | search `const settings = ` | Same |
| Yahoo Finance fetch (`fetchYahooBars`) + custom-ticker UI | search `async function fetchYahooBars` | Same |
| Speed slider HUD + speed control | search `setSpeedMult` | Same |
| Mute toggle + persistence | search `setMuted` | Same |
| MACD calc + bars-only renderer | search `function computeMACD` | Optional — Wick Rider may not need MACD |
| Time & sales side panel | search `pushTnsRow` | Restyle for wireframe aesthetic |
| `_tbw_test.mjs` testing pattern | local file in TBW | Mirror as `_wick_test.mjs` once core logic exists |

## Aesthetic summary (from `inspo.gif` screenshots)

- Pure black background
- 1–2px white/grey lines
- Multiple small panels with thin frames arranged around the edges
- Each panel has tiny mono-font labels: "workflow", "001", "Procedural generation", "Random shapes / Processing", "Mouse tracking", section dividers
- Wireframe 3D shapes (rotating organic blobs, polyhedra, mesh surfaces) inside panels
- Curved line plots, dot scatter plots, grid surfaces
- Bottom-right caption: "Other workspaces are reflections of the workflow of digital artist or specialised applications" — set the tone
- Total opposite of CRT phosphor (no scanlines, no flicker, no chromatic fringe). Clean, calm, slick.

## How to run (once Phase 1 ships)

```bash
cd wick_rider
python -m http.server 9001   # different port from TBW (8089) and Plushie Rush (8088)
# Open http://localhost:9001
```

## Files (planned)

| File | Purpose |
|---|---|
| `index.html` | The game. Single-file. |
| `STATE.md` | This file. |
| `TODO.md` | Phased checklist. |
| `CLAUDE.md` | Architecture + run commands for cold-start agents. |
| `_wick_test.mjs` | Unit tests (mirrors TBW's pattern). Local only — gitignored. |
| `data/` | Optional: pre-fetched bar bundles for offline play. May not need if Yahoo fetch works. |
| `.gitignore` | Standard pattern from TBW. |
| `wick_launch.bat` | Local launcher (port 9001). Gitignored. |
