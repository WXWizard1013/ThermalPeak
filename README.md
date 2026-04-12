<div align="center">
  <img src="https://raw.githubusercontent.com/WXWizard1013/ThermalPeak/main/Pics/logo-white.png" alt="ThermalPeak Logo" width="500"/>

  **Scan The Heat, Trade The Peak.**

---

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Telegram API](https://img.shields.io/badge/Telegram-Bot_API-0088cc.svg?logo=telegram)](https://core.telegram.org/bots/api)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-asyncpg-336791.svg?logo=postgresql)](https://www.postgresql.org/)
[![v2.7](https://img.shields.io/badge/version-v2.7-orange.svg)]()

</div>

ThermalPeak is a Telegram-based trading bot that finds edges in Polymarket's daily temperature markets. It pulls live order book prices from Polymarket's CLOB API, runs them against bias-corrected weather forecasts from GFS, NOAA, and AVWX, and fires a signal when the math says the market is wrong. Everything — signals, positions, risk settings, city exclusions — is managed from your Telegram chat.

---

## How It Works

Each day Polymarket lists markets like *"Will the highest temperature in Tokyo exceed 19°C on April 1?"*. ThermalPeak discovers every active D+1 market across 47 cities, prices each bucket using a normal distribution over GFS model output (with multi-model consensus from ECMWF, ICON, UKMET, and GEM), and compares that fair value against the live market price. When the edge clears the threshold and Kelly sizing produces a bet above the minimum, it fires a signal.

The signal must pass two gates before firing:

1. **Consensus gate** — all five models (GFS, ECMWF, ICON, UKMET, GEM) must agree within 2.5°C. If model spread exceeds this, the city is skipped regardless of edge.
2. **Per-model min-prob gate** — each model independently computes `P(temp lands in bucket)` using norm.cdf. The signal only fires if every model gives that bucket ≥ 55% probability. A single dissenting model blocks the trade. Regional wx (NOAA/BMKG/MSS/CWA) is also fed into this gate as a hard veto.

The model accounts for:
- **Per-city σ calibration** — GFS uncertainty varies by climate type. Stable tropical cities (Singapore, Lagos) use 1.0–1.2°C σ. Orographic/coastal cities (LA, Seattle, Denver) use 2.8–3.0°C σ, making it much harder to fire signals where GFS systematically underperforms.
- **Station bias** — each city is mapped to its exact Polymarket resolution station (e.g. KLGA not JFK, KBKF not KDEN, RCSS not RCTP). Station-specific temperature offsets vs GFS grid are hardcoded per city.
- **Fee drag** — Polymarket charges ~2% per trade. Kelly sizing and EV calculations factor this in.
- **Spread filter** — signals only fire when the YES ask/bid spread is ≤ 4¢.

---

## Features

**Market Discovery**
- Discovers all active D+1 temperature markets across 47 cities at startup and on a configurable scan interval
- Slug fetch covers all 47 cities as fallback when Polymarket's tag API returns no results
- Fetches live prices directly from Polymarket's CLOB order book (not cached Gamma data)
- On-demand full refresh (GFS + NOAA + CLOB) triggered by `/signals`

**Forecasting**
- GFS via Open-Meteo — baseline 48h forecast for all cities
- ECMWF, ICON, UKMET, GEM — four additional models run in parallel
- NOAA NWS — human-edited corrections for US cities; veto model in per-model gate
- BMKG — Indonesian meteorological service for Jakarta; veto model
- MSS (NEA) — Meteorological Service Singapore for Singapore; veto model
- CWA — Taiwan Central Weather Administration for Taipei; veto model
- AVWX METAR/TAF — aviation weather for short-range confirmation near resolution
- ICON boost for European cities — ICON weighted higher over European terrain
- Per-city σ calibration — tiered GFS uncertainty by climate type and terrain complexity
- norm.cdf scoring — fair probability calculated over the full bucket range

**Signal Quality Filters**
- Consensus gate — all models must agree within 2.5°C spread
- Per-model min-prob gate — every model must assign ≥55% to the bucket; one dissenter blocks
- Edge threshold: 10% minimum; edge cap at 13%
- Minimum entry floor at 14%
- Spread liquidity filter — ≤ 4¢ ask/bid spread required
- One signal per city per scan (highest-edge bucket only)
- City exclusion system — cities with 10+ SL losses trigger an exclusion prompt

**Take Profit & Stop Loss**
- TP price fixed at entry as an absolute level, stored per-trade — survives bot restarts
- Three-tier trailing stop: +15% gain locks floor at +5%, +35% locks at +20%
- Profit floor protection — prevents profitable positions reversing into full SL losses
- High-confidence signals use a separate higher TP tier

**Risk Management**
- Kelly Criterion sizing with configurable fraction and max bet
- Bayesian Kelly with Beta(3,3) prior and 35% WR floor
- Stop-loss: poll-based % drop check every 30 minutes
- Daily drawdown limit — scanning pauses automatically if equity limit is breached
- `/abort` emergency kill-switch — stops scanning, resolves all open positions, blocks new trades

**Portfolio**
- Paper trading with full trade log (CSV or PostgreSQL)
- Live unrealised PnL on open positions via CLOB refresh
- Auto-resolve: checks pending positions against live prices every 30 minutes
- Sharpe ratio calculated across resolved trades

---

## Commands

| Command | What it does |
| :--- | :--- |
| `/signals` | Full refresh then scan all 47 cities for edges right now |
| `/vol` | Live volume table sorted by liquidity (paginated, 18 per page) |
| `/brief` | Cities where GFS disagrees with the market favourite by 3°C+ |
| `/cities` | Browse all 47 active D+1 markets, odds, METAR, and GFS forecast |
| `/pnl` | PnL card as image + unrealised PnL on open positions |
| `/pos` | Open positions with live prices and UPnL |
| `/log` | Recent trade history |
| `/export` | Download full trade log as CSV |
| `/exclude` | View and manage excluded cities |
| `/settings` | Adjust all signal, risk, and protection parameters |
| `/status` | Bot health, last scan time, active markets, current settings |
| `/drift` | Check if GFS forecasts have shifted since last run |
| `/accu` | Win/loss breakdown by city |
| `/summ` | Today's summary — signals fired, win rate, net PnL (00:00–23:59 UTC) |
| `/resolveall` | Force-check all pending positions against live prices |
| `/abort` | 🚨 Emergency stop — halt scanning, resolve all positions, block new trades |
| `/resetlog` | Wipe the trade log and start fresh |
| `/pause` / `/cont` | Pause and resume the scan loop |
| `/version` | Changelog and build info |
| `/help` | Full command list |

---

## Settings Reference

All adjustable via `/settings` in Telegram.

**Signals**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Volume filter | $1,000 | Minimum bucket volume to include in scans |
| Edge threshold | 10% | Minimum edge (fair − market) to fire a signal |
| Min entry price | 14% | Minimum market price to enter |
| Scan interval | 1h | How often CLOB prices are refreshed |

**Sizing**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Kelly fraction | 15% | Fraction of Kelly formula applied to bet sizing |
| Max bet | $100 | Hard cap per position |

**Risk**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Take profit | 60% | Close when up this % from entry |
| TP high conf | 80% | TP threshold for high-confidence signals |
| Stop loss | 30% | Close when down this % from entry |
| High conf stars | 8★ | Star rating that triggers the higher TP |
| Slippage tolerance | 2% | Deducted from fair value before edge calc |
| Max daily drawdown | 10% | Pause signals if today's losses hit this % of equity |

**Protection**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| SL min floor | $5 | Minimum absolute $ drop before SL fires. Both % threshold AND floor must be breached. Set to 0 to disable. |

---

## City Coverage

47 cities across all major timezones, each mapped to the exact ICAO station Polymarket uses for resolution:

New York (KLGA), London (EGLC), Tokyo (RJTT), Shanghai (ZSPD), Singapore (WSSS), Paris (LFPG), Seoul (RKSI), Hong Kong (HKO†), Toronto (CYYZ), Warsaw (EPWA), Tel Aviv (LLBG), Lucknow (VILK), Madrid (LEMD), Sao Paulo (SBGR), Buenos Aires (SAEZ), Miami (KMIA), Chicago (KORD), Dallas (KDFW), Atlanta (KATL), Seattle (KSEA), Ankara (LTAC), Munich (EDDM), Milan (LIMC), Wellington (NZWN), Taipei (RCSS‡), Beijing (ZBAA), Shenzhen (ZGSZ), Wuhan (ZHHH), Chongqing (ZUCK), Chengdu (ZUUU), San Francisco (KSFO), Austin (KAUS), Denver (KBKF‡), Houston (KHOU), Los Angeles (KLAX), Istanbul (LTFM), Moscow (UUWW‡), Mexico City (MMMX), Busan (RKPK), Panama City (MPMG), Amsterdam (EHAM), Kuala Lumpur (WMKK), Helsinki (EFHK), Jakarta (WIHH), Cape Town (FACT), Jeddah (OEJN), Lagos (DNMM)

† Hong Kong resolves via HKO Observatory directly, not Wunderground.  
‡ Corrected in v2.7 — previous versions used wrong station.

---

## Performance (Paper Trading)

As of Apr 12, 2026 — data collection phase, targeting 200 resolved trades before live capital deployment.

| Metric | Value |
| :--- | :--- |
| Resolved trades | ~107 |
| Win rate | ~43% |
| Net PnL | +$87.51 |
| Avg edge | ~10.5% |
| Breakeven WR | ~32% |

v2.7 introduced the per-model min-prob gate which is expected to reduce signal frequency ~30% while improving win rate on remaining signals — full validation at 200 resolved trades.

---

Access is locked to a private whitelist — this bot is not open to the public.

---

*⚠️ Not financial advice. Prediction markets are volatile and algorithmic trading carries real financial risk. Use at your own discretion.*
