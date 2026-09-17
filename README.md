# DRAM-Breakdown

A single-page dashboard for tracking the daily price attribution of the **Roundhill Memory ETF (CBOE: DRAM)**. It fetches live prices for each underlying holding, calculates each position's contribution to the fund's daily move, and presents the results in a clean, mobile-friendly layout.

Live at: **https://coreycousins.github.io/DRAM-Breakdown/DRAMInvestmentBreakdown.html**

Learn the market: **https://coreycousins.github.io/DRAM-Breakdown/DRAMMarketPrimer.html** — a plain-language primer on memory chips, the AI-driven supercycle, and how this fund is built, for anyone newer to the space.

---

## Glossary

A few terms that come up throughout this page and dashboard, for anyone newer to the memory-chip market or fund mechanics:

| Term | Meaning |
|---|---|
| **DRAM** | Dynamic Random-Access Memory — the fast, short-term memory chip in every computer, phone, and server that temporarily holds data the processor is actively using. Loses its contents when powered off. |
| **NAND** | A type of flash memory that stores data permanently (unlike DRAM) — the chip inside SSDs, USB drives, and phone storage. |
| **HBM** | High-Bandwidth Memory — a stacked, high-speed variant of DRAM built specifically for AI accelerators (like Nvidia GPUs), where moving huge amounts of data quickly is the bottleneck. Currently the highest-margin, fastest-growing corner of the memory market. |
| **NOR flash** | An older, slower flash memory type still used where reliability matters more than speed — automotive, industrial, and embedded electronics. |
| **Fabless** | A chip company (e.g. GigaDevice, Phison) that designs chips but outsources actual manufacturing to a foundry, rather than owning its own fabrication plants. |
| **ADR** | American Depositary Receipt — a US-traded certificate representing shares of a foreign company (e.g. SK hynix's `SKHY`), letting US investors trade it without a foreign brokerage account. |
| **NAV** | Net Asset Value — the fund's total assets minus liabilities, divided by shares outstanding; effectively the fund's "fair value" per share, which its market price tracks closely for an ETF. |
| **Expense ratio** | The fund's annual management fee, expressed as a % of assets (DRAM's is 0.65%/year), deducted automatically from NAV rather than billed separately. |
| **Total-return swap** | A derivative contract where the fund receives the full return (price moves + dividends) of a stock from a counterparty (here, Goldman Sachs — the "GS" in the swap tickers), without directly owning the shares. Lets the fund get equity-like exposure to hard-to-access or restricted stocks. |
| **Notional exposure** | The economic size of a swap position — i.e. how much stock-price movement the fund is exposed to — as opposed to how much cash it actually put up for it. |
| **Collateral** | Assets (here, US Treasury bills) posted to back a swap obligation, in case the fund owes money to its swap counterparty. This is why DRAM holds so many T-bills despite being a memory-stock fund. |
| **Modified market-cap weighting** | A weighting scheme based on company size (bigger companies get bigger weights) but with adjustments — here, a 25% cap on any single issuer — to avoid excessive concentration. |

---

## How to Use It

### Summary Strip

At the top of the page, a summary strip shows:

| Field | Description |
|---|---|
| **Est. Fund Move** | Sum of all weighted contributions from holdings with live prices |
| **Coverage** | Percentage of fund weight currently covered by live or entered prices — can exceed 100% (see *How DRAM uses swaps* below) |
| **Top Contributor** | Holding with the largest positive contribution today |
| **Top Drag** | Holding with the largest negative contribution today |
| **Fund NAV** | Live DRAM price from Yahoo Finance |
| **Reported Move** | DRAM's own daily % change, per Yahoo Finance |

### Holdings Table

Each row shows one holding with:
- **Ticker** — exchange-standard symbol (e.g. `MU US`, `005930 KS`)
- **Issuer** — company name, with the Yahoo Finance ticker in brackets (e.g. `[MU]`)
- **Weight** — consolidated economic exposure as a percentage of fund NAV
- **Day %** — today's price change for the holding
- **Sector** — sector classification for the holding (`DRAM/NAND`, `Storage/Controller`, `Unlisted`, `Cash & Collateral`)
- **Contrib** — weighted contribution to the fund's daily move (`weight × day%`)
- **Bar** — visual bar scaled relative to the largest contributor today (minimum scale: 0.10%)

Rows are colour-coded: green for positive contributors, amber for positions requiring manual entry.

### Manual Entry (Amber Rows)

CXMT Corporation (a private Chinese DRAM maker with no public ticker) and the fund's Cash & T-Bill Collateral position show an input field instead of a live price. Type a value in `±0.00%` format (e.g. `+0.50%` or `-1.20%`) and it will be included in the estimated fund move and coverage calculations immediately.

### Holding Detail Popup

Click any row to open a popup with a description of that holding and a **Recent News** link that opens a Google News search for that company in a new tab.

### Refresh

Click **↻ Refresh Prices** in the header to re-fetch all live prices. The status indicator shows how many holdings loaded successfully and the time of the last update.

### Debug Panel

A collapsed debug panel at the bottom of the page logs each fetch attempt. Expand it to see which tickers loaded, which failed, and the specific error for any failures (e.g. `FAIL XYZ: HTTP 404`).

---

## How DRAM Uses Swaps

Unlike a plain equity basket, DRAM gets most of its memory-stock exposure through **total-return swaps** rather than holding shares outright. Each major holding (Samsung, Micron, SK hynix, CXMT) shows up in the fund's disclosed holdings as up to three separate line items:

1. A direct share or ADR position
2. A "swap NM" (notional) line referencing the same company
3. A "swap Gold-L" line, also referencing the same company

All three move with the same underlying stock price, so this dashboard **consolidates them into a single row per issuer**, summing the weights. This is the economically meaningful view for attribution — it tells you how much the fund's NAV moves for a 1% move in Samsung, not how the position is structured internally.

Because swap notional exposure sits on top of the US Treasury bills that collateralize it, the fund's total disclosed weight comes to **~125% of NAV** rather than 100%. This is expected, not a data error — it means DRAM has roughly 125% notional exposure to memory stocks, backed by ~25% of NAV held in T-bill collateral. The Coverage number in the summary strip is allowed to exceed 100% for this reason; only the visual progress bar clamps its fill width at 100%.

---

## Technical Details

### Architecture

The entire application is a single self-contained HTML file (`DRAMInvestmentBreakdown.html`) with no build system, no dependencies, and no backend. Everything runs in the browser.

### Holdings Data

Holdings are defined as a static JavaScript array near the top of the `<script>` block. Each entry has:

```js
{ ticker: "MU", display: "MU US", issuer: "Micron Technology, Inc.", weight: 0.2534, manual: false, sector: { label: "DRAM/NAND", cls: "memory" }, desc: "..." }
```

- `ticker` — Yahoo Finance symbol used for price fetching (null for manual positions)
- `display` — label shown in the table
- `weight` — consolidated portfolio weight as a decimal (e.g. `0.2534` = 25.34% of NAV)
- `manual` — if `true`, shows a text input instead of fetching a price
- `sector` — object with `label` (display text) and `cls` (`memory` | `storage` | `cash` | `unlisted`) for colour-coding
- `desc` — description shown in the holding detail popup

Source data as of 2026-09-16 from stockanalysis.com/etf/dram/holdings/, cross-checked against roundhillinvestments.com/etf/dram/ and tipranks.com/etf/dram/holdings.

### Price Fetching (Yahoo Finance)

Live prices are fetched from the **Yahoo Finance v8 chart API**:

```
https://query1.finance.yahoo.com/v8/finance/chart/{ticker}?interval=1d&range=1d
```

Because the page is served from a different origin (GitHub Pages), direct requests are blocked by CORS. Requests are routed through a self-hosted **Cloudflare Worker** first, falling back to public CORS proxies (corsproxy.io, allorigins.win, codetabs.com) if the worker is unavailable.

The proxy returns the raw Yahoo JSON. The relevant fields extracted are:
- `meta.regularMarketPrice` — current price
- `meta.chartPreviousClose` — prior close (fallback: `meta.previousClose`)

Daily % change is calculated as `(price - prev) / prev * 100`.

All holdings are fetched in parallel via `Promise.allSettled()`. Failures are caught gracefully — the holding shows `N/A` and the error is logged to the debug panel.

### Fund NAV

Unlike a Canadian mutual fund ticker that isn't carried by Yahoo Finance, **DRAM trades directly on CBOE and is fully covered by Yahoo**. The fund's own NAV and daily % change are fetched with the exact same Yahoo v8 chart API call used for every individual holding — no HTML scraping is needed.

### Attribution Calculation

For each holding with a known day %:

```
contribution = weight × dayPct
```

The estimated fund move is `Σ(contribution)` across all covered holdings. Coverage is `Σ(weight)` of covered holdings, expressed as a percentage of NAV — this can exceed 100% because of the swap notional exposure described above.

### Bar Chart Scaling

Bar widths are scaled relative to the largest absolute contributor on the current day:

```js
const barScale = Math.max(maxAbsContrib, 0.10);
barWidth = Math.abs(contrib) / barScale * 100   // capped at 100%
```

The 0.10% floor prevents flat days from exaggerating small moves — without it, a 0.001% contribution would still fill 100% of the bar if it happened to be the day's biggest.

### Rendering

`render()` is called after every fetch and after every manual input change. It rebuilds the holdings table from scratch each time, then applies bar widths in a second pass once the maximum contribution is known.
