---
description: Check the Infleux MCP connection, show who is signed in, and report which Infleux data this account can reach.
argument-hint: (no arguments)
---

Verify the Infleux connection and report it plainly.

1. Call `ping_core` to confirm Core API connectivity.
2. Call `whoami` to identify the signed-in user.
3. Probe read access with one cheap call: `find_brands(limit: 1)`.

Report:

- **Connection** — endpoint `https://mcp.infleux.io/mcp`, reachable or not.
- **Signed in as** — name and email from `whoami`.
- **Access** — whether brand/campaign data came back, and note any `Missing scope`
  response as a permissions boundary of this Infleux account rather than a failure.

If a call returns 401, the OAuth session has expired: tell the user to run `/mcp`, pick
**infleux**, and re-authenticate. Never ask them to paste a token.

Keep it to a few lines. This is a health check, not a report.
