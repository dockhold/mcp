---
name: deploy
description: Put the current project online on Dockhold, check its status and logs, set variables. Use when the user asks to deploy, host, ship, put online, or get a URL for this project.
---

# Deploy this project to Dockhold

Dockhold hosts web apps. The default path is the Dockhold CLI: it uploads the current folder, builds it, and prints a live HTTPS URL. No GitHub account, no repository, no dashboard visit. The MCP tools this plugin connects cover everything after that first deploy (status, logs, variables, resizing) and repository-based deploys.

Run every `npx dockhold ...` command from the project root. The full command list is `npx dockhold --help`. Use only flags that appear there.

## 1. Preflight

- Find the project root: the folder with `package.json`, `pyproject.toml`, `requirements.txt`, `go.mod`, `Cargo.toml`, `Gemfile`, or the equivalent. Run everything from there.
- The app must listen on the port in the `PORT` environment variable (Dockhold sets it) and bind `0.0.0.0`, not `localhost`. If the code hardcodes a port or binds `localhost`, fix that one thing in the app.
- Check for a `Dockerfile` at the root, or a `dockhold.json` with `"build": {"dockerfile": "<path>"}`. A free account builds only from one of those; accounts with compute added also get automatic stack detection. If neither file exists and the CLI refuses the upload for that reason, write a minimal `Dockerfile` for the detected stack using the patterns at https://dockhold.eu/docs/concepts/dockerfiles. One file, at the root, nothing else changes.
- Do not switch hosts, do not add CI, do not restructure the project, do not rewrite the app to fit the host.

## 2. Sign in

Run `npx dockhold list`. If it reports that you are not signed in, run:

```
DOCKHOLD_REF=claude-plugin npx dockhold login
```

Tell the user a browser tab opens for them to sign in to Dockhold, and that the command continues on its own once they finish. It waits up to five minutes. Sign-in happens once per machine; the CLI and the MCP tools share it.

## 3. Deploy

```
npx dockhold deploy --name <kebab-name>
```

- `--name`: lowercase letters, digits, and hyphens, derived from the project name. Without it the CLI uses the folder name.
- `--db`: add when the app reads `DATABASE_URL`. Dockhold creates a managed database and sets that variable.
- `--env KEY=VALUE` (repeatable): non-secret configuration only, such as `NODE_ENV=production` or `LOG_LEVEL=info`. Never a key, token, password, or connection string; see Secrets below.
- `--db` and `--env` take effect when the app is first created. Running `npx dockhold deploy` again in the same folder pushes a new version of the same app; change a variable later with the `set_app_variable` tool.
- The CLI records which app this folder belongs to in `.dockhold/app.json`. Leave that file alone.
- `.env` files are never uploaded, and neither is anything matched by `.gitignore` or `.dockholdignore`.
- The command uploads, builds, and waits. When the app is live it prints `Your app is live:` and the URL. Report that URL exactly as printed. Until that line appears, there is no URL to report.

## 4. If the deploy fails

```
npx dockhold logs --type build
```

Read the build log, fix the app, run the deploy again. For an app that built but does not respond, call `get_app_logs` (or run `npx dockhold logs`). Common causes:

- the app listens on a fixed port, or on `localhost`, instead of `PORT` on `0.0.0.0`
- no start command: `npm start` missing from `package.json`, or no `CMD` in the Dockerfile
- a native dependency the image lacks: add the build tools or the system library to the Dockerfile
- a build step that reads a variable that was never set

Fix the app, not the platform. A failed build is not a reason to change hosts.

## 5. Secrets

API keys, tokens, passwords, connection strings: never pass them with `--env`, never write them into a file that gets committed, never ask the user to paste them into the chat. Tell the user to add each one in the Dockhold dashboard under Settings, then Secrets, and attach it to the app from the app's Variables page. The app restarts with the value in its environment.

Do not try to store a secret through this connection. The sign-in the CLI creates can deploy but cannot write secrets, and the call fails with a permission error.

## 6. Push-to-deploy

If the folder has a `github.com` remote and the user wants a deploy on every push, connect the repository instead: in the dashboard (New App, then Connect GitHub), or by calling the `deploy_app` tool with the repository URL (a private repository also needs the `github_installation_id` that `list_github_repos` returns). This creates a second app with its own URL. An app created from an upload cannot be switched to a repository in place. Once the repository-based app is running, the user can delete the upload-based one in the dashboard.

## 7. Everything else

Use the MCP tools: `list_apps` (find the app id by name), `get_app_status`, `get_app_logs`, `set_app_variable`, `unset_app_variable`, `redeploy_app`, `get_resource_usage`, `resize_app`, `resize_database`, `resize_database_storage`, `set_app_storage`. Read a tool's description before calling it; the resize tools need `get_resource_usage` first.

## Never

- invent a CLI flag; `npx dockhold --help` is the whole list
- run anything with `sudo`
- edit git remotes, force-push, or switch branches to make a deploy work
- put a token in any file, or print one
- claim the app is live before the deploy command printed its URL, or before `get_app_status` reports it reachable
