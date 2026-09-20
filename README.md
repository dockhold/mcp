# Dockhold for your AI coding tool

Put the project you are working on online without leaving your editor. Install once, sign in once in the browser, then say "put this online". Your AI tool deploys the current folder (or a GitHub repo) to [Dockhold](https://dockhold.eu) and reports a live HTTPS URL. It can also read status and logs, set variables, and resize apps and managed databases.

No token to copy. Sign-in happens through the browser, the same way as the [Dockhold CLI](https://github.com/dockhold/cli), and every client below shares it.

- **Docs:** https://dockhold.eu/docs
- **Registry name:** [`eu.dockhold/dockhold`](https://registry.modelcontextprotocol.io/v0.1/servers?search=eu.dockhold)
- **Requirements:** Node.js 18 or newer (for `npx`)

## Install

Every install below starts the same local MCP server, `npx -y dockhold mcp`. It forwards each request to Dockhold with the sign-in the CLI stored on your machine. Nothing sensitive goes into your editor config.

### Claude Code

```
/plugin marketplace add dockhold/mcp
/plugin install dockhold@dockhold
```

This installs the MCP server and a `deploy` skill that knows the whole flow: preflight, sign-in, deploy, and what to do when a build fails.

### Cursor

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=dockhold&config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsImRvY2tob2xkIiwibWNwIl19)

The button opens this link in Cursor:

```
cursor://anysphere.cursor-deeplink/mcp/install?name=dockhold&config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsImRvY2tob2xkIiwibWNwIl19
```

Or add it by hand to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "dockhold": {
      "command": "npx",
      "args": ["-y", "dockhold", "mcp"]
    }
  }
}
```

### VS Code

```sh
code --add-mcp '{"name":"dockhold","command":"npx","args":["-y","dockhold","mcp"]}'
```

### Any other MCP client

Use the same server entry in your client's config:

```json
{
  "mcpServers": {
    "dockhold": {
      "command": "npx",
      "args": ["-y", "dockhold", "mcp"]
    }
  }
}
```

If your client gives a server only a few seconds to start, the first `npx -y dockhold` download can be too slow for it. Install the CLI once with `npm i -g dockhold` and the same entry starts instantly.

## Sign in

The first time a tool needs your account it answers: `Not signed in to Dockhold. Run "npx dockhold login" in a terminal, then try again.` Run that command, a browser tab opens, and you are done. The sign-in lasts 90 days and is shared by the CLI and every client above. Nothing else to configure.

## What it looks like

> **You:** put this online
>
> **AI:** Checked the project: it listens on `PORT` and has a Dockerfile. You are not signed in yet, so I am running `npx dockhold login`. A browser tab will open.
>
> *(you sign in, the tab says you are done)*
>
> **AI:** Signed in. Deploying with `npx dockhold deploy --name recipe-box`. Uploading, building...
>
> **AI:** Your app is live: https://recipe-box-a1b2c3.dockhold.app
>
> **You:** show me the logs
>
> **AI:** *(calls `get_app_logs`)* The last 100 lines: the server started on port 8080 and has served 3 requests. No errors.

Local folders deploy through the CLI, which needs no GitHub account. A GitHub repository deploys through the `deploy_app` tool, and pushes to it redeploy the app.

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
| `unset_app_variable` | deploy | Remove a plain environment variable and roll it out |
| `resize_app` | deploy | Change an app's memory |
| `resize_database` | deploy | Change a managed database's memory |
| `resize_database_storage` | deploy | Grow a managed database's disk |
| `set_app_storage` | deploy | Give an app storage that survives restarts and deploys, or grow it |
| `remove_app_storage` | deploy | Remove an app's storage and erase its files |
| `set_app_secret`, `unset_app_secret`, `bind_app_secret`, `list_secrets`, `list_app_secrets` | secrets | Store, attach, detach, and list encrypted secrets |

The browser sign-in has the `read` and `deploy` scopes. The `secrets` tools need a dashboard token with the `secrets` scope (see the remote server below); through the browser sign-in they answer with a permission error, and secrets are added in the dashboard's Secrets section instead.

## Remote server (no CLI)

For a client that cannot run a local command, the same tools are served over Streamable HTTP at `https://api.dockhold.eu/mcp`. Create a token in the [Dockhold dashboard](https://app.dockhold.eu) under Settings, then API tokens (read-only, read+deploy, or with secrets), and pass it as a bearer:

```json
{
  "type": "streamable-http",
  "url": "https://api.dockhold.eu/mcp",
  "headers": { "Authorization": "Bearer dh_mcp_..." }
}
```

Claude Code: `claude mcp add --transport http dockhold https://api.dockhold.eu/mcp --header "Authorization: Bearer dh_mcp_..."`. This is the one setup that puts a token in a config file, so keep that file out of version control.

## Getting started with Dockhold

Dockhold is the easiest way to put your app online: push from GitHub or deploy from a folder, always on, secure by default. Sign in at [app.dockhold.eu](https://app.dockhold.eu). The free tier needs no card.
