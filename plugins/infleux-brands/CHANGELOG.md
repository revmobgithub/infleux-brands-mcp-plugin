# Changelog

All notable changes to the Infleux for Brands plugin are documented here.
This project follows [Semantic Versioning](https://semver.org/).

## [1.0.0] — 2026-09-01

First public release.

### Added

- Remote MCP server `https://mcp.infleux.io/mcp` over Streamable HTTP, authenticated with
  OAuth 2.0 / OIDC (PKCE + dynamic client registration) — no tokens to configure.
- Skills: `infleux-brands-guide`, `campaign-performance`, `pre-campaign-builder`.
- Commands: `/infleux-status`, `/infleux-campaigns`, `/infleux-performance`,
  `/infleux-pending`.
- Agent: `infleux-campaign-analyst` (read-only).
