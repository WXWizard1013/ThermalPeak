<div align="center">
  <img src="https://raw.githubusercontent.com/WXWizard1013/ThermalPeak/main/Pics/logo-white.png" alt="ThermalPeak Logo" width="500"/>

  **Scan the heat, trade the peak.**

---

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Telegram API](https://img.shields.io/badge/Telegram-Bot_API-0088cc.svg?logo=telegram)](https://core.telegram.org/bots/api)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-asyncpg-336791.svg?logo=postgresql)](https://www.postgresql.org/)
[![v2.9](https://img.shields.io/badge/version-v2.9-orange.svg)]()

</div>

ThermalPeak is a private Telegram bot for scanning Polymarket D+1 weather markets and running a mode-aware `Paper` / `Shadow` / `Live` workflow around the highest-edge temperature buckets. It compares live Polymarket CLOB prices against a weather stack built from GFS, multi-model consensus, regional forecast sources, and TAF confirmation, then manages signals, positions, execution queues, wallet readiness, exports, and diagnostics directly from Telegram.

## Overview

The bot focuses on markets like:

`Will the highest temperature in Hong Kong be 28°C or above tomorrow?`

For each active city/date, it:

1. Discovers the live D+1 market and its bucket prices from Polymarket.
2. Builds a forecast stack:
   GFS baseline, ECMWF, ICON, UKMET, GEM, plus regional guidance when available.
3. Maps the city to the exact Polymarket resolution station and applies station bias + city sigma.
4. Scores each bucket with `norm.cdf`.
5. Applies signal-quality gates, EV checks, Kelly sizing, and spread filters.
6. Logs the trade or execution intent plus a separate forecast-history snapshot for later calibration.

## Execution Modes

- `Paper`: fully local paper positions. `/pnl`, `/pos`, `/log`, and `/resolveall` operate on paper trades only.
- `Shadow`: stages execution intents and review rows without submitting orders to the exchange. Good for validating queue logic, sizing, and operator flows.
- `Live`: tracks live review, submission, resting orders, open positions, cancels, and flatten requests. Actual exchange submission requires wallet, signer, allowance, RPC, and `LIVE_TRADING_ENABLED=1`.

If wallet or live infra is not configured, live mode can still queue and review rows, but it will not actually submit or flatten on Polymarket.

## Forecast Stack

**Core models**
- `GFS` via Open-Meteo
- `ECMWF`
- `ICON`
- `UKMET`
- `GEM`

**Regional sources**
- `NOAA` for US cities
- `HKO` for Hong Kong
- `BMKG` for Jakarta
- `MSS` for Singapore
- `CWA` for Taipei
- `SMG` for Guangzhou and Shenzhen

**Short-range confirmation**
- `TAF` via AVWX with AviationWeather fallback
- `METAR` via AVWX with AviationWeather fallback

Notes:
- Regional sources are not all treated equally.
- `SMG` is a soft regional source: light blend, no hard veto.
- The others act like stronger national-agency guidance.

## Signal Logic

ThermalPeak does not fire on raw edge alone.

**Main gates**
- `Consensus gate`: the 5-model spread must stay within `3.5°C`.
- `Per-model min-prob gate`: each model must clear a relative probability threshold based on the bucket's maximum possible probability, not a flat absolute cutoff.
- `Regional veto`: strong regional sources can block trades when they clearly point away from the bucket.
- `EV gate`: expected value must be positive after fee drag.
- `Spread filter`: entry spread must be `<= 4¢`.

**Calibration inputs**
- Per-city `bias`
- Per-city `sigma`
- Exact resolution-station mapping
- TAF weighting that increases near resolution

**Default entry filters**
- Edge threshold: `10%`
- Minimum entry price: `14%`
- Minimum bucket volume: `$1,000`

## Telegram Commands

| Command | What it does |
| :--- | :--- |
| `/start` | Choose `Paper`, `Shadow`, or `Live`, and open the launch screen |
| `/home` | Open the dashboard for the current mode |
| `/signals` | Full refresh, then scans for current edge signals |
| `/vol` | D+1 bucket volume table by city |
| `/cities` | Browse active cities, forecasts, and bucket odds |
| `/pnl` | Mode-aware PnL or queue summary card |
| `/pos` | Current-mode book or open positions |
| `/log` | Trade history |
| `/export` | Downloads both trade CSV and forecast-history CSV |
| `/status` | Bot health, scan status, pricing/model summary |
| `/wallet` | Live wallet, allowance, gas, and signer readiness checks |
| `/orders` | Shadow and live execution journal |
| `/liveaudit` | Compact live truth report with attention-needed rows, exposure, and recent realized closes |
| `/settings` | Signal, sizing, and risk controls |
| `/sync` | Refresh forecast-history resolution and calibration data |
| `/debug city` | Source diagnostics for one city |
| `/exclude` | Manage excluded cities |
| `/accu` | City-level accuracy / win-loss view |
| `/summ` | Daily summary |
| `/version` | Version notes |
| `/help` | Compact command list |
| `/pause` / `/cont` | Pause or resume scanning |
| `/resolveall` | Resolve paper positions or clear the current shadow/live local book |
| `/resetlog` | Reset the trade log |
| `/abort` | Mode-aware emergency stop: resolve paper, cancel shadow rows, or cancel/flatten tracked live rows |

## Telegram UI

- `/start` opens the mode chooser and a compact launch panel.
- `/home` renders the active mode dashboard with contextual back routing across inline flows.
- The `/start` inline buttons keep emoji labels for quick scanning.
- The Telegram slash-command menu uses plain-text descriptions with no emoji.
- `Status`, `Log`, and `Positions` are intentionally not shown on the launch panel.
- Legacy `/brief` and `/drift` commands are removed from both the command menu and bot flows.

## Storage and Exports

ThermalPeak stores three core datasets:

**Trade log**
- Paper positions
- Entry price
- Fair value
- Kelly size
- PnL and outcome

**Execution journal**
- Shadow intents
- Live review and blocked rows
- Tracked live orders and live positions
- Fill counts, cost basis, proceeds, and realized close metrics
- CSV fallback file: `execution_orders.csv`

**Forecast history**
- One snapshot per `city + temp_date`
- GFS, consensus, regional, TAF, and final blended forecast
- Bias, sigma, model spread, model count, source stack, and `hours_left`
- Whether the city/date produced a trade
- Resolution-state tracking for all rows, not just traded rows:
  `resolution_status`, `resolved_bucket`, `resolved_bucket_source`, and `resolved_at`
- Calibration fields:
  `actual_temp_c`, `actual_source`, `actual_temp_quality`,
  `actual_temp_bucket_derived_c`, `actual_temp_bucket_method`,
  `calibration_eligible`, `calibration_note`, and model error fields

`/sync` forces a full forecast-history refresh pass so you can update proposed/final outcomes and calibration rows before exporting.

For calibration, the cleanest subset is:
- `resolution_status = final`
- `calibration_eligible = 1`
- `actual_temp_quality = bucket_exact`

`/export` sends:
- `thermal_peak_trades.csv`
- `thermal_peak_history.csv`

If `DATABASE_URL` is set, PostgreSQL is used.
If not, the bot falls back to local CSV files, including `forecast_history.csv` and `execution_orders.csv`.

## Required Environment Variables

**Required**
- `TELEGRAM_TOKEN`

**Recommended**
- `DATABASE_URL`
- `AVWX_TOKEN`

**Live trading and wallet checks**
- `POLYMARKET_WALLET_ADDRESS`
- `POLYMARKET_FUNDER_ADDRESS`
- `POLYMARKET_PRIVATE_KEY`
- `POLYMARKET_API_KEY`
- `POLYMARKET_API_SECRET`
- `POLYMARKET_API_PASSPHRASE`
- `POLYGON_RPC_URL`
- `POLYMARKET_USDC_ADDRESS`
- `POLYMARKET_SPENDER`
- `LIVE_TRADING_ENABLED`

**Optional**
- `TELEGRAM_CHAT_ID`
- `POLYMARKET_CHAIN`
- `POLYMARKET_CHAIN_ID`
- `POLYMARKET_CLOB_HOST`
- `POLYMARKET_SIGNATURE_TYPE`
- `LIVE_MIN_GAS_NATIVE`
- `LIVE_EXECUTOR_INTERVAL_SEC`
- `LIVE_EXIT_SLIPPAGE`
- `LIVE_WATCHDOG_INTERVAL_SEC`
- `CWA_API_TOKEN`
- `CWA_TOKEN`
- `CWA_AUTHORIZATION`
- `CWA_API_KEY`
- `SMG_XML_BASE_URL`

## Current City Coverage

ThermalPeak currently covers 50 active weather-market cities:

New York, London, Singapore, Shanghai, Buenos Aires, Miami, Chicago, Ankara, Los Angeles, Tokyo, Paris, Toronto, Moscow, Hong Kong, Seoul, Seattle, Atlanta, Dallas, Denver, Houston, Sao Paulo, Wellington, Madrid, Warsaw, Taipei, Milan, Tel Aviv, Lucknow, Munich, Jakarta, San Francisco, Austin, Chengdu, Chongqing, Beijing, Shenzhen, Wuhan, Istanbul, Mexico City, Busan, Panama City, Amsterdam, Kuala Lumpur, Helsinki, Cape Town, Jeddah, Lagos, Guangzhou, Karachi, and Manila.

## Notable v2.9 Additions

- Fixed paper `NO` entry logging so stored entry price matches the traded side instead of the mirrored `YES` price.
- Added legacy normalization for older open paper `NO` rows and recalculated TP values from the corrected entry.
- `/pos` now renders the tracked side directly as `YES x% → y%` or `NO x% → y%`.
- Live execution rows now retain richer fill, cost-basis, proceeds, and realized-close metrics.
- Added `/liveaudit` for compact live operator review: readiness, attention-needed rows, open exposure, and recent realized live closes.
- Visible bot strings, cards, and startup logs are aligned to `v2.9`.

## Status

This bot is private and whitelist-only.

It is built around calibration-first iteration, operator review, and staged execution rather than blind automation.

`Paper` and `Shadow` are suitable for day-to-day testing. `Live` is now exchange-aware, but true live submission still depends on wallet, signer, allowance, RPC, and runtime readiness being correctly configured.

---

*Not financial advice. Prediction markets are volatile, and any automated or semi-automated workflow carries real risk.*
