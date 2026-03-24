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

### v2.4 — Risk Redesign, Price Accuracy & Bug Fixes (Mar 24, 2026)
* **SL bug fixed (critical)** — `sl_min_floor` default of $5 was silently blocking SL on virtually all positions. With typical kelly sizes of $7–12, a 30% loss generates only $2–3 in absolute dollar terms — below the $5 floor. SL was effectively requiring a 50–75% price crash, not the configured 30%. Default is now $0 (disabled). The per-bucket volume filter handles cheap-entry noise upstream.
* **TP profit calculation fixed** — TP was recording profit at the current poll price, not the limit price. If the market moved from 13¢ to 40¢ between 5-minute polls, profit was overstated based on 40¢ instead of the stored TP limit. Now always uses `tp_price` for PnL calculation.
* **Nested timeout race fixed** — `check_resolved_markets()` was wrapped in a 20s outer timeout while the inner CLOB fetch also used a 20s timeout. When the outer fired first, the inner received a `CancelledError` that wasn't caught, silently aborting all position checks. Outer timeout raised to 45s.
* **TP matrix: confidence × hours-to-close** — replaced the flat dual-TP system (standard 60% / high-conf 80%) with a matrix that accounts for both signal confidence and time remaining. High confidence (8★+) with >12h left → 80%; any position with <6h remaining → 90% (hold to resolution). The <6h threshold is set high because temperature markets close before the daily maximum is recorded — the price rockets after market close, not before. Premature exits before resolution clip gains.
* **Volume filter moved to per-bucket** — the old city-level `ev_vol` filter ($10K) summed all buckets and let thin individual buckets through. A city with $10K spread across 20 buckets averages $500 per bucket. Filter now checks the specific bucket being signalled. Default lowered to $1K (~100× minimum kelly).
* **True mid price from (bestAsk + bestBid) / 2** — `token.price` from the CLOB API was being used as mid, but it does not reliably equal the true midpoint. Mid is now computed directly from `(bestAsk + bestBid) / 2` with fallback to `token.price` if either side is unavailable.
* **UPnL uses bestBid (real exit value)** — UPnL in the position detail card now reflects `bestBid` (what you'd actually receive selling right now) instead of mid. On a position with bid 58¢ and ask 63¢, mid would show -1% while the honest exit-based UPnL shows -9%.
* **Market price display updated** — position card now shows `YES ask X¢ / bid X¢ · NO ask X¢ · mid X¢` so all four relevant prices are visible at a glance.
* **Refresh triggers immediate TP/SL check** — tapping Refresh on any position card now runs `check_resolved_markets()` immediately after rendering. Previously the check only ran on the 5-minute poll, meaning a position that crossed TP/SL between polls could sit open for up to 5 minutes even after a manual refresh.
* **Resolve check interval: 30min → 5min** — automated TP/SL polling tightened significantly.
* **Slippage removed from signal scoring** — the flat 2% fair value haircut was equivalent to silently raising the edge threshold from 8% to ~10.2% without indicating this anywhere. Removed entirely from paper mode. Will be rebuilt as ask-price comparison in the live execution layer.
* **Protection sub-menu removed** — `sl_min_floor`, `slippage_pct`, `tp_high`, and `high_conf_stars` removed from Settings. Risk sub-menu simplified to three controls: Take Profit (base), Stop Loss, Max Daily Drawdown.
* **Positions button added to signal alerts** — signal notification now includes a `📋 Pos` button for immediate position check after a signal fires.
* **Loading messages auto-delete** — `/vol` and `/signals` loading indicators now delete themselves after the result is sent, keeping the chat clean.
