# State and Persistence

GenericAgent Launch keeps state in several layers. Understanding which layer owns which fact makes debugging much faster.

## State Layers

| Layer | Location | Owns |
| --- | --- | --- |
| Process env | shell, `.env.development`, `.env.production`, explicit env paths | Secrets, mode, ports, payment keys, DB credentials, deployment credentials. |
| Deployment config | `genericagent.config.json` or `multica.config.json` | Provider, source archive/git settings, base URLs, port ranges, remote paths, service naming. |
| App database | PostgreSQL or memory driver | Users, sessions, orders, deployments, instances, products, analytics. |
| Mock deployment state | configured `mockRootDir` | Local simulated workspaces, generated `.env`, mock service files, deployment JSON. |
| Remote workspace state | SSH target under configured `baseDir` | Real workspace app, runtime `.env`, `.multica/multica.json`, systemd service. |
| Router state | `MULTICA_ROUTER_ROUTES_DIR` | JSON route records mapping instance names to console ports and runtime state. |
| Browser state | cookies and local runtime UI state | Auth session and guest order access. |

## Database Tables

`server-lib/app-database.mjs` creates and migrates the app schema.

Important tables:

| Table | Purpose |
| --- | --- |
| `users` | Accounts and roles. |
| `sessions` | Auth sessions. |
| `orders` | Plan/model/channel/payment/provisioning truth. |
| `deployments` | Workspace deployment metadata, console URL, service info, run logs. |
| `agent_instances` | Current runtime state, version, upgrade status, instance metadata. |
| `creem_products` | Cached Creem product IDs by pricing lookup key. |
| `analytics_sessions` | Visitor/session analytics. |
| `analytics_events` | Captured events. |

## Memory Driver

For tests and lightweight local development:

```bash
MULTICA_POSTGRES_DRIVER=memory
MULTICA_POSTGRES_MEMORY_ID=genericagent-dev
```

The memory driver behaves like PostgreSQL through the app's async database wrapper, but it is not a production persistence layer.

## PostgreSQL Driver

For production and realistic integration testing:

```bash
MULTICA_POSTGRES_HOST=127.0.0.1
MULTICA_POSTGRES_DB=multica_app
MULTICA_POSTGRES_USER=multica_app
MULTICA_POSTGRES_PASSWORD=replace-with-password
MULTICA_POSTGRES_PORT=5432
```

Optional tunnel settings are supported for PostgreSQL access through the deployment host.

## Order State

Order state spans payment and provisioning:

- `payment_status`
- `deployment_status`
- `deployment_message`
- `deployment_eta_minutes`
- `included_deployments`

Do not infer workspace health from `orders` alone. Check `deployments` and `agent_instances` for runtime details.

## Deployment State

`deployments` stores:

- target server,
- workspace path,
- console URL,
- public endpoint,
- runtime user,
- service name,
- encrypted console token,
- run logs.

`agent_instances` stores:

- instance identity,
- runtime state,
- selected model/channel/plan,
- version,
- upgrade status.

## Router Records

When router mode is active, deployment writes route records to:

```text
MULTICA_ROUTER_ROUTES_DIR
```

The router reads those files to proxy `/instances/<instanceName>/` traffic to the correct console port.

If console routing fails, inspect the route record before changing frontend code.

## Practical Debugging

| Problem | Check first |
| --- | --- |
| Wrong value after restart | env file and process env protection rules |
| Config edit has no effect | config resolution order |
| Order visible in UI but no workspace | `orders`, then `deployments` |
| Console URL exists but fails | router record or direct console port |
| Upgrade button stuck | `agent_instances.upgrade_status` and deployment run logs |
| Analytics missing | `analytics_sessions`, `analytics_events`, and client event call |
