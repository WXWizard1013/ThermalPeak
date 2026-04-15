## Release History

### v2.7.2 — New Cities (Guangzhou, Karachi, Manila), Resolution Notes & Spread Bug Fix (Apr 15, 2026)

**City Expansion — 47 → 50**
- **Guangzhou** added — ZGGG (Baiyun Intl). σ=1.6°C, bias=+0.5°C (industrial UHI, station north of city). Confirmed via Polymarket market rules Apr 2026.
- **Karachi** added — OPKC (Jinnah Intl). σ=1.5°C, bias=0.0°C (coastal; Arabian Sea sea breeze already captured by GFS grid). Confirmed Apr 2026.
- **Manila** added — RPLL (Ninoy Aquino Intl). σ=1.2°C, bias=+0.3°C (Metro Manila UHI, near city center). Confirmed Apr 2026.
- All three were already appearing in the market scanner (tag fetch finding them) but had no ICAO/coords, so `city_cache` had no entry → `find_opportunities()` skipped them silently.

**Resolution Station Notes — new `CITY_RESOLUTION_NOTE` dict**
- Every `/cities` detail card now shows `📍 [station description]`, `σ`, and `bias` directly below the city header.
- 13 cities with explicit notes covering non-obvious stations (KLGA not KJFK, EGLC not EGLL, KBKF not KDEN, HKO Observatory for HK, RCSS for Taipei, etc.).
- All other cities auto-display as `[ICAO] — Wunderground resolution station`.

**Bug Fix — Spread Check "?" in Logs**
- The spread-check log was printing `?` as the city name because the bucket dict (built at line 892) never carried a `city_key` field.
- `bucket.get("city_key", bucket.get("city","?"))` always fell through to `"?"`.
- Fixed: spread check now uses the outer `for city_key, info in event_cache.items()` loop variable directly.

**`normalize_city` aliases added**
- `"canton"` → `"guangzhou"`, `"guangzhou city"` → `"guangzhou"`
- `"karachi city"` → `"karachi"`
- `"metro manila"` → `"manila"`, `"ncr"` → `"manila"`, `"manila city"` → `"manila"`

**`NEW_CITIES`** updated to `{"guangzhou", "karachi", "manila"}` — 🆕 section in `/cities` reflects current version additions.

---

### v2.7.1 — Relative Min-Prob Gate Fix, Consensus Gate 2.5→3.5°C & Trade Log Reset (Apr 13, 2026)

**Critical Gate Fix — Absolute → Relative Min-Prob Threshold**
- v2.7 shipped with `MIN_PROB_THRESHOLD = 0.55` (absolute probability). This was physically impossible for any 1°C bucket at σ=2.0°C (max possible P ≈ 19.7%). Every 1°C bucket was being vetoed unconditionally.
- Fixed: gate now uses `MIN_PROB_RATIO = 0.65` — each model must give ≥65% of the *maximum possible* probability for that bucket width/sigma. Normalises correctly across 1°C buckets (max ~20%) and open-ended buckets (max ~70%).
- A second bug: the regional wx veto kept an absolute `0.40` floor, which blocked all US 2°F buckets (max P ≈ 28%). Fixed to relative `REGIONAL_VETO_RATIO = 0.40` against regional max, not absolute probability.

**Consensus Gate Raised 2.5 → 3.5°C**
- Spring model transition globally pushing normal mid-latitude spread to 3.0–4.0°C. At 2.5°C, 77% of cities were blocked before reaching the min-prob gate.
- Cities with genuinely chaotic spread (Seoul 9.3°C, Jeddah 9.1°C, Munich 6.9°C) still blocked at 3.5°C. 14 additional cities now pass to the min-prob gate for per-model validation.

**`/resetlog` PostgreSQL fix**
- `/resetlog` was only deleting the CSV file. PostgreSQL trade records survived the reset.
- Fixed: `DELETE FROM trades` executed before CSV removal. Trade count correctly resets to 0.

**Trade Log Reset — Clean Dataset**
- Trade log reset to 0 on Apr 13 after gate calibration. Previous 87-trade pre-calibration dataset archived.
- First live signal under new architecture: Hong Kong 28°C YES at 23.5¢ (Apr 13, 2026).

---

### v2.7 — Station Audit, Per-Model Gate & 47-City Coverage (Apr 12, 2026)

**Signal Architecture**
- **Per-model min-probability gate** — replaces binary bucket voting (3-of-5 majority). Each NWP model independently computes `_bucket_prob(model_temp, sigma, lo, hi)`. Gate blocks if any model gives less than the relative threshold fraction of the max possible probability for that bucket (see v2.7.1 for calibration fix).
- **Regional wx hard veto** — NOAA/BMKG/MSS/CWA probability fed directly into the min-prob gate as a named model. Human-edited national forecast pointing away from the bucket = block.
- **CWA (Taiwan) integrated** — Central Weather Administration added for Taipei. Endpoint: `opendata.cwa.gov.tw` Songshan District forecast. 6h cache. Graceful skip when `CWA_API_TOKEN` env var not set.

**Station Corrections (full crosscheck — all 47 cities vs live Polymarket market rules)**
- **Denver** — KDEN → **KBKF** (Buckley Space Force Base, Aurora CO). Coords corrected from (39.86, −104.67) to (39.717, −104.752). ~18km SE of KDEN; different microclimate.
- **Moscow** — duplicate dict entry removed. UUEE (Sheremetyevo) was silently overwriting UUWW (Vnukovo) due to Python dict key collision. Single UUWW entry retained.
- **Taipei** — RCTP → **RCSS** (Taipei Songshan Airport) for April 2026+ markets. Coords corrected to (25.069, 121.553). `CITY_BIAS_C` 0.0 → +1.0°C (urban basin vs coastal Taoyuan plain).

**City Expansion — 44 → 47**
- **Cape Town** — FACT (Cape Town Intl). σ=2.0°C, bias=−0.5°C.
- **Jeddah** — OEJN (King Abdulaziz Intl). σ=1.5°C, bias=0.0°C.
- **Lagos** — DNMM (Murtala Muhammed Intl). σ=1.2°C, bias=+0.5°C.

**UI**
- `/cities` — 3-per-row buttons. "Or type the city name directly." added. 🆕 New cities section on page 1. Cold-start fallback trimmed to confirmed-active cities (removed Dubai, Sydney, Berlin, Mumbai, Bangkok, Boston, Phoenix, Changsha).
- `/vol` — PAGE_SIZE 20→18 (clean 6×3 grid).
- **Arrows** — `bucket_with_c()` renders "or higher"→↑ and "or below"→↓ globally across all views.
- Slug 0-stubs log spam suppressed (was 516 lines/40min at day boundary).
- `NEW_CITIES` constant introduced — new city detection no longer breaks when cities are added to `KNOWN_CITIES`.



### v1.0 — Initial Alpha (Mar 16, 2026)
* Proof-of-concept. Four cities (New York, London, Singapore, Shanghai), basic GFS scanning, text-only Telegram alerts, manual CSV logging, trades resolved by hand.

### v2.0 — Major Overhaul (Mar 18, 2026)
* **City coverage** expanded from 4 to 30 cities worldwide.
* **Full Telegram UI** — replaced plain-text alerts with a proper inline keyboard interface for navigating markets, managing positions, and adjusting settings.
* **Automated trade management** — auto Take-Profit (+60%) and Stop-Loss (-40%) introduced.
* **Live CLOB pricing** — switched from cached Gamma data to Polymarket's live order book.
* **Rebuilt probability engine** — norm.cdf replaces the old sigmoid scorer. Factors in 2% fee drag, applies edge caps, weights TAF near expiry.
* **PostgreSQL backend** — persistent trade logging with async connection pooling. CSV fallback.

### v2.1 — Analytics & Maintenance (Mar 19, 2026)
* `/accu` — per-city accuracy tracking. `/summ` — daily PnL summary. `/resolveall` — force-sweep pending positions. `/resetlog` — wipe trade history.

### v2.2 — Stability (Mar 20, 2026)
* Refined data pipelines and API request handling. Minor UI navigation improvements.

### v2.3 — Settings, Risk, Card & 30-City Coverage (Mar 21–22, 2026)
* **Pure inline keyboard UI** — bottom keyboard stripped entirely.
* **Overhauled /settings** — conversational parameter input with `/cancel` support.
* **Risk management suite** — TP (standard + high-conf tier), SL, slippage tolerance, max daily drawdown.
* **Shareable PnL card** — auto-generated HD PNG on every `/pnl`.
* **30-city coverage** — fixed market discovery for Chinese mainland markets.
* **Station bias corrections** — HKO, KLGA, EGLC, ZSPD, CWA 46692.
* **NOAA NWS blending** — 55/45 with GFS for US cities.
* **Hours-ahead gate** — 16–72h window replaces strict UTC date matching.

### v2.3.1 — Risk Engine, UI Polish & 35-City Coverage (Mar 23, 2026)
* **35-city coverage** — San Francisco (KSFO) and Austin (KAUS) added.
* **Limit-order TP** — fixed at entry, stored per-trade, survives restarts.
* **SL min floor** — absolute ¢ floor alongside % SL. Default 5¢.
* **Duplicate signal suppression** — auto-scan stays silent if no new positions opened.
* **Settings → Reset to Default** button.

### v2.4 — Signal Quality, ICAO Fixes & 38-City Coverage (Mar 24–27, 2026)
* **38-city coverage** — Istanbul (LTFM), Moscow (UUEE), Mexico City (MMMX).
* **ICAO corrections** — KJFK → KLGA, EGLL → EGLC, ZSSS → ZSPD.
* **Edge cap at 13%** — very high edge = GFS wrong, not market.
* **Min entry floor 14%** — below this had 17–27% win rate.
* **Per-bucket volume filter**, **spread filter** (≤ 4¢), **NO bet guards** (75–92% YES), **fair value ceiling 80%**, **city deduplication**.

### v2.4.1 — Profit Floor & Reliability (Mar 27, 2026)
* **Profit floor protection**, **TP cap fix**, **SL bypass fix**, **`/pos` command**.

### v2.4.2 — Position Detail & UPnL (Mar 28, 2026)
* **Individual position detail card** — bid/ask, Kelly, UPnL, GFS, NOAA, METAR, Polymarket link.
* **UPnL mid-price fix**. **Resolve/Remove buttons**. **Positions pagination** at 8 per page.

### v2.4.3 — Backtest Notebook v1.0 (Mar 28–29, 2026)
* **Backtest notebook** (`thermalpeak_backtest.ipynb`) — Colab parameter sweeps against real resolved trades.

### v2.4.4 — UI Polish & City Expansion (Mar 29–30, 2026)
* **`bucket_with_c()`** — inline °C next to °F outcomes. **`/vol` and `/cities` pagination** at 20 per page.

### v2.4.4.1 — Signal Quality & City Risk Management (Apr 1, 2026)
* **Edge threshold raised to 10%**.
* **Per-city σ calibration** — tight (1.0–1.5°C) for stable tropical, default (1.8–2.2°C) mid-latitude, wide (2.5–3.0°C) orographic/coastal.
* **City exclusion system** — `/exclude` + automated 10-SL-loss alert. Persisted via PostgreSQL.

### v2.5 — Multi-Model Consensus & Dead Zone Pause (Apr 2, 2026)
* **5-model gate** — 3 of 5 required (GFS, ECMWF, ICON, UKMET, GEM).
* **ICON boost** for European cities. **Dead zone pause** 22:00–02:00 UTC. **`/random`** command.

### v2.6 — 44-City Expansion & Station Audit (Apr 3, 2026)
* **44-city coverage** — Busan (RKPK), Panama City (MPMG), Amsterdam (EHAM), Kuala Lumpur (WMKK), Helsinki (EFHK), Jakarta (WIHH).
* ICON boost extended to Amsterdam and Helsinki. Per-city σ for new cities. Slug fetch fallback.

### v2.6.1 — Full Resolution Station Audit (Apr 3, 2026)
* **Station corrections** — Moscow UUEE → UUWW (Vnukovo), Houston KIAH → KHOU (Hobby), Taipei coords → RCTP. Jakarta WIHH and Panama City MPMG verified.

### v2.6.2 — Emergency Abort, Daily Summary Fix & UI Fixes (Apr 9, 2026)
* **`/abort` emergency command** — two-step confirmed kill-switch. Stops scanning, resolves all open positions at live price, blocks new trades. `/cont` resumes.
* **`🚨 Abort` on `/start`** — one tap from home screen with `× Close` beside it.
* **`/summ` day-scoped** — queries only today's trades (00:00–23:59 UTC), not all-time.
* **Consensus gate raised 1.5 → 2.5°C** — April spring transition was blocking every city at 1.5°C.
* **Market discovery hardened** — slug fetch now covers all 44 cities (was 7). Removed `closed=True` filter incorrectly blocking active markets. `process_event` no longer requires `active=True`.
* **`× Close` button fixed** — was missing its handler; now correctly deletes the message on all views.
* **`/vol` °C strip fixed** — regex now correctly removes `(X.X°C)` suffix including the degree symbol.
* **GFS Drift alert bold** — auto-drift now sends with `parse_mode=Markdown`.
* **GFS schedule in UTC** — replaced WIB times with `00:00 · 06:00 · 12:00 · 18:00 UTC`.
* **`/cities` fallback fixed** — showed only 12 cities when event cache was empty; now shows all 44.
