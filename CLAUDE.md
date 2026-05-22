# Memory & AI Cycle Dashboard

A static single-page dashboard tracking the AI-driven memory semiconductor cycle. Watches memory makers (Micron, SK Hynix, Samsung, SanDisk), the HBM supply chain (TSMC, Amkor, Advantest), AI demand engines (NVIDIA, AMD, Broadcom, Marvell, Nokia), and hyperscaler capex (MSFT, GOOG, META, AMZN).

This dashboard is independent research — explicitly **not investment advice**. Has a prominent disclaimer at the top.

## Stack

- Vanilla HTML / CSS / JS — no frameworks, no build pipeline beyond a single `sed` substitution at deploy time
- Self-contained `index.html` (~1700 lines, ~64KB)
- Google Fonts (Space Grotesk for display, JetBrains Mono for numerics)
- Finnhub free-tier API for live stock prices

## Deploy

- **Hosted on Cloudflare Workers (Static Assets)**, connected to this GitHub repo. The project used to be on Cloudflare Pages — CF rebranded the product, same underlying infra. The failing CI check named `Workers Builds: ai-buildout` is the CF deploy.
- Cloudflare auto-deploys on every push to `main`
- Production branch: `main`
- Worker config lives in `wrangler.jsonc` at the repo root (committed) — declares the whole repo as the static asset directory
- Build command (configured in Cloudflare dashboard → Workers → ai-buildout → Settings → Build):
  ```
  sed -i "s|__FINNHUB_KEY_PLACEHOLDER__|$FINNHUB_KEY|g" index.html
  ```
  This is *not* in `wrangler.jsonc` — `wrangler.jsonc` only configures the Worker runtime, not preprocessing. If the dashboard build command ever gets unset, live prices break and the page silently falls back to snapshot mode.
- `FINNHUB_KEY` env var must be set in Cloudflare dashboard → Variables and Secrets (Production scope)

**Important:** Never commit the actual Finnhub key to this repo. Only the placeholder `__FINNHUB_KEY_PLACEHOLDER__` should appear in `index.html`. Cloudflare substitutes the real key at build time.

Local dev with the CF stack: `wrangler dev` will serve the site locally using the committed `wrangler.jsonc`. `wrangler deploy` deploys directly (bypasses GitHub).

## Data layers

| Layer | Refresh model |
|---|---|
| US-listed stock prices | Live, via Finnhub `/quote` on every page load. Free tier: 60 calls/min. |
| OTC stock prices (HXSCL, SSNLF, ATEYY) | Static — Finnhub free tier doesn't reliably cover OTCs. Flagged `skipLiveFetch: true` in the `stocks` array. |
| Earnings Pulse (last quarter actuals) | Static — manually updated in the `earnings` JS array |
| Earnings Preview (consensus estimates) | Static — manually updated in the `upcomingEarnings` JS array |
| Cycle stage, signals, status banner | Static — editorial judgment, in JS / HTML |
| Catalyst calendar | Static — in HTML |

Live refresh degrades gracefully: if the key isn't substituted or any fetch fails, the page falls back to the snapshot values committed to the repo. The badge in the header switches from green "Live" to yellow "Snapshot" to indicate state.

## Visual identity

Retro brutalist aesthetic — think PostHog meets Allium:
- Cream paper background (`--bg-cream: #f5f0e1`) with radial-gradient dot grid
- 2px solid black borders + hard non-blurred shadows (`4px 4px 0 var(--ink)`)
- Hover state: cards "lift" — shadow grows, card translates `(-2px, -2px)`
- Space Grotesk for headlines and display copy
- JetBrains Mono for all numerics, tickers, prices, signal copy, dates
- Saturated sticker palette for tags: sky-blue (HBM), green (DRAM), yellow (NAND), pink (packaging), purple (equipment), mint (foundry), coral (GPU), teal (ASIC), lavender (hyperscaler)
- Sub-section headers rendered as small pill-shaped colored stickers
- Status banners get a black "STATUS" / "DISCLAIMER" tag floating over the top-left corner

All color variables are defined in `:root` at the top of the `<style>` block. The system uses CSS custom properties throughout so palette tweaks happen in one place.

## File structure

```
.
├── index.html                  the entire dashboard (CSS + JS inlined)
├── wrangler.jsonc              Cloudflare Workers config (static assets)
├── README.md                   user-facing project description
├── CLAUDE.md                   this file — project context for agents
└── .gitignore
```

## Conventions

- All numeric content (prices, %, dates, EPS, revenue) uses the mono font
- All "what to watch" / takeaway copy is mono, small, on a `border-top: var(--border-thin)` separator
- Sub-section grouping is done via a `category` field on stock/earnings objects + a `stockSections` array that drives `renderStocks()` / `renderEarnings()` / `renderUpcomingEarnings()`
- Status badges (`beat`/`miss`/`inline`/`upcoming`) live in the `earnings-badge` class with color variants
- Don't introduce frameworks. Don't add npm. Don't add a build step beyond the existing `sed`. The whole point is "static file, deploys instantly, no infra."

## Anti-patterns

- ❌ Real API key in committed code (only `__FINNHUB_KEY_PLACEHOLDER__`)
- ❌ Browser-side stock fetch from APIs without CORS (Yahoo direct, Stooq) — use Finnhub
- ❌ Adding npm / Vite / Webpack / etc. — this should stay a single-file static site
- ❌ Removing the disclaimer banner
- ❌ Adding tracking pixels or analytics that would require a privacy notice
