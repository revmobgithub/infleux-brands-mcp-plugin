---
description: Build a performance read-out for an Infleux campaign — creators, clicks, conversions and spend for a period.
argument-hint: [campaign name] [period, e.g. "last 30 days" or "sep 2026"]
---

Produce a performance report for: **$ARGUMENTS**

Follow the `campaign-performance` skill. In short:

1. Resolve the campaign name with `find_campaigns(nameRegex: ...)`. If more than one
   plausible match comes back, ask which one before querying further.
2. Read the model with `get_campaign(id)` — window, conversion type, payout.
3. Pull, for the requested period (`YYYYMMDD`, max 31 days per call — split and say so if
   the user asked for longer):
   - `find_actions(campaignId, pauseState: "active", limit: 100)`
   - `query_athena_clicks(template: "origin_breakdown", campaignId, periodStart, periodEnd)`
   - `query_athena_spend(template: "campaign_spend_by_ids", campaignIds: [campaignId], periodStart, periodEnd)`
4. Add `find_campaign_views(campaignId)` when the question is about creator adoption.

If no period was given, default to the last 30 days and state that you did.

Report: headline sentence, a metrics table, where in the funnel any drop starts, quality
flags from the click origin (as signals, never as a fraud verdict), and one concrete next
step. Name the window and the source of each number.
