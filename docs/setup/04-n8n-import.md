# Setup Guide 04 — n8n Workflow Import & Activation

## Overview

This guide covers importing the 6 workflow JSON files, configuring credentials and IDs, and activating them in the correct order.

All workflow files are in: `n8n-workflows/`

---

## Activation Order

> ⚠️ **Activate in this exact order.** Do NOT activate Workflow A first.

```
1. Workflow B  →  dashboard webhook must be live before A
2. Workflow D  →  runs 30 min before A daily
3. Workflow F  →  runs 10 min before A daily (post expiry)
4. Workflow E  →  runs 1 hour before A on Mondays
5. Workflow C  →  runs on Sundays (activate when Shopify sync is needed)
6. Workflow A  →  main daily generator — activate last
```

---

## Step 1 — Import Workflow B (Approval & Publisher)

1. In n8n, click **+ New Workflow** → **⋮ menu** → **Import from File**
2. Select `workflow-B-approval-publisher.json`

### Configure Workflow B nodes

| Node | What to update |
|------|---------------|
| **Dashboard Approval Webhook** | No changes needed — listens at `/webhook/amgo-approve` |
| **Fetch Workflow Config** | Update URL to point to your dashboard (`/api/settings`) |
| **Fetch Post from API** | Update URL to your dashboard (`/api/posts/{postId}`) |
| **Post Image via Blotato** | Replace `REPLACE_WITH_BLOTATO_IG_ACCOUNT_ID` + credential → `Blotato_AMGO` |
| **Update DB — POSTED** | Update URL to your dashboard (`/api/posts/{postId}`) |
| **Update DB — REJECTED** | Update URL to your dashboard (`/api/posts/{postId}`) |
| **Trigger Workflow A (Regenerate)** | Update URL to your n8n manual trigger webhook |
| **All Telegram nodes** | Credential → `Telegram_AMGO_Bot`, set `chatId` to your Telegram chat ID |

### Activate Workflow B

Toggle the **Active** switch ON. Note the webhook URL:
```
https://your-n8n.com/webhook/amgo-approve
```
This is the URL your dashboard's `N8N_TRIGGER_APPROVE_PATH` will call.

---

## Step 2 — Import Workflow D (Cricket Monitor)

1. **Import** `workflow-D-cricket-monitor.json`
2. Update the **Fetch Workflow Config** URL to your dashboard
3. Update the **Fetch Today Cricket Matches** node with your `CricAPI_AMGO` credential
4. Update **Write Match Day to DB** and **Write Normal Day to DB** URLs to your dashboard
5. Set `Telegram_AMGO_Bot` credential + chat ID on all Telegram nodes
6. **Activate**

---

## Step 3 — Import Workflow F (Post Auto-Expiry)

1. **Import** `workflow-F-post-expiry.json`
2. Update the **Fetch Workflow Config** URL to your dashboard
3. Update **Fetch Stale Pending Posts** URL to your dashboard (`/api/posts?status=PENDING&before=...`)
4. Update **Expire Post** URL to your dashboard (`/api/posts/{postId}`)
5. Set `Telegram_AMGO_Bot` credential + chat ID on Telegram nodes
6. **Activate**

---

## Step 4 — Import Workflow E (Hashtag Intelligence)

1. **Import** `workflow-E-hashtag-intelligence.json`
2. Update the **Fetch Workflow Config** URL to your dashboard
3. Set `OpenAI_AMGO` credential on the GPT-4o-mini node
4. Update **Fetch Upcoming Cricket Matches** URL to your dashboard
5. Update **Sync Hashtags to DB** URL to your dashboard (`/api/hashtags/sync`)
6. Set `Telegram_AMGO_Bot` credential + chat ID on Telegram nodes
7. **Activate**

---

## Step 5 — Import Workflow C (Shopify Sync) — Optional

> ⚠️ Workflow C currently writes to Google Sheets and is not yet fully integrated with the PostgreSQL dashboard. Activate once the Shopify integration is completed.

1. **Import** `workflow-C-shopify-sync.json`
2. Set `Shopify_AMGO` credential on the Shopify HTTP Request node
3. Set your Shopify store domain in the URL
4. Set `Telegram_AMGO_Bot` + chat ID on Telegram nodes
5. **Leave inactive** until Shopify→DB integration is complete

---

## Step 6 — Import Workflow A (Content Generator) — Last

1. **Import** `workflow-A-content-generator.json`

### Configure Workflow A nodes

| Node | What to update |
|------|---------------|
| **Fetch Workflow Config** | Update URL to your dashboard (`/api/settings`) |
| **Fetch Themes from DB** | Update URL to your dashboard (`/api/settings?key=dayThemes`) |
| **Fetch Cricket Match Status** | Update URL to your dashboard (`/api/cricket-matches?date=today`) |
| **GPT-4o Caption & Prompt Generator** | Credential → `OpenAI_AMGO` |
| **Flux 2 Pro Image Generator** | Credential → `FalAI_AMGO` (HTTP Header Auth with Fal.ai key) |
| **Upload Image to imgbb** | Add imgbb API key (as query param `key`) |
| **Log to Database (POST /api/posts)** | URL → your dashboard `/api/posts`; Header `x-webhook-secret` → your `N8N_WEBHOOK_SECRET` |
| **Telegram — Preview Notification** | Credential → `Telegram_AMGO_Bot`, set `chatId` |
| **Send Error Alert** | Credential → `Telegram_AMGO_Bot`, set `chatId` |

> **Cron schedule**: `0 22 * * *` = 8:00 AM AEST. To change:
> - 9 AM AEST → `0 23 * * *`
> - 7 AM AEST → `0 21 * * *`

### Activate Workflow A

Toggle **Active** ON. It fires on the next 8:00 AM AEST.

---

## Manual Test

To test immediately without waiting for the schedule:

1. Open Workflow A in n8n
2. Click **Test workflow** → **Execute workflow**

Within ~60 seconds you should receive:
- 📸 A Telegram photo notification
- A pending post appearing in the dashboard

Then open the dashboard — the pending post card should appear with 3 action buttons (Approve, Regenerate, Reject).

---

## Verification Checklist

- [ ] Workflow B activated (webhook URL: `/webhook/amgo-approve`)
- [ ] Workflow D activated (runs 7:30 AM AEST daily)
- [ ] Workflow F activated (runs 7:50 AM AEST daily)
- [ ] Workflow E activated (runs 7:00 AM AEST Mondays)
- [ ] Workflow A activated (runs 8:00 AM AEST daily)
- [ ] Manual test: Telegram photo notification received
- [ ] Dashboard shows pending post card
- [ ] Dashboard Approve button posts to Instagram (or updates DB if Blotato not yet connected)
- [ ] PostgreSQL Database row updates from `PENDING` → `POSTED`

---

## Troubleshooting

| Issue | Solution |
|-------|---------|
| No Telegram notification received | Check `Telegram_AMGO_Bot` credential, check chat ID (must be numeric, e.g. `-100123456789` for groups) |
| Dashboard approve does nothing | Check `N8N_TRIGGER_APPROVE_PATH` in `.env.local` matches `/webhook/amgo-approve` |
| Database not updating | Verify dashboard URL and `N8N_WEBHOOK_SECRET` are correctly configured in n8n HTTP Request nodes |
| Flux image generation fails | Check `FalAI_AMGO` credential — ensure value is `Key fal_xxxx...` with the "Key " prefix |
| Blotato returns 401 | Re-check API key; ensure Instagram is connected in Blotato |
| Post expiry not running | Check Workflow F is active; verify `/api/posts?status=PENDING&before=...` URL and webhook secret |
