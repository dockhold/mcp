# Dockhold MCP server

Deploy and operate web apps on [Dockhold](https://dockhold.eu) straight from your AI coding tool. Deploy a GitHub repo to a live HTTPS URL, read status and logs, set variables, and resize apps and managed databases.

This is a **remote** MCP server. There is nothing to install or run locally; your AI tool connects to our hosted endpoint. This repository holds the public metadata and setup instructions.

- **Endpoint:** `https://api.dockhold.eu/mcp` (Streamable HTTP)
- **Registry name:** [`eu.dockhold/dockhold`](https://registry.modelcontextprotocol.io/v0.1/servers?search=eu.dockhold)
- **Auth:** Bearer token. Create one in the [Dockhold dashboard](https://app.dockhold.eu) under Settings, then API tokens. Choose read-only or read+deploy. Tokens look like `dh_mcp_...`
- **Docs:** https://dockhold.eu/docs

## Setup

Replace `dh_mcp_...` with your token in the snippets below.

### Claude Code

```sh
claude mcp add --transport http dockhold https://api.dockhold.eu/mcp \
  --header "Authorization: Bearer dh_mcp_..."
```

### Cursor

Add to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in your project):

```json
{
  "mcpServers": {
    "dockhold": {
      "url": "https://api.dockhold.eu/mcp",
      "headers": { "Authorization": "Bearer dh_mcp_..." }
    }
  }
}
```

### VS Code

```sh
code --add-mcp '{"name":"dockhold","type":"http","url":"https://api.dockhold.eu/mcp","headers":{"Authorization":"Bearer dh_mcp_..."}}'
```

### Any other MCP client

```json
{
  "type": "streamable-http",
  "url": "https://api.dockhold.eu/mcp",
  "headers": { "Authorization": "Bearer dh_mcp_..." }
}
```

### Installing as a plugin

If your tool installs this repo as a plugin (Open Plugins standard), the bundled `.mcp.json` connects automatically. Set the `DOCKHOLD_API_TOKEN` environment variable to your `dh_mcp_...` token first.

## Tools

| Tool | Scope | What it does |
|---|---|---|
| `list_apps` | read | List your apps with id, status, and URL |
| `get_app_status` | read | Deploy status, URL, last commit, error message for one app |
| `get_app_logs` | read | Recent runtime logs for one app |
| `get_resource_usage` | read | Account pools (compute, database RAM, storage) and per-app sizes |
| `list_github_repos` | read | Your connected GitHub repos, with the id needed to deploy private ones |
| `deploy_app` | deploy | Deploy a GitHub repo as a live web app |
| `deploy_group` | deploy | Deploy several connected services in one go |
| `redeploy_app` | deploy | Zero-downtime restart of an app |
| `set_app_variable` | deploy | Set a non-secret environment variable and roll it out |
| `resize_app` | deploy | Change an app's memory |
| `resize_database` | deploy | Change a managed database's memory |
| `resize_database_storage` | deploy | Grow a managed database's disk |

Read tools work with a read-only token. Deploy tools need a read+deploy token. Secrets never go through MCP; set those in the dashboard's encrypted vault.

## Getting started with Dockhold

Dockhold is the easiest way to put your app online: push from GitHub, always on, secure by default. Sign in at [app.dockhold.eu](https://app.dockhold.eu), deploy a repo, get a live HTTPS URL. The free tier needs no card.

No GitHub repo yet? Deploy straight from a folder on your computer with the CLI: `npx dockhold login`, then `npx dockhold deploy`. Details at [github.com/dockhold/cli](https://github.com/dockhold/cli).
