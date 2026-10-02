---
description: List the live Infleux campaigns for a brand, with window, payout and conversion model.
argument-hint: "[brand name] (omit for all live campaigns)"
---

List live Infleux campaigns for: **$ARGUMENTS**

- With a brand name: call `find_live_campaigns_for_brand(brandNameRegex: "$ARGUMENTS")`.
  On `status: "ambiguous_brand"`, show the matching brands and ask which one — do not
  pick for the user. On `status: "no_brand"`, retry `find_brands` with a shorter
  substring before concluding nothing matched.
- With no argument: call `find_campaigns(onlyActiveCampaigns: true, limit: 100)` for every
  live campaign across brands.

Present a table: campaign name, brand, `startsAt` → `endsAt`, conversion type, payout.
Sort by start date, most recent first.

Check `pagination.hasMore` before you summarize. If more rows exist, say how many of
`pagination.total` you are showing and offer to page through the rest.

Close by offering the obvious next step: the full briefing for one of them
(`get_campaign`), or a performance read-out.
