# Provisioning Automation

This repository automates the operational lifecycle of GenericAgent workspaces. The important automation is mock/SSH deployment, router updates, and lifecycle commands.

## Runtime Modes

| Mode | How to enable | What happens |
| --- | --- | --- |
| Mock deployment | `deployment.provider = "mock"` | Writes local simulated workspace state under `mockRootDir`. |
| SSH deployment | `deployment.provider = "ssh"` plus credentials | Writes real workspace files, env, systemd service, and optional router record. |
| Router mode | `MULTICA_ROUTER_BASE_URL` and routes dir configured | Console URLs route through the instance router. |
| Manual deployment queue | `MULTICA_DEPLOYMENT_MODE=manual` | Payment confirmation queues work without automatic completion messaging. |

## Deployment Runtime

Main implementation:

```text
server-lib/deployment-runtime.mjs
```

Public functions:

- `executeConfiguredDeployment`
- `upgradeConfiguredDeployment`
- `stopConfiguredDeployment`
- `uninstallConfiguredDeployment`
- `createDeploymentPlanPreview`
- `listConfiguredMulticaVersions`

## Deployment Plan

For each instance, the runtime derives:

- instance name,
- runtime user,
- systemd service name,
- workspace path,
- app path,
- env path,
- console port,
- console token,
- console URL,
- public endpoint,
- selected model/channel/plan env.

These values are returned to the app and saved into deployment tables.

## Mock Automation

Mock mode creates local files that mirror the shape of remote provisioning:

```text
<mockRootDir>/instances/<instanceName>/.env
<mockRootDir>/instances/<instanceName>/app/deployment.json
<mockRootDir>/systemd/<serviceName>.service
```

Use mock mode before changing real SSH scripts.

## SSH Automation

SSH mode renders and executes remote Bash scripts.

It can:

- install archive or git source,
- create runtime users,
- write `.env`,
- write `.multica/multica.json`,
- install dependencies,
- build the runtime,
- write a systemd unit,
- open console ports when needed,
- write router route files,
- restart services.

Credentials are resolved from password or private key env variables.

## Lifecycle Automation

Workspace lifecycle actions map to API routes:

| Action | API route | Runtime function |
| --- | --- | --- |
| Deploy | payment/deployment queue | `executeConfiguredDeployment` |
| Upgrade | `POST /api/orders/:id/multica-upgrade` | `upgradeConfiguredDeployment` |
| Stop | `POST /api/orders/:id/multica-stop` | `stopConfiguredDeployment` |
| Uninstall | `POST /api/orders/:id/multica-uninstall` | `uninstallConfiguredDeployment` |
| Delete | `POST /api/orders/:id/multica-delete` | uninstall plus app state update |

## Operational Scripts

| Script | Use |
| --- | --- |
| `scripts/deploy-order-to-real-server.mjs` | Trigger a real deployment from an existing paid order. |
| `scripts/package-remote-multica-instance.mjs` | Package a remote workspace template/archive. |
| `scripts/reset-dev-environment.mjs` | Clear development records and clean tracked instances. |
| `scripts/setup-postgres-server.mjs` | Prepare PostgreSQL server settings. |
| `scripts/bootstrap-remote-multica-postgres.mjs` | Bootstrap shared remote PostgreSQL databases. |
| `scripts/dev-tunnel-*.sh` | Manage development tunnels. |

## Safety Rules

- Test plan generation with mock mode first.
- Keep deployment commands idempotent where possible.
- Never log raw channel tokens or upstream API keys.
- Treat router route files as production traffic control records.
- Keep stop and uninstall paths separate; stop should preserve recoverable state, uninstall removes it.
