# Installing the Dockhold MCP server (instructions for an AI agent)

Dockhold deploys a folder or a GitHub repository to a live HTTPS URL. This repository holds setup metadata only.

## Rules

- Do not clone this repository, run `npm install`, or build anything. The server is the published npm package `dockhold`, started with `npx`.
- Do not ask the user for an API key or token. None goes into the config. Sign-in happens once in the browser.
- Requirement: Node.js 18.19 or newer (`node --version`). If it is older or missing, tell the user and stop.

## Steps

1. Add this entry under `mcpServers` in your MCP settings file (for Cline: `cline_mcp_settings.json`). Keep any existing servers.

   ```json
   "dockhold": {
     "command": "npx",
     "args": ["-y", "dockhold", "mcp"],
     "env": { "DOCKHOLD_REF": "cline" },
     "disabled": false,
     "autoApprove": []
   }
   ```

2. Check the server: its tool list should include `list_apps`, `deploy_app` and `get_app_logs`. The tool list works before sign-in. The first `npx` download can take up to a minute. If the client gives up before the server starts, run `npm i -g dockhold` once and retry; the same entry then starts instantly.

3. Sign-in. Run `npx dockhold login` in a terminal. It opens a browser tab and waits up to five minutes while the user signs in, so use a timeout of at least five minutes. Tell the user a tab is opening. When it prints that sign-in succeeded, the MCP server uses it immediately, with no restart. The sign-in lasts 90 days and is shared by every client on the machine.

4. Confirm: call `list_apps`. A new account returns an empty list. That is success.

## If something fails

- A tool answers "Not signed in to Dockhold": step 3 was not done or has expired. Run `npx dockhold login` again.
- `npx` not found: Node.js is not installed. Tell the user; do not install it without asking.
- The user wants to deploy the folder they are in: `npx dockhold deploy` from the project root (timeout of at least ten minutes). See `AGENTS.md` in this repository for the full deploy workflow.
