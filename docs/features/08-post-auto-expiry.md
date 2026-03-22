# Feature 08 — Post Auto-Expiry (Workflow F)

## Overview

Every day at **7:50 AM AEST** (10 minutes before Workflow A generates new content), Workflow F automatically expires any PENDING posts that are older than 23 hours. This keeps the dashboard clean and prevents stale yesterday's posts from cluttering the approval queue when today's fresh content arrives.

---

## Why This Matters

Without auto-expiry:
- Unapproved posts accumulate in the dashboard
- Yesterday's content is still showing as PENDING when today's post arrives
- The operator could accidentally approve outdated content
- The dashboard becomes cluttered over time

With auto-expiry:
- Dashboard is always clean at 8 AM — only today's fresh posts appear as PENDING
- Post status lifecycle is complete: PENDING → POSTED / REJECTED / **EXPIRED**
- Historical data is preserved (expired posts are visible in the calendar view, just not the approval queue)

---

## How It Works

### Workflow F — Post Auto-Expiry (Daily)

**Runs**: Every day at 7:50 AM AEST (21:50 UTC), 10 minutes before Workflow A

```
Schedule Trigger (7:50 AM AEST)
  ↓
Fetch Workflow Config (from /api/settings)
  ↓
Fetch Stale Pending Posts
  GET /api/posts?status=PENDING&before=<23h ago ISO timestamp>
  ↓
Extract Stale Posts (Code node)
  ↓
Expire Post (for each post)
  PATCH /api/posts/{postId}  →  { status: "EXPIRED" }
  ↓
Aggregate Results
  ↓
Telegram — Posts Expired summary
```

If no posts are stale, Workflow F completes silently without sending a Telegram message.

---

## Expiry Logic

The 23-hour threshold means:
- A post generated at 8:00 AM today will expire at 7:00 AM the next day (23 hours later)
- Workflow F runs at 7:50 AM, catching all posts older than 23 hours
- This gives the operator a full day to review before expiry

```
Workflow A runs at 8:00 AM → creates PENDING post
                              ↓
                         Operator has until ~7:00 AM next day
                              ↓
Workflow F runs at 7:50 AM → expires any PENDING posts older than 23h
                              ↓
Workflow A runs at 8:00 AM → creates fresh PENDING post (clean dashboard)
```

---

## API Interaction

**Fetch stale posts:**
```
GET /api/posts?status=PENDING&before=2026-03-22T21:00:00.000Z
Headers:
  x-webhook-secret: {N8N_WEBHOOK_SECRET}
```

**Expire each post:**
```
PATCH /api/posts/{postId}
Headers:
  x-webhook-secret: {N8N_WEBHOOK_SECRET}
Body:
  { "status": "EXPIRED" }
```

---

## Telegram Notification

After processing, Workflow F sends a summary to Telegram:

```
🗑️ Post Auto-Expiry Complete

Expired 1 stale pending post(s):
• AMGO_2026-03-21_DAILY

Next content generation in 10 minutes.
```

If 0 posts are expired, no Telegram message is sent.

---

## Error Handling

If the API call fails or any step errors, the Error Trigger node:
1. Fetches the current workflow config
2. Sends a Telegram error alert with details

This ensures the operator is aware if expiry fails (so stale posts can be manually resolved before Workflow A runs).

---

## n8n Nodes Used

- `Schedule Trigger` — daily cron at `21 21 * * *` (7:50 AM AEST = 21:50 UTC)
- `HTTP Request` — Fetch Workflow Config (`/api/settings`)
- `HTTP Request` — Fetch Stale Pending Posts (`/api/posts?status=PENDING&before=...`)
- `Code` — Extract Stale Posts (builds list of post IDs to expire)
- `HTTP Request` — Expire Post (PATCH `/api/posts/{postId}` per post)
- `Aggregate` — Collect all results
- `Telegram` — Posts Expired summary
- `Error Trigger` + `HTTP Request` + `Telegram` — error alert pipeline

---

## Schedule Context

| Time (AEST) | Workflow | Purpose |
|-------------|----------|---------|
| 7:30 AM | D | Cricket match detection |
| **7:50 AM** | **F** | **Post expiry cleanup** |
| 8:00 AM | A | Daily content generation |

Workflow F always runs before Workflow A, ensuring a clean slate for the day's new content.
