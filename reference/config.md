# Configuration Reference

This is a compact reference for settings developers touch most often.

## Files

| File | Status | Purpose |
| --- | --- | --- |
| `.env.development` | local | Development env loaded by default. |
| `.env.production` | production | Production env loaded by `--mode production`. |
| `genericagent.config.json` | preferred | Deployment provider/source/runtime config. |
| `multica.config.json` | legacy | Still supported for compatibility. |
| `deploy/multica.env.example` | example | Same-host production env template. |
| `deploy/vercel-frontend.env.example` | example | Split frontend/API env template. |

## Env Path Priority

| Order | Variable or file |
| --- | --- |
| 1 | existing `process.env` values |
| 2 | automatic `.env.development` or `.env.production` |
| 3 | `GENERICAGENT_ENV_PATH` |
| 4 | `MULTICA_ENV_PATH` |

Existing process variables are not overwritten by loaded files.

## Config Path Priority

| Order | Variable or file |
| --- | --- |
| 1 | `GENERICAGENT_CONFIG_PATH` |
| 2 | `MULTICA_CONFIG_PATH` |
| 3 | `genericagent.config.json` |
| 4 | `multica.config.json` |

## Core App Env

| Variable | Required | Notes |
| --- | --- | --- |
| `PORT` | no | Defaults to `5175`. |
| `APP_ORIGIN` | production yes | Comma-separated allowed origins and return URL base. |
| `NODE_ENV` | production yes | Usually `production` in deployed service env. |
| `MULTICA_DATA_DIR` | production recommended | Data root for operational files. |
| `MULTICA_TOKEN_SECRET` | yes | Encrypts/signs sensitive runtime values. |
| `MULTICA_CONFIG_SECRET` | yes | Used for config/secret encryption. |
| `ADMIN_ALLOWED_EMAILS` | optional | Comma-separated admin email allowlist. |

## Database Env

| Variable | Notes |
| --- | --- |
| `MULTICA_POSTGRES_DRIVER=memory` | Uses memory driver for tests/local dev. |
| `MULTICA_POSTGRES_MEMORY_ID` | Names the memory DB instance. |
| `MULTICA_POSTGRES_HOST` | PostgreSQL host. |
| `MULTICA_POSTGRES_DB` | Database name. |
| `MULTICA_POSTGRES_USER` | Database user. |
| `MULTICA_POSTGRES_PASSWORD` | Database password. |
| `MULTICA_POSTGRES_PORT` | Defaults to `5432` when omitted in config builder. |
| `MULTICA_POSTGRES_SSLMODE` | Optional SSL mode. |
| `MULTICA_POSTGRES_USE_SSH_TUNNEL` | Optional tunnel behavior. |

## Payment Env

| Variable | Provider | Notes |
| --- | --- | --- |
| `PAYMENT_PROVIDER` | all | `creem` or PayPal flow depending on app config. |
| `CREEM_ENV` | Creem | `test` or `live`. |
| `API_TEST_KEY` | Creem | Test key. |
| `API_PROD_KEY` | Creem | Live key. |
| `CREEM_WEBHOOK_SECRET` | Creem | Webhook signature secret. |
| `PAY_CLIENT_ID` | PayPal | PayPal client ID. |
| `PAY_SECRET` | PayPal | PayPal secret. |
| `PAYPAL_ENV` | PayPal | Sandbox/live/auto behavior. |
| `PAYPAL_WEBHOOK_ID` | PayPal | Webhook verification ID. |

## Deployment Env

| Variable | Notes |
| --- | --- |
| `MULTICA_ALLOW_SIMULATED_DEPLOYMENT` | Allows local simulated provisioning behavior. |
| `MULTICA_DEPLOYMENT_MODE` | `manual` changes payment/deployment messaging. |
| `MULTICA_DEPLOY_HOST` | SSH host. |
| `MULTICA_DEPLOY_PORT` | SSH port. |
| `MULTICA_DEPLOY_USERNAME` | SSH username. |
| `MULTICA_DEPLOY_ROOT_PASSWORD` | Password credential. |
| `MULTICA_AGENT_DEPLOY_PRIVATE_KEY_PATH` | Private key path. |
| `MULTICA_DEPLOY_PRIVATE_KEY_PASSPHRASE` | Optional key passphrase. |
| `MULTICA_DEPLOY_RUN_LOCAL` | Forces local command execution for local targets. |

## Router Env

| Variable | Notes |
| --- | --- |
| `MULTICA_ROUTER_BASE_URL` | Launch server uses this to build console URLs. |
| `MULTICA_ROUTER_ROUTES_DIR` | Directory for route JSON files. |
| `MULTICA_ROUTER_SHARED_TOKEN` | Shared access token. |
| `MULTICA_ROUTER_HOST` | Router listen host. |
| `MULTICA_ROUTER_PORT` | Router listen port. |
| `MULTICA_CONSOLE_PORT_BASE` | Base console port. |
| `MULTICA_CONSOLE_PORT_RANGE` | Deterministic allocation range. |

## Model Proxy Env

| Variable | Notes |
| --- | --- |
| `MULTICA_MODEL_PROXY_BASE_URL` | Upstream model provider base URL. |
| `MULTICA_MODEL_PROXY_INTERNAL_BASE_URL` | Internal URL exposed to deployed workspaces. |
| `MULTICA_MODEL_PROXY_ALLOWED_REMOTE_ADDRESSES` | Remote IP allowlist. |
| `MULTICA_MODEL_PROXY_MODEL_MAP_JSON` | Map public catalog model IDs to provider model IDs. |
| `MULTICA_MODEL_PROXY_PROVIDER_ID` | Generated provider ID. |
| `MULTICA_MODEL_PROXY_API` | Provider API mode. |
| `MULTICA_MODEL_PROXY_CHAT_MAX_TOKENS_CAP` | Chat max token cap. |
| `QS_KEY` | Server-only upstream model API key. |

## Deployment Config Fields

| Field | Notes |
| --- | --- |
| `deployment.provider` | `mock` or `ssh`. |
| `deployment.targetServer` | Display/record target name. |
| `deployment.consoleBaseUrl` | Base console URL. |
| `deployment.publicBaseUrl` | Base public workspace URL. |
| `deployment.consolePortBase` | Deterministic port base. |
| `deployment.consolePortRange` | Port allocation range. |
| `deployment.mockRootDir` | Mock deployment root. |
| `genericagent.sourceType` | `archive` or `git`. |
| `genericagent.archivePath` | Local/remote archive path. |
| `genericagent.archiveUrl` | Archive URL. |
| `genericagent.repoUrl` | Git source URL. |
| `genericagent.repoRef` | Git branch/tag/ref. |
| `genericagent.baseDir` | Remote base directory. |
| `genericagent.servicePrefix` | systemd service prefix. |
| `genericagent.runtimeUserPrefix` | Runtime user prefix. |
| `genericagent.installCommand` | Runtime install command. |
| `genericagent.buildCommand` | Runtime build command. |
| `genericagent.startCommand` | Runtime start command. |
