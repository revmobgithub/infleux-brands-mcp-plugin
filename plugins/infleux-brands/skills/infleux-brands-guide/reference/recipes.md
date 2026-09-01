# Worked flows

Each recipe is the shortest correct path. Stop and ask the user whenever a step returns
`ambiguous_brand`, more than one plausible campaign, or `hasMore: true` on data you are
about to summarize as complete.

## "What is my brand running right now?"

```
find_live_campaigns_for_brand(brandNameRegex: "acme")
  → status: "ok"        → report campaigns (name, window, payout, conversion type)
  → status: "ambiguous_brand" → show the candidates, ask which one
  → status: "no_brand"  → retry find_brands with a shorter nameRegex
```

For the whole platform instead of one brand:
`find_campaigns(onlyActiveCampaigns: true, limit: 100)`, then page with `skip: nextSkip`.

## "Send me the briefing for campaign X"

```
find_campaigns(nameRegex: "x")        → confirm the campaign with the user
get_campaign(id: <campaignId>)        → briefing, rules, payouts, dates
```

Report the rules as written. Do not paraphrase payout terms or deadlines loosely —
creators and finance both read them literally.

## "How is campaign X performing?"

```
get_campaign(id)                                        → window + conversion model
find_actions(campaignId, pauseState: "active", limit: 100)  → creators actually running it
query_athena_clicks(template: "origin_breakdown", campaignId, periodStart, periodEnd)
query_athena_spend(template: "campaign_spend_by_ids", campaignIds: [id], periodStart, periodEnd)
```

Keep the window under 31 days. If the user asks for a longer range, split it into
consecutive queries and say that you did.

## "Where are the clicks coming from?" / fraud smell check

```
query_athena_clicks(template: "origin_breakdown", campaignId | actionId | brandId,
                    topN: 20, periodStart, periodEnd)
```

Read the breakdown as *signals*, not verdicts: a single IP or user agent dominating, or a
city with no relation to the audience, is worth flagging to the Infleux team — it is not
proof of fraud on its own. Say that distinction out loud.

To rank creators by clicks from one place:
`query_athena_clicks(template: "by_geo", city | state | country, minClicks: 30)`.

## "Did my tracking link work?"

```
find_actions(campaignId, ...)             → pick the actionId
query_action_tracker_events(actionId, includeSamples: true)
```

Returns aggregates across clicks, conversions and skipped conversions. Zero conversions
with healthy clicks usually means the advertiser's postback is not firing — report both
numbers rather than a conclusion.

## "How much have we spent this month?"

```
find_advertiser_brands(...)  → advertiserBrandId
query_athena_spend(template: "advertiser_brand_spend", advertiserBrandId,
                   periodStart: "20260901", periodEnd: "20260930")
find_budgets(advertiserBrandId)          → the budget the spend runs against
```

Spend templates: `advertiser_brand_spend`, `campaign_spend_by_ids`,
`campaign_spend_by_advertiser_brand`. Pick by what the user already gave you.

## "Who is waiting on us?"

```
find_campaign_queue(campaignId, queueStage: "brand")             → creators awaiting brand review
find_content_approval_queue(campaignId, approvalStage: "brand")  → content awaiting brand review
```

Both tools are **read-only**. Claude cannot approve or reject — point the user to the
Infleux dashboard for the decision itself, and offer to summarize what is pending.

## "Run campaign X again next month"

```
get_campaign(id)                                     → confirm this is the right source
clone_campaign_to_pre_campaign(campaignId)           → PREVIEW (no confirmed)
   ↳ ask the user for startsAt / endsAt — the clone never copies dates
clone_campaign_to_pre_campaign(campaignId, confirmed: true, startsAt, endsAt, name?)
```

The clone lands as a `pending` pre-campaign with margin zeroed and a versioned name
suggestion (v3 → v4). It does **not** publish or approve anything.

## "Change the briefing on the draft"

```
find_pre_campaigns(status: "pending", brandId)   → locate the draft
get_pre_campaign(id)                             → current state
update_pre_campaign(id, updates: { ... })        → PREVIEW (no confirmed)
   ↳ show the patch, get explicit approval
update_pre_campaign(id, updates: { ... }, confirmed: true)
```

Send nested objects (`payouts`, `collectContentReels`) whole — a partial sub-object
overwrites the rest. Published pre-campaigns are rejected by the API; say so plainly
instead of looking for a workaround.
