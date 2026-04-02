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
* **Shareable PnL card** — generates a styled PNG card with the thermal gradient branding, Net PnL, win rate, Sharpe, trade stats, and the thermal win rate bar.
* **30-city coverage** — fixed market discovery to correctly surface all 30 active cities including Chinese mainland markets (Chengdu, Chongqing, Beijing, Shenzhen, Wuhan).
* **Station bias corrections** — hardcoded micro-climate offsets for HKO Observatory (Hong Kong), CWA 46692 (Taipei city center), KLGA (LaGuardia), EGLC (London City), and ZSPD (Pudong coastal sea-breeze effect).
* **NOAA NWS blending** — human-edited US forecasts blended 55/45 with GFS for US cities.
* **Hours-ahead gate** — replaced strict UTC date matching with a 16–72h window to correctly handle Asian markets that resolve at noon UTC.

### v2.3.1 — Risk Engine, UI Polish & 35-City Coverage (Mar 23, 2026)
* **35-city coverage** — added San Francisco (KSFO) and Austin (KAUS), both confirmed against Polymarket resolution rules and added to NOAA coverage.
* **Limit-order TP** — take-profit price is now fixed at entry as an absolute level (`entry × (1 + tp_threshold)`) and stored per-trade in the database/CSV. Survives bot restarts.
* **SL min floor (Protection)** — configurable absolute ¢ floor that must be breached alongside the % stop-loss before SL fires. Default 5¢, set to 0 to disable.
* **Duplicate signal suppression** — auto-scan alerts only fire for genuinely new signals.
* **PnL card auto-generated on `/pnl`** — HD 1280×680 card sent automatically every time `/pnl` is called.
* **On-demand forecast refresh in `/signals`** — GFS, NOAA, and CLOB prices all refreshed live.
* **TP/SL setting changes trigger immediate position check**.
* **Settings → Reset to Default** — one-tap button restores all 11 settings to factory values with a confirmation summary.

### v2.4 — Signal Quality, ICAO Fixes & 38-City Coverage (Mar 24–27, 2026)
* **38-city coverage** — added Istanbul (LTFM, new IST airport), Moscow (UUEE Sheremetyevo), and Mexico City (MMMX Benito Juárez).
* **ICAO station corrections** — fixed three wrong station assignments: KJFK → KLGA (New York resolves at LaGuardia), EGLL → EGLC (London resolves at City Airport), ZSSS → ZSPD (Shanghai resolves at Pudong).
* **Edge cap lowered to 13%** — data showed signals with >15% edge had 0W/7L. High edge = GFS vs market disagreement = GFS is wrong, not the market.
* **Minimum entry floor at 14%** — trades below 14% market price had 17–27% win rate at −$20 PnL.
* **Per-bucket volume filter** — volume now checked on individual buckets, not the city total.
* **Spread liquidity filter** — signals only fire when the YES ask/bid spread is ≤ 4¢.
* **NO bet guards** — NO signals only allowed when YES price is in the 75–92% window.
* **Fair value bounds** — ceiling at 80% fair value prevents oversized Kelly on near-certain outcomes.
* **City deduplication** — one signal per city per scan, keeping the highest-edge bucket.

### v2.4.1 — Profit Floor & Reliability (Mar 27, 2026)
* **Profit floor protection** — positions up +30% that reverse now lock in at least +5% profit instead of risking a full SL.
* **TP capped at threshold** — fixed bug where TP was not capped at the configured threshold price, causing overpayment on wins.
* **SL bypass fixed** — fixed critical bug where the resolved-market branch bypassed SL entirely, causing full-stake losses instead of capped losses.
* **`/pos` command** — new command showing all open positions with live UPnL, entry price, current price, and resolution date.

### v2.4.2 — Position Detail & UPnL (Mar 28, 2026)
* **Individual position detail** — tap any position in `/pos` to see full detail: bid/ask, Kelly size, UPnL, volume, GFS forecast, NOAA, METAR, resolution date, and a direct Polymarket link.
* **UPnL calculation fixed** — unrealised PnL now uses mid-price consistently between the list view and detail view.
* **Resolve / Remove buttons** — each position detail has inline buttons to manually resolve or remove the position.
* **Positions pagination** — `/pos` paginates at 8 positions per page when more than 8 are open.

### v2.4.3 — Backtest Notebook v1.0 (Mar 28–29, 2026)
* **Backtest notebook** (`thermalpeak_backtest.ipynb`) — Google Colab notebook for historical parameter sweeps. Fetches GFS archive data (Open-Meteo) and Polymarket closed markets, builds a combined dataset, and sweeps entry floor × edge threshold combinations.

### v2.4.4 — UI Polish & City Expansion (Mar 29–30, 2026)
* **`bucket_with_c()` display** — inline °C conversion shown next to °F outcomes throughout the UI.
* **`/pos` list cleaned up** — removed °C conversion from the compact positions list.
* **`/vol` and `/cities` pagination** — both paginated at 20 per page with `Page 1 / 2` header and navigation buttons.
* **3 new cities** — Istanbul (LTFM), Moscow (UUEE), Mexico City (MMMX). Total active coverage: 38 cities.

### v2.4.4.1 — Signal Quality & City Risk Management (Apr 1, 2026)
* **Edge threshold raised to 10%** — based on 41-trade analysis. Trades in the 8–9% band had materially lower win rate and were diluting the signal pool.
* **Per-city σ calibration** — replaced the fixed 2.0°C GFS uncertainty with a tiered `CITY_SIGMA_C` lookup table:
  * **Tight σ (1.0–1.5°C):** Singapore, Lucknow, Taipei, Miami, Tokyo, Buenos Aires, Sao Paulo.
  * **Default σ (1.8–2.2°C):** New York, Chicago, London, Seoul, Paris, Shanghai, Ankara, Toronto, etc.
  * **Wide σ (2.5–3.0°C):** Los Angeles, San Francisco, Seattle, Denver, Dallas, Houston, Austin, Milan, Warsaw, Munich — orographic terrain, marine layers, and fast-moving frontal systems where GFS consistently underperforms.
* **City exclusion system** — new `/exclude` command and automated exclusion workflow:
  * After 10 SL losses on any city, bot sends an alert: *"10 losses in [City] — exclude city?"* with **Yes / No** inline buttons. Repeats every 10 losses.
  * `/exclude` command shows all excluded cities (tap to re-include) and a grid of active cities to manually add.
  * Exclusions are persisted to the `excluded_cities` PostgreSQL table and loaded on startup.

### v2.5 — Multi-Model Consensus, Bayesian Sizing & Trailing SL (Apr 1, 2026)
* **Multi-model consensus gate** — ECMWF (`ecmwf_ifs025`) and ICON (`icon_global`) fetched via Open-Meteo every 4h alongside GFS. Signals only fire when the spread between all available models is ≤ 1.5°C. ECMWF-dominant weighting globally; ICON-boosted for EU/MENA cities (Warsaw, Milan, Munich, Madrid, Ankara, Tel Aviv, Istanbul, Moscow).
* **Bayesian Kelly sizing** — per-city win rate estimated using a Beta(3,3) prior blended with observed trade history. Cities below 35% posterior WR get scaled-down Kelly. Bayesian multiplier shown on every signal card.
* **City win rates pre-loaded at startup** — `get_pnl_summary()` called during `scan_loop()` initialisation so `city_accuracy` is populated before the first scan.
* **Three-tier trailing stop-loss** — positions ratchet up their floor as they move in favour:
  * Tier 1: +15% gain → floor locks at +5%
  * Tier 2: +35% gain → floor locks at +20%
  * Fast-move: spread ≤ 5¢ and position up → existing `be_floor` fires first
* **Per-city max 2 open positions** — scanner skips a city already holding 2 PENDING positions.
* **City detail enhanced** — `/cities` now shows ECMWF + ICON for EU cities, NOAA for US cities, ECMWF for all others.
* **/random** — AI-generated weather fact on demand, powered by Claude Haiku. Tap 🔀 New Fact for a fresh one every time.

### v2.6 — 5-Model Ensemble, Ladder Sizing & Entry Timing (Apr 3, 2026)
* **5-model consensus gate** — UKMET (`ukmo_seamless`) and Canadian GEM (`gem_seamless`) added alongside GFS, ECMWF, and ICON. All five models fetched every 4h via Open-Meteo. Spread gate remains 1.5°C — now requires broader agreement before a signal fires. Weighted blend: ECMWF-dominant globally, ICON/UKMET-boosted for EU cities, GEM adds North American mesoscale correction.
* **2-bucket temperature ladder** — instead of one bucket per city, the scanner now opens the primary signal plus the best adjacent bucket when it clears 8% minimum edge. Kelly is split edge-weighted across both legs — total city exposure preserved. Based on the temperature laddering strategy used by top Polymarket weather traders (neobrother).
* **Entry timing gate (20–48h window)** — signals only fire when the market is 20–48 hours from resolution. Before 48h model variance is too high; after 20h prices are already efficient. Confirmed as the sweet spot by research into top Polymarket weather trader behaviour.
* **Dynamic Kelly scaling by model count** — Kelly scales with how many models are available and agreeing: 5 models → 100%, 4 → 80%, 3 → 60%, 2 → 40%, 1 → 30%. High-conviction 5-model signals get full size; thin-data signals get penalised automatically.
