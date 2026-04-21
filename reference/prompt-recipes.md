# Prompt Recipes

These prompts are for agent-assisted development in this repository. They are written to force the assistant to read the right files before changing code.

## Trace Checkout

```text
Trace the checkout flow in this repository from the React action to order persistence and payment confirmation. Read src/lib/api.ts, src/App.tsx, server-lib/api/order-routes.mjs, server-lib/payment-helpers.mjs, and api/launch-checkout.mjs. Summarize the exact route sequence and identify where to add tests for a changed checkout field.
```

## Add A Plan Or Model

```text
Add a new catalog entry safely. Start from shared/catalog.mjs, then check src/content/catalog.ts and any checkout tests that depend on pricing. Keep frontend and backend catalog behavior consistent.
```

## Change Deployment Env Injection

```text
I need to add one environment variable to every provisioned workspace. Read server-lib/deployment-runtime.mjs and explain where the plan environment is built. Then make the smallest change and add or update a test if the existing suite has deployment coverage for generated env values.
```

## Debug Console URL

```text
Debug why a deployed workspace console URL is wrong. Read server-lib/deployment-runtime.mjs, server-lib/multica-router-helpers.mjs, server-lib/multica-instance-router.mjs, and the order routes that create console sessions. Tell me whether the bug is direct-port URL generation, router route record generation, or stored deployment metadata.
```

## Add An API Route

```text
Add a new API endpoint using the existing route style. Read server-lib/api/index.mjs and server-lib/api/route-utils.mjs, then inspect the closest existing route module. Implement the route with the same error and JSON response conventions, and add a focused Node test.
```

## Harden Env Loading

```text
Review env loading behavior. Read server-lib/env-loader.mjs and tests around env loading. Verify automatic mode files, explicit env path variables, and process.env precedence. Add a regression test before changing behavior.
```

## Prepare Production Deployment

```text
Prepare a production deployment checklist for this exact repository. Read deploy/multica.env.example, deploy/genericagent.nginx.example, deploy/*.service.example, server.mjs, server-router.mjs, and guide/deployment.md. Do not include real secrets.
```

## Investigate Payment Failure

```text
Investigate a payment failure without exposing secrets. Read server-lib/payment-helpers.mjs, server-lib/api/webhook-routes.mjs, server-lib/api/order-routes.mjs, and relevant tests. Identify whether the issue is provider credentials, return URL, webhook verification, order lookup, or payment status transition.
```

## Add Troubleshooting Docs

```text
A developer hit this error: <paste sanitized error>. Find the source file that throws it, explain the likely cause, and add a concise troubleshooting entry with the exact fix and verification command.
```

## Good Prompt Habits

- Name the flow you are changing: checkout, auth, deployment, router, analytics, catalog, or frontend content.
- Include the route or script command if you know it.
- Ask the assistant to read the closest test before editing.
- Never paste real API keys, passwords, tokens, or private keys.
- Ask for `npm run build` and `npm test` verification after code changes.
