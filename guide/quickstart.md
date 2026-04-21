# Quickstart

Use this path when you want to prove the app works before making changes.

## 1. Boot in Development Mode

```bash
npm install
cp genericagent.config.example.json genericagent.config.json
npm run dev
```

Open the URL printed by the server. The fallback is `http://localhost:5175`, but `.env.development` can override the port.

## 2. Confirm the Catalog Loads

The frontend and backend share plan, model, and channel data from:

```text
shared/catalog.mjs
```

The browser fetches runtime/catalog data through:

```text
GET /api/catalog
GET /api/runtime
```

If pricing, models, or channels look wrong, start with `shared/catalog.mjs`, then check `src/content/catalog.ts`.

## 3. Create a Local Launch Order

The normal stateful order path is:

```text
POST /api/launch-orders
GET  /api/orders
GET  /api/orders/:id
```

In development mode, an unpaid order can be marked paid through:

```text
POST /api/orders/:id/pay
```

That endpoint is intentionally hidden in production.

## 4. Try Stateless Checkout

There is also a database-free checkout handler:

```text
POST /api/launch-checkout
```

The route is implemented in:

```text
api/launch-checkout.mjs
server-lib/api/order-routes.mjs
```

The test `test/stateless-checkout.test.mjs` shows the expected Creem request shape without hitting the real Creem API.

## 5. Exercise Mock Deployment

For local development, set:

```bash
MULTICA_ALLOW_SIMULATED_DEPLOYMENT=true
```

and use a config whose `deployment.provider` is `mock`.

Mock deployments write predictable local state under the configured `mockRootDir`, usually:

```text
./data/mock-remote
```

The implementation lives in `server-lib/deployment-runtime.mjs`.

## 6. Read the First Three Files

For a fast mental model, read these in order:

1. `server.mjs`
2. `server-lib/api/order-routes.mjs`
3. `server-lib/deployment-runtime.mjs`

They explain the request path from browser action to order state to workspace provisioning.

## 7. Verify Before Editing Deeply

```bash
npm run build
npm test
```

If either fails before your change, record that baseline before editing.

## Good First Tasks

| Task | Start file |
| --- | --- |
| Change one price or included deployment count | `shared/catalog.mjs` |
| Change homepage or FAQ copy | `src/content/site-content.tsx` |
| Add a small API response field | `server-lib/api/*-routes.mjs` |
| Add a deployment env var | `server-lib/deployment-runtime.mjs` |
| Add a new troubleshooting case | `guide/troubleshooting.md` |
