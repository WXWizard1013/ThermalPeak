## Release History

### v1.0 — Initial Alpha (Mar 16, 2026)
* Proof-of-concept. Four cities (New York, London, Singapore, Shanghai), basic GFS scanning, text-only Telegram alerts, manual CSV logging, trades resolved by hand.

### v2.0 — Major Overhaul (Mar 18, 2026)
* **City coverage** expanded from 4 to 30 cities worldwide.
* **Full Telegram UI** — replaced plain-text alerts with a proper inline keyboard interface for navigating markets, managing positions, and adjusting settings.
* **Automated trade management** — auto Take-Profit (+60%) and Stop-Loss (-40%) introduced. Bot now tracks and resolves open positions without manual input.
* **Live CLOB pricing** — switched from cached Gamma data to Polymarket's live order book. D+1 market isolation ensures prices are never stale.
* **Rebuilt probability engine** — norm.cdf replaces the old sigmoid scorer. Factors in Polymarket's 2% fee drag, applies edge caps to filter noise, and weights TAF heavier near market expiry.
* **PostgreSQL backend** — persistent trade logging with async connection pooling. CSV fallback for lightweight deployments.

### v2.1 — Analytics & Maintenance (Mar 19, 2026)
* `/accu` — per-city accuracy tracking against actual Polymarket resolutions.
* `/summ` — daily performance and PnL summary.
* `/resolveall` — force-sweep all pending expired positions at once.
* `/resetlog` — wipe trade history and start clean.

### v2.2 — Stability (Mar 20, 2026)
* Refined data pipelines and API request handling for faster, more consistent market polling.
* Minor UI navigation improvements.

### v2.3 — Settings, Risk, Card & 30-City Coverage (Mar 21–22, 2026)
* **UI overhaul** — stripped the bottom keyboard entirely. Pure inline keyboard interface throughout.
* **Overhauled /settings** — two-level menu (Signals / Risk). Every parameter is now typed in conversationally instead of tapping preset buttons. Includes `/cancel` support.
* **Risk management suite** — configurable Take Profit (standard + high-confidence tier), Stop Loss, High Conf Stars threshold, Slippage Tolerance, and Max Daily Drawdown. Daily drawdown gate pauses scanning automatically when the limit is hit.
* **Slippage tolerance** — deducted from fair value before any edge calculation. Default 2%, max 10%.
* **Sharpe ratio** — calculated across all resolved trades and displayed in `/pnl`. Shows N/A until 10+ resolved trades.
* **Shareable PnL card** (`/card`) — generates a styled PNG card with the thermal gradient branding, Net PnL, win rate, Sharpe, trade stats, and the thermal win rate bar. Triggered via `/card` or the Share Card button in `/pnl`.
* **30-city coverage** — fixed market discovery to correctly surface all 30 active cities including Chinese mainland markets (Chengdu, Chongqing, Beijing, Shenzhen, Wuhan). Root cause: Polymarket uses a different question format ("Will the highest temperature in Chengdu be X°C on...") that required a separate parser and direct slug fetch.
* **Station bias corrections** — hardcoded micro-climate offsets for HKO Observatory (Hong Kong), CWA 46692 (Taipei city center), KLGA (LaGuardia), EGLC (London City), and ZSPD (Pudong coastal sea-breeze effect).
* **NOAA NWS blending** — human-edited US forecasts blended 55/45 with GFS for US cities.
* **Hours-ahead gate** — replaced strict UTC date matching with a 16–72h window to correctly handle Asian markets that resolve at noon UTC.

### v2.3.1 — Risk Engine, UI Polish & 35-City Coverage (Mar 23, 2026)
* **35-city coverage** — added San Francisco (KSFO) and Austin (KAUS), both confirmed against Polymarket resolution rules and added to NOAA coverage. Total active city count: 35.
* **Limit-order TP** — take-profit price is now fixed at entry as an absolute level (`entry × (1 + tp_threshold)`) and stored per-trade in the database/CSV. Survives bot restarts. When the TP threshold is changed in settings, all open positions are immediately recalculated and checked against the new level — closes any position already past the updated target.
* **SL min floor (Protection)** — new settings category. Configurable absolute ¢ floor that must be breached alongside the % stop-loss before SL fires. Protects cheap entries (e.g. 10¢) from being stopped out by normal market noise. Both conditions must be met simultaneously. Default 5¢, set to 0 to disable.
* **Duplicate signal suppression** — auto-scan alerts now only fire for genuinely new signals. If the same edge persists across scans with no new positions opened, the loop stays silent. `/signals` still shows all edges including already-open ones, labelled `📌 Already open — monitoring`.
* **PnL card auto-generated on `/pnl`** — removed the separate `/card` command. The HD 1280×680 card is now sent automatically every time `/pnl` is called. Pill centering fixed to measure actual text width — "In drawdown" no longer clips.
* **On-demand forecast refresh in `/signals`** — GFS, NOAA, and CLOB prices are all refreshed live when `/signals` is called manually. Previously only CLOB was refreshed, meaning edge calculations ran on hours-old forecast data.
* **TP/SL setting changes trigger immediate position check** — changing any risk threshold now runs `check_resolved_markets()` instantly with a retry, instead of waiting up to 30 minutes for the next scheduled check.
* **SL alert format unified** — Stop Loss alerts now show `SL@30% + 3¢ floor (6★)` matching the Take Profit format. Fixed double-negative loss display (`Loss: --$X`) and missing `daily_loss` update on SL hits.
* **Position detail overhauled** — shows TP as a fixed limit price (`TP: Limit 19%`), SL regime including floor, market price for YES and NO from CLOB bestAsk (independent values, not forced to sum to 100), and retry on failed price fetches.
* **Settings → Reset to Default** — one-tap button restores all 11 settings to factory values with a confirmation summary.
* **`/vol` loading indicator** — bot immediately replies "📡 Fetching latest market volumes..." before the 415-market CLOB fetch, so the user knows it's working.
* **All `_FakeUpdate` classes patched** — `reply_photo` added to every inline button proxy class. Previously `/pnl` via the Refresh button or home screen errored with `type object 'message' has no attribute 'reply_photo'`.
* **SL `daily_loss` tracking fixed** — SL hits now correctly increment the daily drawdown counter in both DB and CSV paths. Previously only market-resolved losses were counted.
