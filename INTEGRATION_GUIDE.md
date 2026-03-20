# Integration Guide — Bias Dashboard + Greeks Engine
## acidrocks0-star / nse-scanner

---

## STEP 1 — Add the files to your repo

You have two new HTML files from this session:
- `bias-dashboard.html`  — Market bias, S/R zones, OI chain for all indices
- `greeks-picks.html`    — Greeks engine, Gamma Blast detector, option picks

### 1a. Clone / open your repo locally

```bash
git clone https://github.com/acidrocks0-star/nse-scanner.git
cd nse-scanner
```

### 1b. Copy both files into your repo root (or a `pages/` folder)

```
nse-scanner/
├── index.html              ← your existing scanner
├── bias-dashboard.html     ← NEW: paste the downloaded file here
├── greeks-picks.html       ← NEW: paste the downloaded file here
└── ... rest of your files
```

If you prefer a subfolder:
```
nse-scanner/
├── index.html
└── pages/
    ├── bias-dashboard.html
    └── greeks-picks.html
```

---

## STEP 2 — Link them from your existing scanner (index.html)

Open your `index.html` and add navigation links.

### Option A — Simple nav links (quickest)

Find your existing nav/header section and add:

```html
<!-- Add inside your <nav> or <header> -->
<a href="bias-dashboard.html" class="nav-link">Bias Dashboard</a>
<a href="greeks-picks.html"   class="nav-link">Greeks + Picks</a>
```

### Option B — If your app uses a sidebar

```html
<div class="sidebar-section">
  <p class="sidebar-label">Options Tools</p>
  <a href="bias-dashboard.html">
    <span class="icon">◈</span> Bias Scanner
  </a>
  <a href="greeks-picks.html">
    <span class="icon">◈</span> Greeks Engine
  </a>
</div>
```

### Option C — If your app is a Single Page App (React/Vue/vanilla JS router)

Add routes instead of href links:

```javascript
// In your router config
{ path: '/bias',   component: 'bias-dashboard.html' },
{ path: '/greeks', component: 'greeks-picks.html'   },
```

Or if using an iframe embed approach:
```html
<iframe
  id="bias-frame"
  src="bias-dashboard.html"
  style="width:100%;height:100vh;border:none">
</iframe>
```

---

## STEP 3 — Add "back to scanner" link inside the new pages

Open `bias-dashboard.html`. Find this line near the top (~line 8):

```html
<div class="logo">acid<span>rocks</span> / nse-scanner <span style="color:var(--faint)">·</span> bias</div>
```

Replace with:
```html
<div class="logo">
  <a href="index.html" style="color:var(--muted);text-decoration:none;font-size:11px;margin-right:12px">← Scanner</a>
  acid<span>rocks</span> / nse-scanner <span style="color:var(--faint)">·</span> bias
</div>
```

Do the same in `greeks-picks.html`. Find:
```html
<div class="logo">acid<span>rocks</span> <span class="pipe">/</span> nse-scanner <span class="pipe">/</span> <span style="color:var(--gamma)">greeks</span></div>
```

Replace with:
```html
<div class="logo">
  <a href="index.html" style="color:var(--muted);text-decoration:none;font-size:11px;margin-right:12px">← Scanner</a>
  acid<span>rocks</span> <span class="pipe">/</span> nse-scanner <span class="pipe">/</span> <span style="color:var(--gamma)">greeks</span>
</div>
```

---

## STEP 4 — Wire up LIVE data (the most important step)

Right now both files use hardcoded sample values. You need to replace them
with real data. Here is exactly where and what to change.

### 4a. Locate the INDICES array in bias-dashboard.html

Search for this block (around line 450):

```javascript
const INDICES = [
  {
    id:'nifty50', name:'Nifty 50', short:'NIFTY',
    ...
    price:22547, chg:0.62, vix:14.2, pcr:1.08, maxPain:22500,
    ...
  },
  ...
```

### 4b. Locate the INDICES array in greeks-picks.html

Search for this block (around line 320):

```javascript
const INDICES = [
  {id:'nifty50', ... price:22547, chg:0.62, vix:14.2, pcr:1.08, ...dte:3},
  ...
```

### 4c. Replace with a live fetch function

Delete the hardcoded `const INDICES = [...]` block in BOTH files and
replace it with this live fetch wrapper:

```javascript
// ── LIVE DATA FETCH (replaces hardcoded INDICES) ──────────────────────
const INDEX_CONFIG = [
  { id:'nifty50',   name:'Nifty 50',      short:'NIFTY',      exchange:'NSE', lot:75,  step:50,  weeklyExpiry:true,  expiry:'Tue',     nseKey:'NIFTY 50',        chainSymbol:'NIFTY'     },
  { id:'banknifty', name:'Bank Nifty',     short:'BANKNIFTY',  exchange:'NSE', lot:30,  step:100, weeklyExpiry:false, expiry:'Monthly', nseKey:'NIFTY BANK',      chainSymbol:'BANKNIFTY' },
  { id:'finnifty',  name:'Fin Nifty',      short:'FINNIFTY',   exchange:'NSE', lot:65,  step:50,  weeklyExpiry:false, expiry:'Monthly', nseKey:'NIFTY FIN SERVICE',chainSymbol:'FINNIFTY'  },
  { id:'midcap',    name:'Midcap Nifty',   short:'MIDCPNIFTY', exchange:'NSE', lot:120, step:25,  weeklyExpiry:false, expiry:'Monthly', nseKey:'NIFTY MIDCAP SELECT',chainSymbol:'MIDCPNIFTY'},
  { id:'sensex',    name:'Sensex',         short:'SENSEX',     exchange:'BSE', lot:10,  step:100, weeklyExpiry:true,  expiry:'Thu',     nseKey:'SENSEX',           chainSymbol:'SENSEX'    },
  { id:'bankex',    name:'Bankex',         short:'BANKEX',     exchange:'BSE', lot:15,  step:100, weeklyExpiry:false, expiry:'Monthly', nseKey:'BANKEX',           chainSymbol:'BANKEX'    },
  { id:'niftynxt',  name:'Nifty Next 50',  short:'NIFTYNXT50', exchange:'NSE', lot:25,  step:100, weeklyExpiry:false, expiry:'Monthly', nseKey:'NIFTY NEXT 50',    chainSymbol:'NIFTYNXT50'},
  { id:'niftyit',   name:'Nifty IT',       short:'NIFTYIT',    exchange:'NSE', lot:25,  step:100, weeklyExpiry:false, expiry:'Monthly', nseKey:'NIFTY IT',         chainSymbol:'NIFTYIT'   },
];

// NSE requires a cookie/session — use a proxy or your existing backend
async function fetchNSEIndices() {
  try {
    // OPTION 1: If you already have an NSE proxy in your scanner backend
    const res = await fetch('/api/nse/allIndices');
    const data = await res.json();
    return data.data; // array of { indexSymbol, last, percentChange, ... }
  } catch(e) {
    console.warn('Live fetch failed, using fallback data', e);
    return null;
  }
}

async function fetchVIX() {
  try {
    const res = await fetch('/api/nse/vix'); // your proxy
    const data = await res.json();
    return parseFloat(data.last);
  } catch(e) { return 14.5; }
}

async function fetchPCR(symbol) {
  // PCR = total put OI / total call OI from option chain
  try {
    const res = await fetch(`/api/nse/optionchain?symbol=${symbol}`);
    const data = await res.json();
    const records = data.records?.data || [];
    let totalCallOI = 0, totalPutOI = 0;
    records.forEach(r => {
      if(r.CE) totalCallOI += r.CE.openInterest || 0;
      if(r.PE) totalPutOI  += r.PE.openInterest || 0;
    });
    return totalPutOI / (totalCallOI || 1);
  } catch(e) { return 1.0; }
}

function calcMaxPain(chainData) {
  // Max pain = strike where total option loss is minimised
  const records = chainData?.records?.data || [];
  const strikes = [...new Set(records.map(r => r.strikePrice))].sort((a,b)=>a-b);
  let minLoss = Infinity, maxPainStrike = strikes[0];
  strikes.forEach(targetStrike => {
    let loss = 0;
    records.forEach(r => {
      if(r.CE) loss += Math.max(0, r.strikePrice - targetStrike) * r.CE.openInterest;
      if(r.PE) loss += Math.max(0, targetStrike - r.strikePrice) * r.PE.openInterest;
    });
    if(loss < minLoss) { minLoss = loss; maxPainStrike = targetStrike; }
  });
  return maxPainStrike;
}

function calcDTE(expiryDateStr) {
  // expiryDateStr format from NSE: "25-Jan-2024"
  const expiry = new Date(expiryDateStr);
  const today  = new Date();
  return Math.max(0, Math.ceil((expiry - today) / (1000*60*60*24)));
}

async function buildLiveINDICES() {
  const [indicesData, vix] = await Promise.all([fetchNSEIndices(), fetchVIX()]);

  const INDICES = await Promise.all(INDEX_CONFIG.map(async cfg => {
    // Find this index in the allIndices response
    const live = indicesData?.find(d => d.indexSymbol === cfg.nseKey);

    // Fetch option chain for PCR + max pain
    let pcr = 1.0, maxPain = 0, callWall = 0, putWall = 0, dte = 7;
    try {
      const chainRes = await fetch(`/api/nse/optionchain?symbol=${cfg.chainSymbol}`);
      const chainData = await chainRes.json();
      const records = chainData.records?.data || [];

      // PCR
      let tCallOI = 0, tPutOI = 0;
      const oiByStrike = {};
      records.forEach(r => {
        tCallOI += r.CE?.openInterest || 0;
        tPutOI  += r.PE?.openInterest || 0;
        if(!oiByStrike[r.strikePrice]) oiByStrike[r.strikePrice] = {call:0, put:0};
        oiByStrike[r.strikePrice].call += r.CE?.openInterest || 0;
        oiByStrike[r.strikePrice].put  += r.PE?.openInterest || 0;
      });
      pcr = tPutOI / (tCallOI || 1);

      // Max pain
      maxPain = calcMaxPain(chainData);

      // Call wall = strike with highest call OI
      let maxC = 0, maxP = 0;
      Object.entries(oiByStrike).forEach(([strike, oi]) => {
        if(oi.call > maxC) { maxC = oi.call; callWall = parseFloat(strike); }
        if(oi.put  > maxP) { maxP = oi.put;  putWall  = parseFloat(strike); }
      });

      // DTE
      const expDates = chainData.records?.expiryDates || [];
      if(expDates[0]) dte = calcDTE(expDates[0]);

    } catch(e) { /* use defaults */ }

    const price = live?.last || cfg.price || 22000;
    const prevClose = live?.previousClose || price;
    const chg = live ? parseFloat(live.percentChange) : 0;

    return {
      ...cfg,
      price, prevClose, chg,
      vix,
      pcr: parseFloat(pcr.toFixed(2)),
      maxPain: maxPain || Math.round(price/cfg.step)*cfg.step,
      callWall: callWall || Math.round(price/cfg.step)*cfg.step + cfg.step*3,
      putWall:  putWall  || Math.round(price/cfg.step)*cfg.step - cfg.step*3,
      atr: Math.round(price * 0.008),
      dte,
      high: live?.dayHigh || price,
      low:  live?.dayLow  || price,
    };
  }));

  return INDICES;
}

// ── INIT: replace the synchronous render with async ───────────────────
async function init() {
  // Show loading state
  document.getElementById('pages').innerHTML =
    '<div style="padding:40px;text-align:center;font-family:var(--mono);color:var(--muted)">Fetching live data...</div>';

  const INDICES = await buildLiveINDICES();
  // ... rest of your render/nav build code using INDICES
  buildUI(INDICES);
}

init();
```

---

## STEP 5 — Set up your NSE proxy backend

NSE blocks direct browser requests (CORS). You need a small server-side
proxy. Add this to your existing backend (Node/Python/whatever you use).

### If your scanner already has a Node.js backend:

```javascript
// Add to your existing Express server (server.js / app.js)
const https = require('https');

const NSE_HEADERS = {
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0',
  'Accept': '*/*',
  'Accept-Language': 'en-US,en;q=0.9',
  'Referer': 'https://www.nseindia.com/',
  'Connection': 'keep-alive',
};

// Session cookie cache (NSE requires a cookie)
let nseSession = '';

async function getNSECookie() {
  return new Promise((resolve) => {
    const req = https.get('https://www.nseindia.com', { headers: NSE_HEADERS }, res => {
      nseSession = (res.headers['set-cookie'] || []).map(c => c.split(';')[0]).join('; ');
      resolve(nseSession);
    });
    req.on('error', () => resolve(''));
  });
}

async function nseGet(path) {
  if (!nseSession) await getNSECookie();
  return new Promise((resolve, reject) => {
    const options = {
      hostname: 'www.nseindia.com',
      path,
      headers: { ...NSE_HEADERS, Cookie: nseSession },
    };
    https.get(options, res => {
      let body = '';
      res.on('data', chunk => body += chunk);
      res.on('end', () => {
        try { resolve(JSON.parse(body)); }
        catch(e) { reject(e); }
      });
    }).on('error', reject);
  });
}

// ── ROUTES ──
app.get('/api/nse/allIndices', async (req, res) => {
  try {
    const data = await nseGet('/api/allIndices');
    res.json(data);
  } catch(e) {
    // Refresh cookie and retry once
    nseSession = '';
    try { res.json(await nseGet('/api/allIndices')); }
    catch(e2) { res.status(500).json({ error: e2.message }); }
  }
});

app.get('/api/nse/vix', async (req, res) => {
  try {
    const data = await nseGet('/api/allIndices');
    const vix = data.data?.find(d => d.indexSymbol === 'INDIA VIX');
    res.json({ last: vix?.last || 14.5 });
  } catch(e) { res.status(500).json({ error: e.message }); }
});

app.get('/api/nse/optionchain', async (req, res) => {
  const symbol = req.query.symbol || 'NIFTY';
  try {
    const data = await nseGet(`/api/option-chain-indices?symbol=${symbol}`);
    res.json(data);
  } catch(e) { res.status(500).json({ error: e.message }); }
});
```

### If your scanner uses Python (Flask/FastAPI):

```python
# Add to your existing Flask/FastAPI app

import requests
import json

NSE_HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0",
    "Accept": "*/*",
    "Accept-Language": "en-US,en;q=0.9",
    "Referer": "https://www.nseindia.com/",
}

session = requests.Session()
session.headers.update(NSE_HEADERS)

def refresh_nse_cookie():
    session.get("https://www.nseindia.com", timeout=10)

# Flask routes:
@app.route("/api/nse/allIndices")
def all_indices():
    try:
        r = session.get("https://www.nseindia.com/api/allIndices", timeout=10)
        return r.json()
    except:
        refresh_nse_cookie()
        r = session.get("https://www.nseindia.com/api/allIndices", timeout=10)
        return r.json()

@app.route("/api/nse/optionchain")
def option_chain():
    symbol = request.args.get("symbol", "NIFTY")
    try:
        url = f"https://www.nseindia.com/api/option-chain-indices?symbol={symbol}"
        r = session.get(url, timeout=10)
        return r.json()
    except Exception as e:
        return {"error": str(e)}, 500

@app.route("/api/nse/vix")
def vix():
    try:
        r = session.get("https://www.nseindia.com/api/allIndices", timeout=10)
        data = r.json()
        vix_entry = next((d for d in data["data"] if d["indexSymbol"] == "INDIA VIX"), None)
        return {"last": vix_entry["last"] if vix_entry else 14.5}
    except Exception as e:
        return {"error": str(e)}, 500
```

---

## STEP 6 — Auto-refresh during market hours

Add this to both HTML files, just before the closing `</script>` tag:

```javascript
// ── AUTO REFRESH ─────────────────────────────────────────────────────
function isMarketOpen() {
  const now = new Date(new Date().toLocaleString('en-US', {timeZone: 'Asia/Kolkata'}));
  const h = now.getHours(), m = now.getMinutes();
  const day = now.getDay(); // 0=Sun, 6=Sat
  if(day === 0 || day === 6) return false;
  const timeVal = h * 60 + m;
  return timeVal >= 9*60+15 && timeVal <= 15*60+30;
}

// Refresh every 3 minutes during market hours
setInterval(() => {
  if(isMarketOpen()) {
    init(); // re-runs the full fetch + render
  }
}, 3 * 60 * 1000);
```

---

## STEP 7 — Commit and push

```bash
# From your repo root
git add bias-dashboard.html greeks-picks.html
git commit -m "feat: add bias dashboard and greeks engine with gamma blast detector"
git push origin main
```

---

## STEP 8 — Test checklist

Run through this before going live:

- [ ] Both pages open without console errors
- [ ] Nav tabs switch between all 8 indices
- [ ] Back arrow on both pages returns to index.html
- [ ] `/api/nse/allIndices` returns JSON (test in browser/Postman)
- [ ] `/api/nse/optionchain?symbol=NIFTY` returns option chain data
- [ ] Spot prices update automatically after 09:15 IST
- [ ] Gamma blast banner appears for NIFTY/SENSEX on expiry days (Tue/Thu)
- [ ] Option pick scores change when PCR or VIX changes
- [ ] S/R canvas draws on every index page

---

## OPTIONAL — GitHub Pages deployment

If you host this on GitHub Pages (no backend), you cannot use the NSE proxy
above. Instead, use a free serverless function:

### Vercel (free tier, 5 seconds timeout)

1. Create `api/nse-proxy.js` in your repo:

```javascript
export default async function handler(req, res) {
  const { path } = req.query;
  const response = await fetch(`https://www.nseindia.com/api/${path}`, {
    headers: {
      'User-Agent': 'Mozilla/5.0 Chrome/120.0',
      'Referer': 'https://www.nseindia.com/',
      'Accept': '*/*',
    }
  });
  const data = await response.json();
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.json(data);
}
```

2. In the HTML files, change your fetch URLs to:
```javascript
const res = await fetch(`/api/nse-proxy?path=allIndices`);
const res = await fetch(`/api/nse-proxy?path=option-chain-indices%3Fsymbol%3DNIFTY`);
```

3. Deploy: `vercel --prod`

---

## Quick reference — what each file does

```
bias-dashboard.html
  ├── Overview page        All 8 indices at a glance, click to drill down
  ├── Per-index pages      Bias verdict + buying/selling zones + S/R map + OI chain
  └── Calculator page      Manual input → instant bias + zone computation

greeks-picks.html
  ├── Overview page        Gamma score for all indices, condition count badges
  ├── Per-index pages
  │   ├── Condition banners  Gamma blast / Delta run / Vega expansion / Theta crush
  │   ├── Greek meters       Delta, Gamma, Theta, Vega, IV — live gauges
  │   ├── Condition matrix   6 conditions, Active/Forming/Inactive
  │   ├── Option picks       Best call + put strike with full greeks + targets
  │   ├── Gamma blast panel  When active: expected move, triggers, strategy
  │   ├── Greeks skew table  IV, gamma, vega, theta across ±5 strikes
  │   └── Delta/Gamma chart  Visual profile across strikes
  └── Black-Scholes engine  erf() → normCDF() → full BS pricing, client-side
```

---

*Generated for acidrocks0-star/nse-scanner — March 2026*
