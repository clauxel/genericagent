# Architecture

GenericAgent Launch is a single Node control plane with a React/Vite frontend and a deployment runtime that can provision remote GenericAgent workspaces.

## High-Level Topology

```text
Browser
  -> React/Vite app in src/
  -> server.mjs
  -> server-lib/api/*
  -> PostgreSQL or memory driver
  -> payment providers
  -> deployment runtime
  -> mock state or SSH workspace host
  -> optional instance router
```

## Frontend Layer

Main files:

| Path | Responsibility |
| --- | --- |
| `src/App.tsx` | Main app shell, page routing, checkout/console views, and UI state. |
| `src/main.tsx` | React mount. |
| `src/styles.css` | Site-wide styling. |
| `src/content/site-content.tsx` | Page copy, solution/comparison/FAQ/legal content. |
| `src/content/catalog.ts` | Catalog presentation helpers. |
| `src/lib/api.ts` | Browser API wrapper. |
| `src/lib/routing.ts` | Client route helpers. |
| `src/lib/seo.ts` | Client-side metadata helpers. |

The frontend is product-facing and operations-facing. It includes public marketing pages, pricing, checkout, console access, account state, admin views, and support entry points.

## Server Layer

`server.mjs` is the main entrypoint.

It handles:

- runtime mode and env loading,
- PostgreSQL or memory database initialization,
- HTTP helpers and security headers,
- API route mounting,
- Vite dev middleware,
- static asset serving,
- same-origin frontend fallback,
- server-rendered metadata for public routes,
- console proxy helpers.

API route composition starts in:

```text
server-lib/api/index.mjs
```

Route modules:

| Module | Responsibility |
| --- | --- |
| `auth-routes.mjs` | Register, login, logout, current user. |
| `catalog-routes.mjs` | Shared catalog and runtime metadata. |
| `order-routes.mjs` | Launch orders, checkout sessions, console links, lifecycle actions, payment confirmation. |
| `webhook-routes.mjs` | Creem and PayPal webhook entrypoints. |
| `analytics-routes.mjs` | Event capture and admin analytics queries. |
| `admin-routes.mjs` | User/order admin operations. |
| `model-proxy-routes.mjs` | Internal model proxy endpoint for deployed workspaces. |

## Data Layer

The database is created by `createAppDatabase` in:

```text
server-lib/app-database.mjs
```

Core tables:

- `users`
- `sessions`
- `orders`
- `deployments`
- `agent_instances`
- `creem_products`
- `analytics_sessions`
- `analytics_events`

Production uses PostgreSQL. Tests and simple local development can use the memory driver through `MULTICA_POSTGRES_DRIVER=memory`.

## Deployment Runtime

Key files:

```text
server-lib/deployment-config.mjs
server-lib/deployment-runtime.mjs
scripts/deploy-order-to-real-server.mjs
scripts/package-remote-multica-instance.mjs
```

The runtime supports:

- mock deployment into local `mockRootDir`,
- SSH deployment to a remote host,
- archive or git source modes,
- generated runtime env files,
- generated `.multica/multica.json`,
- systemd services,
- deterministic console port allocation,
- router route records,
- upgrade, stop, and uninstall lifecycle commands.

## Router Layer

The instance router is optional.

Entrypoint:

```text
server-router.mjs
```

Implementation:

```text
server-lib/multica-instance-router.mjs
server-lib/multica-router-helpers.mjs
```

It proxies:

```text
/instances/<instanceName>/
```

to the console port recorded in route JSON files.

## Shared Catalog

Plans, models, channels, discounts, and pricing live in:

```text
shared/catalog.mjs
```

This file affects both frontend display and backend checkout/payment calculations, so changes should be tested carefully.

## Extension Points

| Goal | Edit |
| --- | --- |
| Add a page or content section | `src/content/site-content.tsx`, `src/App.tsx` |
| Add a model, channel, or plan | `shared/catalog.mjs` |
| Add an API route | `server-lib/api/index.mjs` and a route module |
| Add order behavior | `server-lib/api/order-routes.mjs` |
| Add a payment provider behavior | `server-lib/payment-helpers.mjs`, `server-lib/api/webhook-routes.mjs` |
| Add deployment env or script logic | `server-lib/deployment-runtime.mjs` |
| Change config resolution | `server-lib/deployment-config.mjs`, `server-lib/env-loader.mjs` |
| Add persistence | `server-lib/app-database.mjs` and tests |
| Add operations script | `scripts/` and `package.json` |

## Architectural Biases Worth Keeping

- Keep catalog data shared instead of duplicating prices between frontend and backend.
- Keep API routes thin and push reusable behavior into helpers.
- Keep deployment plan construction inspectable because it writes real remote machines.
- Keep mock mode working because it makes deployment changes testable without SSH.
- Keep legacy `MULTICA_*` compatibility until the full runtime and deployment path migrates.
- Add tests around checkout, persistence, deployment, and env loading before broad refactors.
