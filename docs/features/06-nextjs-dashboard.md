# Feature 06 — Next.js Dashboard

## Overview

The AMGO Sports Content Engine Dashboard is a **Next.js 16 web application** that provides full visibility and control over the Instagram automation. It is the operational nerve centre — showing what's been posted, what's pending, managing products and hashtags, and allowing on-demand content generation.

---

## Pages

### 1. Dashboard Home (`/`)

The first thing you see when you open the app. Designed for **daily use in under 30 seconds**.

**Stats Row** — 4 metric cards:
- Posts This Month (vs 30 target)
- Approval Rate (% of generated posts that were published)
- Posts Rejected (quality control metric)
- Current Streak (consecutive published days)

**Pending Approval Card** — appears when a post is awaiting review:
- Full-size image preview
- Caption text
- Hashtag preview
- Post ID and date
- **3 Action Buttons**: ✅ Approve Image · ✏️ Regenerate · ❌ Reject
- Copy caption + hashtags button

**Recent Posts Table** — last 5 posts with status badges and Instagram links

**Generate Now Button** — manually triggers Workflow A outside of the 8 AM schedule

---

### 2. Calendar (`/calendar`)

Full history of all generated posts in a scrollable list, sorted newest first.

**Per row**:
- Thumbnail image, date, theme badge, status badge, caption preview, Instagram link

**Click to expand** — shows full image, complete caption, hashtags, and image generation prompt (for reference)

**Use cases**:
- Review what was posted last week
- Check image generation prompts that produced great images (for reference with future posts)
- Find the Instagram link for a specific date

---

### 3. Products (`/products`)

Grid of all products synced from the AMGO Sports Shopify store.

**Per product card**:
- Product image (from Shopify CDN)
- Product title, brand, price
- "Feature" button — triggers an instant Product Spotlight post for that product
- External link to the product page on amgosports.com.au

**Sync from Shopify button** — manually triggers Workflow C to re-sync the product library

**Use cases**:
- Manually feature a new product that just arrived
- Check what's in the synced product library
- Trigger ad-hoc product spotlight outside of Monday

---

### 4. Hashtags (`/hashtags`)

Displays the current AI-curated hashtag library, organised by category.

**Categories (colour-coded)**:
- 🟣 Brand (`#amgosports`, `#amgosportsau`, etc.)
- 🔵 Product (`#cricketbat`, `#SSbats`, `#SG`, etc.)
- 🟢 Cricket (`#cricket`, `#cricketaustralia`, `#bbllive`, etc.)
- 🟠 Regional (`#hillsshire`, `#sydneycricket`, `#nswcricket`, etc.)
- 🩷 Trending (refreshed weekly based on current season)

**AI Refresh button** — manually triggers Workflow E to regenerate the hashtag library using GPT-4o

---

### 5. Settings (`/settings`)

Configuration panel for the content engine.

**Post Timing**: Set the daily post time in AEST (note: must also update the n8n cron expression)

**Feature Toggles**:
- Telegram Notifications (default: ON)
- AI Reel Generation (default: ON)
- Auto-Approve Posts (default: OFF — dangerous, posts without review)

**Connection Status**: Shows which services need credentials configured (checks `.env.local`)

---

## Tech Stack Details

### Data Architecture

```
PostgreSQL Database (source of truth)
    ↑ populated by n8n workflows A/B/C/D/E/F (via API)
    ↓ read by Next.js Server Components + TanStack Query
Next.js 16 Dashboard
    ↓ action buttons call
API Routes (/api/*)
    ↓ trigger
n8n Webhooks → workflows execute
```

**Why Server Components + TanStack Query?**
- Dashboard home revalidates automatically after actions
- No loading spinners on page load — content is always pre-rendered
- Approval/action buttons use TanStack Query mutations with optimistic updates

### ISR Revalidation per Page

| Page | Revalidation | Rationale |
|------|-------------|-----------|
| `/` | 60 seconds | Needs to show new pending posts quickly |
| `/calendar` | 2 minutes | Post history doesn't change that often |
| `/products` | 5 minutes | Products sync weekly |
| `/hashtags` | 1 hour | Refreshed weekly |
| `/settings` | No ISR | Client-side only (Zustand persisted) |

---

## Component Architecture

```
src/
├── components/
│   ├── ui/                    ← Design system primitives
│   │   ├── Badge.tsx          ← Status/theme badges
│   │   ├── Button.tsx         ← All button variants (primary/secondary/danger/success/ghost)
│   │   └── Card.tsx           ← Glassmorphism card container
│   │
│   ├── layout/
│   │   ├── Sidebar.tsx        ← Navigation sidebar with active state
│   │   ├── QueryProvider.tsx  ← TanStack Query context wrapper
│   │   └── SettingsPanel.tsx  ← Settings form (uses Zustand)
│   │
│   ├── dashboard/
│   │   ├── StatsGrid.tsx      ← 4-card stats row
│   │   ├── PendingPostCard.tsx← Approval card with image + action buttons
│   │   ├── GenerateButton.tsx ← Manual trigger button
│   │   ├── RecentPostsTable.tsx ← Last 5 posts table
│   │   └── Toast.tsx          ← Lightweight toast notifications
│   │
│   ├── calendar/
│   │   └── CalendarGrid.tsx   ← Post list with expand panel
│   │
│   ├── products/
│   │   └── ProductsGrid.tsx   ← Product card grid
│   │
│   └── hashtags/
│       └── HashtagsPanel.tsx  ← Category-grouped hashtag display
```

---

## Design System

### Dark Theme Colour Palette

| Token | Value | Use |
|-------|-------|-----|
| Background | `#0a0a0f` | Page background |
| Surface | `rgba(255,255,255,0.03)` | Card glass surface |
| Border | `rgba(255,255,255,0.07)` | Card borders |
| Primary | `#7c3aed` (violet-600) | Primary actions, active nav |
| Success | `#10b981` (emerald-500) | POSTED status, Approve button |
| Warning | `#f59e0b` (amber-500) | PENDING status |
| Danger | `#ef4444` (red-500) | REJECTED, Reject button |

### Glassmorphism

All cards use the `.glass` CSS class:
```css
background: rgba(255, 255, 255, 0.03);
backdrop-filter: blur(12px);
border: 1px solid rgba(255, 255, 255, 0.07);
```

---

## API Routes

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/stats` | Session | Dashboard stats |
| `GET/POST` | `/api/posts` | Session / Webhook secret | Fetch posts; n8n ingest |
| `GET/PATCH` | `/api/posts/[postId]` | Session / Webhook secret | Fetch/update post |
| `POST` | `/api/generate` | Session | Trigger Workflow A |
| `POST` | `/api/approve/[postId]` | Session | Approve post (image) or regenerate |
| `POST` | `/api/reject/[postId]` | Session | Reject post |
| `GET/POST` | `/api/products` | Session / Webhook secret | Products from Database |
| `GET/POST/DELETE` | `/api/hashtags` | Session | Manage individual hashtags |
| `POST` | `/api/hashtags/sync` | Webhook secret | Bulk atomic replace (Workflow E) |
| `GET/POST` | `/api/cricket-matches` | None / Webhook secret | Cricket match status |
| `GET/PUT` | `/api/settings` | Session / Webhook secret | App settings |
| `GET/POST` | `/api/users` | Admin | User management |
| `GET` | `/api/audit-log` | Session | Audit trail |

---

## Deployment

The app is deployed via **Coolify** (Docker, `output: "standalone"`). See [DEPLOYMENT.md](../../DEPLOYMENT.md) for the full deployment guide.

```bash
# Local development
npm run dev       # → http://localhost:3000
npm run build     # Verify production build
npm run lint      # Run ESLint
```
