---
name: campaign-performance
description: Build a performance read-out for Infleux campaigns — clicks, conversions, active creators, spend against budget, and click-origin quality — and turn it into a report the brand can act on. Use when someone asks how a campaign is doing, why results dropped, whether a tracking link is firing, where clicks come from, how much a brand has spent, or asks for a weekly/monthly campaign report.
---

# Campaign performance analysis

A performance answer that cites one number is usually wrong. Volume without spend hides
efficiency; spend without click origin hides quality. Pull the picture, then interpret.

## Frame the question first

Establish three things before querying — ask if the user did not say:

- **Which campaign(s)** — resolve names to IDs (`find_campaigns`, `find_brands`).
- **Which window** — `YYYYMMDD` dates. Analytics tools default to the last 7 days and
  cap at 31 days per call. Longer ranges mean consecutive calls; say when you split.
- **What "performing" means here** — installs, conversions, clicks, or spend efficiency.
  `get_campaign(id)` tells you the conversion model the campaign actually pays on.

## The standard pull

```
get_campaign(campaignId)                     # window, payout, conversion type
find_actions(campaignId, pauseState: "active", limit: 100)
query_athena_clicks(template: "origin_breakdown", campaignId, periodStart, periodEnd)
query_athena_spend(template: "campaign_spend_by_ids", campaignIds: [campaignId],
                   periodStart, periodEnd)
```

Add when relevant:

- `find_campaign_views(campaignId)` — how many creators opened the campaign in the app.
  Views high but actions low means the briefing or payout is not converting creators.
- `find_budgets(advertiserBrandId)` — the budget the spend runs against.
- `query_action_tracker_events(actionId)` — per-link clicks vs conversions vs skipped,
  for a specific creator or a test link.
- `query_athena_clicks(template: "by_geo", city|state|country)` — which creators drive
  clicks from a target region.

## Reading the numbers honestly

- **Funnel order**: views → actions (creators running) → clicks → conversions → spend.
  A drop is easiest to explain by naming the first stage where it appears.
- **Clicks without conversions** points at the advertiser's postback, not at the
  creators. Report both figures and name the hypothesis as a hypothesis.
- **`spendExceeded` on an action is not a budget overrun** — it is an informational
  threshold flag. Never report it as overspending.
- **Origin concentration** — one IP, one user agent, or an implausible city dominating
  `origin_breakdown` is a signal worth escalating to the Infleux team. It is not proof of
  fraud, and it is not yours to conclude. Flag it as a pattern to review.
- **Paused actions** (`pauseType` set) still hold historical clicks. Compare like with
  like when explaining a week-over-week drop.
- **Pagination**: `find_actions` caps at 100 rows. Check `pagination.hasMore` before
  saying "N creators are running this" — use `pagination.total`.

## Reporting

Lead with the answer, then the evidence:

1. **Headline** — one sentence: what happened in the window, versus what was expected.
2. **Numbers** — a small table: active creators, views, clicks, conversions, spend, and
   the derived rate that matters for this campaign's model (CPI, CPA, CTR).
3. **Movement** — what changed and where in the funnel it started.
4. **Quality flags** — origin concentration or tracking anomalies, stated as signals.
5. **Next step** — one concrete action, and whether it needs the Infleux dashboard.

State the window and the source of every number. When a figure is missing because the
account lacks a scope, say which data is out of reach rather than working around it.
