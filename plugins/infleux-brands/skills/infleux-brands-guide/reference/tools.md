# Infleux MCP tool reference

Tools are exposed to Claude as `mcp__plugin_infleux-brands_infleux__<tool_name>`.
The **Scope** column is the permission the signed-in Infleux account needs; a call
without it returns `Missing scope: <scope>` (403).

## Session

| Tool | Scope | Purpose |
| --- | --- | --- |
| `ping_core` | — | Connectivity + session check |
| `whoami` | — | Signed-in user identity |
| `get_query_reference` | — | Query contracts and field guides by topic |

## Brands and advertisers

| Tool | Scope | Purpose |
| --- | --- | --- |
| `find_brands` | `brands:read` | Resolve brand name → `brandId` |
| `find_advertisers` | `brands:read` | Resolve advertiser name → `advertiserId` |
| `find_advertiser_brands` | `brands:read` | Advertiser↔brand link → `advertiserBrandId` |

## Campaigns

| Tool | Scope | Purpose |
| --- | --- | --- |
| `find_campaigns` | `campaigns:read` | List campaigns; `onlyActiveCampaigns: true` for live |
| `find_live_campaigns_for_brand` | `campaigns:read` | Live campaigns for one named brand |
| `get_campaign` | `campaigns:read` | One campaign: briefing, rules, payouts |
| `get_campaign_casting` | `campaigns:read` | Creators cast for a campaign |
| `find_campaign_queue` | `campaigns:read` | Interest/approval funnel (`cm`/`am`/`brand`) |
| `find_content_approval_queue` | `campaigns:read` | Content pending review (read-only) |
| `find_campaign_views` | `campaigns:read` | `opened-campaign` events, raw or summarized |
| `find_campaigns_available` / `get_campaign_available` | `campaigns:read` | Availability for a given influencer |

## Performance and money

| Tool | Scope | Purpose |
| --- | --- | --- |
| `find_actions` / `get_action` | `actions:read` | Creator participations and tracked links |
| `query_athena_clicks` | `analytics:read` | Click analytics: `by_geo`, `origin_breakdown` |
| `query_action_tracker_events` | `analytics:read` | Clicks / conversions / skipped for one action |
| `query_athena_spend` | `budgets:read` | Spend templates (server-side SQL only) |
| `find_budgets` | `budgets:read` | Monthly budgets per `advertiserBrandId` |

## Pre-campaign pipeline

| Tool | Scope | Purpose |
| --- | --- | --- |
| `find_pre_campaigns` | `pre_campaigns:read` | Drafts before launch (`status: "pending"` = queue) |
| `get_pre_campaign` | `pre_campaigns:read` | One draft in full |
| `clone_campaign_to_pre_campaign` | `pre_campaigns:write` | New edition from a published campaign |
| `update_pre_campaign` | `pre_campaigns:write` | Patch a draft (blocked once published) |

## Not available to brand accounts

`find_influencers`, `query_spine_influencers`, `get_influencer`, `find_influencer_events`,
`find_influencer_tags` (`influencers:read`), `query_influencer_earnings`
(`influencer_earnings:read`), and `find_automated_castings` / `get_automated_casting`
(`automated_castings:read`) are reserved for Infleux staff accounts. A brand account
calling them gets a 403 — report it as a permissions boundary, not a failure.

## Resources and prompts

The server also publishes MCP resources — `infleux://about`, `infleux://glossary`,
`infleux://spine-select` — and prompt templates such as `live_campaigns_for_brand`,
`campaign_rules_and_briefing`, `campaign_queue_for_campaign`,
`content_approval_queue_for_campaign`, `campaign_views`, `clone_campaign` and
`edit_pre_campaign`. Users reach prompts through `/mcp` in Claude Code.

## Shared conventions

- **Period arguments** (`periodStart`, `periodEnd`) are `YYYYMMDD` strings. Athena click
  and tracker queries default to the last 7 days and cap at a 31-day span.
- **IDs** are Mongo ObjectIds. Resolve names first; never invent an ID.
- **Athena SQL** is built server-side from named templates. There is no raw-SQL escape
  hatch, by design.
