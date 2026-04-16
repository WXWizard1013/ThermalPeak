<div align="center">
  <img src="https://raw.githubusercontent.com/WXWizard1013/ThermalPeak/main/Pics/logo-white.png" alt="ThermalPeak Logo" width="500"/>

  **Scan The Heat, Trade The Peak.**

---

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Telegram API](https://img.shields.io/badge/Telegram-Bot_API-0088cc.svg?logo=telegram)](https://core.telegram.org/bots/api)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-asyncpg-336791.svg?logo=postgresql)](https://www.postgresql.org/)
[![v2.7.3](https://img.shields.io/badge/version-v2.7.3-orange.svg)]()

</div>

ThermalPeak is a private Telegram bot for scanning Polymarket D+1 weather markets and paper-trading the highest-edge temperature buckets. It compares live Polymarket CLOB prices against a weather stack built from GFS, multi-model consensus, regional forecast sources, and TAF confirmation, then manages signals, positions, exports, and diagnostics directly from Telegram.

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
6. Logs the trade and a separate forecast-history snapshot for later calibration.

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
| `/signals` | Full refresh, then scans for current edge signals |
| `/vol` | D+1 bucket volume table by city |
| `/brief` | GFS vs market-favorite disagreement view |
| `/cities` | Browse active cities, forecasts, and bucket odds |
| `/pnl` | Paper-trade summary card + unrealized PnL |
| `/pos` | Open positions |
| `/log` | Trade history |
| `/export` | Downloads both trade CSV and forecast-history CSV |
| `/status` | Bot health, scan status, pricing/model summary |
| `/settings` | Signal, sizing, and risk controls |
| `/debug city` | Source diagnostics for one city |
| `/exclude` | Manage excluded cities |
| `/drift` | GFS drift check |
| `/accu` | City-level accuracy / win-loss view |
| `/summ` | Daily summary |
| `/version` | Version notes |
| `/help` | Compact command list |
| `/pause` / `/cont` | Pause or resume scanning |
| `/resolveall` | Force-check all pending positions |
| `/resetlog` | Reset the trade log |
| `/abort` | Emergency stop and resolve-all flow |

## Storage and Exports

ThermalPeak stores two different datasets:

**Trade log**
- Opened positions
- Entry price
- Fair value
- Kelly size
- PnL and outcome

**Forecast history**
- One snapshot per `city + temp_date`
- GFS, consensus, regional, TAF, and final blended forecast
- Bias, sigma, model spread, model count, source stack
- Whether the city/date produced a trade

`/export` sends:
- `thermal_peak_trades.csv`
- `thermal_peak_history.csv`

If `DATABASE_URL` is set, PostgreSQL is used.
If not, the bot falls back to local CSV files.

## Required Environment Variables

**Required**
- `TELEGRAM_TOKEN`

**Recommended**
- `DATABASE_URL`
- `AVWX_TOKEN`

**Optional**
- `TELEGRAM_CHAT_ID`
- `CWA_API_TOKEN`
- `CWA_TOKEN`
- `CWA_AUTHORIZATION`
- `CWA_API_KEY`
- `SMG_XML_BASE_URL`

## Current City Coverage

ThermalPeak currently covers 50 active weather-market cities:

New York, London, Singapore, Shanghai, Buenos Aires, Miami, Chicago, Ankara, Los Angeles, Tokyo, Paris, Toronto, Moscow, Hong Kong, Seoul, Seattle, Atlanta, Dallas, Denver, Houston, Sao Paulo, Wellington, Madrid, Warsaw, Taipei, Milan, Tel Aviv, Lucknow, Munich, Jakarta, San Francisco, Austin, Chengdu, Chongqing, Beijing, Shenzhen, Wuhan, Istanbul, Mexico City, Busan, Panama City, Amsterdam, Kuala Lumpur, Helsinki, Cape Town, Jeddah, Lagos, Guangzhou, Karachi, and Manila.

## Notable v2.7.3 Additions

- Cleaned city cards and position detail cards
- `HKO` regional forecast for Hong Kong
- `SMG` regional forecast for Guangzhou and Shenzhen
- `CWA` handling fixed for Taipei
- `/debug city` diagnostics
- Dual-file `/export`
- Forecast-history logging for calibration
- `/signals` display refreshed to show GFS, seed, regional, TAF, final forecast, and expected error

## Status

This bot is private and whitelist-only.

It is built around paper trading, diagnostics, and calibration-first iteration rather than blind automation.

---

*Not financial advice. Prediction markets are volatile, and any automated or semi-automated workflow carries real risk.*
