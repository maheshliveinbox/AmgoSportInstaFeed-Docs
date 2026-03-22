# Setup Guide 05 — Dashboard Deployment (Vercel)

## Overview

The Next.js dashboard is designed for **zero-config deployment on Vercel** — connect your GitHub repo, add environment variables, and it deploys automatically. This guide covers the full deployment process.

---

## Option A — Vercel (Recommended, Free)

### Step 1 — Push to GitHub

```bash
cd d:\Codes\WorkflowAutomations\AmgoSportInstaFeed

# Initialise git if not already
git init
git add .
git commit -m "feat: AMGO Sports Content Engine v1.0"

# Create a GitHub repo at github.com → New repository
# Name: amgo-sports-content-engine
git remote add origin https://github.com/YOUR_USERNAME/amgo-sports-content-engine.git
git push -u origin main
```

### Step 2 — Import on Vercel

1. Go to [vercel.com](https://vercel.com) → **Add New → Project**
2. Select **Import Git Repository**
3. Select `amgo-sports-content-engine`
4. Framework preset: **Next.js** (auto-detected)
5. Root directory: `./` (the dashboard is the root)
6. Click **Deploy** (initial build will fail — env vars needed)

### Step 3 — Add Environment Variables

In Vercel → Project → **Settings → Environment Variables**

Add all variables from your `.env.local` file:

| Name | Value |
|------|-------|
| `NEXTAUTH_SECRET` | (from `openssl rand -base64 32`) |
| `NEXTAUTH_URL` | `https://your-project.vercel.app` |
| `DATABASE_URL` | `postgresql://user:password@host:5432/db` |
| `SEED_ADMIN_EMAIL` | `admin@amgosports.com.au` |
| `SEED_ADMIN_PASSWORD` | your secure password |
| `N8N_BASE_URL` | `https://your-n8n.app.n8n.cloud` |
| `N8N_WEBHOOK_SECRET` | your webhook secret |
| `N8N_TRIGGER_GENERATE_PATH` | `/webhook/amgo-manual-trigger` |
| `N8N_TRIGGER_APPROVE_PATH` | `/webhook/amgo-approve` |
| `N8N_TRIGGER_PRODUCT_FEATURE_PATH` | `/webhook/amgo-product-feature` |
| `N8N_TRIGGER_HASHTAG_REFRESH_PATH` | `/webhook/amgo-hashtag-refresh` |
| `N8N_TRIGGER_SHOPIFY_SYNC_PATH` | `/webhook/amgo-shopify-sync` |
| `SHOPIFY_STORE_DOMAIN` | `amgosports.myshopify.com` |
| `SHOPIFY_ACCESS_TOKEN` | `shpat_...` |
| `NEXT_PUBLIC_APP_URL` | `https://your-project.vercel.app` |



### Step 4 — Redeploy

After adding all env vars:
1. Go to **Deployments** tab
2. Click the **⋮** menu on the latest deployment
3. Click **Redeploy**

The dashboard will be live at `https://your-project.vercel.app`

---

## Option B — Self-Hosted (Docker)

If you prefer to host alongside n8n on your VPS:

```dockerfile
# Dockerfile (already present in the project)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json
EXPOSE 3000
CMD ["npm", "start"]
```

```bash
docker build -t amgo-dashboard .
docker run -p 3000:3000 --env-file .env.local amgo-dashboard
```

Configure Nginx as reverse proxy:
```nginx
server {
    listen 443 ssl;
    server_name dashboard.amgosports.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

---

## Custom Domain

### Vercel Custom Domain

1. Vercel → Project → **Settings → Domains**
2. Add domain: `dashboard.amgosports.com.au`
3. Add a `CNAME` record in your DNS:
   - Name: `dashboard`
   - Value: `cname.vercel-dns.com`

Update `NEXTAUTH_URL` and `NEXT_PUBLIC_APP_URL`:
```
NEXTAUTH_URL=https://dashboard.amgosports.com.au
NEXT_PUBLIC_APP_URL=https://dashboard.amgosports.com.au
```

---

## NeuralNode AI Showcase Setup

To feature this project on [neuralnodeai.com](https://neuralnodeai.com):

### Dashboard Demo URL
Make the dashboard publicly accessible (without password for demo purposes):

```bash
# In .env.local for demo environment
DASHBOARD_PASSWORD=  # Leave empty to disable auth
```

Or create a read-only demo mode that shows the UI but disables action buttons.

### Recommended Showcase Assets

1. **Screen recording** of the full flow:
   - 8 AM → Telegram photo notification arrives
   - Email preview received with "Review in Dashboard" button
   - Dashboard shows pending post card → click ✅ Approve
   - Instagram post goes live
   - Dashboard updates to POSTED

2. **Architecture diagram** — use the ASCII diagram from `docs/README.md`

3. **Key metrics to highlight**:
   - Setup time: ~2 hours
   - Monthly AI cost: ~AUD $17
   - Time saved: ~15 hours/month (2 posts/week × 30 min each)
   - Content ROI: 30 professional posts for the cost of one hour of a social media manager

---

## Local Development

```bash
cd d:\Codes\WorkflowAutomations\AmgoSportInstaFeed

# Install dependencies (if not already)
npm install

# Copy and fill env vars
cp .env.local.example .env.local
# Edit .env.local with your values

# Start development server
npm run dev
# → http://localhost:3000
```

---

## Build Verification

Before deploying, always verify the build is clean:

```bash
npm run build
# Should output:
# ✓ Compiled successfully
# ✓ TypeScript
# 13 routes generated
```
