<div align="center">
  <h1>🏏 AMGO Sports Content Engine.</h1>
  <p><strong>AI-powered Instagram automation for AMGO Sports</strong></p>
  <p>
    <a href="https://amgosports.com.au">amgosports.com.au</a> ·
    <a href="https://www.instagram.com/amgosports">@amgosports</a> ·
    Built by <a href="https://neuralnodeai.com">NeuralNode AI</a>
  </p>
  <br/>
  <img src="https://img.shields.io/badge/Next.js-16-black?style=flat-square&logo=next.js" />
  <img src="https://img.shields.io/badge/TypeScript-5-blue?style=flat-square&logo=typescript" />
  <img src="https://img.shields.io/badge/n8n-automation-orange?style=flat-square" />
  <img src="https://img.shields.io/badge/GPT--4o-captions-green?style=flat-square&logo=openai" />
  <img src="https://img.shields.io/badge/Flux%202%20Pro-images-purple?style=flat-square" />
</div>

---

## What It Does

Every day at **8:00 AM AEST**, this system automatically:

1. 🤖 **Generates** a professional Instagram post (caption + hashtags + HD image) using GPT-4o & Flux 2 Pro
2. 🏏 **Detects** cricket match days and generates an **additional** match-day post alongside the regular daily post
3. 📲 **Notifies** via Telegram (photo preview) with a direct dashboard link
4. ✅ **Posts** to Instagram via Blotato when you approve in the dashboard — zero manual work required
5. 📊 **Logs** everything securely to a **PostgreSQL** database (via Prisma)
6. 🗑️ **Expires** unapproved posts automatically at 7:50 AM the next day (before new content arrives)

---

## Features

| Feature | Description |
|---------|-------------|
| 🤖 **Daily AI Content** | GPT-4o captions + Flux 2 Pro HD images via Fal.ai, 7 rotating themes |
| 🏏 **Cricket Intelligence** | Daily match detection via CricAPI — generates an extra match-day post alongside the regular theme post |
| 🛍️ **Shopify Integration** | Product library synced weekly, real product photos in posts |
| 📲 **Telegram Notifications** | Telegram photo preview with direct dashboard link — approve from the dashboard |
| 🗑️ **Auto Post Expiry** | Unapproved PENDING posts are automatically expired at 7:50 AM before new content arrives |
| 🏷️ **Smart Hashtags** | GPT-4o-mini refreshes 30 hashtags weekly based on cricket season |
| 📊 **Dashboard** | Next.js 16 control panel for stats, calendar, products, hashtags, settings |

---

## Project Structure

```
./                           # Project root
├── src/                     # Next.js 15 App Router source
│   ├── app/                 # Pages + API routes
│   ├── components/          # UI, layout, dashboard, calendar, products, hashtags
│   ├── lib/                 # Prisma DB client, n8n webhook client, utils
│   ├── stores/              # Zustand state management
│   ├── types/               # TypeScript definitions
│   └── config/              # Constants, routes, brand config
├── n8n-workflows/           # All 6 n8n workflow JSON files (import these)
├── docs/                    # Full feature & setup documentation
│   ├── features/            # 7 detailed feature docs
│   └── setup/               # 5 step-by-step setup guides
├── Dockerfile               # Multi-stage build for Coolify/Docker
└── .env.local.example       # Copy to .env.local and fill in
```

---

## Quick Start

```bash
# 1. Clone and install
git clone https://github.com/YOUR_USERNAME/amgo-sports-content-engine.git
cd amgo-sports-content-engine
npm install

# 2. Configure environment
cp .env.local.example .env.local
# Fill in your API keys in .env.local

# 3. Run the dashboard
npm run dev
# → http://localhost:3000
```

---

## n8n Workflows

Import these JSON files into your n8n instance (in order):

| Workflow | File | Runs | Status |
|----------|------|------|--------|
| B — Approval & Publisher | `workflow-B-approval-publisher.json` | On dashboard action | Active |
| D — Cricket Monitor | `workflow-D-cricket-monitor.json` | 7:30 AM AEST daily | Active |
| F — Post Auto-Expiry | `workflow-F-post-expiry.json` | 7:50 AM AEST daily | Active |
| E — Hashtag Intelligence | `workflow-E-hashtag-intelligence.json` | 7:00 AM AEST Mondays | Active |
| C — Shopify Sync | `workflow-C-shopify-sync.json` | 6:00 AM AEST Sundays | Inactive |
| A — Content Generator | `workflow-A-content-generator.json` | 8:00 AM AEST daily | Active |

> See [`n8n-workflows/README.md`](n8n-workflows/README.md) for activation instructions.

---

## Documentation

Full documentation lives in [`docs/`](docs/README.md):

**Features**
- [Daily AI Content Generation](docs/features/01-daily-content-generation.md)
- [Cricket Match Intelligence](docs/features/02-cricket-match-intelligence.md)
- [Shopify Product Integration](docs/features/03-shopify-product-integration.md)
- [Telegram Notification & Approval Flow](docs/features/04-telegram-approval-flow.md)
- [Next.js Dashboard](docs/features/06-nextjs-dashboard.md)
- [Hashtag Intelligence](docs/features/07-hashtag-intelligence.md)
- [Post Auto-Expiry](docs/features/08-post-auto-expiry.md)

**Setup Guides**
- [Prerequisites & Accounts](docs/setup/01-prerequisites.md)
- [Credentials Configuration](docs/setup/02-credentials.md)
- [Database Setup (PostgreSQL)](docs/setup/03-database-setup.md)
- [n8n Workflow Import](docs/setup/04-n8n-import.md)
- [Vercel / Coolify Deployment](docs/setup/05-vercel-deployment.md)

---

## Tech Stack

**Dashboard:** Next.js 16 · TypeScript · Tailwind CSS v4 · TanStack Query · Prisma ORM
**AI:** GPT-4o · GPT-4o-mini · Flux 2 Pro (Fal.ai)
**Automation:** n8n · Blotato · Telegram Bot API
**Data:** PostgreSQL · Shopify Admin API · CricAPI · imgbb
**Hosting:** Coolify (Docker)

---

## Monthly Cost

| Service | Cost |
|---------|------|
| GPT-4o (30 posts) | ~AUD $2 |
| GPT-4o-mini (hashtags, weekly) | ~AUD $0.01 |
| Flux 2 Pro via Fal.ai (30 images) | ~AUD $4–6 |
| **Total AI cost** | **~AUD $6–8/month** |

Blotato, n8n, and Coolify hosted separately.

---

<div align="center">
  <sub>Built with ❤️ by <a href="https://neuralnodeai.com">NeuralNode AI</a></sub>
</div>
