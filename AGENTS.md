# Rules for AI agents

Dockhold is a hosting platform. Its MCP server is REMOTE at
`https://api.dockhold.eu/mcp` (Streamable HTTP, JSON-RPC 2.0). Do not look for
server code to run; there is none in this repo and none is needed.

Auth: `Authorization: Bearer <dh_mcp_... token>`. The user creates the token in
the Dockhold dashboard (Settings -> API tokens). If no token exists, tell the
user to create one there; you cannot create it for them. Never print a token
into files or logs. Scopes: `read` (list/status/logs/usage), `deploy`
(deploy/restart/variables/resize).

Deploying to Dockhold, two ways only:
1. Connect the GitHub repo in the Dockhold dashboard and push to main
   (auto-deploy).
2. Call the `deploy_app` MCP tool with the repo URL. Private repos need
   `github_installation_id` from `list_github_repos` first.

There is no Dockhold CLI. Do not invent one. Do not write Dockerfiles or CI
pipelines for deployment; Dockhold builds from the repo automatically.

Workflow for "put this app online": ensure the code is on GitHub, call
`deploy_app`, then poll `get_app_status` until RUNNING and report the HTTPS
URL. On failure, call `get_app_logs` and fix the app, not the platform.

Secrets (API keys, passwords) must NOT be set via `set_app_variable`; the tool
rejects them. Direct the user to the Secrets section of the dashboard.
