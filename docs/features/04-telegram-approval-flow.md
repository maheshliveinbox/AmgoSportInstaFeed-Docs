# Feature 04 — Notification & Approval Flow

## Overview

Each day after a post is generated, the operator receives a **Telegram notification** with a photo preview. All approval actions (approve, regenerate, reject) happen exclusively in the **Next.js Dashboard**. This makes the dashboard the single source of truth for content decisions.

> **Note:** Email notifications were removed in favour of Telegram-only notifications. The dashboard provides a clean, centralised approval interface.

---

## Flow Diagram

```
8:00 AM AEST — Workflow A runs
         ↓
  Generate post(s) (GPT-4o + Flux 2 Pro)
         ↓
  Upload to imgbb → permanent URL
         ↓
  POST /api/posts → DB (Status: PENDING)
         ↓
  Telegram — sendPhoto notification
  "Check the dashboard to review and approve"
         ↓
  Operator opens dashboard
         ↓
  Dashboard pending card shows:
  ┌─────────────────────────────────────────────┐
  │  [Image]  Caption  Hashtags                 │
  │  [✅ Approve Image]  [✏️ Regenerate]          │
  │  [❌ Reject]                                 │
  └─────────────────────────────────────────────┘
         ↓ (on action)
  Dashboard → POST /api/approve/:postId  (approve / regenerate)
           OR POST /api/reject/:postId   (reject)
         ↓
  Workflow B executes chosen action
         ↓
  Telegram: outcome alert (✅ Posted / ❌ Rejected / ✏️ Regenerating / ⚠️ Error)
```

On **match days**, two Telegram notifications are sent — one per post — and both appear as separate PENDING cards in the dashboard.

---

## Telegram Notification (Workflow A — on generation)

Workflow A sends a **sendPhoto** message (no inline buttons):

```
🏏 AMGO Sports — Today's Post is Ready!

🆕 Brand & New Arrivals
📅 Tuesday, 2026-03-15

Check the dashboard to review and approve.

🔗 Dashboard: https://dashboard.amgosports.com.au
```

**Purpose:** Quick visual check. The image appears natively in the notification. Head to the dashboard to take action.

---

## Dashboard Approval Actions

`src/components/dashboard/PendingPostCard.tsx` renders the pending post card with 3 actions:

| Button | API call | Action value sent to n8n | What happens |
|--------|----------|--------------------------|-------------|
| ✅ Approve Image | `POST /api/approve/[postId]` | `action: "approve_image"` | Posts static image to Instagram via Blotato |
| ✏️ Regenerate | `POST /api/approve/[postId]` | `action: "regenerate"` | Workflow B triggers Workflow A → new preview in ~60s |
| ❌ Reject | `POST /api/reject/[postId]` | _(no body needed)_ | Marks as REJECTED in DB, no post today |

**Approve/Regenerate API call format** (`src/app/api/approve/[postId]/route.ts`):
```json
POST /api/approve/{postId}
Body: {
  "action": "approve_image",
  "approved_by": "Dashboard"
}
```

**Reject API call format** (`src/app/api/reject/[postId]/route.ts`):
```json
POST /api/reject/{postId}
Body: {}
```

Both routes call `src/lib/n8n.ts` → forward to Workflow B's webhook.

---

## Workflow B — Dashboard Webhook

`n8n-workflows/workflow-B-approval-publisher.json` listens at:
```
POST https://your-n8n.com/webhook/amgo-approve
```

Workflow B fetches the workflow config, then routes on the `action` field:

| Action | Route taken |
|--------|-------------|
| `approve_image` | Fetch post from DB → Blotato image post → Wait 30s → Poll status → Update DB to POSTED → Telegram alert |
| `regenerate` | Trigger Workflow A webhook → Telegram "Regenerating..." alert |
| `reject` | Update DB to REJECTED → Telegram rejection alert |

**Error handling:** n8n Error Trigger node → fetches config → Telegram error alert with details.

---

## Outcome Notifications (Telegram — sent by Workflow B)

| Outcome | Telegram message |
|---------|-----------------|
| ✅ Posted (image) | "Posted to Instagram! 🏏 [Theme] [Date] 🔗 link" |
| ❌ Rejected | "Post Rejected — [post_id]. Next post: tomorrow 8 AM." |
| ✏️ Regenerating | "Regenerating Post... Check Telegram for new preview shortly." |
| ⚠️ Error | "AMGO Sports — Publish Error: [error details]" |

---

## Post Auto-Expiry (Workflow F)

If a post remains PENDING and is not approved or rejected, **Workflow F** automatically expires it at **7:50 AM AEST** (10 minutes before new content arrives). This prevents the dashboard from accumulating stale PENDING posts.

See [Post Auto-Expiry](./08-post-auto-expiry.md) for full details.

---

## Implementation Status

### Fully Implemented
- [x] Workflow A: Telegram sendPhoto notification on generation
- [x] Workflow B: approve_image, regenerate, reject routing
- [x] Dashboard `PendingPostCard` with 3 action buttons
- [x] `POST /api/approve/[postId]` — handles approve_image, regenerate
- [x] `POST /api/reject/[postId]` — handles rejection
- [x] Post status lifecycle: PENDING → POSTED / REJECTED / FAILED / EXPIRED
- [x] Audit log on all actions
- [x] Telegram outcome alerts (success, reject, regenerate, error)
- [x] Post auto-expiry (Workflow F)
- [x] Dual post generation on match days (daily + match day posts)
