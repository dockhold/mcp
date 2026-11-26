# Rules for AI agents

Dockhold is a hosting platform: it takes a folder or a GitHub repository, builds
it, and serves it at a live HTTPS URL. This repository holds the plugin and
setup metadata. There is no server code here and none is needed.

## Two paths, one sign-in

1. **A local folder: the CLI.** `dockhold` on npm. `npx dockhold deploy`
   uploads the current folder, builds it, and prints the URL when the app is
   live. This is the default for "put this online": it needs no GitHub account
   and no repository. Other commands: `logs`, `list`, `open`. Do not invent
   flags that are not in `npx dockhold --help`.
2. **A GitHub repository, and every other operation: the MCP tools.**
   `deploy_app` deploys a repository by URL (a private repository also needs
   `github_installation_id` from `list_github_repos`). Status, logs,
   variables, resizing, storage, and restarts all go through the tools; call
   `list_apps` first to get an app id.

Both use the same sign-in, except in the Claude Code plugin (below). `npx dockhold login` opens the browser once and
stores an access token on the machine. The MCP server this repository
configures is `npx -y dockhold mcp`, a local process that forwards each
request to Dockhold with that stored token. No token is pasted into any
client config, and you must never write one into a file or print one. If a
tool call reports that the user is not signed in, run `npx dockhold login` in
a terminal (a browser tab opens) and call the tool again.

The Claude Code plugin lives in `claude-plugin/`, the folder submitted to
the Claude plugin directory. Its `.mcp.json` names the remote server below,
never a local command: the directory accepts a server that runs on the
user's computer only from Verified organizations. Keep that folder to the
plugin's own files (manifest, `.mcp.json`, skills, icon); the directory
refuses scripts and instruction files in it. Its skill is a copy of
`skills/deploy/SKILL.md` that differs only in the sign-in lines: change both
in the same PR. Raise the `version` in both `plugin.json` files with any
change to either plugin.

## Workflow for "put this app online"

If the code is only a local folder, run `npx dockhold deploy` and report the
URL it prints. If the code is on GitHub, call `deploy_app` with the
repository URL, poll `get_app_status` until it is reachable, and report the
HTTPS URL. Deploys on every push happen only when the repository is
connected: pass the `github_installation_id` that `list_github_repos` returns
(if the repository is not listed, the user connects it in the dashboard
first). A repository deployed by URL alone builds once and does not follow
pushes. On failure, read the build log
(`npx dockhold logs --type build`) or the runtime log (`get_app_logs`) and fix
the app, not the platform. Never report a URL as live before the deploy has
finished.

Builds: a `Dockerfile` at the project root is used when present, or the one
named under `"build": {"dockerfile": "<path>"}` in `dockhold.json`. Without
either, Dockhold builds the stacks it recognises on every account (Next.js
standalone, Vite, Node with a `start` script, static sites, FastAPI), and
accounts with compute added get automatic builds for any other stack. A stack
it does not recognise on a free account is declined within seconds with the
fix named; apply it, or add a `Dockerfile` (examples:
https://dockhold.eu/docs/concepts/dockerfiles). Do not write CI pipelines for
deployment.

## Secrets

API keys, passwords, and connection strings never go through
`set_app_variable` (the tool rejects them) or `--env`. Direct the user to the
Secrets section of the dashboard, where they add the value and attach it to
the app. Do not store a secret through the tools: the value would pass through the
conversation, and the CLI's sign-in cannot write secrets at all.

## Clients that need a remote server

For a client that cannot run a local command and needs a remote HTTP MCP
server, the same tools are served at `https://api.dockhold.eu/mcp`
(Streamable HTTP). A client with OAuth sign-in (ChatGPT, claude.ai, Claude
Code) connects by that URL alone: the user signs in when a tool first needs
the account and approves what it may do. Never ask for a token for that path.
For CI, scripts or a client that cannot sign in, use a bearer token the user
creates under API tokens in the dashboard sidebar. That setup carries a
token in a config file; treat it as a secret.
