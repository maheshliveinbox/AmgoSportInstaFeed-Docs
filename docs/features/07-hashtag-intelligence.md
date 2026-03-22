# Feature 07 — Smart Hashtag Intelligence

## Overview

Every **Monday at 7:00 AM AEST**, Workflow E uses **GPT-4o-mini** to generate a fresh, season-aware set of Instagram hashtags tailored to AMGO Sports' audience. The hashtag library is stored in PostgreSQL and automatically used in every generated post. The result is a **dynamic hashtag strategy** that stays relevant throughout the cricket season rather than using the same static 25 tags every day.

---

## Why Smart Hashtags?

Static hashtag sets have well-documented problems on Instagram:

| Problem | Impact |
|---------|--------|
| Shadow-banning overused tags | Posts invisible to non-followers |
| Seasonal irrelevance | #BigBash in off-season signals low-quality content |
| Missing trending opportunities | New viral tags go unnoticed |
| No category diversity | Algorithm needs mix of large + small reach tags |

A weekly AI refresh solves all of these by:
- Removing oversaturated tags
- Adding cricket-season relevant tags (e.g. `#BBL` during Big Bash season)
- Incorporating trending community hashtags
- Maintaining optimal tag mix by category

---

## How It Works

### Workflow E — Hashtag Intelligence Refresh (Weekly)

**Runs**: Every Monday at 7:00 AM AEST (21:00 UTC Sunday), or manually via `/webhook/amgo-hashtag-refresh`

```
Schedule Trigger (or Manual Webhook)
  ↓
Fetch Workflow Config (from /api/settings)
  ↓
Fetch Upcoming Cricket Matches (/api/cricket-matches?upcoming=7)
  ↓
Build Context for GPT-4o (season detection, match calendar)
  ↓
GPT-4o-mini Hashtag Generator
  ↓
Parse & Validate Hashtags
  ↓
Sync Hashtags to DB (POST /api/hashtags/sync — atomic replace)
  ↓
Build Confirmation Summary
  ↓
Telegram: "🏷️ Hashtag library refreshed for week of {date} — {count} tags synced"
```

### GPT-4o-mini Hashtag Prompt

```
You are a professional Instagram hashtag strategist for AMGO Sports,
a premium cricket equipment retailer in Hills Shire, NSW, Australia.

Current date: {date}
Current month: {month}
Upcoming cricket matches: {matches_from_cricapi}

Generate exactly 30 optimised Instagram hashtags for AMGO Sports.
Categorise them as:
- Brand (5 tags): AMGO Sports specific branded tags
- Product (8 tags): Cricket equipment, gear, and brands stocked (SS, SG, CA, GM, DSC, TON, Kookaburra)
- Cricket (8 tags): General cricket tags relevant RIGHT NOW given current series
- Regional (5 tags): Australian-specific, Hills Shire, NSW cricket community tags
- Trending (4 tags): Currently high-performing cricket Instagram tags for {month}

Rules:
- All tags must start with #
- No spaces in tags
- Mix of high-reach (1M+) and niche (10K-100K) tags
- Australian English where applicable
- Include current BBL/Test/ODI series tags if active season

Respond as a valid JSON object:
{
  "brand": ["#tag1", ...],
  "product": ["#tag1", ...],
  "cricket": ["#tag1", ...],
  "regional": ["#tag1", ...],
  "trending": ["#tag1", ...]
}
```

### Atomic Database Sync

The `/api/hashtags/sync` endpoint atomically replaces all 30 hashtags in a single database transaction — deleting all existing hashtags and inserting the new 30. This prevents stale tags from accumulating.

### How Tags Are Used in Posts

When Workflow A generates a post, it reads from the hashtag library via the Dashboard API. The 25 hashtags included in each post caption come from this weekly-refreshed set.

---

## Example Hashtag Sets by Cricket Season

### Big Bash Season (December — February)
```
Brand:    #amgosports #amgosportsau #amgocricket #trybeforeyoubuy #hillsshirecricket
Product:  #cricketbat #cricketgear #cricketequipment #SSbats #SGbats #cricketbats #batsman #cricketshop
Cricket:  #BBL #BigBash #BigBashLeague #BBLfinals #T20cricket #T20 #cricketaustralia #cricket
Regional: #nswcricket #sydneycricket #hillsshire #australiacricket #cricketAUS
Trending: #BBL16 #CricketFreaks #CricketFamily #GrassrootsCricket
```

### Test Cricket Season (November — January)
```
Cricket:  #TestCricket #TheAshes #AUSvIND #TestMatch #RedBall #cricketaustralia #cricket #baggygreen
Trending: #AshesTest #cricketfans #CricketWar #TestNation
```

### Off-Season (March — September)
```
Cricket:  #cricket #crickettraining #cricketpractice #cricketcoaching #cricketers #grassrootscricket #cricket365 #cricketlife
Trending: #cricketoffseason #cricketworkout #battraining #bowlingcoach
```

---

## Manual Refresh from Dashboard

On the dashboard's **Hashtags page**, the **"Refresh with AI"** button manually triggers Workflow E. Use this when:
- A major cricket series just started
- A trending cricket moment is happening (unexpected viral match)
- You want to test a new season's hashtag strategy before Monday

---

## Database: Hashtag Library

**Schema:**
```
Category | Hashtag | Week | CreatedAt
```

**Example rows:**
```
Brand    | #amgosports        | 2026-W12 | 2026-03-16
Brand    | #amgosportsau      | 2026-W12 | 2026-03-16
Cricket  | #BBL               | 2026-W12 | 2026-03-16
Trending | #cricketfreaks     | 2026-W12 | 2026-03-16
```

The `week` field tracks when each hashtag set was generated, visible in the dashboard Hashtags page.

---

## Cost

One GPT-4o-mini API call per week:
- Input: ~500 tokens (system + prompt)
- Output: ~400 tokens (30 hashtags as JSON)
- Cost: ~$0.001 USD per week = **effectively free**

GPT-4o-mini was chosen over GPT-4o for this task as hashtag generation doesn't require the full capability of GPT-4o — the smaller model performs equivalently at a fraction of the cost.
