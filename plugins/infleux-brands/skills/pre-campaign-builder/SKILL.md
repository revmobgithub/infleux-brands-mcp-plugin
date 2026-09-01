---
name: pre-campaign-builder
description: Draft and edit Infleux pre-campaigns safely — clone a published campaign into a new edition, adjust the briefing, payouts or content rules, and walk the mandatory preview-then-confirm flow. Use when someone wants to run a campaign again, prepare next month's edition, change a draft before launch, or check what is sitting in the pre-campaign approval queue.
---

# Building and editing pre-campaigns

A pre-campaign is the draft that precedes a live campaign. It is the only part of Infleux
this plugin can write to, and every write is gated behind an explicit confirmation.

## The confirmation contract — never skip it

`clone_campaign_to_pre_campaign` and `update_pre_campaign` both accept `confirmed`.
Without `confirmed: true` they change nothing and return a preview.

1. Call **without** `confirmed`.
2. Show the user the preview: which pre-campaign, and exactly which fields change.
3. Wait for an explicit yes.
4. Call again with `confirmed: true` and the same arguments.

This holds even when the user's phrasing sounds like a standing order ("just clone it",
"go ahead and update"). The preview is cheap; an unwanted draft on a brand's pipeline is
not. If the user re-confirms after seeing the preview, proceed without further hedging.

## Cloning a campaign into a new edition

```
find_campaigns(nameRegex: "...")            → confirm the source campaign with the user
get_campaign(id)                            → check payouts, rules, conversion type
clone_campaign_to_pre_campaign(campaignId)  → preview
```

Two things the clone deliberately does not do, and that you must handle:

- **Dates are not copied.** `startsAt` and `endsAt` are required (ISO datetimes) when
  creating with `confirmed: true`. Ask the user for the window — do not assume next month.
- **Margin is zeroed** and the name gets a version suggestion (v3 → v4). Override with
  `name` / `nameForInfluencer` / `monthYear` when the user wants different wording.

The result is a `pending` pre-campaign. It is not published and not approved — say so, and
point to the Infleux dashboard for the launch itself.

## Editing a draft

```
find_pre_campaigns(status: "pending", brandId)   → locate it
get_pre_campaign(id)                             → read current values
update_pre_campaign(id, updates: {...})          → preview
update_pre_campaign(id, updates: {...}, confirmed: true)
```

Commonly patched fields: `name`, `payouts`, `needContentApproval`, `aboutTheBrand`,
`whatToDo`, `collectContentReels`.

**Nested objects are replaced, not merged.** Send `payouts` or `collectContentReels`
complete, built from what `get_pre_campaign` returned, or you will silently drop the keys
you left out.

**Locked drafts**: a pre-campaign with `status: published`, or one that already has a
`campaignId`, is rejected by the API — the campaign is live. Tell the user plainly; there
is no supported path to edit a published campaign from here. Core API records the change
history for everything you do write.

## Reviewing the pipeline

`find_pre_campaigns(status: "pending")` is the approval queue. Useful filters:
`brandId`, `campaignConversionsType`, `budgetGte`, `startsAtGte` / `endsAtLte`.
Use `get_pre_campaign(id)` for detail rather than `fullDocument: true` on the list.

## Writing briefing copy

When you draft `aboutTheBrand` or `whatToDo`, write for the creator who will read it in
the app: concrete, short, and specific about what to show and what to avoid. Mirror the
source campaign's tone when cloning. Always show the copy to the user in the preview step
— you are drafting it, they are approving it.
