# Robotics Supply Chain Map

Interactive robotics supply chain explorer — industrial robots, cobots, humanoids.

---

## Quick Fix: CORS Error on GitHub Pages

If you see:
```
blocked by CORS policy: No 'Access-Control-Allow-Origin' header
```

**The Worker is crashing before it can set CORS headers.** Cloudflare returns its own
error page (no CORS headers) instead of your Worker's response, so the browser reports
it as a CORS error — but the real problem is inside the Worker.

**Most common causes:**

| Cause | Symptom in Worker logs | Fix |
|---|---|---|
| `FINNHUB_KEY` secret not added | `Cannot read env.FINNHUB_KEY` | Add the secret (Step 3 below) |
| Old Worker code without try/catch | Any runtime error | Redeploy `worker.js` from this repo |
| Worker never deployed | 404 from Cloudflare | Deploy the Worker (Step 2 below) |

**How to see the real error:**
Cloudflare dashboard → Workers → your worker → **Logs** tab → click a failed request.
You'll see the actual exception message.

---

## Setup (5 minutes)

### Step 1 — Get a free Finnhub key

1. [finnhub.io/register](https://finnhub.io/register) — free, no credit card
2. Copy your API key from the dashboard

---

### Step 2 — Deploy the Worker

1. [workers.cloudflare.com](https://workers.cloudflare.com) → Create Worker
2. Copy the entire contents of **`worker.js`** (in this repo) into the editor
3. Click **Save and Deploy**
4. Note your Worker URL: `https://robotics-proxy.yourname.workers.dev`

> The `worker.js` in this repo wraps everything in try/catch so CORS headers are
> **always** returned — even if the Worker crashes — so you see real errors instead
> of misleading CORS messages.

---

### Step 3 — Add your Finnhub key as an encrypted secret

Worker dashboard → **Settings → Variables and Secrets → Add variable**

| Field | Value |
|---|---|
| Variable name | `FINNHUB_KEY` |
| Value | your Finnhub API key |
| Type | **Secret** ← important, encrypts it |

Click **Deploy** after saving.

**Verify it worked:** Go to Worker → **Logs**, then click your ticker in the browser.
You should see a 200 response. If you see `FINNHUB_KEY secret not configured` the
secret wasn't saved — try again.

---

### Step 4 — Set your Worker URL in the HTML

Open `robotics-supply-chain.html`, find line ~1296:

```js
var WORKER_URL = '';  // ← e.g. 'https://robotics-proxy.yourname.workers.dev'
```

Replace with your Worker URL:

```js
var WORKER_URL = 'https://robotics-proxy.yourname.workers.dev';
```

Safe to commit — it's just a URL, not a secret.

---

### Step 5 — Push to GitHub Pages

```bash
git add robotics-supply-chain.html worker.js README.md
git commit -m "fix: add worker and set WORKER_URL"
git push
```

---

## Troubleshooting

| Error | Real cause | Fix |
|---|---|---|
| CORS blocked | Worker crashing (no CORS headers on error) | Check Worker Logs for real error |
| `FINNHUB_KEY secret not configured` | Secret not added | Redo Step 3 |
| "No data found for XYZ" | `quote.c` is null | Symbol not on Finnhub; try without exchange suffix |
| Stats all `—`, price shows | Market closed | Normal — `c:0` falls back to previous close |
| 429 Too Many Requests | Rate limit | Worker fans out internally; wait 60s |

---

## Security

- `WORKER_URL` in HTML → safe to commit, just a URL
- `FINNHUB_KEY` → encrypted in Cloudflare, never in code or Git
- To restrict to your domain only, change in `worker.js`:
  ```js
  'Access-Control-Allow-Origin': 'https://yourusername.github.io'
  ```

Cloudflare free tier: 100k requests/day · Finnhub free tier: 60 calls/min
