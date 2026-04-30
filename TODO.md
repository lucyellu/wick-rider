# Wick Rider — TODO

Phased build. Each phase is a clear deliverable; mark done with `[x]` as you go.

## Phase 0 — Planning + scaffolding ✅

- [x] Create `wick_rider/` folder
- [x] Write `STATE.md` (cold-pickup pointer + decisions log + reuse map)
- [x] Write `TODO.md` (this file)
- [x] Write `CLAUDE.md` (architecture for next-session pickup)
- [ ] Initial empty `index.html` skeleton (deferred to Phase 1)

## Phase 1 — Visual scaffold (the wireframe aesthetic) ✅

Goal: open `index.html`, see the inspo.gif vibe — black bg, thin white panels around edges, tiny labels, blank canvas in middle. No game logic yet.

- [x] `index.html` skeleton: `<head>` with mono fonts (VT323, Share Tech Mono, IBM Plex Mono)
- [x] CSS palette + reset (black bg, near-white text, monochrome with green/red accents)
- [x] Outer frame: corner cropmarks + edge tick marks
- [x] Top strip: title block (`WICK RIDER · v0.1`), TICKER, PORTFOLIO + $/% P/L row, LIVE indicator + session timer (4 panels in a flex row)
- [x] Left strip: 3 vertical panels — POSITION (with exposure bar), ORDER BOOK (placeholder), INVENTORY (cash/positions/bp)
- [x] Right strip: TAPE (time & sales placeholder), LEADERBOARD RANK, CONTROLS reference
- [x] Bottom strip: SELL / BUY buttons (compact 50px tall, color-coded green/red)
- [x] Center: chart canvas with 12×8 grid placeholder + "AWAITING TICKS" text
- [x] Tiny labels everywhere — every panel has `000 · app` / `001 · sym` / etc. overlapping the top border, plus chart-corner sub-labels
- [x] Pulsing LIVE dot (green) in top-right
- [x] No CRT effects — clean wireframe only
- [x] Mobile portrait media query — collapses to single-column flow with chart still dominant; hides BOOK/CONTROLS/LIVE on mobile, shrinks buttons but keeps tappable

**Acceptance:** loads, looks like the inspo gif, no errors. ✅ Ships at port 9001. No interactivity required yet (BUY/SELL buttons flash on click but do nothing else).

**Run:** `cd wick_rider && python -m http.server 9001` then open `http://localhost:9001`.

## Phases 2–5 (combined first pass) — gameplay MVP shipped ✅

Lifted TBW's mechanic + dropped in the new candle renderer in one push so user could feel the loop. Specifics of what's in:

- **Candle renderer** (Phase 2): wireframe-outlined candles, no solid fills. Green-up / red-down strokes only. Wicks, bodies, current-price marker on developing bar with dashed extension to right edge. Faint 4-row grid. Avg-entry dashed horizontal line when in position. Right-edge price scale (hi/lo).
- **Developing-bar animation** (Phase 2): the rightmost candle is "live" — body and wick grow as 15 synthesized intra-bar ticks fire over the bar's window. At 60×, that's ~67ms per intra-tick (15 per second). Inner glow + slightly thicker stroke distinguishes the developing bar.
- **Tick synthesis** (Phase 3): `synthesizeBarTicks(o, h, l, c, 15)` walks o → c, plants h and l at random intermediate indices, fills the rest with linearly-interpolated noise clamped to [l, h].
- **Yahoo Finance fetch** (Phase 3): lifted from TBW. Custom ticker via prompt overlay on load. Default SPY. Cache per-ticker.
- **Position model** (Phase 4): full TBW stacking model — `applyOrder`, `closePosition`, `setPosition`, exposure cap at ±100%. Both `state.netShares` (signed) + `state.avgEntry` are canonical.
- **HUD updates** (Phase 4): PORTFOLIO total + $/% P/L (color-coded green/red), POSITION panel (LONG/SHORT/FLAT + exposure %, qty + avg, animated exposure bar), INVENTORY (cash, positions, buying power), SESSION timer, current price + bar number in chart corners.
- **Tape (time & sales)** (Phase 4): right-side panel updates on each trade — last 8 actions in a compact list.
- **Insufficient-buying-power toast** (Phase 4): fixed-position red banner near buttons; fires `sfxAttack` on rejection.
- **Game loop** (Phase 5): `intraTick()` advances the developing bar, locks it when ticks are exhausted, advances to next. Auto-ends on `state.curBarIdx >= allBars.length`. End-of-session summary modal with replay / new ticker buttons. Buy-and-hold benchmark in stats.
- **Audio** (partial Phase 6): `envTone` engine + `sfxBuy` / `sfxSell` / `sfxCoin` / `sfxAttack` / `sfxBarClose`. Coin/attack fires on portfolio-direction-change at bar close. M key mutes (persisted).
- **Inputs**: ↑/W/→/D = BUY, ↓/S/←/A = SELL, Esc/Q = close out, M = mute. Keyboard ignores text input fields.
- **Ticker prompt overlay** on first load — type a ticker (default SPY), press Enter or click "OPEN MARKET". Error message if symbol can't be fetched.
- **Lazy audio init** on first user gesture (Chrome autoplay policy compliance).

**Acceptance:** ✅ load page → ticker prompt → Enter → candles develop in real-time at 60× → BUY/SELL works, exposure stacks to 100% then rejects → ~60s later session ends with summary.

**Status:** gameplay loop is complete and playable. Remaining work in Phase 6+ is polish.

## Round 2 additions ✅

- [x] **Space bar = close all** (also Esc, Q — Space is canonical)
- [x] **Volume sub-pane** under main chart (outlined wireframe bars, color-coded up/down). Collapsible via `−` button in panel corner.
- [x] **MACD histogram sub-pane** (12·26·9, bars-only — no signal line per the wireframe aesthetic). Collapsible.
- [x] **B/W monochrome mode** — toggleable via `[B/W]` pill in the chart panel label OR `B` key. Persisted in localStorage. Up = white, down = grey. Both DOM and canvas pick up the swap.
- [x] **Zoom out** — `VISIBLE_BARS` bumped 60 → 80 so candles read denser/calmer.
- [x] **Trade markers on chart** — green ▲ below price for BUY, red ▼ above for SELL, hollow ◇ for CLOSE. Drawn at the bar where each order filled. Persisted across the session and visible while bars scroll.
- [x] **CLOSE button** in the bottom strip (between SELL and BUY) — works on desktop *and* mobile so users without a keyboard can flatten.
- [x] **Mobile portrait** — all three buttons stay tap-friendly; subpane heights shrink (vol 40px, macd 50px) so the chart stays dominant; existing media query already collapses side panels.
- [x] **Controls panel** updated with `space` (close) and `b` (b/w) keys.

## Phase 2 (kept for reference) — Candle renderer (the core mechanic visual)

Goal: canvas draws candlesticks. The most-recent candle is the "developing" one — its body and wick grow as ticks come in within the bar's window.

- [ ] Coordinate system: x = bar index, y = price. Auto-scale price range with margin.
- [ ] Static candles: O/H/L/C four-line candles (thin wireframe — no fill, just outlined rect for body + thin lines for wicks)
- [ ] Filled bodies: green stroke if close > open, red stroke if close < open. Bodies stay outlined (wireframe), not solid filled. Optional: thin diagonal hatch fill on hover.
- [ ] Developing candle: track `currentBarTicks[]` — array of mid-bar prices. Body grows from open to current; wick stretches to running high/low.
- [ ] Smooth interpolation: rather than jumpy redraws, interpolate the developing candle's geometry frame-to-frame for slick feel.
- [ ] X-axis time labels (HH:MM tiny mono labels under the chart)
- [ ] Y-axis price labels on the right (tiny mono, current-price highlighted)
- [ ] Crosshair on hover (thin white dashed lines, price + time labels)
- [ ] When a bar closes (1 minute elapses at current speed), it locks in. New developing candle starts at next index.
- [ ] Auto-scroll: as new bars come in, the chart scrolls left to keep the developing candle ~70% of the way across.

**Acceptance:** play a test bundle (or random walk simulation), see candles fill smoothly. Watching one bar form is satisfying.

## Phase 3 — Data layer

Goal: feed the candle renderer with real or simulated tick data.

- [ ] Lift `fetchYahooBars(ticker)` from TBW (1-min bars for past day)
- [ ] **Tick simulation within bars:** Yahoo gives O/H/L/C per minute; we need *intra-bar ticks* to make candles "develop". Synthesize them: linear walk from open → close with random excursions to high/low. ~10–20 simulated ticks per bar.
- [ ] Speed: at 60× speed, 1 minute of data plays in 1 second. So a single bar (60 sec real time) → 1 sec at 60×, with ~15 ticks → tick every ~66 ms.
- [ ] Speed slider 1×–60× (lift from TBW). Re-derives tick interval.
- [ ] Custom ticker input (lift from TBW menu).
- [ ] Bundle fetch fallback (`data/bars_<sym>_<date>.json`) for offline / SPY default.
- [ ] **Live mode hook (deferred):** stub for connecting to Alpaca WebSocket; no implementation yet.

**Acceptance:** type "AAPL", press play, see real AAPL candles forming at 60× speed.

## Phase 4 — Position model + UI hookup

Goal: BUY/SELL works, P/L tracks, position panel updates.

- [ ] Lift TBW position model (`applyOrder`, `setPosition`, `closePosition`, `state.netShares`/`avgEntry`)
- [ ] Lift TBW settings system (`settings.stepPct`, `settings.speedMult`, `saveSettings()`)
- [ ] Wire keyboard: ↑/W/→/D = BUY one step, ↓/S/←/A = SELL one step, Esc/Q = close out
- [ ] Wire mobile buttons (compact, bottom strip)
- [ ] Render: position panel (LONG/SHORT/FLAT + qty + avg + exposure %)
- [ ] Render: P/L panel (huge total, $ + % below)
- [ ] Render: time & sales feed (tape — every BUY/SELL/CLOSE row)
- [ ] Insufficient buying power toast (lift from TBW)
- [ ] Visualize position on chart: thin horizontal line at avg entry price; current-bar marker (▲ if long, ▼ if short)

**Acceptance:** play, click BUY 4×, see position fill to 100%, see avg entry line on chart, P/L updates as candles develop.

## Phase 5 — Game loop + scoring

Goal: a 60-bar session plays end-to-end, score gets recorded.

- [ ] Session timer: 60 bars at current speed. At 60×, that's ~1 minute. At 1×, that's ~1 hour.
- [ ] End-of-session: closes any open position at last close, summary modal pops.
- [ ] Summary modal: trades count, win rate, total P/L, vs-benchmark.
- [ ] Optional: max-drawdown game-over (e.g. -25% = "blown up", session ends early).
- [ ] Auto-submit to Firebase leaderboard for sessions > 10 sec (lift from TBW).
- [ ] **Important:** tag entries `game: 'wick'` so they don't pollute TBW's leaderboard.
- [ ] Replay button on summary modal.

**Acceptance:** complete a session, see summary, see entry on global leaderboard.

## Phase 6 — Audio + polish

- [ ] Lift TBW audio module verbatim (`envTone`, `sfxCoin`, `sfxAttack`, ambient hum, etc.)
- [ ] **New: candle-close sound** — quick low blip on each bar closing (so you feel the rhythm even with eyes elsewhere)
- [ ] Coin SFX on portfolio up-tick (already in TBW)
- [ ] Attack SFX on portfolio down-tick (already in TBW)
- [ ] Mute toggle + persistence (lift from TBW)
- [ ] Smooth animations: lerp the developing candle, fade in new candles (60fps, no judder)
- [ ] Procedural trail / particles? — optional, only if it doesn't fight the wireframe aesthetic. Consider thin fading lines from cursor position rather than literal particles.

**Acceptance:** plays well, sounds tight, feels slick.

## Phase 7 — Mobile + deploy

- [ ] Mobile portrait media query — collapse panels to top + bottom strips, chart fills middle
- [ ] Touch-friendly button sizes (min 80px tap target)
- [ ] Test on actual phone if possible
- [ ] `git init`, push to `lucyellu/wick-rider` GitHub repo
- [ ] Netlify deploy (drag-and-drop or repo connect)
- [ ] **If submitting to Vibe Jam:** add `<script async src="https://vibej.am/2026/widget.js"></script>` back into `<head>` (it's removed from TBW per user request)
- [ ] Submit URL to vibej.am/2026 before 2026-05-01 13:37 UTC

**Acceptance:** live URL works, plays on phone, mobile leaderboard submission works.

## Phase 8 — Live trading harness mode (deferred, post-jam)

Goal: same UI, real broker. The standalone-harness payoff.

- [ ] Settings toggle: GAME / LIVE
- [ ] LIVE mode: connect to Alpaca paper trading WS (use TBW's existing key flow)
- [ ] Real ticks → real developing candles
- [ ] BUY/SELL → real paper orders via Alpaca REST
- [ ] Real P/L from Alpaca account, override the simulated portfolio value
- [ ] Add a clear "PAPER TRADING" red banner so it can never be mistaken for live $$$
- [ ] (Stretch) IBKR bridge mode using `C:\Users\lucyl\Desktop\ib\ibkr_bridge.py` for users with TWS

**Acceptance:** flip to LIVE, place a paper order on Alpaca, see it execute in their dashboard, confirm position syncs.
