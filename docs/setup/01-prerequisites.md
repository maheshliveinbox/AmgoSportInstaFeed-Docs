# Setup Guide 01 — Prerequisites & Accounts

## Overview

Before setting up the AMGO Sports Content Engine, you need accounts and access for **8 services**. This guide tells you exactly what to sign up for, what plan you need, and where to find the credentials.

---

## Required Services

| Service | Purpose | Cost | Sign Up |
|---------|---------|------|---------|
| **n8n** | Workflow automation engine | Free (self-hosted) | [n8n.io](https://n8n.io) |
| **Coolify** | Self-hosted PaaS for Dashboard & PostgreSQL | Free (self-hosted) | [coolify.io](https://coolify.io) |
| **OpenAI** | GPT-4o (captions) + GPT-4o-mini (hashtags) | Pay-per-use ~$2 AUD/mo | [platform.openai.com](https://platform.openai.com) |
| **Fal.ai** | Flux 2 Pro image generation | Pay-per-use ~$4-6 AUD/mo | [fal.ai](https://fal.ai) |
| **Telegram** | Approval previews and notifications | Free | [telegram.org](https://telegram.org) |
| **imgbb** | Permanent image hosting | Free | [imgbb.com](https://imgbb.com) |
| **Blotato** | Instagram posting API | ~$29 USD/mo | [blotato.com](https://blotato.com) |
| **CricAPI** | Cricket match schedule data | Free | [cricapi.com](https://cricapi.com) |
| **Shopify** | Product catalogue sync | Requires Shopify store | Admin access needed |

---

## n8n Setup

### Option A — Cloud (Recommended for getting started)

1. Go to [n8n.io](https://n8n.io) → **Start for free**
2. Create account → you'll get a 14-day free trial
3. Pro plan (~$24/month) needed for multiple active workflows
4. Note your instance URL: `https://your-name.app.n8n.cloud`

### Option B — Self-Hosted (Recommended for production)

If you have a VPS (Hetzner, DigitalOcean, etc.):

```bash
# Using Docker Compose
docker compose up -d

# Or with npm (Node.js 18+)
npm install -g n8n
n8n start
```

n8n will be available at `http://your-server-ip:5678`

> ⚠️ **Self-hosted n8n must be publicly accessible** for Telegram webhooks to work. Either expose port 5678 or put it behind a reverse proxy (Nginx/Caddy) with HTTPS.

---

## OpenAI Setup

1. Go to [platform.openai.com](https://platform.openai.com)
2. Sign up or log in
3. Go to **API Keys** → **Create new secret key**
4. Name it: `AMGO Sports Content Engine`
5. Copy the key (shown only once): `sk-proj-...`
6. **Add billing**: Settings → Billing → Add payment method
   - Recommended: Add $20 AUD credit to start
   - Enable auto-recharge when balance drops below $5

> **Estimated usage**: ~$3-5 AUD/month for 30 posts

---

## Telegram Setup

1. Download Telegram on your phone (if not already)
2. Search for `@BotFather` in Telegram
3. Send `/newbot` and follow prompts:
   - Display name: `AMGO Sports Content Bot`
   - Username: `amgosports_content_bot` (must be unique globally)
4. Save the bot token: `7123456789:ABCDEFGhijklmnopqrSTUVwxyz`
5. Start a chat with your new bot (search by username)
6. Get your chat ID by visiting:
   `https://api.telegram.org/bot{TOKEN}/getUpdates`

---

## imgbb Setup

1. Go to [imgbb.com](https://imgbb.com)
2. Create a free account
3. Go to [api.imgbb.com](https://api.imgbb.com) → **Get API Key**
4. Copy your API key

---

## CricAPI Setup

1. Go to [cricapi.com](https://cricapi.com)
2. Sign up for a free account
3. Free tier: 100 API calls/day (workflow uses 1/day)
4. Copy your API key from the dashboard

---

## Verification Checklist

Before proceeding to credential setup, confirm you have:

- [ ] n8n instance running and accessible via HTTPS
- [ ] OpenAI API key (`sk-proj-...`)
- [ ] Fal.ai API key (for Flux 2 Pro image generation)
- [ ] Telegram bot token (`123456789:ABCdef...`)
- [ ] Telegram chat ID (a negative number for group chats)
- [ ] PostgreSQL Database deployed on Coolify
- [ ] imgbb API key
- [ ] Blotato account with Instagram connected
- [ ] CricAPI free account + API key
- [ ] Shopify Admin API access token (if using product integration)
