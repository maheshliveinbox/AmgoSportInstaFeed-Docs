# AMGO Sports — n8n Workflows

All 6 n8n workflow JSON files are in this folder. Import them into your n8n instance in the order listed below.

> For full setup instructions see [Setup Guide 04](../docs/setup/04-n8n-import.md)

---

## Workflows

| File | Workflow | Trigger | Runs |
|------|----------|---------|------|
| `workflow-A-content-generator.json` | **A — Daily Content Generator** | Schedule + Dashboard button | 8:00 AM AEST daily |
| `workflow-B-approval-publisher.json` | **B — Approval & Publisher** | Dashboard webhook POST | On approve/reject button |
| `workflow-C-shopify-sync.json` | **C — Shopify Product Sync** | Schedule + Dashboard button | 6:00 AM AEST Sundays |
| `workflow-D-cricket-monitor.json` | **D — Cricket Match Intelligence** | Schedule | 7:30 AM AEST daily |
| `workflow-E-hashtag-intelligence.json` | **E — Hashtag Intelligence** | Schedule + Dashboard button | 7:00 AM AEST Mondays |
| `workflow-F-post-expiry.json` | **F — Post Auto-Expiry** | Schedule | 7:50 AM AEST daily |

---

## Approval Architecture

> **No Telegram inline buttons are used.** Telegram is notification-only.

```
Workflow A generates post
  → Telegram: photo preview + post details
  → Dashboard: operator approves/rejects/regenerates
  → Dashboard calls POST /webhook/amgo-approve
  → Workflow B executes the action
  → Telegram: outcome alert (posted/rejected/regenerating)
```

---

## Activation Order

> ⚠️ **Activate in this exact order.**

```
1. Activate Workflow B  →  dashboard webhook ready
2. Activate Workflow D  →  runs 30 min before A
3. Activate Workflow E  →  runs 1 hr before A on Mondays
4. Activate Workflow C  →  syncs products before Monday's post
5. Activate Workflow F  →  post expiry cleanup (activate before A)
6. Activate Workflow A  →  main daily generator (activate last)
```

---

## Timing Summary (AEST)

```
Sunday    06:00  → Workflow C: Shopify product sync
Monday    07:00  → Workflow E: Hashtag intelligence refresh
Daily     07:30  → Workflow D: Cricket match detection
Daily     07:50  → Workflow F: Auto-expire stale PENDING posts
Daily     08:00  → Workflow A: Content generation + notifications
On demand        → Workflow B: Triggered by dashboard actions
```

---

## Credentials Required

| Credential Name | Type | Used By |
|----------------|------|---------|
| `OpenAI_AMGO` | OpenAI API | A, E |
| `FalAI_AMGO` | HTTP Header Auth | A |
| `Imgbb_AMGO` | HTTP Query Auth | A |
| `Telegram_AMGO_Bot` | Telegram API | A, B, C, D, E, F |
| `GoogleSheets_AMGO` | Google Sheets OAuth2 | C |
| `Blotato_AMGO` | HTTP Header Auth | B |
| `CricAPI_AMGO` | HTTP Query Auth | D |

> Shopify is accessed via n8n environment variables (`SHOPIFY_STORE_DOMAIN`, `SHOPIFY_ACCESS_TOKEN`) — no named n8n credential needed.

---

## Dashboard .env Webhook Paths

```bash
N8N_TRIGGER_GENERATE_PATH=/webhook/amgo-manual-trigger
N8N_TRIGGER_APPROVE_PATH=/webhook/amgo-approve
N8N_TRIGGER_PRODUCT_FEATURE_PATH=/webhook/amgo-product-feature
N8N_TRIGGER_HASHTAG_REFRESH_PATH=/webhook/amgo-hashtag-refresh
N8N_TRIGGER_SHOPIFY_SYNC_PATH=/webhook/amgo-shopify-sync
```

---

## Placeholder Replacements

Before activating, search and replace in each JSON file:

| Placeholder | Replace with |
|------------|-------------|
| `REPLACE_WITH_OPENAI_CREDENTIAL_ID` | Your OpenAI_AMGO credential ID |
| `REPLACE_WITH_FALAI_CREDENTIAL_ID` | Your FalAI_AMGO credential ID |
| `REPLACE_WITH_IMGBB_CREDENTIAL_ID` | Your Imgbb_AMGO credential ID |
| `REPLACE_WITH_TELEGRAM_CREDENTIAL_ID` | Your Telegram_AMGO_Bot credential ID |
| `REPLACE_WITH_TELEGRAM_CHAT_ID` | Your numeric Telegram chat ID |
| `REPLACE_WITH_GSHEETS_CREDENTIAL_ID` | Your GoogleSheets_AMGO credential ID |
| `REPLACE_WITH_GOOGLE_SHEET_ID` | Your Google Sheet ID |
| `REPLACE_WITH_BLOTATO_CREDENTIAL_ID` | Your Blotato_AMGO credential ID |
| `REPLACE_WITH_BLOTATO_IG_ACCOUNT_ID` | Instagram account ID from Blotato |
| `REPLACE_WITH_CRICAPI_CREDENTIAL_ID` | Your CricAPI_AMGO credential ID |
| `REPLACE_WITH_DASHBOARD_URL` | Your deployed dashboard URL |
| `YOUR_N8N_URL` | Your n8n instance base URL |

> **Tip:** Use Ctrl+H in VS Code to batch-replace across all files at once.
