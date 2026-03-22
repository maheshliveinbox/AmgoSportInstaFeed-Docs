# 🚀 AMGO Sports Content Engine — Deployment Guide

This guide covers all steps to deploy the AMGO Sports Content Engine from scratch. The stack uses **PostgreSQL** (self-hosted on Coolify) for all data, **Prisma** as the ORM, and **Next.js** for the dashboard.

---

## 🟢 Phase 1: Prerequisites & Accounts

Before starting, you need accounts and access for the following services:

1. **Coolify**: Self-hosted PaaS for hosting the dashboard and PostgreSQL ([coolify.io](https://coolify.io)).
2. **n8n**: Workflow automation engine ([n8n.io](https://n8n.io)). Cloud (~$24/mo) or Self-Hosted (Free).
3. **OpenAI**: GPT-4o (captions) + GPT-4o-mini (hashtags). API key at [platform.openai.com](https://platform.openai.com). Add billing (~$2 AUD/mo).
4. **Fal.ai**: Flux 2 Pro image generation. API key at [fal.ai](https://fal.ai). Add billing (~$4-6 AUD/mo).
5. **Telegram**: For notifications. Use `@BotFather` to create a bot. Save the **Bot Token** and **Chat ID**.
6. **imgbb**: Permanent image hosting. Free API key at [api.imgbb.com](https://api.imgbb.com).
7. **Blotato**: Instagram posting API. Connect your Instagram account at [blotato.com](https://blotato.com).
8. **CricAPI**: Cricket match data. Free API key at [cricapi.com](https://cricapi.com).
9. **Shopify**: Store domain and Admin API Access Token (for Workflow C when enabled).

---

## 🗄️ Phase 2: PostgreSQL Database (Coolify)

### 2.1 Create PostgreSQL Service in Coolify

1. In Coolify, go to your **Project → New Resource → Database → PostgreSQL**.
2. Set a name (e.g. `amgo-postgres`) and a strong root password.
3. Click **Deploy**. Wait for it to go green.
4. Navigate to the service → **Connection** tab.
5. Copy the **Internal Connection URL** — it will look like:
   ```
   postgresql://postgres:your-password@amgo-postgres:5432/postgres
   ```
   > Use the **internal** URL (not the public one) if the dashboard is on the same Coolify host.

### 2.2 Push Schema & Seed Admin User

After the dashboard is deployed (Phase 5), run these commands once from Coolify's **Terminal** for the dashboard service:

```bash
# Create all tables from Prisma schema
npx prisma db push

# Seed the initial admin user (uses SEED_ADMIN_EMAIL / SEED_ADMIN_PASSWORD env vars)
npx prisma db seed
```

> After seeding, `SEED_ADMIN_EMAIL` and `SEED_ADMIN_PASSWORD` can remain in env — they are only used on seed, not at runtime.

---

## 🟠 Phase 3: Dashboard Configuration (.env)

In Coolify, go to your dashboard service → **Environment Variables** and add all the following:

```bash
# ── Authentication ──────────────────────────────────────────
NEXTAUTH_SECRET=        # Generate: openssl rand -base64 32
NEXTAUTH_URL=           # Your public dashboard URL, e.g. https://dashboard.amgosports.com.au

# ── Database ────────────────────────────────────────────────
DATABASE_URL=           # PostgreSQL connection string from Phase 2.1
                        # e.g. postgresql://postgres:password@amgo-postgres:5432/postgres

# ── Admin User Seed (first-time setup only) ─────────────────
SEED_ADMIN_EMAIL=       # e.g. admin@amgosports.com.au
SEED_ADMIN_PASSWORD=    # Choose a strong password (will be bcrypt-hashed in DB)

# ── n8n Webhooks ────────────────────────────────────────────
N8N_BASE_URL=           # e.g. https://your-n8n.app.n8n.cloud
N8N_WEBHOOK_SECRET=     # A shared secret for securing webhook calls
N8N_TRIGGER_GENERATE_PATH=/webhook/amgo-manual-trigger
N8N_TRIGGER_APPROVE_PATH=/webhook/amgo-approve
N8N_TRIGGER_PRODUCT_FEATURE_PATH=/webhook/amgo-product-feature
N8N_TRIGGER_HASHTAG_REFRESH_PATH=/webhook/amgo-hashtag-refresh
N8N_TRIGGER_SHOPIFY_SYNC_PATH=/webhook/amgo-shopify-sync

# ── Shopify ─────────────────────────────────────────────────
SHOPIFY_STORE_DOMAIN=amgosports.myshopify.com
SHOPIFY_ACCESS_TOKEN=   # shpat_your-access-token

# ── External APIs ───────────────────────────────────────────
OPENAI_API_KEY=         # sk-your-key
FAL_API_KEY=            # From fal.ai (Flux 2 Pro image generation)
TELEGRAM_BOT_TOKEN=     # From @BotFather
TELEGRAM_CHAT_ID=       # Your chat/group ID
CRICAPI_KEY=            # From cricapi.com
IMGBB_API_KEY=          # From imgbb.com
BLOTATO_API_KEY=        # From blotato.com

# ── App Config ──────────────────────────────────────────────
NEXT_PUBLIC_APP_URL=    # Same as NEXTAUTH_URL
NEXT_PUBLIC_APP_NAME=AMGO Sports Content Engine
```

---

## 🔵 Phase 4: n8n Credentials & Workflows

### 4.1 Create Credentials in n8n

Go to n8n → **Credentials** and add:

1. `OpenAI_AMGO` — OpenAI API key
2. `Telegram_AMGO_Bot` — Telegram Bot API key
3. `FalAI_AMGO` — HTTP Header Auth: Name `Authorization`, Value: `Key your-fal-api-key`
4. `imgbb_AMGO` — HTTP Header Auth: Name `key`, Value: your imgbb key
5. `Blotato_AMGO` — HTTP Header Auth: Name `blotato-api-key`, Value: your key
6. `Shopify_AMGO` — HTTP Header Auth: Name `X-Shopify-Access-Token`, Value: your token
7. `CricAPI_AMGO` — HTTP Query Auth: Name `apikey`, Value: your key

> ✅ **No email (Gmail) credentials needed** — notifications are Telegram-only.

### 4.2 Import Workflows

Import the JSON files from the `n8n-workflows/` directory in this **exact order**:

1. **Workflow B** (`workflow-B-approval-publisher.json`)
2. **Workflow D** (`workflow-D-cricket-monitor.json`)
3. **Workflow F** (`workflow-F-post-expiry.json`)
4. **Workflow E** (`workflow-E-hashtag-intelligence.json`)
5. **Workflow C** (`workflow-C-shopify-sync.json`) — optional, leave inactive until Shopify integration is needed
6. **Workflow A** (`workflow-A-content-generator.json`)

For each workflow, configure:
- Credentials (mapped to the ones created above)
- The `NEXT_PUBLIC_APP_URL` in any HTTP Request nodes pointing to your dashboard

> **n8n → Dashboard integration:** Workflow A writes new posts directly to your dashboard via `POST {NEXT_PUBLIC_APP_URL}/api/posts` with the `x-webhook-secret` header set to your `N8N_WEBHOOK_SECRET`.

> **Testing:** Trigger Workflow A manually. Within ~60s you should see a Telegram notification and a pending post appear on the dashboard.

---

## 🟣 Phase 5: Dashboard Deployment (Coolify)

### 5.1 Connect Repository

1. In Coolify, go to your project → **New Resource → Application → From a Git Repository**.
2. Connect your GitHub/GitLab repo.
3. Set the **Build Pack** to `Dockerfile`.
4. Set the **Port** to `3000`.

### 5.2 Configure Build

Coolify will auto-detect the `Dockerfile`. Ensure:
- **Root directory**: `/` (default)
- The environment variables from Phase 3 are all set on the service.

### 5.3 Deploy

1. Click **Deploy**. Coolify builds the Docker image.
2. Once green, open the **Terminal** tab and run:
   ```bash
   npx prisma db push
   npx prisma db seed
   ```
3. Navigate to your dashboard URL. Log in with your `SEED_ADMIN_EMAIL` + `SEED_ADMIN_PASSWORD`.

### 5.4 Custom Domain (Optional)

In Coolify → your app service → **Domains**, add your custom domain (e.g. `dashboard.amgosports.com.au`). Coolify will handle SSL via Let's Encrypt automatically.

---

## 👤 Managing Users

Additional dashboard users can be created via the API (admin only):

```bash
curl -X POST https://your-dashboard-url/api/users \
  -H "Content-Type: application/json" \
  -H "Cookie: next-auth.session-token=your-session" \
  -d '{"name":"Jane Smith","email":"jane@amgosports.com.au","password":"SecurePass123","role":"ADMIN"}'
```

Available roles: `ADMIN`, `EDITOR`, `VIEWER`

---

## 🗂️ Project Structure

```
AmgoSportInstaFeed/
├── prisma/
│   ├── schema.prisma       # Database schema (all tables)
│   └── seed.ts             # Seeds initial admin user
├── src/
│   ├── lib/
│   │   ├── prisma.ts       # Singleton Prisma client
│   │   ├── db.ts           # All data access functions
│   │   └── n8n.ts          # n8n webhook triggers
│   ├── app/api/            # All Next.js API routes
│   └── ...
├── Dockerfile
└── DEPLOYMENT.md
```

---

**🎉 Deployment Complete!** The dashboard is live, backed by PostgreSQL, with secure credentials-based authentication. Trigger Workflow A from n8n or the dashboard to generate your first post.
