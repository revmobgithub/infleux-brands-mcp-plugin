---
description: Show what is waiting on the brand in Infleux — creators in the campaign queue and content awaiting review.
argument-hint: [campaign or brand name]
---

Show everything pending brand review for: **$ARGUMENTS**

1. Resolve the target: `find_campaigns(nameRegex: ...)` for a campaign, or
   `find_brands(nameRegex: ...)` → `find_live_campaigns_for_brand` for a brand. Confirm
   with the user if the match is ambiguous.
2. For each campaign in scope:
   - `find_campaign_queue(campaignId, queueStage: "brand")` — creators awaiting approval
     to join.
   - `find_content_approval_queue(campaignId, approvalStage: "brand")` — content awaiting
     review.
3. Also surface `find_pre_campaigns(status: "pending", brandId)` when the user asked about
   a brand — drafts waiting to launch belong in the same picture.

Group the output by campaign, with counts first and the oldest items listed by
`createdAt` so nothing sits forgotten.

Both queue tools are read-only: Claude cannot approve or reject. Say that once, and point
the user to the Infleux dashboard for the decisions. Offer to summarize any single item in
more detail.
