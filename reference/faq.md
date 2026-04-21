# FAQ

## Is this the upstream GenericAgent runtime?

No. This repository is the hosted launch/control-plane layer. It presents, sells, provisions, and manages GenericAgent workspaces.

The upstream runtime is product context, not the main source tree documented here.

## Why are there still so many `MULTICA_*` names?

The codebase has moved its public positioning to GenericAgent, but deployment scripts, runtime templates, env variables, tests, and remote paths still preserve legacy Multica naming.

Treat those names as active compatibility contracts until the whole deployment path migrates.

## Do I need PostgreSQL locally?

Not for basic UI/API work.

Use:

```bash
MULTICA_POSTGRES_DRIVER=memory
```

Use real PostgreSQL for production-like payment, webhook, order, deployment, admin, or analytics work.

## Which port should I open?

Use the value of `PORT` in your active environment.

If no `PORT` is set, `server.mjs` uses `5175`.

## Where do I change prices, plans, models, or channels?

Start with:

```text
shared/catalog.mjs
```

Then check frontend presentation in:

```text
src/content/catalog.ts
```

## Where do I add an API endpoint?

Add a route module under:

```text
server-lib/api/
```

and mount it from:

```text
server-lib/api/index.mjs
```

## How do I test checkout without a real payment provider?

Use stateful orders plus the development-only route:

```text
POST /api/orders/:id/pay
```

For stateless Creem checkout behavior, read:

```text
test/stateless-checkout.test.mjs
```

## What is mock deployment?

Mock deployment writes local simulated workspace state under `mockRootDir` instead of using SSH.

It is the safest way to test deployment logic before touching a real host.

## What is router mode?

Router mode lets users access stable instance URLs while workspace consoles run on private or high-numbered ports.

The router reads route JSON files from `MULTICA_ROUTER_ROUTES_DIR` and proxies `/instances/<instanceName>/`.

## What should I run before committing?

```bash
npm run build
npm test
```

For UI-heavy changes, also run the dev server and check the affected page manually.
