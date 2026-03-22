# Feature 01 — Daily AI Content Generation

## Overview

The core engine of the AMGO Sports Content Engine. Every day at **8:00 AM AEST**, an n8n workflow automatically generates one or two complete, publication-ready Instagram posts using a chain of AI models.

- **Normal day:** 1 post generated (`AMGO_YYYY-MM-DD_DAILY`)
- **Match day:** 2 posts generated — daily theme + Match Day Inspiration (`AMGO_YYYY-MM-DD_DAILY` + `AMGO_YYYY-MM-DD_MATCH`)

---

## How It Works

### Step 1 — Fetch Workflow Config

Workflow A starts by fetching the `workflow_config` settings object from the Dashboard API (`/api/settings`). This provides dynamic configuration such as the app URL and n8n webhook secret without hardcoding values into the workflow.

---

### Step 2 — Fetch Themes from DB

The current set of daily themes is read from the Dashboard API (`/api/settings?key=dayThemes`). Themes are stored in the database, allowing them to be updated from the dashboard settings page without modifying the workflow.

---

### Step 3 — Fetch Cricket Match Status

An API call to `/api/cricket-matches?date=today` checks whether Workflow D flagged today as a match day. The response determines whether one or two posts are generated.

---

### Step 4 — Theme Selection (Code Node)

A JavaScript code node determines today's theme based on the day of the week:

| Day | Theme | Emoji | Focus |
|-----|-------|-------|-------|
| Sunday | Behind the Scenes | 🎬 | Store atmosphere, team passion, Try Before You Buy program |
| Monday | Product Spotlight | 🏏 | Premium bat or gear — craftsmanship, performance, willow quality |
| Tuesday | Cricket Tips | 💡 | Batting/bowling technique, skill development |
| Wednesday | Brand & New Arrivals | 🆕 | Brand heritage, newly arrived stock |
| Thursday | Cricket Lifestyle | 🏃 | Australian cricket culture, Hills Shire community |
| Friday | Weekend Deal | 🔥 | Promotional energy, weekend cricketers |
| Saturday | Match Day Inspiration | 🏆 | Elite cricket imagery, match-day motivation |

**On match days:** The code node outputs **two items** — the scheduled daily theme (item #1) and a "Match Day Inspiration" theme (item #2). n8n runs the full downstream pipeline independently for each item in parallel.

---

### Step 5 — Caption & Prompt Generation (GPT-4o)

GPT-4o receives a structured system prompt defining AMGO Sports' brand voice and generates:

```json
{
  "caption": "200-280 character Instagram caption with hook, value prop, CTA, and 3-5 emojis",
  "hashtags": "Exactly 25 hashtags starting with #amgosports #amgosportsau",
  "dalle_prompt": "Detailed image generation prompt for professional sports product photography",
  "alt_text": "Short descriptive alt text for accessibility",
  "theme_tag": "One-word category: PRODUCT | TIPS | LIFESTYLE | PROMO | BRAND | MATCHDAY | BTS"
}
```

**Brand Voice Rules (enforced in system prompt):**
- Australian English (colour, favourite, practise, centre)
- Audience: Serious cricketers aged 16–45
- 3–5 emojis maximum — professional, not playful
- Always include a subtle CTA (DM us / Link in bio / Visit amgosports.com.au)
- Brands stocked: SS, SG, SF, CA, GM, DSC, TON, Kookaburra
- Never use US English or generic clichés

---

### Step 6 — Image Generation (Flux 2 Pro via Fal.ai)

The image prompt from GPT-4o is submitted to **Flux 2 Pro** via the Fal.ai HTTP API:

- **Model**: `fal-ai/flux-pro/v1.1`
- **Output**: High-quality 1024×1024 image
- **Auth**: `FAL_API_KEY` environment variable

> Flux 2 Pro produces professional-grade product imagery suitable for Instagram. It replaced DALL·E 3 to provide higher quality at lower cost.

---

### Step 7 — Image Upload to imgbb

The Flux-generated image URL is uploaded to **imgbb.com** for a permanent, public URL. This permanent URL is stored in the database and sent to Blotato for Instagram publishing.

**Why imgbb?** Free, no account required for API usage, permanent URLs, no bandwidth limits for this use case.

---

### Step 8 — Database Logging

The post is immediately logged to the **PostgreSQL Database** via `POST /api/posts` with `x-webhook-secret` header auth. Status is set to `PENDING`. The `postId` uses a suffix:
- `AMGO_YYYY-MM-DD_DAILY` — the regular daily theme post
- `AMGO_YYYY-MM-DD_MATCH` — the match-day post (only on match days)

---

### Step 9 — Telegram Notification

A **sendPhoto** message is sent to the configured Telegram chat with the full-size image and post details:

```
🏏 AMGO Sports — Today's Post is Ready!

🆕 Brand & New Arrivals
📅 Tuesday, 2026-03-15

Check the dashboard to review and approve.

🔗 Dashboard: https://dashboard.amgosports.com.au
```

> **Notification only** — no inline buttons. All approval actions happen exclusively in the Next.js dashboard.

On match days, **two Telegram notifications** are sent (one per post).

---

### Step 10 — Awaiting Dashboard Action

After the notification is sent, the workflow ends. The post(s) are stored as `PENDING` in the database. The operator opens the dashboard and takes one of 3 actions (Approve, Regenerate, or Reject), which triggers **Workflow B** via a POST webhook.

---

## AI Model Choices

### Why GPT-4o (not GPT-3.5)?

| Consideration | Detail |
|--------------|--------|
| Quality | GPT-4o produces significantly more nuanced, on-brand copy |
| JSON compliance | Very reliable structured JSON output |
| Cricket knowledge | Strong understanding of cricket terminology and brands |
| Cost | ~$0.05 per post (negligible vs value) |

### Why Flux 2 Pro (not DALL·E 3)?

| Consideration | Detail |
|--------------|--------|
| Quality | Flux 2 Pro produces professional-grade imagery comparable to DALL·E 3 HD |
| Cost | Lower per-image cost than DALL·E 3 HD |
| API access | Direct HTTP API via Fal.ai — simple integration |
| Photorealism | Excellent for sports product photography |

---

## Caption Structure

All generated captions follow this structure:

```
[HOOK — 5 words max that stop the scroll]

[VALUE — 1-2 sentences: what's special, what the cricketer gains]

[CTA — DM us / Link in bio / Visit amgosports.com.au]

[2-3 emojis woven naturally]
```

Example output:
> "The perfect mid-season upgrade. 🏏 SS Ton players' edition — hand-crafted English willow, extra-hard pressing for that satisfying power on the cut shot. Built for cricketers who take their game seriously. DM us to book a Try Before You Buy session. 🔥 #amgosports"

---

## Error Handling

| Failure Point | Fallback Behaviour |
|--------------|-------------------|
| OpenAI API down | Error Trigger node → Telegram error alert with details |
| Fal.ai API error | Error Trigger node → Telegram error alert |
| imgbb upload failure | Error Trigger node → Telegram error alert |
| Database write failure | Error Trigger node → Telegram error alert |

All errors are caught by n8n's Error Trigger node which fetches the current config and sends a detailed Telegram alert.

---

## n8n Nodes Used

- `Schedule Trigger` — daily cron at `0 22 * * *` (8 AM AEST = 22:00 UTC)
- `Webhook` — manual trigger at `/webhook/amgo-manual-trigger`
- `HTTP Request` — Fetch Workflow Config (`/api/settings`)
- `HTTP Request` — Fetch Themes from DB (`/api/settings?key=dayThemes`)
- `HTTP Request` — Fetch Cricket Match Status (`/api/cricket-matches?date=today`)
- `Code` — Select Today Theme (outputs 1 or 2 items based on match day)
- `OpenAI` (langchain) — GPT-4o caption & prompt generation
- `Code` — Parse AI Output
- `HTTP Request` — Flux 2 Pro Image Generator (Fal.ai API)
- `Set` — Merge Caption + Image URL
- `HTTP Request` — Upload Image to imgbb
- `Code` — Build Complete Post Package
- `HTTP Request` — Log to Database (`POST /api/posts`)
- `HTTP Request` — Download Image for Telegram
- `Telegram` — Preview Notification (photo message)
- `Error Trigger` + `HTTP Request` + `Telegram` — error alert pipeline
