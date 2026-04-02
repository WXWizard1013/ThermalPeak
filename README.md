<div align="center">
  <img src="https://raw.githubusercontent.com/WXWizard1013/ThermalPeak/main/Pics/logo-white.png" alt="ThermalPeak Logo" width="500"/>

  **Scan The Heat, Trade The Peak.**

---

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Telegram API](https://img.shields.io/badge/Telegram-Bot_API-0088cc.svg?logo=telegram)](https://core.telegram.org/bots/api)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-asyncpg-336791.svg?logo=postgresql)](https://www.postgresql.org/)
[![v2.6](https://img.shields.io/badge/version-v2.6-orange.svg)]()

</div>

ThermalPeak is a Telegram-based trading bot that finds edges in Polymarket's daily temperature markets. It pulls live order book prices from Polymarket's CLOB API, runs them against a 5-model weather consensus (GFS, ECMWF, ICON, UKMET, GEM), and fires a signal only when all available models agree and the math says the market is wrong. Everything — signals, positions, risk settings, city exclusions — is managed from your Telegram chat.

---

## How It Works

Each day Polymarket lists markets like *"Will the highest temperature in Tokyo exceed 19°C on April 1?"*. ThermalPeak discovers every active D+1 market across 38 cities, prices each bucket using a normal distribution over a blended 5-model forecast, and compares that fair value against the live market price. When the edge clears the threshold, the consensus gate passes, the entry window is correct, and Kelly sizing produces a bet above the minimum, it fires a signal.

The model accounts for:
- **5-model consensus** — GFS, ECMWF, ICON, UKMET, and Canadian GEM must agree within 1.5°C before a signal can fire. Model disagreement = uncertainty = skip.
- **Entry timing gate** — signals only fire in the 20–48h window before resolution. This is the sweet spot confirmed by top Polymarket weather traders.
- **Temperature laddering** — the primary signal plus the best adjacent bucket (≥8% edge) are opened simultaneously with edge-weighted Kelly. Based on neobrother's strategy.
- **Dynamic Kelly by model count** — 5 models agreeing = full Kelly. 4 models = 80%. Rewards high-conviction signals.
- **Per-city σ calibration** — GFS uncertainty is not the same everywhere. Stable tropical cities (Singapore, Lucknow) use a tight 1.0–1.2°C σ. Orographic and coastal cities (LA, Seattle, Denver) use a wide 2.8–3.0°C σ that makes it harder to generate false signals.
- **Bayesian Kelly sizing** — position sizes are scaled by a Beta-prior estimate of each city's win rate. New cities start conservative; cities with track records pull toward their actual performance.
- **Station bias** — LaGuardia runs warmer than city-center GFS, London City Airport runs warmer than Heathrow, etc. Each city has a hardcoded correction.
- **Fee drag** — Polymarket charges ~2% per trade. Kelly sizing and EV calculations factor this in.
- **Spread filter** — signals only fire when the YES ask/bid spread is ≤ 4¢.

---

## Features

**Market Discovery**
- Discovers all active D+1 temperature markets across 38 cities at startup and on a configurable scan interval
- Fetches live prices directly from Polymarket's CLOB order book (not cached Gamma data)
- On-demand full refresh (all models + CLOB) triggered by `/signals` — no stale data

**Forecasting**
- GFS via Open-Meteo — baseline 48h forecast for all cities
- ECMWF (`ecmwf_ifs025`) via Open-Meteo — global high-resolution model, refreshed every 4h
- ICON (`icon_global`) via Open-Meteo — European mesoscale model, boosted for EU/MENA cities
- UKMET (`ukmo_seamless`) via Open-Meteo — UK Met Office global model
- Canadian GEM (`gem_seamless`) via Open-Meteo — North American mesoscale correction
- NOAA NWS — human-edited corrections for US cities, blended 55/45 with GFS
- AVWX METAR/TAF — aviation weather for short-range confirmation near resolution
- Per-city σ calibration — tiered GFS uncertainty by climate type and terrain complexity
- norm.cdf scoring — fair probability calculated over the bucket range, not a single threshold

**Signal Quality Filters**
- 5-model consensus gate — all available models must agree within 1.5°C or city is skipped
- Entry timing gate — signals only fire 20–48h before resolution
- Edge threshold: 10% minimum
- Edge cap at 13% — very high edge = GFS vs market disagreement = GFS is probably wrong
- Minimum entry floor at 14% — market prices below this had 17–27% win rate historically
- Spread liquidity filter — ≤ 4¢ ask/bid spread required
- Max 2 open positions per city — prevents concentration during volatile periods
- City exclusion system — cities with 10+ SL losses trigger an exclusion prompt

**Signal Sizing**
- 2-bucket temperature ladder — primary + adjacent bucket (≥8% edge), edge-weighted Kelly split
- Dynamic Kelly scaling — scales 30–100% based on number of models agreeing
- Bayesian Kelly multiplier — per-city Beta posterior win rate scales position size
- Hard max bet cap and minimum bet floor

**Take Profit & Stop Loss**
- TP price is fixed at entry as an absolute level: `entry × (1 + tp_threshold)`
- Stored per-trade in the database — survives bot restarts
- Three-tier trailing SL — ratchets up the floor as positions move in favour (+15% → lock +5%, +35% → lock +20%)
- Profit floor protection — prevents profitable positions from reversing into full SL losses
- High-confidence signals use a separate higher TP tier

**Risk Management**
- Daily drawdown limit — scanning pauses automatically if you breach your equity limit
- SL min floor — absolute ¢ floor that must also be breached before SL fires

**Portfolio**
- Paper trading with full trade log (CSV or PostgreSQL)
- Live unrealised PnL on open positions via CLOB price refresh
- Auto-resolve: checks pending positions against live prices every 3 minutes
- Sharpe ratio calculated across resolved trades

---

## Commands

| Command | What it does |
| :--- | :--- |
| `/signals` | Full refresh then scan all 38 cities for edges right now |
| `/vol` | Live volume table sorted by liquidity (paginated, 20 per page) |
| `/brief` | Cities where GFS disagrees with the market favourite by 3°C+ |
| `/cities` | Browse all active D+1 markets, odds, METAR, and GFS forecast (paginated) |
| `/pnl` | PnL card as image + unrealised PnL on open positions |
| `/pos` | Open positions with live prices and UPnL |
| `/log` | Recent trade history |
| `/export` | Download full trade log as CSV |
| `/exclude` | View and manage excluded cities |
| `/settings` | Adjust all signal, risk, and protection parameters |
| `/status` | Bot health, last scan time, active markets, current settings |
| `/drift` | Check if GFS forecasts have shifted since last run |
| `/accu` | Win/loss breakdown by city |
| `/summ` | Daily summary — signals fired, win rate, net PnL |
| `/resolveall` | Force-check all pending positions against live prices |
| `/resetlog` | Wipe the trade log and start fresh |
| `/pause` / `/cont` | Pause and resume the scan loop |
| `/random` | AI-generated weather fact (tap 🔀 for a new one) |
| `/version` | Changelog and build info |
| `/help` | Full command list |

---

## Settings Reference

All adjustable via `/settings` in Telegram.

**Signals**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Volume filter | $10K | Minimum bucket volume to include in scans |
| Edge threshold | 10% | Minimum edge (fair − market) to fire a signal |
| Min entry price | 14% | Minimum market price to enter |
| Scan interval | 1h | How often CLOB prices are refreshed |

**Sizing**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Kelly fraction | 15% | Fraction of Kelly formula applied to bet sizing |
| Max bet | $25 | Hard cap per position |

**Risk**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Take profit | 60% | Close when up this % from entry — stored as fixed limit price per trade |
| Stop loss | 30% | Close position when down this % from entry |
| Profit floor | +5% | Minimum profit locked in when a position hits +30% gain |
| Max daily drawdown | 10% | Pause signals if today's losses hit this % of equity |

---

## City Coverage

38 cities across all major timezones, each mapped to the exact ICAO station Polymarket uses for resolution:

New York (KLGA), London (EGLC), Tokyo (RJTT), Shanghai (ZSPD), Singapore (WSSS), Paris (LFPG), Seoul (RKSI), Hong Kong (HKO), Toronto (CYYZ), Warsaw (EPWA), Tel Aviv (LLBG), Lucknow (VILK), Madrid (LEMD), Sao Paulo (SBGR), Buenos Aires (SAEZ), Miami (KMIA), Chicago (KORD), Dallas (KDFW), Atlanta (KATL), Seattle (KSEA), Ankara (LTAC), Munich (EDDM), Milan (LIMC), Wellington (NZWN), Taipei (RCTP), Beijing (ZBAA), Shenzhen (ZGSZ), Wuhan (ZHHH), Chongqing (ZUCK), Chengdu (ZUUU), San Francisco (KSFO), Austin (KAUS), Denver (KDEN), Houston (KIAH), Los Angeles (KLAX), Istanbul (LTFM), Moscow (UUEE), Mexico City (MMMX)

---

## Performance (Paper Trading)

As of Apr 3, 2026 — 68 trades logged since Mar 27, 2026:

| Metric | Value |
| :--- | :--- |
| Resolved trades | 64 |
| Win rate | ~39% |
| Net PnL | +$38.17 |
| Avg edge | 10.1% |
| Breakeven WR | 31.9% |

Strong performers: Chicago (83% WR), New York (50% WR), Lucknow, Tokyo, Ankara, Taipei.
Weak performers: LA, Seattle, Denver, Houston (systematic GFS coastal/orographic bias — wide σ assigned, consensus gate filters heavily).

---

Access is locked to a private whitelist — this bot is not open to the public.

---

*⚠️ Not financial advice. Prediction markets are volatile and algorithmic trading carries real financial risk. Use at your own discretion.*
