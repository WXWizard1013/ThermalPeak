<div align="center">
  <img src="https://raw.githubusercontent.com/WXWizard1013/ThermalPeak/main/Pics/logo-white.png" alt="ThermalPeak Logo" width="500"/>

  **Scan The Heat, Trade The Peak.**

---

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Telegram API](https://img.shields.io/badge/Telegram-Bot_API-0088cc.svg?logo=telegram)](https://core.telegram.org/bots/api)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-asyncpg-336791.svg?logo=postgresql)](https://www.postgresql.org/)
</div>

ThermalPeak bridges the gap between meteorological forecasting...

ThermalPeak bridges the gap between meteorological forecasting and predictive markets. By synthesizing live Polymarket order books with hyper-local, bias-corrected weather models, it identifies execution edges and automates D+1 temperature trading directly via Telegram.

---

## ✨ System Architecture & Features

* **Live Market Discovery**: Asynchronously polls Polymarket's Gamma and CLOB APIs to identify active D+1 daily temperature markets and fetch live Layer-2 order book liquidity.
* **Algorithmic Weather Arbitrage**: Aggregates forecast data from Open-Meteo, NOAA NWS, and AVWX (METAR/TAF).
* **Micro-Climate Bias Correction**: Employs strict ICAO station mapping to match exact Polymarket resolution rules (e.g., `KLGA` for New York, not JFK; `EGLC` for London, not Heathrow).
* **Dynamic Risk Management**: Utilizes the Kelly Criterion for dynamic position sizing. Includes auto Stop-Loss (SL) and dynamic Take-Profit (TP) features.
* **Persistent State**: Fast, asynchronous database logging using PostgreSQL (`asyncpg`), with a seamless fallback to local `.csv` logging for lightweight deployments.
* **Zero-Trust Telegram UI**: Fully operated via Telegram inline keyboards. Locked down via a hardcoded `WHITELIST` of authorized User IDs.

---

## 🎛️ Command Reference

The bot is entirely controlled via Telegram. Below are the available commands:

| Command | Description |
| :--- | :--- |
| `/signals` | Scan weather models vs. Polymarket odds to find trading edges. |
| `/pos` | View currently active and pending market positions. |
| `/pnl` | Display detailed Profit & Loss statements and win rates. |
| `/vol` | Check live market volumes and liquidity across tracked events. |
| `/drift` | Analyze forecast drift (changes in model outputs over time). |
| `/cities` | View the tracked ICAO station database and accuracy metrics. |
| `/settings` | Adjust Kelly fraction, bet limits, and SL/TP thresholds. |
| `/log` | Read recent execution logs directly in chat. |
| `/clearlog`| Purge temporary log history. |
| `/accu` | Display historical accuracy of weather models vs actual resolutions. |
| `/status` | Check system uptime, API health, and database connection. |
| `/resolveall` | Force-check resolution for all pending expired markets. |
| `/export` | Export PnL and trade history as a downloadable `.csv` file. |
| `/pause` | Halt automated market polling. |
| `/cont` | Resume automated market polling. |
| `/brief` | Get a quick summarized daily/weekly performance brief. |
| `/summ` | View a comprehensive performance summary. |
| `/version` | Show current ThermalPeak build and runtime environment. |
| `/help` | Display the help menu and command list. |


---
*Disclaimer: This software is for educational and research purposes only. Automated trading involves significant risk.*
