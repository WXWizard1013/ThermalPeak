<div align="center">
  <img src="https://raw.githubusercontent.com/WXWizard1013/ThermalPeak/main/Pics/logo-white.png" alt="ThermalPeak Logo" width="500"/>

  **Scan The Heat, Trade The Peak.**

---

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Telegram API](https://img.shields.io/badge/Telegram-Bot_API-0088cc.svg?logo=telegram)](https://core.telegram.org/bots/api)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-asyncpg-336791.svg?logo=postgresql)](https://www.postgresql.org/)
[![Pillow](https://img.shields.io/badge/Pillow-imaging-yellow.svg)](https://python-pillow.org/)

</div>

ThermalPeak is a Telegram-based trading bot that finds edges in Polymarket's daily temperature markets. It pulls live order book prices from Polymarket's CLOB API, runs them against bias-corrected weather forecasts from GFS, NOAA, and AVWX, and fires a signal when the math says the market is wrong. Everything — signals, positions, risk settings — is managed from your Telegram chat.

---

## How It Works

Each day Polymarket lists markets like *"Will the highest temperature in Tokyo exceed 14°C on March 23?"*. ThermalPeak discovers every active D+1 market across 30 cities, prices each bucket using a normal distribution over GFS model output (with NOAA and TAF corrections), and compares that fair value against the live market price. When the edge clears the threshold and Kelly sizing produces a bet above the minimum, it fires a signal.

The model accounts for:
- **Station bias** — LaGuardia runs warmer than city-center GFS, London City Airport runs warmer than Heathrow, etc. Each city has a hardcoded correction.
- **Slippage** — your configured slippage tolerance is deducted from fair value before the edge calculation, so you're not fooling yourself about execution cost.
- **Fee drag** — Polymarket charges ~2% per trade. Kelly sizing and EV calculations factor this in.

---

## Features

**Market Discovery**
- Discovers all active D+1 temperature markets across 30 cities at startup and on a configurable scan interval
- Fetches live prices directly from Polymarket's CLOB order book (not cached Gamma data)
- Handles cities like Chengdu that Polymarket files under non-standard tags via direct slug fetch

**Forecasting**
- GFS via Open-Meteo — baseline 48h forecast for all cities
- NOAA NWS — human-edited corrections for US cities, blended 55/45 with GFS
- AVWX METAR/TAF — aviation weather for short-range confirmation near resolution
- norm.cdf scoring — fair probability calculated over the bucket range, not a single threshold

**Risk Management**
- Kelly Criterion position sizing with configurable fraction and max bet
- Per-trade take-profit (standard and high-confidence tiers based on signal star rating)
- Stop-loss monitoring on all open positions
- Daily drawdown limit — scanning pauses automatically if you breach your equity limit for the day
- Slippage tolerance deducted from fair value before any signal fires

**Portfolio**
- Paper trading with full trade log (CSV or PostgreSQL)
- Live unrealised PnL on open positions via CLOB price refresh
- Auto-resolve: checks pending positions against live prices for TP/SL hits every 30 minutes
- Sharpe ratio calculated across resolved trades (shows N/A until 10+ resolved)
- Shareable PnL card generated as a PNG image — dark card, thermal gradient, all key stats

**Settings**
All risk and signal parameters are adjustable at runtime from the Telegram UI — no redeploy needed. Changes take effect immediately.

---

## Commands

| Command | What it does |
| :--- | :--- |
| `/signals` | Scan all 30 cities for edges right now |
| `/vol` | Live volume table sorted by liquidity |
| `/brief` | Cities where GFS disagrees with the market favourite by 3°C+ |
| `/cities` | Browse all active D+1 markets, odds, METAR, and GFS forecast |
| `/pnl` | Full PnL breakdown — wins, losses, Sharpe, unrealised |
| `/card` | Generate and send a shareable PnL image card |
| `/pos` | Open positions with live prices and UPnL |
| `/log` | Recent trade history |
| `/export` | Download full trade log as CSV |
| `/settings` | Adjust all signal and risk parameters |
| `/status` | Bot health, last scan time, active markets, current settings |
| `/drift` | Check if GFS forecasts have shifted since last run |
| `/accu` | Win/loss breakdown by city |
| `/summ` | Daily summary — signals fired, win rate, net PnL |
| `/resolveall` | Force-check all pending positions against live prices |
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
| Volume filter | $10K | Minimum market volume to include in scans |
| Kelly fraction | 15% | Fraction of Kelly formula applied to bet sizing |
| Max bet | $100 | Hard cap per position |
| Edge threshold | 8% | Minimum edge (fair − market) to fire a signal |
| Scan interval | 1h | How often CLOB prices are refreshed |

**Risk**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Take profit | 60% | Close position when up this % on entry price |
| TP high conf | 80% | TP threshold for signals rated high-conf stars or above |
| Stop loss | 30% | Close position when down this % |
| High conf stars | 8★ | Star rating that triggers the higher TP |
| Slippage tolerance | 2% | Deducted from fair value before edge calc |
| Max daily drawdown | 10% | Pause signals if today's losses hit this % of equity |

---

## City Coverage

30 cities across all major timezones, each mapped to the exact ICAO station Polymarket uses for resolution:

New York (KLGA), London (EGLC), Tokyo (RJTT), Shanghai (ZSPD), Singapore (WSSS), Paris (LFPG), Seoul (RKSI), Hong Kong (HKO), Toronto (CYYZ), Warsaw (EPWA), Tel Aviv (LLBG), Lucknow (VILK), Madrid (LEMD), Sao Paulo (SBGR), Buenos Aires (SAEZ), Miami (KMIA), Chicago (KORD), Dallas (KDFW), Atlanta (KATL), Seattle (KSEA), Ankara (LTAC), Munich (EDDM), Milan (LIMC), Wellington (NZWN), Taipei (RCTP), Beijing (ZBAA), Shanghai (ZSPD), Shenzhen (ZGSZ), Wuhan (ZHHH), Chongqing (ZUCK), Chengdu (ZUUU)

---

## Setup

**Environment variables required:**

```
TELEGRAM_TOKEN=
TELEGRAM_CHAT_ID=
AVWX_TOKEN=
DATABASE_URL=        # optional, falls back to CSV
```

**Dependencies (`requirements.txt`):**
```
python-telegram-bot==21.9
aiohttp==3.9.5
asyncpg
Pillow
```

**Font files** (place in project root alongside `main.py`):
```
Inter_18pt-Bold.ttf
Inter_18pt-Regular.ttf
```
Download from [fonts.google.com/specimen/Inter](https://fonts.google.com/specimen/Inter).

The bot is locked to a whitelist of Telegram user IDs defined in `WHITELIST` inside `main.py`. Add your user ID there before deploying.

---

*⚠️ Not financial advice. Prediction markets are volatile and algorithmic trading carries real financial risk. Use at your own discretion.*
