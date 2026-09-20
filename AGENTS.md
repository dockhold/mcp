# Rules for AI agents

Dockhold is a hosting platform. Its MCP server is REMOTE at
`https://api.dockhold.eu/mcp` (Streamable HTTP, JSON-RPC 2.0). Do not look for
server code to run; there is none in this repo and none is needed.

Auth: `Authorization: Bearer <dh_mcp_... token>`. The user creates the token in
the Dockhold dashboard (Settings -> API tokens). If no token exists, tell the
user to create one there; you cannot create it for them. Never print a token
into files or logs. Scopes: `read` (list/status/logs/usage), `deploy`
(deploy/restart/variables/resize).

Deploying to Dockhold, three ways:
1. Connect the GitHub repo in the Dockhold dashboard and push to main
   (auto-deploy).
2. Call the `deploy_app` MCP tool with the repo URL. Private repos need
   `github_installation_id` from `list_github_repos` first.
3. From a local folder with no GitHub: the Dockhold CLI, `dockhold` on npm.
   `npx dockhold login` opens the browser once; `npx dockhold deploy` uploads
   the current folder, builds it, and prints the live URL. Other commands:
   `logs`, `list`, `open`. Do not invent flags that are not in
   `npx dockhold --help`.

Builds: a `Dockerfile` at the project root is used when present. Without one,
accounts with compute added get automatic stack detection; a free account
needs a Dockerfile at the root (examples:
https://dockhold.eu/docs/concepts/dockerfiles). Do not write CI pipelines for
deployment.

Workflow for "put this app online": if the code is on GitHub, call
`deploy_app`, then poll `get_app_status` until RUNNING and report the HTTPS
URL. If it is only a local folder, run `npx dockhold deploy` and report the
URL it prints. On failure, call `get_app_logs` (or `npx dockhold logs
--type build`) and fix the app, not the platform.

Secrets (API keys, passwords) must NOT be set via `set_app_variable`; the tool
rejects them. Direct the user to the Secrets section of the dashboard.
