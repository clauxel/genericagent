# Configuration

Configuration has two layers:

1. environment files and process environment,
2. deployment config JSON.

The code that matters most is:

```text
server-lib/env-loader.mjs
server-lib/deployment-config.mjs
```

## Environment Loading

`server.mjs` resolves runtime mode from:

```bash
node server.mjs --mode production
```

If no mode is provided, it uses `development`.

Automatic env files:

| Mode | File |
| --- | --- |
| development | `.env.development` |
| production | `.env.production` |

Explicit env files:

| Variable | Status |
| --- | --- |
| `GENERICAGENT_ENV_PATH` | Preferred |
| `MULTICA_ENV_PATH` | Legacy alias |

Multiple explicit paths can be separated with the platform path delimiter.

Existing `process.env` values are protected and are not overwritten by `.env.*`.

## Deployment Config Resolution

The deployment config path is resolved in this order:

1. `GENERICAGENT_CONFIG_PATH`
2. `MULTICA_CONFIG_PATH`
3. `genericagent.config.json`
4. `multica.config.json`

Preferred file shape:

```json
{
  "deployment": {
    "provider": "mock",
    "targetServer": "genericagent-runtime-1",
    "consoleBaseUrl": "https://console.genericagent.local",
    "publicBaseUrl": "https://genericagent.local",
    "consolePortBase": 58000,
    "consolePortRange": 4000,
    "mockRootDir": "./data/mock-remote"
  },
  "genericagent": {
    "sourceType": "archive",
    "archivePath": "/data/multica/templates/multica-template.tar.gz",
    "repoUrl": "https://github.com/multica/multica.git",
    "repoRef": "main",
    "baseDir": "/data/multica",
    "servicePrefix": "multica",
    "runtimeUserPrefix": "mca",
    "installCommand": "npm install --no-audit --no-fund",
    "buildCommand": "npm run build",
    "startCommand": "npm run start"
  }
}
```

The `multica` top-level section is still accepted for compatibility.

## Database Settings

Memory driver for local tests and simple development:

```bash
MULTICA_POSTGRES_DRIVER=memory
MULTICA_POSTGRES_MEMORY_ID=genericagent-dev
```

PostgreSQL driver:

```bash
MULTICA_POSTGRES_HOST=127.0.0.1
MULTICA_POSTGRES_DB=multica_dev
MULTICA_POSTGRES_USER=multica_dev
MULTICA_POSTGRES_PASSWORD=replace-with-password
MULTICA_POSTGRES_PORT=5432
```

Removed settings:

- `MULTICA_DB_PROVIDER=sqlite`
- `MULTICA_DB_PATH`

If those are present, `server-lib/app-database.mjs` throws intentionally.

## Security and Sessions

Required in real environments:

```bash
MULTICA_TOKEN_SECRET=replace-with-long-random-secret
MULTICA_CONFIG_SECRET=replace-with-long-random-secret
ADMIN_ALLOWED_EMAILS=admin@example.com
APP_ORIGIN=https://www.genericagent.example.com,https://genericagent.example.com
```

Guest order access can come from:

- guest cookie,
- `x-multica-guest-token` header,
- `guest_token` query parameter.

## Payments

Provider selection:

```bash
PAYMENT_PROVIDER=creem
```

Creem:

```bash
CREEM_ENV=test
API_TEST_KEY=replace-with-creem-test-key
API_PROD_KEY=replace-with-creem-live-key
CREEM_WEBHOOK_SECRET=replace-with-webhook-secret
```

PayPal:

```bash
PAY_CLIENT_ID=replace-with-client-id
PAY_SECRET=replace-with-secret
PAYPAL_ENV=sandbox
PAYPAL_WEBHOOK_ID=replace-with-webhook-id
```

Payment helpers live in `server-lib/payment-helpers.mjs`.

Webhook routes live in `server-lib/api/webhook-routes.mjs`.

## Deployment Credentials

For SSH provisioning:

```bash
MULTICA_DEPLOY_HOST=1.2.3.4
MULTICA_DEPLOY_PORT=22
MULTICA_DEPLOY_USERNAME=root
MULTICA_DEPLOY_ROOT_PASSWORD=replace-with-root-password
```

or private key:

```bash
MULTICA_AGENT_DEPLOY_PRIVATE_KEY_PATH=/path/to/key
MULTICA_DEPLOY_PRIVATE_KEY_PASSPHRASE=
```

The runtime checks for credentials before executing real SSH deployment.

## Router Mode

Router settings:

```bash
MULTICA_ROUTER_BASE_URL=http://10.128.0.4:19280
MULTICA_ROUTER_ROUTES_DIR=/data/multica/router/routes-prod
MULTICA_ROUTER_SHARED_TOKEN=replace-with-router-token
MULTICA_CONSOLE_PORT_BASE=58000
MULTICA_CONSOLE_PORT_RANGE=4000
```

Router implementation:

```text
server-router.mjs
server-lib/multica-instance-router.mjs
server-lib/multica-router-helpers.mjs
```

## Model Proxy

Common settings:

```bash
MULTICA_MODEL_PROXY_BASE_URL=https://provider.example.com
MULTICA_MODEL_PROXY_INTERNAL_BASE_URL=http://127.0.0.1:5175/api/internal/model-proxy
MULTICA_MODEL_PROXY_ALLOWED_REMOTE_ADDRESSES=10.128.0.4
MULTICA_MODEL_PROXY_MODEL_MAP_JSON={"gpt-5-4":"gpt-5.4"}
QS_KEY=replace-with-upstream-model-api-key
```

`QS_KEY` is intentionally blocked from passthrough into deployed runtime environments.

## Practical Advice

- Keep real secrets out of example docs.
- Use memory driver for UI work and fast tests.
- Use PostgreSQL before testing payment/webhook/order persistence seriously.
- Use mock deployment before touching a real SSH target.
- Treat `MULTICA_*` names as active compatibility names until the code migrates them.
