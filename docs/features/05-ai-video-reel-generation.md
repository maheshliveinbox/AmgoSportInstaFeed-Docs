# Feature 05 — AI Video Reel Generation

## Status: Not Currently Implemented

> ⚠️ **This feature has been removed from the active workflows.** Workflow B no longer includes RunwayML video generation. The approval flow currently supports static image posting only (approve, reject, regenerate).

---

## Background

This feature was previously implemented in Workflow B using RunwayML Gen-3 Alpha. When the operator selected "Approve as Reel", the static image was animated into a 5-second video and posted to Instagram as a Reel.

It was removed during the workflow refactoring to simplify the approval flow and reduce operational complexity and cost.

---

## Previously Implemented Flow (for reference)

1. Operator selected **🎬 Approve as Reel** in the dashboard
2. Workflow B submitted the image to RunwayML `image_to_video` API
3. RunwayML returned a task ID and generated video asynchronously (~30-90s)
4. Workflow B polled the task status every 15 seconds
5. On completion, the video URL was posted to Instagram as a Reel via Blotato
6. Post status updated to `POSTED` with `isReel: TRUE`

---

## Re-implementing Reels

If you want to re-add reel generation, you would need to:

1. Add a RunwayML credential (`RunwayML_AMGO`) in n8n
2. Add a 4th route to Workflow B's Switch node for `approve_reel`
3. Add an "Approve as Reel" button back to `PendingPostCard.tsx`
4. Add the RunwayML polling loop (HTTP Request → Wait → HTTP Request)
5. Use Blotato's video/reel endpoint instead of the image endpoint

The RunwayML API key environment variable (`RUNWAY_API_KEY`) is still present in `.env.local.example` for this purpose.

---

## Why Reels Matter (context)

Instagram's algorithm heavily favours Reels:

| Metric | Static Post | Reel |
|--------|------------|------|
| Typical organic reach | ~5-8% of followers | ~15-25% of followers |
| Discoverability (non-followers) | Low | High (Reels tab) |
| Engagement rate | Baseline | ~2-3× higher |

When re-implemented, cost per reel would be approximately **AUD $0.39** (5s, Gen-3 Turbo).
