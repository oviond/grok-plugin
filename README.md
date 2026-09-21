# Oviond plugin for Grok

Connect Grok to [Oviond](https://www.oviond.com) — the white-label client
reporting platform for marketing agencies. Build and update client reports,
connect marketing datasources, read live campaign numbers, and schedule the
monthly send, all from chat.

## Install

In Grok, open `/plugin`, search for **Oviond**, and install.

On first connection Grok opens Oviond sign-in in the browser. Sign in with your
normal Oviond account. Do not paste an API key or token into chat — the plugin
does not use one.

## What you get

**The hosted Oviond MCP server** — 211 tools across clients, reports, pages,
widgets, datasources, goals, calculated metrics, templates, saved sections,
themes and branding, automations, email, PDF exports, and account settings.
Everything is scoped to the account and the permissions of the user who signed
in.

**Two skills** that tell the agent what order to do things in:

| Skill | For |
|---|---|
| `build-client-report` | Onboarding a client and building their first report — linking datasources, discovering a datasource's metrics and dimensions before configuring widgets, and reading the data back |
| `monthly-client-recap` | Closing out a month — moving the report to the new period, refreshing, checking it, and delivering it by PDF, email or a recurring automation |

## Authentication

The plugin connects only to `https://api.oviond.com`. Authentication is OAuth
2.1 with PKCE and dynamic client registration — no credential is stored in the
plugin.

Network endpoints:

- `https://api.oviond.com/mcp` — hosted MCP (streamable HTTP)
- `https://api.oviond.com/oauth/authorize`, `/token`, `/register`, `/revoke`,
  `/jwks` — OAuth 2.1 + DCR
- `https://api.oviond.com/.well-known/oauth-protected-resource`,
  `/.well-known/oauth-authorization-server` — discovery

Credentials: an Oviond account. The single scope is `mcp`. Tools act with
exactly the permissions of the signed-in user — an admin sees every client, a
client-scoped user sees only theirs.

The plugin ships no hooks, no scripts and no commands. It executes nothing on
your machine: one remote MCP server and two Markdown skills.

## Requirements

An Oviond account — [start a trial](https://www.oviond.com).

Datasources that sign in on the vendor's own website (Google, Meta, LinkedIn,
TikTok) must be connected once in the Oviond app, because that flow needs a
browser. Everything after that works over MCP.

## Docs

- [MCP overview](https://docs.oviond.com/mcp/overview)
- [Connecting](https://docs.oviond.com/mcp/connect)
- [Tool catalog](https://docs.oviond.com/mcp/tools)
- [Playbooks](https://docs.oviond.com/mcp/playbooks)
- [Troubleshooting](https://docs.oviond.com/mcp/troubleshooting)

## Licence

MIT — see [LICENSE](./LICENSE). Use of the hosted MCP server is governed by
[Oviond's terms](https://www.oviond.com/terms).
