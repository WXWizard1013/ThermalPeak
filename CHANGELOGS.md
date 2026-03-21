##  Release History

###  v1.0 — Initial Alpha (Mar 16, 2026)
* **Foundation**: Proof-of-concept launched with 4 initial cities (New York, London, Singapore, Shanghai).
* **Basic Scanning**: Scanned foundational weather models against market thresholds to find basic trading edges.
* **Manual Operation**: Relied on text-only Telegram alerts, local CSV logging, and required users to manually resolve trades as wins or losses.

###  v2.0 — Major Overhaul & Automation (Mar 18, 2026)
* **Global Expansion**: Scaled market coverage from 4 to 36 cities worldwide.
* **Interactive Telegram UI**: Replaced plain-text commands with a sleek, button-driven interface. Easily navigate markets, manage live positions, and adjust bot settings.
* **Automated Trade Management**: Introduced Auto Take-Profit (+60%) and Stop-Loss (-40%). The bot now automatically tracks and resolves open positions.
* **Precision Pricing**: Upgraded to read live order book data, specifically isolating tomorrow's (D+1) markets to guarantee real-time pricing.
* **Smarter Execution Math**: Rebuilt the probability engine. Factors in Polymarket's 2% fee, applies strict edge caps to ignore "too good to be true" signals, and dynamically weighs short-term forecasts heavier near expiration.
* **Cloud Database & Security**: Upgraded to PostgreSQL for secure, persistent trade logging, protected by a strict User ID whitelist.

###  v2.1 — Advanced Analytics (Mar 19, 2026)
* **Model Accuracy Tracking**: Added the `/accu` command to track how often weather models (GFS/NOAA) historically align with actual Polymarket resolutions.
* **Comprehensive Summaries**: Introduced `/summ` to generate deep-dive performance and PnL summaries.
* **Manual Sweeps**: Added `/resolveall` to manually force-check and sweep all pending expired markets at once, freeing up capital faster.
* **Log Maintenance**: Added `/clearlog` to easily purge temporary execution history and keep the chat interface clean.

###  v2.2 — Navigation & Stability (Mar 20, 2026)
* **Hybrid Menus**: Implemented a persistent bottom menu alongside the inline buttons for faster, more intuitive navigation.
* **Quick Access**: Added the `/home` command to instantly return to the main dashboard from any sub-menu.
* **Backend Optimizations**: Refined data pipelines and API request handling for faster, more stable market polling.

###  v2.3 — Interface Streamlining & Edge Cases (Mar 21, 2026)
* **Zero-Clutter UI**: Completely stripped out the bulky bottom-menu keyboards and the `/home` command. The bot is now 100% driven by a clean, seamless Inline Keyboard interface.
* **Advanced Resolution Routing**: Hardcoded specific edge-case resolution rules to match Polymarket's exact settlement sources (e.g., Hong Kong routing to the HKO climate page instead of an airport, and Taipei dynamically routing between CWA and NOAA/RCTP).
* **Expanded Roster**: Verified and mapped new international resolution stations including Buenos Aires, Miami, Chicago, Ankara, and Los Angeles.
