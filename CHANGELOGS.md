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
* **Risk management suite** — configurable Take Profit, Stop Loss, High Conf Stars threshold, Slippage Tolerance, and Max Daily Drawdown.
* **Sharpe ratio** — calculated across all resolved trades and displayed in `/pnl`.
* **Shareable PnL card** — 1280×680 HD dark card with thermal gradient branding.
* **30-city coverage** — fixed market discovery to correctly surface all 30 active cities including Chinese mainland markets.
* **Station bias corrections** — hardcoded micro-climate offsets for HKO, Taipei, KLGA, EGLC, and ZSPD.
* **NOAA NWS blending** — human-edited US forecasts blended 55/45 with GFS for US cities.
* **Hours-ahead gate** — 16–72h window to correctly handle Asian markets that resolve at noon UTC.

### v2.3.1 — Risk Engine, UI Polish & 35-City Coverage (Mar 23, 2026)
* **35-city coverage** — added San Francisco (KSFO) and Austin (KAUS).
* **Limit-order TP** — TP price fixed at entry, stored per-trade, survives restarts.
* **SL min floor (Protection)** — configurable absolute ¢ floor alongside % stop-loss.
* **Duplicate signal suppression** — auto-scan alerts only fire for genuinely new signals.
* **PnL card auto-generated on `/pnl`** — removed separate `/card` command.
* **On-demand forecast refresh in `/signals`** — GFS, NOAA, and CLOB all refreshed live.
* **Settings → Reset to Default** — one-tap button restores all settings.

### v2.4 — Risk Redesign, Price Accuracy & Bug Fixes (Mar 24, 2026)
* **SL bug fixed (critical)** — `sl_min_floor` default $5 was silently blocking SL on virtually all positions. Default now $0.
* **TP profit calculation fixed** — now always uses `tp_price` for PnL, not poll price.
* **TP matrix: confidence × hours-to-close** — replaced flat dual-TP with a matrix. <6h remaining → 90% (hold to resolution).
* **Volume filter moved to per-bucket** — default lowered to $1K.
* **True mid price from (bestAsk + bestBid) / 2**.
* **UPnL uses bestBid (real exit value)**.
* **Resolve check interval: 30min → 5min**.
* **Slippage & Protection settings removed**.
* **Loading messages auto-delete**.

### v2.4.1 — Price Accuracy & Display Fixes (Mar 24, 2026)
* **Ghost bid filter** — UPnL falls back to mid when spread > $0.10.
* **Entry price uses ask** — real cost basis recorded at signal time.
* **Ask/bid swap guard** — inverted values corrected before display.
* **1 decimal place on live probability** — `Now: 11.0%` instead of `11%`.

### v2.4.2 — Additional Fixes & Signal Quality (Mar 25, 2026)
* **fetch_live_yes uses /price endpoint** — eliminates false TP triggers on thin markets.
* **bestAsk removed from all Gamma fallbacks**.
* **SL loss capped at threshold** — simulates real limit stop order execution.
* **Resolve check: 5min → 3min**.
* **Spread-aware breakeven stop** — SL moves to entry at +30% gain with spread ≤ 5¢.
* **4¢ spread filter at signal entry** — blocks thin markets at evaluation time.
* **Resolve All / Clear Log confirmations** added.

### v2.4.3 — Signal Quality, Display Fixes & Infrastructure (Mar 27, 2026)
* **14% minimum entry price floor** — based on 48-trade segmented analysis. Sub-14% → 29% WR; 18-25% → 47% WR.
* **SL alert shows capped exit price** — `Exit: 12.6%` not `Now: 9.0%`.
* **Position buttons numbered sequentially** — #1 through N, not database row IDs.
* **`/drift` command bug fixed** — was always showing zero delta due to premature cache refresh.
* **Drift alert format overhauled** — single combined message with timestamp.
* **PostgreSQL now live** — persistent trade history across restarts and redeployments.

### v2.4.4 — Risk UX, Log Redesign & Display Fixes (Mar 29, 2026)
* **Profit floor SL** — positions reaching +30% gain now lock in +5% profit on reversal instead of breakeven. Exits at `entry × 1.05` and records as TP. Previously exited at entry (zero profit).
* **Profit Floor setting** — new control in Settings → Risk. Adjustable 1–25%, default 5%.
* **Min Entry Price setting** — new control in Settings → Signals. Applies symmetrically to YES (price ≥ floor%) and NO (YES price ≤ 100−floor%) bets. Inline YES/NO info buttons. Default 14%.
* **`/log` redesigned** — tab selector with `✅ Resolved (N)` and `⏳ Pending (N)` buttons. Last 20 per tab. Trade numbers removed from all log lines.
* **`/accu` fixed** — was CSV-only, now reads from PostgreSQL first.
* **City volume in position detail fixed** — now sums all buckets from `event_cache` like `/vol`. Previously showed same value as bucket.
* **TRADES on PnL card** — resolved count only, not including pending.
* **6W / 7L on PnL card** — wins green, losses red.
* **v2.4.4 version string** — color raised to `#666666` for readability.
* **`/pnl` compact UPnL** — single summary line `⏳ 13 open · UPnL +$8.15` + Positions inline button.
* **`÷2` in `/version`** — division symbol replaces `/` to prevent Telegram command registration.
