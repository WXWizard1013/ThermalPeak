## Release History

### v1.0 — Initial Alpha (Mar 16, 2026)
* Proof-of-concept. Four cities (New York, London, Singapore, Shanghai), basic GFS scanning, text-only Telegram alerts, manual CSV logging, trades resolved by hand.

### v2.0 — Major Overhaul (Mar 18, 2026)
* **City coverage** expanded from 4 to 30 cities worldwide.
* **Full Telegram UI** — replaced plain-text alerts with a proper inline keyboard interface for navigating markets, managing positions, and adjusting settings.
* **Automated trade management** — auto Take-Profit (+60%) and Stop-Loss (-40%) introduced.
* **Live CLOB pricing** — switched from cached Gamma data to Polymarket's live order book.
* **Rebuilt probability engine** — norm.cdf replaces the old sigmoid scorer. Factors in 2% fee drag, applies edge caps, weights TAF near expiry.
* **PostgreSQL backend** — persistent trade logging with async connection pooling. CSV fallback.

### v2.1 — Analytics & Maintenance (Mar 19, 2026)
* `/accu` — per-city accuracy tracking. `/summ` — daily PnL summary. `/resolveall` — force-sweep pending positions. `/resetlog` — wipe trade history.

### v2.2 — Stability (Mar 20, 2026)
* Refined data pipelines and API request handling. Minor UI navigation improvements.

### v2.3 — Settings, Risk, Card & 30-City Coverage (Mar 21–22, 2026)
* **Pure inline keyboard UI** — bottom keyboard stripped entirely.
* **Overhauled /settings** — conversational parameter input with `/cancel` support.
* **Risk management suite** — TP (standard + high-conf tier), SL, slippage tolerance, max daily drawdown.
* **Shareable PnL card** — auto-generated HD PNG on every `/pnl`.
* **30-city coverage** — fixed market discovery for Chinese mainland markets.
* **Station bias corrections** — HKO, KLGA, EGLC, ZSPD, CWA 46692.
* **NOAA NWS blending** — 55/45 with GFS for US cities.
* **Hours-ahead gate** — 16–72h window replaces strict UTC date matching.

### v2.3.1 — Risk Engine, UI Polish & 35-City Coverage (Mar 23, 2026)
* **35-city coverage** — San Francisco (KSFO) and Austin (KAUS) added.
* **Limit-order TP** — fixed at entry, stored per-trade, survives restarts.
* **SL min floor** — absolute ¢ floor alongside % SL. Default 5¢.
* **Duplicate signal suppression** — auto-scan stays silent if no new positions opened.
* **Settings → Reset to Default** button.

### v2.4 — Signal Quality, ICAO Fixes & 38-City Coverage (Mar 24–27, 2026)
* **38-city coverage** — Istanbul (LTFM), Moscow (UUEE), Mexico City (MMMX).
* **ICAO corrections** — KJFK → KLGA, EGLL → EGLC, ZSSS → ZSPD.
* **Edge cap at 13%** — very high edge = GFS wrong, not market.
* **Min entry floor 14%** — below this had 17–27% win rate.
* **Per-bucket volume filter**, **spread filter** (≤ 4¢), **NO bet guards** (75–92% YES), **fair value ceiling 80%**, **city deduplication**.

### v2.4.1 — Profit Floor & Reliability (Mar 27, 2026)
* **Profit floor protection**, **TP cap fix**, **SL bypass fix**, **`/pos` command**.

### v2.4.2 — Position Detail & UPnL (Mar 28, 2026)
* **Individual position detail card** — bid/ask, Kelly, UPnL, GFS, NOAA, METAR, Polymarket link.
* **UPnL mid-price fix**. **Resolve/Remove buttons**. **Positions pagination** at 8 per page.

### v2.4.3 — Backtest Notebook v1.0 (Mar 28–29, 2026)
* **Backtest notebook** (`thermalpeak_backtest.ipynb`) — Colab parameter sweeps against real resolved trades.

### v2.4.4 — UI Polish & City Expansion (Mar 29–30, 2026)
* **`bucket_with_c()`** — inline °C next to °F outcomes. **`/vol` and `/cities` pagination** at 20 per page.

### v2.4.4.1 — Signal Quality & City Risk Management (Apr 1, 2026)
* **Edge threshold raised to 10%**.
* **Per-city σ calibration** — tight (1.0–1.5°C) for stable tropical, default (1.8–2.2°C) mid-latitude, wide (2.5–3.0°C) orographic/coastal.
* **City exclusion system** — `/exclude` + automated 10-SL-loss alert. Persisted via PostgreSQL.

### v2.5 — Multi-Model Consensus & Dead Zone Pause (Apr 2, 2026)
* **5-model gate** — 3 of 5 required (GFS, ECMWF, ICON, UKMET, GEM).
* **ICON boost** for European cities. **Dead zone pause** 22:00–02:00 UTC. **`/random`** command.

### v2.6 — 44-City Expansion & Station Audit (Apr 3, 2026)
* **44-city coverage** — Busan (RKPK), Panama City (MPMG), Amsterdam (EHAM), Kuala Lumpur (WMKK), Helsinki (EFHK), Jakarta (WIHH).
* ICON boost extended to Amsterdam and Helsinki. Per-city σ for new cities. Slug fetch fallback.

### v2.6.1 — Full Resolution Station Audit (Apr 3, 2026)
* **Station corrections** — Moscow UUEE → UUWW (Vnukovo), Houston KIAH → KHOU (Hobby), Taipei coords → RCTP. Jakarta WIHH and Panama City MPMG verified.

### v2.6.2 — Emergency Abort, Daily Summary Fix & UI Fixes (Apr 9, 2026)
* **`/abort` emergency command** — two-step confirmed kill-switch. Stops scanning, resolves all open positions at live price, blocks new trades. `/cont` resumes.
* **`🚨 Abort` on `/start`** — one tap from home screen with `× Close` beside it.
* **`/summ` day-scoped** — queries only today's trades (00:00–23:59 UTC), not all-time.
* **Consensus gate raised 1.5 → 2.5°C** — April spring transition was blocking every city at 1.5°C.
* **Market discovery hardened** — slug fetch now covers all 44 cities (was 7). Removed `closed=True` filter incorrectly blocking active markets. `process_event` no longer requires `active=True`.
* **`× Close` button fixed** — was missing its handler; now correctly deletes the message on all views.
* **`/vol` °C strip fixed** — regex now correctly removes `(X.X°C)` suffix including the degree symbol.
* **GFS Drift alert bold** — auto-drift now sends with `parse_mode=Markdown`.
* **GFS schedule in UTC** — replaced WIB times with `00:00 · 06:00 · 12:00 · 18:00 UTC`.
* **`/cities` fallback fixed** — showed only 12 cities when event cache was empty; now shows all 44.
