# AMGO Sports Content Engine — Project Documentation

> Built by [NeuralNode AI](https://neuralnodeai.com) for AMGO Sports

## Table of Contents

1. [Project Overview](#overview)
2. [Architecture](#architecture)
3. [Tech Stack](#tech-stack)
4. [Folder Structure](#folder-structure)
5. [Features](#features)
6. [Setup Guide](#setup-guide)
7. [n8n Workflows](#n8n-workflows)
8. [API Reference](#api-reference)
9. [Deployment](#deployment)
10. [Cost Breakdown](#costs)

---

## Overview

AMGO Sports Content Engine is a **fully automated, AI-powered Instagram content pipeline** that:

- Generates a professional cricket-themed Instagram post **every day at 8 AM AEST**
- Uses **GPT-4o** for captions and hashtags, **DALL·E 3** for images
- Detects **cricket match days** automatically and adjusts content accordingly
- Pulls **real Shopify products** for product spotlight posts
- Generates **AI video reels** via RunwayML for higher Instagram reach
- Sends a **Telegram photo notification** (preview only) + **HTML email with dashboard link**
- **Operator approves via the dashboard** — approve as image, approve as reel, regenerate, or reject
- **Auto-posts to Instagram** via Blotato upon dashboard approval
- Logs everything securely to a **PostgreSQL Database** content calendar
- Provides a **Next.js web dashboard** for full visibility and control

---

## Architecture

```
AMGO Sports Content Engine
├── n8n Workflows (6 total)
│   ├── Workflow A — Daily Content Generator (GPT-4o + DALL·E 3) + Telegram notification
│   ├── Workflow B — Approval Handler & Publisher (Blotato) — dashboard-triggered
│   ├── Workflow C — Shopify Product Sync (weekly)
│   ├── Workflow D — Cricket Schedule Monitor (daily)
│   ├── Workflow E — Hashtag Intelligence Refresh (weekly)
│   └── Workflow F — Post Auto-Expiry (daily, runs before A)
│
├── Next.js Dashboard (this project)
│   ├── /           Dashboard home + stats + pending approval
│   ├── /calendar   Full content calendar with post details
│   ├── /products   Shopify product library + feature button
│   ├── /hashtags   AI hashtag library + manual refresh
│   └── /settings   Config, feature toggles, API status
│
└── Data Layer
    ├── PostgreSQL Database (content calendar, products, hashtags, users)
    ├── imgbb (image hosting)
    └── Blotato (Instagram scheduling)
```

---

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | Next.js App Router | 15.x |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | 4.x |
| Data Fetching | TanStack Query (React Query) | 5.x |
| State | Zustand (persisted) | 5.x |
| Icons | Lucide React | latest |
| Charts | Recharts | latest |
| Animation | Framer Motion | latest |
| Validation | Zod | latest |
| Google API | googleapis | latest |
| HTTP Client | axios | latest |
| Bundler | Turbopack | (Next.js built-in) |

---

## Folder Structure

```
./                                 # Project root
├── docs/                          ← You are here
│   ├── README.md                  ← This file
│   ├── features/                  ← Feature docs
│   └── setup/                     ← Setup guides
│
├── public/
│
├── src/
│   ├── app/                       ← Next.js App Router pages
│   │   ├── layout.tsx             ← Root layout (dark theme, sidebar)
│   │   ├── globals.css            ← Global styles (glassmorphism, glows)
│   │   ├── page.tsx               ← Dashboard home
│   │   ├── calendar/page.tsx      ← Content calendar
│   │   ├── products/page.tsx      ← Shopify products
│   │   ├── hashtags/page.tsx      ← Hashtag library
│   │   ├── settings/page.tsx      ← App settings
│   │   └── api/                   ← API route handlers
│   │       ├── stats/route.ts
│   │       ├── posts/route.ts
│   │       ├── generate/route.ts
│   │       ├── approve/[postId]/route.ts
│   │       ├── reject/[postId]/route.ts
│   │       ├── products/route.ts
│   │       └── hashtags/route.ts
│   │
│   ├── components/
│   │   ├── ui/                    ← Reusable primitives (Badge, Button, Card)
│   │   ├── layout/                ← Sidebar, QueryProvider, SettingsPanel
│   │   ├── dashboard/             ← StatsGrid, PendingPostCard, GenerateButton
│   │   ├── calendar/              ← CalendarGrid
│   │   ├── products/              ← ProductsGrid
│   │   └── hashtags/              ← HashtagsPanel
│   │
│   ├── lib/
│   │   ├── utils.ts               ← cn(), date helpers, badge classes
│   │   ├── sheets.ts              ← Google Sheets API client
│   │   └── n8n.ts                 ← n8n webhook trigger client
│   │
│   ├── stores/
│   │   └── dashboard.ts           ← Zustand stores (dashboard + settings)
│   │
│   ├── types/
│   │   └── index.ts               ← All TypeScript types
│   │
│   └── config/
│       └── constants.ts           ← App constants, routes, brand config
│
├── .env.local.example             ← Copy to .env.local and fill in
├── .prettierrc                    ← Prettier config
└── package.json
```

---

## Features

See `docs/features/` for detailed documentation on each feature:

- [Daily Content Generation](features/01-daily-content-generation.md)
- [Cricket Match Intelligence](features/02-cricket-match-intelligence.md)
- [Shopify Product Integration](features/03-shopify-product-integration.md)
- [Notification & Approval Flow](features/04-telegram-approval-flow.md)
- [AI Video Reel Generation](features/05-ai-video-reel-generation.md)
- [Dashboard](features/06-nextjs-dashboard.md)
- [Hashtag Intelligence](features/07-hashtag-intelligence.md)

---

## Setup Guide

See `docs/setup/` for step-by-step guides:

1. [Prerequisites & Accounts](setup/01-prerequisites.md)
2. [Credentials Setup](setup/02-credentials.md)
3. [Google Sheets Setup](setup/03-google-sheets.md)
4. [n8n Workflow Import](setup/04-n8n-import.md)
5. [Dashboard Deployment (Vercel)](setup/05-vercel-deployment.md)

---

## n8n Workflows

| File | Workflow | Trigger |
|------|----------|---------|
| `workflow-A-content-generator.json` | Content Generator | Daily 8 AM AEST + dashboard button |
| `workflow-B-approval-publisher.json` | Approval & Publisher | Dashboard POST webhook |
| `workflow-C-shopify-sync.json` | Shopify Product Sync | Weekly Sunday 6 AM + dashboard button |
| `workflow-D-cricket-monitor.json` | Cricket Match Monitor | Daily 7:30 AM AEST |
| `workflow-E-hashtag-intelligence.json` | Hashtag Intelligence | Weekly Monday 7 AM + dashboard button |
| `workflow-F-post-expiry.json` | Post Auto-Expiry | Daily 7:50 AM AEST |

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/stats` | Dashboard stats (posts, approval rate, streak) |
| `GET` | `/api/posts` | All posts from Database |
| `POST` | `/api/generate` | Trigger Workflow A to generate a new post |
| `POST` | `/api/approve/:postId` | Send approval action to Workflow B (body: `{action: "approve"|"approve_reel"|"reject"|"regenerate"}`) |
| `GET` | `/api/products` | All products from Database |
| `GET` | `/api/hashtags` | Hashtag library from Database |

---

## Deployment

### Local Development
```bash
cp .env.local.example .env.local
# Fill in your credentials
npm run dev
```

### Coolify (Recommended — self-hosted)
1. Push to GitHub
2. Coolify → New Resource → Application → GitHub repo
3. Set Build Pack: `Dockerfile`, Port: `3000`
4. Add env vars from `.env.local.example`
5. Deploy

### Vercel (Alternative)
1. Push to GitHub
2. Import to Vercel → Add env vars from `.env.local`
3. Deploy

### Environment Variables Required
See `.env.local.example` for full list.

---

## Costs

| Service | Monthly Cost |
|---------|-------------|
| GPT-4o captions | ~AUD $2 |
| DALL·E 3 HD (30 images) | ~AUD $3.60 |
| RunwayML Gen-3 (30 × 5s reels) | ~AUD $11.25 |
| imgbb, Telegram | Free |
| Coolify/Vercel hosting | Free/Included |
| **Total AI Cost** | **~AUD $17/month** |

Blotato and n8n billed separately.
