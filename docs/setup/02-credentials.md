# Setup Guide 02 — Credentials Configuration

## Overview

This guide walks through creating all credentials in n8n and configuring `.env.local` for the Next.js dashboard. Complete this after setting up all accounts (see Setup Guide 01).

---

## n8n Credentials

Open your n8n instance → **Credentials** tab → **Add Credential**

### 1. OpenAI_AMGO

- **Type**: OpenAI API
- **Name**: `OpenAI_AMGO`
- **API Key**: `sk-proj-your-openai-key`

Used by: Workflow A (GPT-4o captions), Workflow E (GPT-4o-mini hashtag refresh)

---

### 2. Telegram_AMGO_Bot

- **Type**: Telegram API
- **Name**: `Telegram_AMGO_Bot`
- **Access Token**: `7123456789:ABCDEFGhijklmno...`

Used by: Workflow A (photo preview), Workflow B (outcome alerts), Workflow C/D/E/F (status notifications)

> **Note**: Telegram is used for notifications only — no inline buttons or callback webhooks. All approvals happen via the dashboard.

---

### 3. imgbb_AMGO

- **Type**: HTTP Header Auth
- **Name**: `imgbb_AMGO`
- **Header Name**: `key`
- **Header Value**: `your-imgbb-api-key`

> Note: imgbb uses a query parameter, not header, but n8n passes it as form data. The HTTP Request node in Workflow A handles this directly.

---

### 4. Blotato_AMGO

- **Type**: HTTP Header Auth
- **Name**: `Blotato_AMGO`
- **Header Name**: `blotato-api-key`
- **Header Value**: `your-blotato-api-key`

**Getting Blotato API key:**
1. Log in to [blotato.com](https://blotato.com)
2. Settings → API → Generate Key
3. Settings → Connected Accounts → Connect Instagram Business Account
4. Note the **Social Account ID** (needed in Workflow B node)

---

### 5. Shopify_AMGO

- **Type**: HTTP Header Auth
- **Name**: `Shopify_AMGO`
- **Header Name**: `X-Shopify-Access-Token`
- **Header Value**: `shpat_your-shopify-token`

Used by: Workflow C (Shopify product sync)

---

### 6. CricAPI_AMGO

- **Type**: HTTP Query Auth (or passed in URL directly in the HTTP Request node)
- **Name**: `CricAPI_AMGO`
- **Param Name**: `apikey`
- **Param Value**: `your-cricapi-key`

Used by: Workflow D (cricket match detection)

---

### 7. FalAI_AMGO (for Flux 2 Pro images)

- **Type**: HTTP Header Auth
- **Name**: `FalAI_AMGO`
- **Header Name**: `Authorization`
- **Header Value**: `Key your-fal-api-key`

**Getting Fal.ai API key:**
1. Sign up at [fal.ai](https://fal.ai)
2. Go to **Account → API Keys** → Generate key
3. Format for the credential value: `Key fal_xxxx...` (with the word "Key" prefix)

Used by: Workflow A (Flux 2 Pro image generation)

---

## Dashboard Environment Variables

```bash
# Copy to .env.local in the project root
cp .env.local.example .env.local
```

Then fill in:

```bash
# Authentication
NEXTAUTH_SECRET=run-openssl-rand-base64-32-to-generate
NEXTAUTH_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://postgres:your-password@amgo-postgres:5432/amgo
SEED_ADMIN_EMAIL=admin@amgosports.com.au
SEED_ADMIN_PASSWORD=your-secure-admin-password

# n8n Integration
N8N_BASE_URL=https://your-n8n.app.n8n.cloud
N8N_WEBHOOK_SECRET=choose-a-random-secret
N8N_API_KEY=your-n8n-api-key
N8N_TRIGGER_GENERATE_PATH=/webhook/amgo-manual-trigger
N8N_TRIGGER_APPROVE_PATH=/webhook/amgo-approve
N8N_TRIGGER_PRODUCT_FEATURE_PATH=/webhook/amgo-product-feature
N8N_TRIGGER_HASHTAG_REFRESH_PATH=/webhook/amgo-hashtag-refresh
N8N_TRIGGER_SHOPIFY_SYNC_PATH=/webhook/amgo-shopify-sync

# External APIs
OPENAI_API_KEY=sk-your-key
FAL_API_KEY=your-fal-api-key
SHOPIFY_STORE_DOMAIN=amgosports.myshopify.com
SHOPIFY_ACCESS_TOKEN=shpat_your-token
TELEGRAM_BOT_TOKEN=from-botfather
TELEGRAM_CHAT_ID=your-chat-id
CRICAPI_KEY=your-cricapi-key
IMGBB_API_KEY=your-imgbb-key
BLOTATO_API_KEY=your-blotato-key

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME=AMGO Sports Content Engine
```

### Generating NEXTAUTH_SECRET

```bash
openssl rand -base64 32
# Example output: K7jH9mNqR2vA8sL4pX6wY1cT5uF3eG0d
```

---

## Verification

After setting up all credentials, test each one:

```
n8n → Credentials → click 3 dots → Test Credential
```

All should show a green ✓ success indicator.
