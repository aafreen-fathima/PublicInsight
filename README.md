# PublicInsight

**Turn public noise into product signal.**

PublicInsight scans Reddit threads, Glassdoor reviews, App Store complaints, and Product Hunt comments for any product — and returns clustered themes, classified signals (opportunity / risk / gap), and concrete feature ideas. Everything runs in a single HTML file, no backend required.

---

## Demo

Type a product name (Notion, Figma, Linear, Slack...), pick a data source, click Analyze. Get structured intelligence in ~3 seconds.

---

## Quick Start

### Option 1 — Open directly in browser (simplest)

```bash
git clone https://github.com/yourname/publicinsight.git
cd publicinsight
open publicinsight.html   # macOS
# or just double-click the file in Finder / File Explorer
```

No install needed. The file is self-contained.

### Option 2 — Serve locally (recommended for development)

```bash
# Python (built-in)
python3 -m http.server 3000

# Node.js (proxy server with env support)
node server.js

# Then open http://localhost:3000/
```

### Option 3 — Deploy to static hosting

Upload `publicinsight.html` to any static host:

| Platform | Command / Method |
|----------|-----------------|
| Netlify | Drag & drop the file at netlify.com/drop |
| Vercel | `vercel --prod` (single file deploy) |
| GitHub Pages | Push to repo, enable Pages in Settings |
| Cloudflare Pages | Connect repo or upload via dashboard |
| AWS S3 | `aws s3 cp publicinsight.html s3://your-bucket/ --acl public-read` |

---

## API Key Setup

PublicInsight calls the Anthropic API directly from the browser. You need an API key from [console.anthropic.com](https://console.anthropic.com).

**For local use:** The app sends the key with each request — no storage, no config file needed. You can hardcode it in the HTML for personal use:

```html
<!-- In publicinsight.html, find the fetch call and add the header -->
headers: {
  "Content-Type": "application/json",
  "x-api-key": "sk-ant-YOUR-KEY-HERE",
  "anthropic-version": "2023-06-01"
}
```

**For production:** Never expose your API key in a public deployment. Use a proxy instead — see the [Production Deployment](#production-deployment) section below.

---

## Production Deployment

For a public-facing deployment, you need a thin backend proxy to keep your API key secret.

### Option A — Cloudflare Worker (free tier, zero cold starts)

```js
// worker.js
export default {
  async fetch(request) {
    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Headers": "Content-Type",
          "Access-Control-Allow-Methods": "POST"
        }
      });
    }

    const body = await request.json();
    const response = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "x-api-key": env.ANTHROPIC_API_KEY,  // set in Worker env vars
        "anthropic-version": "2023-06-01"
      },
      body: JSON.stringify(body)
    });

    const data = await response.json();
    return new Response(JSON.stringify(data), {
      headers: { "Content-Type": "application/json", "Access-Control-Allow-Origin": "*" }
    });
  }
};
```

Deploy: `wrangler deploy worker.js`
Set secret: `wrangler secret put ANTHROPIC_API_KEY`

Then update the fetch URL in `publicinsight.html`:
```js
const resp = await fetch("https://your-worker.workers.dev", { ... });
```

### Option B — Express proxy (Node.js)

```bash
npm install express cors node-fetch dotenv
```

```js
// server.js
require("dotenv").config();
const express = require("express");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static("."));  // serves publicinsight.html

app.post("/api/analyze", async (req, res) => {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.ANTHROPIC_API_KEY,
      "anthropic-version": "2023-06-01"
    },
    body: JSON.stringify(req.body)
  });
  const data = await response.json();
  res.json(data);
});

app.listen(3000, () => console.log("Running at http://localhost:3000"));
```

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...
```

```bash
node server.js
```

### Option C — Next.js API route

```js
// pages/api/analyze.js
export default async function handler(req, res) {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.ANTHROPIC_API_KEY,
      "anthropic-version": "2023-06-01"
    },
    body: JSON.stringify(req.body)
  });
  const data = await response.json();
  res.json(data);
}
```

---

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| UI | Vanilla HTML/CSS/JS | Zero build step, single file, universally deployable |
| Fonts | Google Fonts (DM Serif Display, DM Mono, Syne) | Loaded at runtime via CDN |
| Icons | Tabler Icons (webfont) | 5800+ outline icons, CSS-only, no SVG clutter |
| AI | Anthropic Claude Sonnet via `/v1/messages` | Best-in-class instruction following, reliable JSON output |
| Proxy (prod) | Cloudflare Workers / Express / Next.js | Keeps API key server-side |
| Hosting | Any static host | Netlify, Vercel, GitHub Pages, S3, Cloudflare Pages |

No npm packages. No React. No webpack. No TypeScript compilation. No Docker. No database.

---

## Customization

### Change the AI model

```js
model: "claude-opus-4-20250514"  // more powerful, slower
model: "claude-haiku-4-5-20251001" // faster, cheaper for high volume
```

### Extend the output schema

In the system prompt inside `publicinsight.html`, add fields to the JSON schema:

```js
"ideas": [
  {
    "title": "string",
    "desc": "string",
    "effort": "low|medium|high",       // add this
    "impact": "low|medium|high",       // and this
    "team": "string (which team owns)" // and this
  }
]
```

Then update `renderResults()` to display the new fields.

### Add a real data source

To connect real scraped data instead of AI simulation, preprocess your data and inject it into the user message:

```js
const scrapedData = await fetchRedditPosts(product); // your scraper
const userMsg = `Product: "${product}". Here are 200 real Reddit posts: ${scrapedData}. Analyze and return JSON.`;
```

### Add rate limiting

For production, add simple rate limiting to your proxy:

```js
// Cloudflare Worker — using KV for rate limiting
const key = `rate:${request.headers.get("CF-Connecting-IP")}`;
const count = parseInt(await env.KV.get(key) || "0");
if (count > 20) return new Response("Rate limit exceeded", { status: 429 });
await env.KV.put(key, count + 1, { expirationTtl: 3600 });
```

---

## Project Structure

```
publicinsight/
├── publicinsight.html    # The entire product
├── README.md             # This file
├── server.js             # Optional Express proxy for production
├── worker.js             # Optional Cloudflare Worker proxy
├── .env.example          # Environment variable template
└── package.json          # Only needed if using the Express proxy
```

---

## Cost

Each analysis call uses approximately 400–600 input tokens and 600–900 output tokens.

| Model | Cost per analysis (est.) |
|-------|--------------------------|
| Claude Haiku | ~$0.0003 |
| Claude Sonnet | ~$0.004 |
| Claude Opus | ~$0.03 |

For a team running 50 analyses/day on Sonnet: ~$6/month.

---

## License

MIT — do whatever you want with it.
