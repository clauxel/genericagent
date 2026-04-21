# GenericAgent Launch for Developers

**Languages:** **English** | [Simplified Chinese](README.zh-CN.md)

> This documentation pack is for the current GenericAgent Launch repository: a React/Vite frontend plus a Node.js control plane for checkout, provisioning, workspace console access, analytics, payments, and remote GenericAgent workspace operations.

Last checked against the local repository shape on 2026-04-22.

Try GenericAgent online: <https://www.genericagent.org/>

## Developer Usefulness Verdict

The previous version of this folder described the upstream Python GenericAgent runtime and pointed developers at files such as `mykey.py`, `agentmain.py`, and `launch.pyw`. Those files are not the center of this repository.

This version is useful for developers because it answers the questions that actually block work in this codebase:

- how to run the app locally,
- which Node and Vite entrypoints matter,
- how environment and deployment config are loaded,
- where checkout, payment, deployment, console, admin, analytics, and model proxy code lives,
- how PostgreSQL, memory tests, mock deployment, SSH deployment, and router mode fit together,
- how to change plans, models, channels, UI copy, API routes, deployment behavior, and operations scripts,
- how to verify changes with the existing build and test commands.

## What This Repo Owns

GenericAgent Launch is not the upstream agent runtime. It is the hosted launch and operations layer around GenericAgent workspaces.

It owns:

- public marketing, solution, pricing, comparison, FAQ, legal, and SEO surfaces,
- launch checkout, payment confirmation, order state, and provisioning queues,
- workspace console links, lifecycle actions, upgrades, stops, and uninstall flows,
- SSH or mock provisioning through the deployment runtime,
- PostgreSQL-backed application state with a memory driver for tests,
- model proxy wiring, analytics capture, admin tools, and runtime metadata.

The upstream GenericAgent runtime is still an important product reference, but developers working in this repository should start from this launch/control-plane mental model.

## Read This First

| File | Best for |
| --- | --- |
| [guide/installation.md](guide/installation.md) | Clean local setup with Node, config, env, and verification. |
| [guide/quickstart.md](guide/quickstart.md) | Fastest route from clone to running app and first checkout flow. |
| [guide/configuration.md](guide/configuration.md) | Environment loading, `genericagent.config.json`, legacy `MULTICA_*` names, payments, DB, model proxy, and deployment credentials. |
| [guide/deployment.md](guide/deployment.md) | Production topology, same-origin app mode, router mode, SSH provisioning, and release checks. |
| [guide/troubleshooting.md](guide/troubleshooting.md) | Fixes for the failures developers are most likely to hit first. |
| [developer-usefulness-audit.md](developer-usefulness-audit.md) | The 10/10 usefulness checklist and coverage map for this doc set. |
| [developer/architecture.md](developer/architecture.md) | System map, extension points, and where to read or edit code. |
| [features/agent-loop.md](features/agent-loop.md) | The launch order lifecycle from UI request to deployed workspace. |
| [features/frontends.md](features/frontends.md) | Public site, checkout, console, admin, SEO, and API client surfaces. |
| [features/layered-memory.md](features/layered-memory.md) | Persistence model: env, config, DB tables, mock state, remote state, and router records. |
| [features/reflection.md](features/reflection.md) | Provisioning automation: mock mode, SSH mode, router mode, lifecycle operations, and scripts. |
| [reference/runtime-modes.md](reference/runtime-modes.md) | `npm` scripts and runtime entrypoints. |
| [reference/config.md](reference/config.md) | Compact reference for config files and environment variables. |
| [reference/prompt-recipes.md](reference/prompt-recipes.md) | Copy-paste prompts for agent-assisted development in this repo. |
| [reference/faq.md](reference/faq.md) | Practical answers for common developer questions. |

## Five-Minute Local Start

Install dependencies:

```bash
npm install
```

Create a local deployment config:

```bash
cp genericagent.config.example.json genericagent.config.json
```

Create `.env.development`. For UI and API development without a real PostgreSQL server, this is enough to boot the app with the memory driver and simulated provisioning:

```bash
PORT=5175
APP_ORIGIN=http://localhost:5175
MULTICA_POSTGRES_DRIVER=memory
MULTICA_POSTGRES_MEMORY_ID=genericagent-dev
MULTICA_ALLOW_SIMULATED_DEPLOYMENT=true
MULTICA_TOKEN_SECRET=replace-with-a-long-dev-token-secret
MULTICA_CONFIG_SECRET=replace-with-a-long-dev-config-secret
PAYMENT_PROVIDER=creem
CREEM_ENV=test
API_TEST_KEY=
```

Start the local server:

```bash
npm run dev
```

Open the port from `.env.development`. If no `PORT` is set, `server.mjs` falls back to `5175`.

Verify before you push:

```bash
npm run build
npm test
```

## Most Important Files

| Path | Why it matters |
| --- | --- |
| `server.mjs` | Main Node server, env loading, database setup, API router mounting, Vite integration, console proxy, and server-side fallback pages. |
| `src/App.tsx` | Main React application and route rendering surface. |
| `src/content/` | Product copy, page content, FAQ, legal text, and catalog-driven presentation. |
| `src/lib/api.ts` | Browser API client for launch, console, auth, and lifecycle flows. |
| `server-lib/api/` | Route modules for auth, catalog, orders, webhooks, analytics, admin, and model proxy. |
| `server-lib/app-database.mjs` | PostgreSQL or memory driver setup plus schema creation and migrations. |
| `server-lib/deployment-config.mjs` | Config file resolution and normalization. |
| `server-lib/deployment-runtime.mjs` | Mock, SSH, router-aware deployment, upgrade, stop, and uninstall execution. |
| `shared/catalog.mjs` | Plans, models, channels, prices, discounts, and display metadata shared by frontend and backend. |
| `scripts/` | Operational entrypoints for deployment, packaging, DB setup, dev tunnels, and environment reset. |
| `deploy/` | Production env, systemd, router, nginx, and Vercel examples. |
| `test/` | Node test suite covering checkout, deployment, analytics, env loading, visibility, routing, and proxy helpers. |

## High-Value Change Map

| Task | Start here | Then check |
| --- | --- | --- |
| Change pricing, plans, models, or channels | `shared/catalog.mjs` | `src/content/catalog.ts`, checkout tests |
| Change public page copy | `src/content/site-content.tsx` | `src/lib/seo.ts`, screenshot/manual browser check |
| Add or change an API endpoint | `server-lib/api/index.mjs` and one route module | tests under `test/` |
| Change checkout behavior | `server-lib/api/order-routes.mjs` | `server-lib/payment-helpers.mjs`, `api/launch-checkout.mjs` |
| Change Creem or PayPal integration | `server-lib/payment-helpers.mjs`, `server-lib/api/webhook-routes.mjs` | webhook tests and live sandbox settings |
| Change deployment behavior | `server-lib/deployment-runtime.mjs` | `server-lib/deployment-config.mjs`, `scripts/deploy-order-to-real-server.mjs` |
| Change production topology | `deploy/`, `server-router.mjs`, `server-lib/multica-instance-router.mjs` | nginx/systemd/router env examples |
| Change persistence | `server-lib/app-database.mjs` | data serialization helpers and tests |

## Reality Checks

- `genericagent.config.json` is the preferred deployment config file. `multica.config.json` still works for compatibility.
- `GENERICAGENT_ENV_PATH` is the preferred explicit env path. `MULTICA_ENV_PATH` still works.
- Many runtime variables still start with `MULTICA_*`; that is a compatibility layer, not a sign that you are in the wrong repo.
- Local development can use `MULTICA_POSTGRES_DRIVER=memory`; production should use PostgreSQL.
- Mock deployment is for development. SSH deployment writes real workspaces, systemd units, router records, and env files on the target host.
- Do not put real API keys, root passwords, or payment secrets in public documentation or examples.
