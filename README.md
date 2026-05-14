# PublicInsight

**Turn public noise into product signal.**

PublicInsight scans Reddit threads, Glassdoor reviews, App Store complaints, and Product Hunt comments for any product — and returns clustered themes, classified signals (opportunity / risk / gap), and concrete feature ideas. Everything runs in a single HTML file, no backend required.

---


## Quick Start

 Open directly in browser 

```bash
git clone https://github.com/yourname/publicinsight.git
cd publicinsight
open publicinsight.html   # macOS
# or just double-click the file in Finder / File Explorer
```


---

## API Key Setup

PublicInsight calls the Anthropic API directly from the browser. You need an API key from [console.anthropic.com](https://console.anthropic.com).




## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| UI | Vanilla HTML/CSS/JS | Zero build step, single file, universally deployable |
| Fonts | Google Fonts (DM Serif Display, DM Mono, Syne) | Loaded at runtime via CDN |
| Icons | Tabler Icons (webfont) | 5800+ outline icons, CSS-only, no SVG clutter |
| AI | Anthropic Claude Sonnet via `/v1/messages` | Best-in-class instruction following, reliable JSON output |
| Proxy (prod) | Cloudflare Workers / Express / Next.js | Keeps API key server-side |
| Hosting | Any static host | Netlify, Vercel, GitHub Pages, S3, Cloudflare Pages |



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

