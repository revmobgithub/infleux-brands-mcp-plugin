---
name: infleux-brands-guide
description: Ground rules for working with Infleux data — how to resolve brands and campaigns to IDs, which tool answers which question, how pagination and permission scopes behave, and how to read Infleux domain terms (live campaign, action, casting, pre-campaign). Use whenever a request touches Infleux campaigns, brands, influencers, clicks, conversions, spend, castings, content approval, or pre-campaigns, and before the first `mcp__plugin_infleux-brands_infleux__*` call in a conversation.
---

# Working with Infleux data

Infleux is an influencer marketing platform: brands run campaigns, creators pick them up
in the Infleux app and generate tracked links, and the platform pays per performance
(installs, conversions) or on a fixed fee. This plugin connects Claude to the Infleux
Core MCP server (`https://mcp.infleux.io/mcp`) over OAuth — the data you read is the
same live data the Infleux dashboard shows.

Tools reach you as `mcp__plugin_infleux-brands_infleux__<tool_name>`. This guide uses the
bare names (`find_campaigns`, `query_athena_clicks`, …).

## Start every session on solid ground

1. `ping_core` — confirms the session is authenticated and Core API is reachable.
2. `whoami` — returns the signed-in user. Use it when the answer depends on *who* is
   asking (which brand they belong to, what they are allowed to see).

If either fails with a 401, the OAuth session expired: tell the user to run `/mcp`,
select **infleux**, and re-authenticate. Never ask the user to paste a token.

## Names are not IDs — resolve first

Almost every Infleux tool filters by ObjectId, and users speak in names. Resolve before
you query, and never guess an ID:

| The user says | Resolve with | Then pass |
| --- | --- | --- |
| a brand ("Nubank") | `find_brands(nameRegex)` | `brandId` |
| an advertiser | `find_advertisers(nameRegex)` | `advertiserId` |
| a brand under an advertiser | `find_advertiser_brands` | `advertiserBrandId` |
| a campaign name | `find_campaigns(nameRegex)` | `campaignId` |

Shortcut: *"which campaigns is brand X running right now?"* is a single call —
`find_live_campaigns_for_brand(brandNameRegex)`. It returns `status: "ambiguous_brand"`
with candidates when the name matches more than one brand. **Ask the user to pick.**
Do not silently take the first match.

For *all* live campaigns across every brand, use
`find_campaigns(onlyActiveCampaigns: true)` with no `brandId` — not the brand-scoped tool.

## Which tool answers which question

| Question | Tool |
| --- | --- |
| What is running now? | `find_campaigns(onlyActiveCampaigns: true)`, `find_live_campaigns_for_brand` |
| What are the rules / briefing? | `get_campaign(id)` |
| Who was selected for a campaign? | `get_campaign_casting` |
| Who is waiting for approval to join? | `find_campaign_queue` |
| Which content is pending review? | `find_content_approval_queue` |
| How many creators opened the campaign? | `find_campaign_views` |
| Which creators are actively running it? | `find_actions(campaignId, pauseState: "active")` |
| How many clicks, and from where? | `query_athena_clicks` |
| Did my test click/conversion land? | `query_action_tracker_events(actionId)` |
| How much did we spend? | `query_athena_spend`, `find_budgets` |
| What is in the pipeline before launch? | `find_pre_campaigns`, `get_pre_campaign` |
| Draft a new edition of a campaign | `clone_campaign_to_pre_campaign` |
| Change a draft before launch | `update_pre_campaign` |

Field contracts and query grammar for the trickier services live behind
`get_query_reference(topic)` — call it instead of guessing filter shapes.

## Pagination is not optional

Every `find_*` tool returns at most 100 rows (default 50) plus a `pagination` block:
`total`, `count`, `skip`, `limit`, `hasMore`, `nextSkip`.

- Before summarizing, check `hasMore`. If it is `true`, either page through with
  `skip: nextSkip` or state plainly that the answer covers the first N of `total`.
- Never present a truncated page as a complete count. Use `total` for counts.

## Lean by default

List tools return a small projection. Add `select: [...]` for extra fields, and reach for
`fullDocument: true` only when the user genuinely needs everything — full documents are
large and slow. For one record, `get_campaign(id)` / `get_pre_campaign(id)` beats
`fullDocument` on a list call.

## Permissions: a missing tool is not a broken tool

Access is scoped to the signed-in user's Infleux role. A brand/advertiser account can
read brands, campaigns, castings, queues, actions, budgets, spend and click analytics,
and can read *and write* pre-campaigns. It cannot browse the influencer database or
creator earnings — those tools return `Missing scope: <scope>` (HTTP 403).

When that happens, say so directly: the account does not have access to that data, and
their Infleux account manager can review it. Do not retry, and do not try to reach the
same data through another tool.

## Writes are always two-step

`update_pre_campaign` and `clone_campaign_to_pre_campaign` are the only tools that change
anything, and both refuse to write without `confirmed: true`.

1. Call **without** `confirmed` → you get a preview (summary + the exact patch).
2. Show that preview to the user and ask for explicit approval.
3. Only then call again with `confirmed: true`.

Never send `confirmed: true` on the first call, even when the user's request sounds
decisive. Published campaigns cannot be edited at all — a pre-campaign with `status:
published` or an existing `campaignId` is locked.

## Domain terms to read correctly

- **Campaign** — published and running on the platform. **Pre-campaign** — the draft
  pipeline before launch (`status: "pending"` is the approval queue).
- **Live / active campaign** — `onlyActiveCampaigns: true`. This is about the campaign's
  window, not about an influencer's status.
- **Action** — one influencer's participation in one campaign, and the tracked link
  behind it. Active = no `pauseType`; paused = `pauseType` set.
- **Casting** — the selection of creators for a campaign; **campaign queue** is the
  approval funnel (`cm` → `am` → `brand`), **content approval** is the review of the
  content the creator produced.
- `spendExceeded` on an action is an informational threshold flag, **not** a budget
  overrun. Do not report it as one.

Load `reference/recipes.md` for worked end-to-end flows, and `reference/tools.md` for the
full tool list with required scopes.
