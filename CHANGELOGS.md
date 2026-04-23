# Changelog

## v3.0 - Manual Live Workflow, Bot-Managed Exits and Operator Cleanup (Apr 21-23, 2026)

### Live workflow and operator controls
- Expanded the bot into a fuller `Paper` / `Shadow` / `Live` workflow with manual live review, tracked live rows, tracked live positions, archived rows, and richer `/orders` / `/pos` live-book handling.
- Added `/livetest` as a live pre-check for deployment geo, wallet readiness, and queue state.
- Added `/poly` with a one-tap Polymarket weather-market link from Telegram.
- Added live/manual-entry risk controls in `/settings`, including a manual-entry toggle and flat-size workflow for forward testing.
- Removed the old single-open-row canary bottleneck so multiple manual live intents can be reviewed without forced one-at-a-time blocking.

### Wallet, readiness and proxy mode
- Added stronger `/wallet` and `/liveaudit` views for live readiness, exposure, queues, attention-needed rows, and recent realized live closes.
- Added proxy-wallet aware readiness so live review can pass without native gas when the account is using the proxy/funder setup.
- Improved live readiness wording, geo-block messaging, and operator-facing status notes across `/wallet`, `/livetest`, and live review cards.

### Bot-managed exits and live execution
- Added bot-managed TP/SL arming for live positions instead of relying on visible resting exit orders at the exchange.
- Added deep-value live exit behavior with a higher TP target than the standard profile.
- Added explicit live exit-plan rendering on position cards:
  profile, TP, SL mode, logic, and execution style.
- Added safer live exit actions with `Close Pos` confirmation instead of one-tap flattening.

### Partial fills, reconciliation and remainders
- Added closer-to-exchange live reconciliation for open live rows and tracked positions.
- Added partial-fill handling so live rows can show filled shares plus a resting entry remainder.
- Added auto-cancel for small entry remainders near resolution while keeping the filled position alive.
- Improved live share/basis updates so `/pos` and live cards stay closer to exchange truth during partial fills and fills that continue resting on the book.

### Telegram UI cleanup
- Simplified `/start`, `/home`, `/wallet`, `/livetest`, `/liveaudit`, and live review cards to remove extra wording and repetitive readiness noise.
- Added UPnL to `/home` and `/pos`.
- Added city names directly to individual live position headers.
- Changed live position cards to show probability as cents-style entry vs now, instead of only percentages.
- Unified `Sync Live` and `Refresh` into a single refresh action on live position and review cards.
- Moved archived cancelled rows out of the main `/orders` queue into their own archived view.

### Logging and docs
- Visible cards, dashboards, and startup strings were aligned to `v3.0`.
- README and changelog documentation were updated under `v3.0` without bumping to `v3.1`.

---

## v2.9 - Live Audit, Directional Entry Repair and Richer Execution Truth (Apr 20-21, 2026)

### Paper book correctness
- Fixed paper `NO` entries so the stored buy price reflects the actual `NO` side instead of the mirrored `YES` price.
- Added normalization for older open paper `NO` rows and recalculated their TP targets from the corrected directional entry.
- `/pos` now renders tracked price flow as `YES x% → y%` or `NO x% → y%` so the displayed side matches the actual trade.

### Live execution truth
- Live execution rows now retain richer fill metadata:
  entry fill counts, fill timestamps, cost basis, closed proceeds, realized PnL, and realized return.
- Live review, live summary, and recent closed views now use those richer metrics instead of thinner placeholder values.
- Flatten and reconcile paths now persist closer-to-exchange truth for tracked live positions and closes.

### Operator tooling
- Added `/liveaudit` as a compact operator report for live readiness, queue state, attention-needed rows, open exposure, and recent realized closes.
- `/wallet`, `/orders`, `/pnl`, and live detail views now surface live readiness and execution state more clearly.
- Visible strings, cards, and startup logs were aligned to `v2.9`.

---

## v2.8 - Mode-Aware Dashboard, Unified Controls and Execution Journal (Apr 19-20, 2026)

### Modes and dashboard
- Reworked the bot around `Paper`, `Shadow`, and `Live` execution modes.
- `/start` now opens a mode chooser, and `/home` / `/pnl` render mode-aware dashboard cards.
- Restored the exact slogan copy:
  `Scan the heat, trade the peak.`

### Shadow and live workflow
- Added a shadow/live execution journal with review rows, queue states, tracked live orders, and tracked live positions.
- Added `/wallet` for live readiness, allowance, balance, and signer checks.
- Added `/orders` for the shadow/live queue and detailed execution review screens.
- Fixed callback parsing for refs like `id:3`, which stopped shadow-to-live review actions from breaking on colon-containing refs.

### Unified control behavior
- Unified back-button routing across inline flows with contextual back targets.
- Fixed runtime-state normalization so `active_mode=live` is preserved instead of being silently downgraded back to paper.
- Made `/abort` mode-aware:
  paper resolves local positions, shadow cancels prepared rows, and live cancels tracked orders plus queues tracked positions for flatten review.
- Made `/resolveall` mode-aware:
  paper resolves at market, while shadow/live clear local rows and skip exchange-backed live items that still need explicit cancel or flatten handling.

## v2.7.3 - UI Cleanup, Regional Source Expansion, Calibration Sync and Debug Tooling (Apr 17-18, 2026)

### Cards and UI
- City detail cards were cleaned up into a compact station-first format.
- City cards now start from the real station line and remove the old city header, UTC line, and other clutter.
- Position detail cards were simplified to the trading mechanics only:
  direction or bucket, market price, trade, volume, resolves, and logged time.
- `/status` now supports an optional top image panel using `thermal-map-50.png`.
- `/start` now uses the slimmer home screen:
  `Signals`, `Volume`, `PnL`, `Cities`, `Export`, `Sync`, `Pause`, `Help`, `Settings`, and `Abort`.
- `/start` keeps emoji on inline buttons, while Telegram slash-command menu descriptions are plain text only.
- `/start` no longer shows `Status`, `Log`, or `Positions` inline.

### Commands and versioning
- `/random` was removed.
- `/brief` was removed.
- `/drift` was removed, including the old auto-sent drift alerts.
- `/help` no longer shows `/random`, `/brief`, or `/drift`.
- `/sync` was added as a manual forecast-history refresh command.
- Visible version strings were aligned to `v2.7.3`.
- `/version` now uses the correct `Thermal✹Peak` mark.
- `/debug city` was added as a private source-diagnostics view.

### Forecast sources
- `HKO` added as the regional source for Hong Kong.
- `SMG` added for Guangzhou and Shenzhen as a soft regional source.
- `CWA` parsing and fallback handling for Taipei were fixed.
- Moscow METAR/TAF fallback was restored through AviationWeather, preventing the old `N/A` regression.

### Signal flow and diagnostics
- `/signals` on-demand refresh now pulls the fuller forecast stack:
  markets, GFS, regional sources, multimodel forecasts, and TAF.
- `/signals` formatting now reflects the real stack used by the bot:
  raw GFS, seed forecast, regional source, TAF max, final blended forecast, expected error, and source stack.
- `/debug` now shows:
  trade status, action hint, seed forecast, final forecast, expected error, consensus, and cache ages.

### Data and exports
- Added a dedicated `forecast_history` dataset for calibration snapshots.
- The bot now logs one forecast-history row per `city + temp_date`.
- Trade-to-history marking now safely upserts if the history row was not present yet at trade time.
- Added repair passes for older missed history trade marks.
- Forecast-history resolution sync now runs against all rows, not just traded rows.
- Added resolution-state fields:
  `resolution_status`, `resolved_bucket`, `resolved_bucket_source`, and `resolved_at`.
- Added calibration fields:
  `actual_temp_c`, `actual_source`, `actual_temp_quality`,
  `actual_temp_bucket_derived_c`, `actual_temp_bucket_method`,
  `calibration_eligible`, `calibration_note`,
  `error_final_c`, `error_gfs_c`, `error_regional_c`, and `error_taf_c`.
- Winning-bucket actuals are now labeled by quality:
  `bucket_exact`, `bucket_derived_bounded`, or `bucket_derived_open`.
- `/sync` forces a manual forecast-history refresh pass before export when needed.
- `/export` now sends two CSV exports when available:
  `thermal_peak_trades.csv` and `thermal_peak_history.csv`.
- CSV fallback support was added for forecast history alongside PostgreSQL support.

---

## v2.7.2 - New Cities, Resolution Notes and Spread Bug Fix (Apr 15, 2026)

### City expansion
- Coverage increased from 47 to 50 cities.
- Added:
  `Guangzhou (ZGGG)`, `Karachi (OPKC)`, and `Manila (RPLL)`.
- `NEW_CITIES` was updated so the `/cities` page correctly highlights new additions.

### Resolution notes
- Added `CITY_RESOLUTION_NOTE` for non-obvious stations.
- `/cities` detail cards began showing clearer resolution-station context.

### Fixes
- Fixed the spread-check city label bug that logged `?` instead of the real city.
- Added normalize aliases for Guangzhou, Karachi, and Manila variants.

---

## v2.7.1 - Relative Min-Prob Fix, Wider Consensus Gate and Trade Reset Fix (Apr 13, 2026)

### Probability gate fix
- Replaced the broken absolute min-probability threshold with a relative threshold based on each bucket's maximum possible probability.
- Fixed the regional veto to use a relative test instead of an absolute one that was incorrectly blocking narrow buckets.

### Consensus gate
- Raised the consensus gate from `2.5°C` to `3.5°C` to avoid overblocking during spring-transition spread.

### Logging
- Fixed `/resetlog` so PostgreSQL trade data is cleared along with CSV data.
- Reset the trade dataset after recalibration.

---

## v2.7 - Station Audit, Per-Model Gate and 47-City Coverage (Apr 12, 2026)

### Signal architecture
- Added the per-model minimum-probability gate.
- Added stronger regional forecast integration into the gate logic.
- Integrated `CWA` for Taipei.

### Station corrections
- Denver corrected from `KDEN` to `KBKF`.
- Moscow duplicate station entry removed, keeping `UUWW`.
- Taipei updated from `RCTP` to `RCSS` for newer markets.

### City expansion
- Added:
  `Cape Town`, `Jeddah`, and `Lagos`.

### UI
- Improved `/cities`, `/vol`, and bucket formatting.
- Added `NEW_CITIES` support.

---

## v2.6.2 - Emergency Abort, Daily Summary Fix and UI Fixes (Apr 9, 2026)

- Added `/abort` as a two-step emergency stop and resolve-all flow.
- Added the abort button to `/start`.
- Fixed `/summ` to scope correctly to the current UTC day.
- Hardened market discovery for active markets.
- Fixed the `× Close` button handler.
- Fixed the `/vol` Celsius-strip regex.
- Standardized the GFS schedule display in UTC.
- Fixed `/cities` fallback so it showed the full city set instead of only a partial list.

---

## v2.6.1 - Full Resolution Station Audit (Apr 3, 2026)

- Moscow corrected from `UUEE` to `UUWW`.
- Houston corrected from `KIAH` to `KHOU`.
- Taipei coordinates updated.
- Jakarta and Panama City station mappings verified.

---

## v2.6 - 44-City Expansion and Station Audit (Apr 3, 2026)

- Added:
  `Busan`, `Panama City`, `Amsterdam`, `Kuala Lumpur`, `Helsinki`, and `Jakarta`.
- Extended model weighting and slug-fetch fallback coverage.

---

## v2.5 - Multi-Model Consensus and Dead-Zone Pause (Apr 2, 2026)

- Added the 5-model consensus gate:
  `GFS`, `ECMWF`, `ICON`, `UKMET`, `GEM`.
- Added ICON boost behavior for Europe.
- Added the 22:00-02:00 UTC dead-zone pause.
- Added `/random` at the time; later removed in `v2.7.3`.

---

## v2.4.4.1 - Signal Quality and City Risk Management (Apr 1, 2026)

- Raised edge threshold to `10%`.
- Added per-city sigma calibration.
- Added the city exclusion system and `/exclude`.

---

## v2.4.4 - UI Polish and City Expansion (Mar 29-30, 2026)

- Improved `bucket_with_c()`.
- Added pagination to `/vol` and `/cities`.

---

## v2.4.3 - Backtest Notebook v1.0 (Mar 28-29, 2026)

- Added the backtest notebook for parameter sweeps against resolved trades.

---

## v2.4.2 - Position Detail and UPnL (Mar 28, 2026)

- Added individual position detail cards.
- Improved UPnL handling.
- Added position actions and pagination.

---

## v2.4.1 - Profit Floor and Reliability (Mar 27, 2026)

- Added profit-floor protection.
- Fixed TP cap and SL bypass behavior.
- Added `/pos`.

---

## v2.4 - Signal Quality, ICAO Fixes and 38-City Coverage (Mar 24-27, 2026)

- Expanded to 38 cities.
- Corrected several major ICAO mappings.
- Added edge cap, entry floor, per-bucket volume filtering, spread filter, NO-bet guards, fair-value ceiling, and city deduplication.

---

## v2.3.1 - Risk Engine, UI Polish and 35-City Coverage (Mar 23, 2026)

- Expanded to 35 cities.
- Added limit-order TP persistence.
- Added SL minimum-floor behavior.
- Added duplicate-signal suppression.
- Added reset-to-default in settings.

---

## v2.3 - Settings, Risk, Cards and 30-City Coverage (Mar 21-22, 2026)

- Rebuilt the bot around inline-keyboard navigation.
- Overhauled `/settings`.
- Added the risk-management suite.
- Added the shareable PnL card.
- Expanded to 30-city coverage.

---

## v2.2 - Stability (Mar 20, 2026)

- Refined data pipelines and request handling.
- Improved navigation and general reliability.

---

## v2.1 - Analytics and Maintenance (Mar 19, 2026)

- Added `/accu`, `/summ`, `/resolveall`, and `/resetlog`.

---

## v2.0 - Major Overhaul (Mar 18, 2026)

- Expanded city coverage from 4 to 30.
- Added the Telegram UI.
- Added automated trade management.
- Switched to live Polymarket CLOB pricing.
- Rebuilt the probability engine around `norm.cdf`.
- Added PostgreSQL support with CSV fallback.

---

## v1.0 - Initial Alpha (Mar 16, 2026)

- Four-city proof of concept.
- Basic GFS scanning.
- Text-only Telegram alerts.
- Manual CSV logging and manual trade resolution.
