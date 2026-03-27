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

Each day Polymarket lists markets like *"Will the highest temperature in Tokyo exceed 14°C on March 23?"*. ThermalPeak discovers every active D+1 market across 35 cities, prices each bucket using a normal distribution over GFS model output (with NOAA and TAF corrections), and compares that fair value against the live market price. When the edge clears the threshold and Kelly sizing produces a bet above the minimum, it fires a signal.

The model accounts for:
- **Station bias** — LaGuardia runs warmer than city-center GFS, London City Airport runs warmer than Heathrow, etc. Each city has a hardcoded correction.
- **Fee drag** — Polymarket charges ~2% per trade. Kelly sizing and EV calculations factor this in.

---

## Features

**Market Discovery**
- Discovers all active D+1 temperature markets across 35 cities at startup and on a configurable scan interval
- Fetches live prices directly from Polymarket's CLOB order book (not cached Gamma data)
- Handles cities like Chengdu that Polymarket files under non-standard tags via direct slug fetch
- On-demand full refresh (GFS + NOAA + CLOB) triggered by `/signals` — no stale data
- Volume filter applied per-bucket (not city total) — ensures the specific outcome being signalled has real liquidity, not just the event aggregate

**Forecasting**
- GFS via Open-Meteo — baseline 48h forecast for all cities
- NOAA NWS — human-edited corrections for US cities, blended 55/45 with GFS
- AVWX METAR/TAF — aviation weather for short-range confirmation near resolution
- norm.cdf scoring — fair probability calculated over the bucket range, not a single threshold

**Signal Alerts**
- Auto-scan alerts only fire for **new** signals — no duplicate notifications if the same edge persists across scans
- `/signals` shows all current edges including already-open positions, labelled `📌 Already open — monitoring` vs `✅ New position opened`
- Signal notifications include a `📋 Pos` button for immediate position check after a signal fires

**Take Profit (Limit Order Style)**
- TP price is fixed at entry as an absolute level: `entry × (1 + tp_threshold)`
- Stored per-trade in the database/CSV — survives bot restarts
- When you change the TP threshold in settings, all open positions are immediately recalculated and checked against the new level
- **TP scales by confidence × hours-to-close matrix** — high-confidence signals with >12h left run wider (80%); positions with <6h remaining run at 90% (hold to resolution). Temperature markets close before the daily maximum is recorded — the price rockets after market close, not before. Premature exits clip gains.

**Risk Management**
- Kelly Criterion position sizing with configurable fraction and max bet
- Stop-loss monitoring — poll-based % drop check every 5 minutes
- Daily drawdown limit — scanning pauses automatically if you breach your equity limit for the day
- Immediate position check triggered when TP/SL settings change
- Refresh button on any position card triggers an immediate TP/SL check — no need to wait for the next poll

**Portfolio**
- Paper trading with full trade log (CSV or PostgreSQL)
- Live unrealised PnL on open positions — uses **bestBid** when spread ≤ $0.10 (Polymarket's own rule); falls back to mid on thin markets where the bid is stale. Prevents ghost bids distorting UPnL on low-liquidity buckets
- **4¢ spread filter at entry** — signals are rejected if `ask - bid > 4¢` at the moment of evaluation. Blocks thin markets (e.g. Chongqing 12¢ spread) before a position is opened. Grounded in Polymarket's own liquidity rewards qualifying range
- **14% minimum entry price floor** — signals are rejected when `yes_price < 0.14`. Based on 48-trade segmented analysis: sub-14% entries produced 29% win rate (-$12 PnL); entries above 18% produced 47% win rate (+$10 PnL). Raised from previous 10% floor on Mar 27, 2026
- Position detail shows `YES ask X¢ / bid X¢ · NO ask X¢ / bid X¢` — all four relevant prices at a glance
- True mid computed as `(bestAsk + bestBid) / 2` via dedicated `/price` endpoint calls — the `/markets/{cid}` token object does not carry bid/ask fields
- Auto-resolve: checks pending positions against live prices for TP/SL hits every 5 minutes
- Current probability ("Now") displayed with 1 decimal place — `11.0%` instead of `11%`, matching the true mid rather than Polymarket's rounded header display
- Sharpe ratio calculated across resolved trades (shows N/A until 10+ resolved)
- PnL card generated automatically on `/pnl` — 1280×680 HD dark card, thermal gradient, all key stats, centered pill

**Settings**
All risk and signal parameters are adjustable at runtime from the Telegram UI — no redeploy needed. Changes take effect immediately. A **Reset to Default** button restores all settings in one tap.

---

## Kelly Sizing — What Does "Kelly $11" Mean?

When a signal fires, the bot recommends a bet size — e.g. `kelly $11`. This is the dollar amount to place on that position, calculated from the Kelly Criterion:

```
kelly_size = bankroll × kelly_fraction × (edge / fair_value)
```

**Example — Miami YES, edge 12.4%, fair value 26%, bankroll $1,000, kelly fraction 15%:**
```
$1,000 × 0.15 × (0.124 / 0.26) = ~$71 raw Kelly → capped and fractioned to $11
```

In plain terms: out of your $1,000 paper bankroll, the bot is saying *put $11 on this outcome*. If it resolves correctly, profit is roughly `kelly × (1/entry − 1)`. If wrong, you lose the $11.

The number scales with your bankroll — profits accumulate into the bankroll, so bet sizes grow over time. `Max Bet` in Settings caps any single position regardless of what Kelly computes.

---

## Price Lifecycle — Entry, UPnL & Exit

Every position uses three distinct prices. Understanding which price is used where prevents confusion when comparing the bot's numbers to Polymarket's UI.

| Price | Source | Used for |
| :--- | :--- | :--- |
| **YES ask** | `GET /price?side=BUY` | Entry cost basis — recorded in the trade log at signal time |
| **YES bid** | `GET /price?side=SELL` | UPnL exit value — what you'd receive selling right now |
| **YES mid** | `(ask + bid) / 2` | Fair probability reference — used for edge calculation only |
| **NO ask** | `GET /price?side=BUY` | Entry cost for NO positions |
| **NO bid** | `GET /price?side=SELL` | UPnL exit value for NO positions |

**Example — New York YES, entry 22¢ ask, kelly $9.38:**

| Scenario | Price | Formula | Result |
| :--- | :--- | :--- | :--- |
| Entry recorded | YES ask = 22¢ | cost basis | $9.38 notional |
| Current YES bid = 20¢ | exit value | (20 − 22) / 22 × $9.38 | **UPnL −$0.85** |
| TP fires at 34% gain | YES bid ≥ entry × 1.34 = 29.5¢ | limit check | +$3.19 profit |
| SL fires at 30% drop | YES bid ≤ entry × 0.70 = 15.4¢ | poll check | −$2.81 loss |
| Resolves YES | $1.00 | oracle settles | +$33.29 profit |
| Resolves NO | $0.00 | oracle settles | −$9.38 loss |

**SL loss is capped at threshold** — if price crashes past the 30% SL level between polls, the recorded loss is capped at 30% × kelly (not the actual poll price). Simulates a real limit stop order that fills at the threshold, not wherever price happens to be 3 minutes later. SL alert shows `Exit: 12.6%` — the actual simulated exit price — not the noisy poll price.

**Breakeven stop** — once UPnL reaches +30% AND spread ≤ 5¢, the SL automatically moves to entry price. Spread gate prevents noise triggering on thin markets. SL alert shows `breakeven (N★)` to distinguish from a standard `30% (N★)` stop.

**Paper vs live gap (remaining):** In paper mode, TP and SL are poll-based checks every 3 minutes. In live mode, TP should be a resting limit sell order placed at entry, and SL should be a market sell when triggered.

---

## Commands

| Command | What it does |
| :--- | :--- |
| `/signals` | Full refresh then scan all 35 cities for edges right now |
| `/vol` | Live volume table sorted by liquidity |
| `/brief` | Cities where GFS disagrees with the market favourite by 3°C+ |
| `/cities` | Browse all active D+1 markets, odds, METAR, and GFS forecast |
| `/pnl` | PnL card as image + unrealised PnL on open positions |
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
| Vol filter (per bucket) | $1K | Minimum volume on the specific bucket being signalled |
| Edge threshold | 8% | Minimum edge (fair − market) to fire a signal |
| Scan interval | 1h | How often CLOB prices are refreshed |

**Sizing**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Kelly fraction | 15% | Fraction of Kelly formula applied to bet sizing |
| Max bet | $100 | Hard cap per position regardless of Kelly output |

**Risk**

| Setting | Default | What it controls |
| :--- | :--- | :--- |
| Take profit (base) | 60% | Base TP — scaled by confidence × hours-to-close matrix at entry |
| Stop loss | 30% | Close position when down this % from entry (poll-based) |
| Max daily drawdown | 10% | Pause signals if today's losses hit this % of equity |

---

## City Coverage

35 cities across all major timezones, each mapped to the exact ICAO station Polymarket uses for resolution:

New York (KLGA), London (EGLC), Tokyo (RJTT), Shanghai (ZSPD), Singapore (WSSS), Paris (LFPG), Seoul (RKSI), Hong Kong (HKO), Toronto (CYYZ), Warsaw (EPWA), Tel Aviv (LLBG), Lucknow (VILK), Madrid (LEMD), Sao Paulo (SBGR), Buenos Aires (SAEZ), Miami (KMIA), Chicago (KORD), Dallas (KDFW), Atlanta (KATL), Seattle (KSEA), Ankara (LTAC), Munich (EDDM), Milan (LIMC), Wellington (NZWN), Taipei (RCTP), Beijing (ZBAA), Shenzhen (ZGSZ), Wuhan (ZHHH), Chongqing (ZUCK), Chengdu (ZUUU), San Francisco (KSFO), Austin (KAUS), Jakarta (WIII), Denver (KDEN), Houston (KIAH)

---

Access is locked to a private whitelist — this bot is not open to the public. If you're interested in getting access, reach out to the owner directly via Telegram.

---

*⚠️ Not financial advice. Prediction markets are volatile and algorithmic trading carries real financial risk. Use at your own discretion ⚠️*
