# Launch Order Flow

This repository's most important loop is not an LLM turn loop. It is the launch order lifecycle: a user selects a plan, pays, and receives a provisioned GenericAgent workspace.

## The Core Flow

```text
User selects plan/model/channel
  -> frontend submits launch order or stateless checkout
  -> backend validates catalog and communication token
  -> order is created or checkout session is created
  -> payment is confirmed by redirect, capture, webhook, or development shortcut
  -> queuePaidOrder starts provisioning
  -> deployment runtime creates mock or SSH workspace
  -> deployments and agent_instances are updated
  -> console URL becomes available
```

## Files To Read First

| Step | File |
| --- | --- |
| Frontend checkout action | `src/App.tsx`, `src/lib/api.ts` |
| Shared pricing and channel data | `shared/catalog.mjs` |
| Stateful order routes | `server-lib/api/order-routes.mjs` |
| Stateless checkout | `api/launch-checkout.mjs` |
| Payment provider helpers | `server-lib/payment-helpers.mjs` |
| Webhooks | `server-lib/api/webhook-routes.mjs` |
| Provisioning | `server-lib/deployment-runtime.mjs` |
| DB persistence | `server-lib/app-database.mjs` |

## Stateful Order Path

Stateful launch orders use:

```text
POST /api/launch-orders
GET  /api/orders
GET  /api/orders/:id
POST /api/orders/:id/checkout-session
```

Once payment is confirmed, `queuePaidOrder` moves the order toward deployment.

Development-only payment bypass:

```text
POST /api/orders/:id/pay
```

That route is disabled in production.

## Stateless Checkout Path

Stateless checkout is useful when checkout must happen before database-backed order persistence.

Route:

```text
POST /api/launch-checkout
```

Handler:

```text
api/launch-checkout.mjs
```

Test:

```text
test/stateless-checkout.test.mjs
```

## Payment Confirmation

Payment can be confirmed by:

- Creem hosted checkout session,
- Creem redirect confirmation,
- Creem webhook,
- PayPal order capture,
- PayPal webhook,
- development-only manual pay endpoint.

Provider-specific logic is intentionally centralized in `server-lib/payment-helpers.mjs`.

## Deployment Trigger

Deployment is created through helper functions mounted into `order-routes.mjs` from `server.mjs`.

The runtime result updates:

- `orders.deployment_status`
- `deployments.status`
- `deployments.console_url`
- `deployments.public_endpoint`
- `agent_instances.runtime_state`
- `agent_instances.multica_version`

## Lifecycle Actions

Once deployed, users and admins can request:

```text
POST /api/orders/:id/multica-console
POST /api/orders/:id/multica-stop
POST /api/orders/:id/multica-uninstall
POST /api/orders/:id/multica-delete
POST /api/orders/:id/multica-upgrade
POST /api/orders/:id/deployments
```

These routes are the operational surface for workspace lifecycle management.

## Debugging The Flow

When a launch fails, identify the first broken stage:

| Symptom | Likely stage |
| --- | --- |
| Model/channel/price invalid | Catalog selection |
| Checkout URL missing | Payment provider setup |
| Payment confirmed but no workspace | Queue or deployment runtime |
| Workspace exists but console URL fails | Router or console port mapping |
| Upgrade/stop/uninstall fails | Stored deployment metadata or SSH credentials |

Start with the route response, then inspect the matching row in `orders`, `deployments`, and `agent_instances`.
