# Troubleshooting

This page focuses on failures that block developers in the current GenericAgent Launch repository.

## `npm run dev` Starts on the Wrong Port

Check:

```bash
PORT=5175
```

in `.env.development`.

If `PORT` is not set, `server.mjs` uses `5175`. If your env file sets a different port, use that URL.

## Environment Variables Are Ignored

Existing shell variables win over `.env.*` values.

Fix:

- close the shell and reopen it,
- unset the variable,
- or intentionally set `GENERICAGENT_ENV_PATH` to the file you want.

Relevant file:

```text
server-lib/env-loader.mjs
```

## PostgreSQL Configuration Is Missing

Error shape:

```text
Missing PostgreSQL configuration. Set MULTICA_POSTGRES_HOST, MULTICA_POSTGRES_DB, MULTICA_POSTGRES_USER, and MULTICA_POSTGRES_PASSWORD.
```

Fast local fix:

```bash
MULTICA_POSTGRES_DRIVER=memory
MULTICA_POSTGRES_MEMORY_ID=genericagent-dev
```

Production fix:

provide the real `MULTICA_POSTGRES_*` connection variables.

## SQLite Errors

SQLite support has been removed.

Remove:

```bash
MULTICA_DB_PROVIDER=sqlite
MULTICA_DB_PATH=...
```

Use PostgreSQL or the memory driver.

## Config File Looks Ignored

Resolution order:

1. `GENERICAGENT_CONFIG_PATH`
2. `MULTICA_CONFIG_PATH`
3. `genericagent.config.json`
4. `multica.config.json`

If a legacy env variable is set, it may be pointing at the file you are not editing.

## Catalog Data Is Wrong

Plans, models, discounts, and channels come from:

```text
shared/catalog.mjs
```

Frontend presentation helpers also use:

```text
src/content/catalog.ts
```

Run tests after catalog changes because pricing affects checkout behavior.

## Creem Checkout Fails

Check:

```bash
PAYMENT_PROVIDER=creem
CREEM_ENV=test
API_TEST_KEY=...
```

Common mistakes:

- using a live key while `CREEM_ENV=test`,
- leaving the key empty while testing hosted checkout,
- wrong `APP_ORIGIN`, which affects return URLs,
- missing webhook secret in production.

Useful files:

```text
server-lib/payment-helpers.mjs
api/launch-checkout.mjs
test/stateless-checkout.test.mjs
```

## PayPal Checkout Fails

Check:

```bash
PAY_CLIENT_ID=...
PAY_SECRET=...
PAYPAL_ENV=sandbox
```

The helper retries PayPal base URLs in some cases, but invalid credentials still fail.

## Development Payment Shortcut 404s

`POST /api/orders/:id/pay` only exists outside production.

If `runtimeMode` is production or `NODE_ENV=production`, it returns 404.

## Deployment Does Nothing

Check the provider:

```json
{
  "deployment": {
    "provider": "mock"
  }
}
```

Mock deployment writes local state. SSH deployment requires real credentials.

If you expect SSH, verify:

```bash
MULTICA_DEPLOY_HOST=...
MULTICA_DEPLOY_PORT=22
MULTICA_DEPLOY_USERNAME=root
MULTICA_DEPLOY_ROOT_PASSWORD=...
```

or private-key settings.

## SSH Deployment Fails

Common causes:

- target host unreachable,
- bad password or private key path,
- target machine lacks required shell tooling,
- `archivePath` does not exist,
- target base directory permissions are wrong,
- systemd is unavailable or service creation is blocked.

The deployment error includes a debug description from `server-lib/deployment-runtime.mjs` for many failures.

## Console URL Is Wrong

If router mode is enabled, check:

```bash
MULTICA_ROUTER_BASE_URL=...
MULTICA_ROUTER_ROUTES_DIR=...
MULTICA_ROUTER_SHARED_TOKEN=...
```

The router and launch server must agree on the same routes directory and token.

If router mode is not enabled, console URLs are derived from server host and deterministic console port.

## Model Proxy Fails From Remote Workspace

Check:

```bash
MULTICA_MODEL_PROXY_BASE_URL=...
MULTICA_MODEL_PROXY_INTERNAL_BASE_URL=...
MULTICA_MODEL_PROXY_ALLOWED_REMOTE_ADDRESSES=...
QS_KEY=...
```

The upstream API key is `QS_KEY`. It should stay server-side and is not passed directly to remote runtime environments.

## `rg --files` Reports a `.gitignore` Parse Error

This is usually caused by an invalid glob pattern in `.gitignore`.

Fix the malformed ignore line, then rerun `rg --files`. Until then, use `Get-ChildItem` or `rg --no-ignore` as a temporary workaround.

## Build Fails But Dev Server Works

Run:

```bash
npm run build
```

Build includes TypeScript project references plus Vite production bundling. A dev-only browser path can still hide a type error.

## Tests Need a Clean Environment

Tests snapshot and restore parts of `process.env`, but your shell can still affect behavior.

If a test depends on env loading, rerun from a clean shell and avoid exporting production credentials into the test process.
