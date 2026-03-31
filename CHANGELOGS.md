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
* **30-city coverage** — fixed market discovery to correctly surface all 30 active cities including Chinese mainland markets (Chengdu, Chongqing, Beijing, Shenzhen, Wuhan). Root cause: Polymarket uses a different question format that required a separate parser and direct slug fetch.
* **Station bias corrections** — hardcoded micro-climate offsets for HKO Observatory (Hong Kong), CWA 46692 (Taipei city center), KLGA (LaGuardia), EGLC (London City), and ZSPD (Pudong coastal sea-breeze effect).
* **NOAA NWS blending** — human-edited US forecasts blended 55/45 with GFS for US cities.
* **Hours-ahead gate** — replaced strict UTC date matching with a 16–72h window to correctly handle Asian markets that resolve at noon UTC.

### v2.3.1 — Risk Engine, UI Polish & 35-City Coverage (Mar 23, 2026)
* **35-city coverage** — added San Francisco (KSFO) and Austin (KAUS), both confirmed against Polymarket resolution rules and added to NOAA coverage.
* **Limit-order TP** — take-profit price is now fixed at entry as an absolute level (`entry × (1 + tp_threshold)`) and stored per-trade in the database/CSV. Survives bot restarts. When the TP threshold is changed in settings, all open positions are immediately recalculated and checked against the new level.
* **SL min floor (Protection)** — new settings category. Configurable absolute ¢ floor that must be breached alongside the % stop-loss before SL fires. Protects cheap entries from being stopped out by normal market noise. Default 5¢, set to 0 to disable.
* **Duplicate signal suppression** — auto-scan alerts now only fire for genuinely new signals. If the same edge persists across scans with no new positions opened, the loop stays silent.
* **PnL card auto-generated on `/pnl`** — removed the separate `/card` command. The HD 1280×680 card is now sent automatically every time `/pnl` is called.
* **On-demand forecast refresh in `/signals`** — GFS, NOAA, and CLOB prices are all refreshed live when `/signals` is called manually.
* **TP/SL setting changes trigger immediate position check** — changing any risk threshold now runs `check_resolved_markets()` instantly with a retry.
* **Settings → Reset to Default** — one-tap button restores all 11 settings to factory values with a confirmation summary.

### v2.4 — Signal Quality, ICAO Fixes & 38-City Coverage (Mar 24–27, 2026)
* **38-city coverage** — added Istanbul (LTFM, new IST airport), Moscow (UUEE Sheremetyevo), and Mexico City (MMMX Benito Juárez).
* **ICAO station corrections** — fixed three wrong station assignments that were biasing forecasts: KJFK → KLGA (New York resolves at LaGuardia), EGLL → EGLC (London resolves at City Airport), ZSSS → ZSPD (Shanghai resolves at Pudong).
* **Edge cap lowered to 13%** — data showed signals with >15% edge had 0W/7L. High edge = GFS vs market disagreement = GFS is wrong, not the market. Hard cap prevents taking these.
* **Minimum entry floor at 14%** — trades below 14% market price had 17–27% win rate at −$20 PnL. Floor enforced before any signal fires.
* **Per-bucket volume filter** — volume now checked on individual buckets, not the city total. Prevents false positives from cities with high aggregate volume but no liquidity on the specific bucket being traded.
* **Spread liquidity filter** — signals are only fired when the YES ask/bid spread is ≤ 4¢. Fetches CLOB ask and bid for every qualifying signal before logging.
* **NO bet guards** — NO signals only allowed when YES price is in the 75–92% window. Prevents false NO signals at the tails where GFS noise is worst.
* **Fair value bounds** — added ceiling at 80% fair value. Near-certainty signals produce oversized Kelly bets (Lucknow 32°C or below at 92% fair = $41 bet) — capped.
* **City deduplication** — one signal per city per scan, keeping the highest-edge bucket. Prevents duplicate alerts when multiple buckets for the same city all clear the threshold.

### v2.4.1 — Profit Floor & Reliability (Mar 27, 2026)
* **Profit floor protection** — new protection setting. When a position has been open for at least `min_open_hours` and the current price is below the entry (in the red), the position is automatically closed at a small gain floor (`be_floor`) rather than waiting for a full SL. Prevents profitable positions from reversing into losses.
* **TP capped at threshold** — fixed bug where TP was not capped at the configured threshold price, causing overpayment on wins.
* **SL bypass fixed** — fixed critical bug where the resolved-market branch bypassed SL entirely, causing full-stake losses instead of capped losses.
* **`/pos` command** — new command showing all open positions with live UPnL, entry price, current price, and resolution date.

### v2.4.2 — Position Detail & UPnL (Mar 28, 2026)
* **Individual position detail** — tap any position in `/pos` to see full detail: market price, bid/ask, Kelly size, UPnL, volume, GFS forecast, NOAA, METAR, resolution date, and a direct Polymarket link.
* **UPnL calculation fixed** — unrealised PnL now uses mid-price consistently between the list view and detail view.
* **Resolve / Remove buttons** — each position detail has inline buttons to manually resolve or remove the position.
* **Positions pagination** — `/pos` paginates at 8 positions per page when more than 8 are open.

### v2.4.3 — Backtest Notebook v1.0 (Mar 28–29, 2026)
* **Backtest notebook** (`thermalpeak_backtest.ipynb`) — Google Colab notebook for historical parameter sweeps. Fetches GFS archive data (Open-Meteo) and Polymarket closed markets, builds a combined dataset, and sweeps entry floor × edge threshold combinations. Validates against real resolved trades from `thermal_peak_trades.csv`. Fixed Polymarket fetch to use `/events` endpoint (individual `/markets` endpoint slug matching was unreliable).

### v2.4.4 — UI Polish & City Expansion (Mar 29–30, 2026)
* **`bucket_with_c()` display** — inline °C conversion shown next to °F outcomes throughout the UI (e.g. `78-79°F (25.6-26.1°C)`).
* **`/pos` list cleaned up** — removed °C conversion from the compact positions list; raw outcome string only. °C still shown in individual position detail.
* **`/vol` pagination** — volume list now paginates at 20 cities per page with `Page 1 / 2` header and `Next Page →` / `← Prev Page` navigation buttons.
* **`/cities` pagination** — city browser also paginated at 20 per page with the same structure.
* **3 new cities** — Istanbul (LTFM), Moscow (UUEE), Mexico City (MMMX) added to `KNOWN_CITIES`. Total active coverage: 38 cities.

### v2.4.4.1 — Signal Quality & City Risk Management (Apr 1, 2026)
* **Edge threshold raised to 10%** — default edge threshold increased from 8% to 10% based on 41-trade analysis. Trades in the 8–9% band had materially lower win rate and were diluting the signal pool.
* **Per-city σ calibration** — replaced the fixed 2.0°C GFS uncertainty with a tiered `CITY_SIGMA_C` lookup table:
  * **Tight σ (1.0–1.5°C):** Singapore, Lucknow, Taipei, Miami, Tokyo, Buenos Aires, Sao Paulo — stable maritime and tropical climates where GFS is reliable. Tighter σ = sharper fair values = only genuinely mispriced buckets generate signals.
  * **Default σ (1.8–2.2°C):** New York, Chicago, London, Seoul, Paris, Shanghai, Ankara, Toronto, etc.
  * **Wide σ (2.5–3.0°C):** Los Angeles, San Francisco, Seattle, Denver, Dallas, Houston, Austin, Milan, Warsaw, Munich — orographic terrain, marine layers, and fast-moving frontal systems where GFS consistently underperforms. Wider σ = harder to clear 10% edge = systematically biased cities filter themselves out without hard exclusion.
* **City exclusion system** — new `/exclude` command and automated exclusion workflow:
  * After 10 SL losses on any city, bot sends an alert: *"10 losses in [City] — exclude city?"* with **Yes / No** inline buttons. Repeats every 10 losses.
  * `/exclude` command shows all excluded cities (tap to re-include) and a grid of active cities to manually add.
  * Exclusions are persisted to the `excluded_cities` PostgreSQL table and loaded on startup. Excluded cities are skipped entirely during signal scanning.
