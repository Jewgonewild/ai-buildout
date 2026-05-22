# Memory & AI Cycle Dashboard

A single-page dashboard tracking the memory semiconductor cycle, HBM supply chain, hyperscaler capex signals, and AI-driven demand. Stocks watched include Micron, SK Hynix, Samsung, TSMC, NVIDIA, Amkor, Advantest, SanDisk, and Nokia.

**Live site:** `https://<your-username>.github.io/<your-repo>/` (set after deploy)

> **Disclaimer.** This is a personal research dashboard. It is **not investment advice**, not a recommendation to buy or sell securities, and not financial guidance. Stock prices and figures are auto-fetched from public sources and may be delayed or inaccurate. Verify everything on your broker before making any decision. The author holds no liability for any actions taken based on this content.

---

## What's in here

The dashboard renders entirely client-side as one self-contained `index.html` file (CSS, JS, and all data inlined). It contains:

- A cycle-stage indicator showing where the memory market sits in its 6-stage boom/bust pattern.
- A stock watchlist with live prices fetched from [Finnhub](https://finnhub.io)'s free tier.
- An 11-signal cycle-turn dashboard (hyperscaler capex, NVIDIA forward demand, spot DRAM rate-of-change, inventory weeks, etc.) with a traffic-light status.
- A sell-discipline decision framework (trim 25%, trim 50%, exit triggers).
- An upcoming catalysts calendar (earnings dates and macro events).
- A curated resource list — primary data sources, Korean/Chinese/Asia press, X follows.

## How the live prices work

On page load, the JS calls Finnhub's `/quote` endpoint once per US-listed ticker and updates the cards with the current price plus the daily percent change. OTC tickers (Samsung SSNLF, SK Hynix HXSCL, Advantest ATEYY) are flagged `skipLiveFetch` because Finnhub's free tier doesn't reliably cover them; they keep the snapshot values committed to the repo.

If the API key is missing or any fetch fails, the page silently falls back to the snapshot values and shows a yellow "Snapshot" badge in the header. The page never breaks.

## Deploy (GitHub Pages)

The repo is wired for GitHub Pages with a build-time API key injection workflow. The key is stored as a repo secret and substituted into `index.html` when the workflow runs — so it's kept out of git history. (Note: the substituted key is still readable in the deployed page's view-source. If you need real secrecy, route the API call through a serverless proxy like Cloudflare Workers instead.)

### One-time setup

1. **Get a free Finnhub API key.** Sign up at [finnhub.io](https://finnhub.io) — takes about a minute.

2. **Fork or push this repo to GitHub.** Make sure the repo is public if you want to use the free Pages tier.

3. **Add the API key as a repo secret.**
   - Go to **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `FINNHUB_KEY`
   - Value: your Finnhub API key
   - Save.

4. **Enable Pages with the GitHub Actions source.**
   - Go to **Settings → Pages**
   - Under **Source**, choose **GitHub Actions**
   - Save.

5. **Push to `main`.** The workflow at `.github/workflows/deploy.yml` runs automatically. It substitutes the key into `index.html`, then deploys to Pages. Watch progress in the **Actions** tab. First deploy usually takes 1–2 minutes.

6. **Visit your site.** The Pages settings page shows the URL after deploy completes (typically `https://<your-username>.github.io/<your-repo>/`).

### Updating

Any push to `main` triggers a redeploy. To refresh the snapshot data (the fallback values shown when the API is unavailable), edit the `stocks` array and the `signals` array near the bottom of `index.html`, commit, and push.

## Local development

Just open `index.html` in any browser. The page will detect the unsubstituted placeholder and fall back to snapshot mode automatically.

If you want to test live mode locally, copy the file to `index.local.html` (which is already gitignored), paste your key into the `FINNHUB_KEY` constant, and open that copy in a browser.

## File layout

```
.
├── index.html                       # the entire dashboard, self-contained
├── .github/
│   └── workflows/
│       └── deploy.yml              # build-time key injection + Pages deploy
├── .gitignore
└── README.md
```

## Security notes

- The Finnhub free key is rate-limited to 60 calls/minute per key. Worst-case abuse on a public site is your account getting briefly throttled — not financial loss.
- The deployed page contains the API key in its source. Anyone who views source on the live URL can extract it. If you'd prefer the key to be truly hidden, swap the direct Finnhub call for a request to a Cloudflare Worker (or similar serverless function) that holds the secret server-side.
- If a key ever leaks publicly, regenerate it in the Finnhub dashboard and update the `FINNHUB_KEY` secret in GitHub.

## License

This is a personal research project. Code is provided as-is. Feel free to fork and adapt for your own use; please don't repackage the editorial content (signals, decision framework, commentary) without credit.
