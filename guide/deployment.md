# Deployment

GenericAgent Launch can run as a same-origin Node application, a split frontend/API deployment, or a control plane connected to a router and one or more remote workspace hosts.

## Recommended Production Shape

The simplest production topology is:

```text
Nginx
  -> npm run start
  -> server.mjs
  -> same-origin React app, API routes, console proxy, and static assets
  -> PostgreSQL
  -> SSH deployment target and optional instance router
```

In this shape, keep `VITE_API_BASE_URL` unset so the browser calls same-origin APIs.

Useful examples:

```text
deploy/multica.env.example
deploy/genericagent.nginx.example
deploy/multica.service.example
```

## Runtime Commands

Build:

```bash
npm run build
```

Start production server:

```bash
npm run start
```

`npm run start` runs:

```bash
node server.mjs --mode production
```

## Production Environment Baseline

At minimum, provide:

```bash
NODE_ENV=production
PORT=5175
APP_ORIGIN=https://www.genericagent.example.com,https://genericagent.example.com
MULTICA_DATA_DIR=/data/multica/data
MULTICA_CONFIG_PATH=/data/multica/genericagent.config.json
MULTICA_TOKEN_SECRET=replace-with-long-random-token-secret
MULTICA_CONFIG_SECRET=replace-with-long-random-config-secret
MULTICA_POSTGRES_HOST=127.0.0.1
MULTICA_POSTGRES_DB=multica_app
MULTICA_POSTGRES_USER=multica_app
MULTICA_POSTGRES_PASSWORD=replace-with-db-password
MULTICA_POSTGRES_PORT=5432
```

Then add payment, deployment, router, and model proxy variables as needed.

## Deployment Config

Use `genericagent.config.json` unless you are deliberately preserving legacy file names.

Important fields:

| Field | Why |
| --- | --- |
| `deployment.provider` | `mock` for local simulation, `ssh` for real provisioning. |
| `deployment.consoleBaseUrl` | Base console URL if router mode is not deriving instance URLs. |
| `deployment.publicBaseUrl` | Public endpoint base for deployed workspaces. |
| `deployment.consolePortBase` / `consolePortRange` | Deterministic console port allocation range. |
| `deployment.mockRootDir` | Local state root for mock deployment. |
| `genericagent.sourceType` | `archive` or `git`. |
| `genericagent.archivePath` / `archiveUrl` | Template source for archive deployments. |
| `genericagent.repoUrl` / `repoRef` | Runtime source and version for git deployments and upgrades. |
| `genericagent.baseDir` | Remote workspace root. |
| `genericagent.servicePrefix` / `runtimeUserPrefix` | Names for systemd services and runtime users. |

## Router Mode

Router mode is useful when workspace consoles run on private or high-numbered ports, but users should access stable URLs.

Start router:

```bash
npm run router:start
```

Router env:

```bash
MULTICA_ROUTER_HOST=127.0.0.1
MULTICA_ROUTER_PORT=19280
MULTICA_ROUTER_ROUTES_DIR=/data/multica/router/routes-prod
MULTICA_ROUTER_SHARED_TOKEN=replace-with-router-token
```

When `MULTICA_ROUTER_BASE_URL` is configured in the launch server, deployments write route records and console URLs become router URLs.

## SSH Provisioning

Real provisioning is implemented in:

```text
server-lib/deployment-runtime.mjs
```

The runtime:

1. builds a workspace plan,
2. chooses a deterministic runtime user, service name, workspace path, and console port,
3. renders a remote Bash script,
4. uploads or clones the runtime source,
5. writes `.env`,
6. writes `.multica/multica.json`,
7. creates or updates a systemd service,
8. optionally writes a router route record,
9. returns structured JSON to update `deployments` and `agent_instances`.

Credentials can be password-based or private-key-based.

## Payments and Webhooks

Payment confirmation can come from:

- browser-driven confirmation routes,
- Creem webhook,
- PayPal webhook,
- development-only `/api/orders/:id/pay`.

Production should use real webhook secrets and should not rely on development payment bypasses.

## Release Checklist

- `npm run build` passes.
- `npm test` passes.
- `.env.production` or service env contains no placeholder secrets.
- `APP_ORIGIN` matches every production origin.
- PostgreSQL connection works from the server.
- Payment provider keys match test/live mode.
- Webhook secrets are configured.
- SSH target is reachable.
- Deployment config points to the intended archive or git ref.
- Router token and routes directory are shared by launch server and router.
- Nginx or platform routing preserves API, static asset, and console proxy paths.

## Rollback Notes

- If frontend-only changes fail, redeploy the previous build.
- If deployment runtime changes fail, pause provisioning and keep existing instances untouched.
- If router records are wrong, fix the route JSON files or stop router mode until the console URL mapping is known good.
- If DB migration or schema initialization fails, inspect `server-lib/app-database.mjs` and the startup logs before retrying.
