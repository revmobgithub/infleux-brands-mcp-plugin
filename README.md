<p align="center">
  <img src="plugins/infleux-brands/assets/icon-256.png" alt="Infleux" width="96" height="96">
</p>

<h1 align="center">Infleux plugins for Claude</h1>

<p align="center">
  Official Claude Code plugin marketplace for the
  <a href="https://www.infleux.co">Infleux</a> influencer marketing platform.
</p>

---

## Install

```bash
/plugin marketplace add revmobgithub/infleux-brands-mcp-plugin
```

```bash
/plugin install infleux-brands@infleux
```

Then run `/mcp`, pick **infleux**, and sign in with your Infleux account. Authentication
is OAuth — there is no token to copy or store.

## Plugins in this marketplace

| Plugin | Description |
| --- | --- |
| [`infleux-brands`](plugins/infleux-brands) | Query and manage Infleux campaigns from Claude — live campaigns and briefings, clicks, conversions and spend, casting and content-approval queues, and pre-campaign drafts. |

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json          # marketplace manifest ("infleux")
├── plugins/
│   └── infleux-brands/
│       ├── .claude-plugin/
│       │   └── plugin.json       # plugin manifest
│       ├── .mcp.json             # remote MCP server (OAuth)
│       ├── skills/               # infleux-brands-guide, campaign-performance,
│       │                         # pre-campaign-builder
│       ├── commands/             # /infleux-status, /infleux-campaigns,
│       │                         # /infleux-performance, /infleux-pending
│       ├── agents/               # infleux-campaign-analyst (read-only)
│       ├── assets/               # icons
│       ├── README.md
│       ├── CHANGELOG.md
│       └── LICENSE
├── docs/
│   └── SUBMISSAO-MARKETPLACE.md  # how to publish to the Claude plugin directory
├── LICENSE
└── README.md
```

## Local development

Validate the marketplace and plugin manifests:

```bash
claude plugin validate .
```

Install from a local checkout to test before publishing:

```bash
/plugin marketplace add ./
```

```bash
/plugin install infleux-brands@infleux
```

After editing a manifest, refresh the catalog:

```bash
/plugin marketplace update infleux
```

## Security

The plugin ships no executables, hooks or scripts — a manifest, an MCP server URL, and
Markdown. All data access goes through `https://mcp.infleux.io/mcp`, authenticated per
user with OAuth 2.0 / OIDC and authorized server-side against the signed-in account's
Infleux role. No credentials are stored in this repository.

Found a security issue? Email [suporte@infleux.co](mailto:suporte@infleux.co) rather than
opening a public issue.

## License

MIT — see [LICENSE](LICENSE).
