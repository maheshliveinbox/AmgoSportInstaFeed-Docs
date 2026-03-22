# Feature 02 — Cricket Match Intelligence

## Overview

Every morning at **7:30 AM AEST** (30 minutes before the content generator runs), Workflow D checks whether any relevant cricket matches are happening today. If a match is detected, an **additional** Match Day Inspiration post is generated alongside the regular daily theme post — keeping AMGO Sports highly relevant to their community on the most engaged cricket days.

---

## Why This Matters

Cricket match days are the **highest engagement days** for cricket equipment retailers on Instagram. Fans are excited, watching, discussing — and the AMGO Sports audience is no different. Posting only generic product content on a BBL final day is a missed opportunity. This feature ensures:

- **Match days → extra match-day content**, automatically
- The regular daily theme post is still generated (both posts go to PENDING for independent approval)
- No manual calendar management needed
- Content stays culturally relevant and timely
- Higher chance of discovery through trending match hashtags

---

## How It Works

### Step 1 — Fetch Workflow Config

Workflow D first fetches the `workflow_config` from the Dashboard API (`/api/settings`) to get dynamic configuration values.

---

### Step 2 — Fetch Today's Cricket Matches

Workflow D calls the **CricAPI** (`cricapi.com`) to get today's match schedule:

```
GET https://api.cricapi.com/v1/currentMatches
    ?apikey={CRICAPI_KEY}
    &offset=0
```

The response includes all ongoing and upcoming matches globally.

---

### Step 3 — Filter for Australian Matches

The raw response is filtered using a Code node for relevant matches:

**Match criteria (any ONE of these triggers match-day mode):**
- Team name contains: `Australia`, `AUS`, `Australian`
- Series contains: `Big Bash`, `BBL`, `Sheffield Shield`, `Test`
- Venue contains: `Australia`, `Sydney`, `Melbourne`, `Brisbane`, `Perth`, `Adelaide`, `Hobart`

---

### Step 4 — Write Flag to Database

The result is posted to the Dashboard API (`POST /api/cricket-matches`):

**Match detected:**
```
Date       | Is Match Day | Match Title              | Teams               | Venue         | Start Time
2026-03-15 | TRUE         | BBL Final 2026           | Sydney vs Brisbane  | SCG, Sydney   | 19:30 AEST
```

**No match:**
```
Date       | Is Match Day
2026-03-16 | FALSE
```

---

### Step 5 — Telegram Alert (match days only)

When a match is detected, Workflow D sends a Telegram alert:

```
🏏 Match Day Detected! BBL Final 2026 — Sydney Sixers vs Brisbane Heat at SCG

An additional Match Day post will be generated alongside today's regular theme post in 30 minutes.
```

---

### Step 6 — Workflow A Reads the Flag

At the start of Workflow A (content generator), it calls `/api/cricket-matches?date=today`. If `isMatchDay = TRUE`, the theme selector outputs **two items**:
1. The scheduled daily theme (e.g. Wednesday = Brand & New Arrivals)
2. A "Match Day Inspiration" theme referencing the match details

Both items run through the full AI generation pipeline independently, resulting in two separate PENDING posts in the dashboard.

---

## Example Match-Day Output

**Detected match**: BBL Final 2026 — Sydney Sixers vs Brisbane Heat, SCG

**Post 1 (regular daily theme — Wednesday):**
> "Big stock drop this week. 🆕 Fresh arrivals from SS and SG — the bats serious players are buying right now. In store and ready to pick up. Link in bio. 🏏 #amgosports"

**Post 2 (match-day inspiration):**
> "Finals night at the SCG. 🏆 Whether you're watching or playing this weekend — serious cricketers know the gear matters as much as the game. Our range of SS and SG match-day bats are in store now. Who do you have lifting the trophy? 🔥 Link in bio."

Both posts appear in the dashboard as PENDING — the operator approves, rejects, or regenerates each independently.

---

## Configuration

### CricAPI Setup

1. Create a free account at [cricapi.com](https://www.cricapi.com)
2. Free tier: **100 API calls/day** (workflow uses 1/day — well within limits)
3. Copy your API key
4. Add to n8n credential: `CricAPI_AMGO` (HTTP Header Auth → `apikey` header)

### Filtering Logic (editable in Code node)

```javascript
// Adjust these criteria to match your audience
const RELEVANT_TEAMS = ['Australia', 'AUS', 'NSW', 'Queensland', 'Victoria'];
const RELEVANT_SERIES = ['BBL', 'Big Bash', 'Sheffield Shield', 'Test', 'ODI'];
const RELEVANT_VENUES = ['Australia', 'Sydney', 'Melbourne', 'Brisbane', 'Perth'];
```

---

## Fallback Behaviour

| Scenario | Result |
|----------|--------|
| CricAPI is down | Error Trigger → writes fallback (isMatchDay=false) to DB via error path |
| No matches found | Flag written as `isMatchDay=false`, only standard daily theme generated |
| Match found but not Australian | Filtered out, treated as normal day |
| Multiple matches today | First Australian match is used |

---

## Database: Cricket Calendar Matches

**Stored Data:**
```
Date | Is Match Day | Match Title | Teams | Venue | Start Time
```

**Example rows:**
```
2026-03-15 | TRUE  | BBL Final 2026    | SIX vs HEA  | SCG, Sydney | 19:30
2026-03-16 | FALSE |                   |             |             |
2026-03-17 | TRUE  | 3rd Test AUS v IND| AUS vs IND  | MCG, Melb   | 10:30
```

---

## Cost

**Zero additional cost.** CricAPI free tier = 100 calls/day. This workflow uses exactly 1 call per day.
