# Polymarket Sports Edge Finder

A browser-based tool that finds value bets across 8 sports, 30+ leagues and cups on Polymarket. Uses Expected Value (EV) + Kelly Criterion to size bets, compares your model's probability estimates against Polymarket market prices, and lets you follow the top traders.

**Live demo:** deploy to GitHub Pages in ~2 minutes (see below).

---

## Features

- 8 sports: Soccer, Basketball, Tennis, American Football, Baseball, Rugby, Cricket, MMA
- 30+ leagues & cups (UCL, EPL, La Liga, NBA, ATP, NFL, UFC, IPL, Six Nations, and more)
- Market types: Match winner, BTTS, Over/Under, Handicap, Anytime Scorer, First Goal Scorer, Tournament Winner
- Date filters: Today / Tomorrow / This week / All
- EV calculator + Kelly Criterion bet sizing
- Wisdom of crowds: divergence analysis vs Polymarket consensus
- Top traders: copy the sharpest money on the platform
- Model explainer: exactly how probabilities are calculated

---

## Deploy to GitHub Pages (2 minutes)

### Option 1: Upload via GitHub web UI

1. Go to [github.com](https://github.com) and click **New repository**
2. Name it `polymarket-edge-finder`, set to **Public**
3. Click **uploading an existing file**, drag in `index.html`
4. Commit the file
5. Go to **Settings → Pages → Branch: main → / (root) → Save**
6. Your tool is live at `https://YOUR-USERNAME.github.io/polymarket-edge-finder`

### Option 2: Git CLI

```bash
git clone https://github.com/YOUR-USERNAME/polymarket-edge-finder
cd polymarket-edge-finder
cp /path/to/index.html .
git add index.html
git commit -m "Initial deploy"
git push origin main
# Then enable GitHub Pages in repo Settings → Pages
```

---

## Connect Live Polymarket Data (optional)

The tool ships with sample data. To get real live market prices:

### Step 1 — Fetch from Polymarket Gamma API (free, no key)

```js
// Add this to a new file: js/live.js
async function fetchLiveMarkets() {
  const res = await fetch(
    'https://gamma-api.polymarket.com/markets?active=true&tag=sports&limit=100'
  );
  const data = await res.json();
  return data.map(m => ({
    id: m.id,
    match: m.question,
    mktP: parseFloat(m.outcomePrices[0]),  // YES price = implied probability
    vol: parseFloat(m.volume),
    market: m.question,
    sport: inferSport(m.tags),
    // ... map other fields
  }));
}
```

### Step 2 — Add your model's probability estimates

```js
// js/model.js
// Replace ourP with your own probability estimate for each market.
// Free data sources:
//
// Soccer Elo:    https://clubelo.com/API
// Soccer xG:     https://fbref.com  (scrape or use statsbomb open data)
// Tennis Elo:    https://www.tennisabstract.com/blog/2012/03/19/building-a-tennis-elo-system/
// NBA stats:     https://www.basketball-reference.com
// NFL:           https://www.pro-football-reference.com
// Betfair odds:  https://api.betfair.com (free tier available)

async function getModelProbability(matchId, sport) {
  // Your model logic here
  // Return a number between 0 and 1
}
```

### Step 3 — Auto-refresh

Add this to `index.html` before `</body>`:

```html
<script>
  // Refresh every 5 minutes
  setInterval(() => refreshAll(), 5 * 60 * 1000);
</script>
```

---

## Customise Model Weights

Edit the `CONFIG` object at the top of `index.html`:

```js
const CONFIG = {
  MODEL_WEIGHTS: {
    elo: 0.30,       // Elo rating system
    stats: 0.30,     // xG / statistical model
    betfair: 0.25,   // Betfair exchange prices
    crowd: 0.10,     // Polymarket crowd
    h2h: 0.05        // Head-to-head / venue
  },
  KELLY_FRACTION: 0.25,   // Use 25% Kelly (safer than full Kelly)
  BANKROLL: 1000,
};
```

---

## Kelly Criterion — quick reference

**Full Kelly:** `f* = (b·p − q) / b`  
**Fractional Kelly:** multiply by your fraction (default 25%)

- `p` = your estimated win probability
- `q` = 1 − p
- `b` = decimal odds − 1 (e.g. odds of 2.0 → b = 1.0)

Most sharp bettors use 20–33% Kelly to reduce variance while keeping most of the long-run growth benefit.

---

## Expected Value — quick reference

`EV = (p × b) − (1 − p)`

- Positive EV means the bet is mathematically profitable *if your probability estimate is correct*
- The edge is only as good as your model — bad probability estimates = bad EV calculations

---

## Disclaimer

This tool is for informational and educational purposes. Betting involves real financial risk. Past performance of any model or trader does not guarantee future results. Always bet responsibly and within your means.

---

## Roadmap ideas

- [ ] Connect Polymarket WebSocket for real-time price updates
- [ ] Back-test model accuracy against resolved markets
- [ ] Add email/Telegram alerts for high-EV opportunities
- [ ] Portfolio tracker — track your open bets and P&L
- [ ] Dark/light mode toggle

---

## License

MIT — do whatever you want with it.
