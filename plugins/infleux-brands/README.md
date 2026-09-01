<p align="center">
  <img src="assets/icon-256.png" alt="Infleux" width="96" height="96">
</p>

<h1 align="center">Infleux for Brands</h1>

<p align="center">
  Connect Claude to the Infleux influencer marketing platform — campaigns, creators,
  clicks, conversions and spend, from the conversation.
</p>

---

## What it does

[Infleux](https://www.infleux.co) is an influencer marketing platform: brands publish
campaigns, creators pick them up in the Infleux app and generate tracked links, and the
platform pays on performance (installs, conversions) or on a fixed fee.

This plugin gives Claude live, authenticated access to your Infleux data so you can ask
questions in plain language instead of clicking through dashboards:

- *"Which campaigns is Acme running right now, and what do they pay?"*
- *"How did the September campaign perform — clicks, conversions, spend?"*
- *"Where are the clicks coming from? Anything that looks off?"*
- *"What's waiting on us for approval this week?"*
- *"Clone last month's campaign into a draft for October."*

Everything runs against the same live data your Infleux dashboard shows.

## Install

```bash
/plugin marketplace add revmobgithub/infleux-brands-mcp-plugin
```

```bash
/plugin install infleux-brands@infleux
```

Then authenticate:

```bash
/mcp
```

Pick **infleux** and complete the login in the browser. Claude Code registers itself with
the Infleux identity provider automatically — there is no token to copy, no config file to
edit, and nothing to keep in your dotfiles.

Confirm it worked:

```bash
/infleux-status
```

## What you need

- Claude Code with plugin support.
- An **Infleux account** — the same email and password you use on the Infleux platform.
  Ask your Infleux account manager if you do not have one yet.

Your Infleux role decides what you can see. A brand/advertiser account reads campaigns,
castings, approval queues, actions, budgets, spend and click analytics, and can draft
pre-campaigns. Infleux staff accounts see more. When a tool is out of reach, Claude says
so — it does not fail silently.

## Commands

| Command | What it does |
| --- | --- |
| `/infleux-status` | Connection check: who is signed in and what this account can reach |
| `/infleux-campaigns [brand]` | Live campaigns with window, payout and conversion model |
| `/infleux-performance [campaign] [period]` | Full read-out: creators, clicks, conversions, spend |
| `/infleux-pending [campaign or brand]` | Everything waiting on brand review |

## Skills

Claude loads these on its own when a request calls for them.

| Skill | Covers |
| --- | --- |
| `infleux-brands-guide` | Resolving names to IDs, tool selection, pagination, permission scopes, domain vocabulary |
| `campaign-performance` | Funnel analysis, click-origin quality, spend against budget, report structure |
| `pre-campaign-builder` | Cloning campaigns, editing drafts, the preview-then-confirm write flow |

## Agent

`infleux-campaign-analyst` — a read-only analyst for questions that need several queries
stitched together: comparing campaigns or periods, investigating a drop, or assembling a
monthly brand review. It never writes.

## Writes are always confirmed

Two tools change data — `clone_campaign_to_pre_campaign` and `update_pre_campaign` — and
both refuse to write until you approve an explicit preview of the change. Claude shows you
what would change, waits for a yes, and only then commits. Published campaigns cannot be
edited through this plugin at all.

Everything else is read-only.

## Under the hood

| | |
| --- | --- |
| MCP server | `https://mcp.infleux.io/mcp` (remote, Streamable HTTP) |
| Authentication | OAuth 2.0 / OIDC with PKCE and dynamic client registration, via `auth.infleux.io` |
| Credentials | Held by Claude Code's secure storage — never written into this repository or your project files |
| Authorization | Scoped per Infleux role; enforced server-side on every tool call |

The plugin ships no executables, no hooks and no scripts. It is a manifest, an MCP server
URL, and Markdown.

## Support

- Product and account questions: your Infleux account manager, or
  [suporte@infleux.co](mailto:suporte@infleux.co)
- Plugin issues:
  [github.com/revmobgithub/infleux-brands-mcp-plugin/issues](https://github.com/revmobgithub/infleux-brands-mcp-plugin/issues)

## License

MIT — see [LICENSE](LICENSE).
