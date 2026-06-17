# CZ Electricity SPOT Price Dashboard

Single-file HTML dashboard for Czech SPOT electricity prices in 15-minute slots (today + tomorrow).
Built for **ČEZ Distribuce** area, tariff **D57d**.

**Live app:** https://mirrah-v.github.io/EU-CZ-Electricity-Spot-price-calculator/
**Working file:** `w:\electricity-prices-dashboard.html` (copied to `index.html` before each commit)

---

## Price Formulas

### BUY
`( SPOT + SpotFee ) × 1.21 + DistributionFee`
- SPOT: raw quarter-hourly exchange price (CZK/MWh, excl. VAT)
- SpotFee: user-defined distributor spot fee (default 250 CZK/MWh, excl. VAT)
- VAT: 21%, applied to SPOT + SpotFee
- DistributionFee: regulated ČEZ D57d rate, already incl. VAT (added after VAT step)

### SELL
`SPOT − SpotFee`
- No VAT (prosumer/private seller, not VAT-registered)
- No distribution fee (borne by the consumer, not the producer)
- SpotFee is deducted (distributor keeps it for handling the trade)

---

## Distribution Cost Constants — ČEZ D57d 2026 Regulated Rates

| Tariff Band | Incl. VAT | Excl. VAT |
|---|---|---|
| High tariff (VT) | 913.27 CZK/MWh | 754.77 CZK/MWh |
| Low tariff (NT) | 140.97 CZK/MWh | 116.50 CZK/MWh |

Constants are stored **incl. VAT** in code (`DIST_HIGH_VAT`, `DIST_LOW_VAT`) and are added **after** the `× 1.21` step — they are never VAT-multiplied again.

**Low tariff (NT) hours:** 00:00–08:00, 09:00–12:00, 13:00–15:00, 16:00–19:00, 20:00–24:00
**High tariff (VT) hours** (4 × 1-hour windows): 08:00–09:00, 12:00–13:00, 15:00–16:00, 19:00–20:00

---

## Header Controls

| Control | Default | Function |
|---|---|---|
| BUY / SELL glass pill | BUY | Switches price formula for all cards and charts |
| Light / Dark toggle | Syncs to OS | Light = left, Dark = right |
| CZ / EN toggle | EN | Full Czech/English translation |
| CZK / EUR toggle | CZK | Converts all prices; ⓘ shows live CNB rate |

---

## Stat Cards

| Card | BUY | SELL |
|---|---|---|
| Current Price | (spot + fee) × 1.21 + dist | spot − fee |
| Cheapest Hour | cheapest effective BUY slot | cheapest effective SELL slot |
| Most Expensive | most expensive BUY slot | most expensive SELL slot |
| Daily Consumption / Production | kWh input, spread across 96 slots | same, labelled "Production" |
| Estimated Cost / Revenue | consumption × effective price | production × effective price |
| Price Range | min–max effective price today | min–max effective price today |
| Low Price Slots | count of NT slots out of 96 | same |
| Tomorrow vs Today | % delta in daily average effective price | same |
| Distributor Spot Fee | user-defined CZK or EUR/MWh, excl. VAT | same, deducted instead of added |

---

## Bar Charts

- **Today** chart: "Low Tariff Slots" toggle in the header (top-right) highlights NT bands
- **Tomorrow** chart: shows next day's data when available (typically after ~14:00 CET)
- Tooltips: hover (desktop) / tap to pin (mobile); tap elsewhere to dismiss
- Y-axis always anchored at 0 for positive prices; headroom buffer = max(300, spotFee) CZK
- Switching BUY/SELL rescales the axis and redraws all bars

---

## APIs

| Source | URL | Purpose |
|---|---|---|
| Spotová elektřina — current | https://spotovaelektrina.cz/api/v1/price/get-actual-price-czk | Live spot price (CZK) |
| Spotová elektřina — slots | https://spotovaelektrina.cz/api/v1/price/get-prices-json-qh | Today + tomorrow 15-min data |
| CNB exchange rate | https://www.cnb.cz/cs/financni_trhy/devizovy_trh/kurzy_devizoveho_trhu/denni_kurz.xml | Daily EUR/CZK rate |

CNB rate: 3-source fallback chain (direct → corsproxy.io → allorigins.win), 5s timeout each. Fallback: **24.50 CZK/EUR**.

---

## Technology

- Pure single-file HTML — no framework, no build step, no dependencies
- Vanilla JS + CSS (backdrop-filter glass effects, CSS Grid layout)
- PWA: `manifest.json` + `sw.js` service worker — installable as Android home screen app
- Icons: `icons/icon-192.svg` and `icons/icon-512.svg` (dark blue + yellow lightning bolt)
- Responsive: desktop (20-col grid), tablet, mobile portrait (2-col card grid), landscape

---

## Key Functions

| Function | What it does |
|---|---|
| `getEntryEffectivePriceCZK(entry)` | Core price formula — BUY or SELL depending on `isSellMode` |
| `updateCurrentPrice()` | Recalculates and renders the Current Price card |
| `renderChart(id, axisId, data, showTime)` | Draws bar chart + Y-axis for a given dataset |
| `updateCards(today, tomorrow)` | Updates all stat cards |
| `calculateCost()` | Computes estimated cost/revenue from consumption input |
| `reRenderAll()` | Full redraw — call after currency or language change |
| `reRenderCharts()` | Chart-only redraw — call after spot fee or low tariff toggle |
| `applyLanguage()` | Applies all EN/CZ strings to the DOM |
| `setMode(mode)` | Switches BUY/SELL, slides pill, calls reRenderAll() |
| `isLowTariff(hour, minute)` | Returns true if the given time falls in an NT band |

**Key state variables:** `isSellMode`, `isEUR`, `isDark`, `isEN`, `spotFeeCZK`, `EUR_TO_CZK`

**Distribution constants** (top of script): `DIST_HIGH_VAT`, `DIST_HIGH_NOVAT`, `DIST_LOW_VAT`, `DIST_LOW_NOVAT`

**Translations** (`LANG` object): `en` and `cz` keys covering all UI strings, applied via `applyLanguage()`.

---

## Development Workflow

1. Edit `w:\electricity-prices-dashboard.html`
2. `cp w:/electricity-prices-dashboard.html /tmp/elec-repo/index.html`
3. `cd /tmp/elec-repo && git add index.html && git commit -m "..." && git push`
4. GitHub Pages auto-deploys; PWA on phone updates on next visit/refresh
