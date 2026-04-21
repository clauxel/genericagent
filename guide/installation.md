# Installation

This guide gets a developer from a clean checkout to a running GenericAgent Launch app.

## Prerequisites

| Tool | Recommended | Why |
| --- | --- | --- |
| Node.js | Current LTS or newer | Runs `server.mjs`, Vite, and the test suite. |
| npm | Bundled with Node | The repository scripts are defined in `package.json`. |
| Git | Current | Clone, diff, and update the project. |
| PostgreSQL | Production and realistic integration work | The app database stores users, sessions, orders, deployments, instances, products, and analytics. |
| Bash | Deployment scripts and remote provisioning | Several scripts and remote deployment commands assume a Bash shell. On Windows, use Git Bash or WSL for those flows. |

For UI and API development, PostgreSQL can be replaced by the memory driver.

## 1. Install Dependencies

```bash
npm install
```

The key dependencies are React, Vite, TypeScript, `pg`, `ssh2`, `undici`, and `lucide-react`.

## 2. Create Local Deployment Config

```bash
cp genericagent.config.example.json genericagent.config.json
```

PowerShell:

```powershell
Copy-Item genericagent.config.example.json genericagent.config.json
```

The preferred config file is `genericagent.config.json`.

Legacy compatibility still exists:

- `multica.config.json`
- top-level `multica` section
- several `MULTICA_*` env variable names

## 3. Create Local Environment File

`server.mjs` loads one automatic env file by mode:

| Mode | Automatic file |
| --- | --- |
| development | `.env.development` |
| production | `.env.production` |

Use this minimal development file when you do not need a real database or real remote provisioning:

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

Important behavior:

- values already present in `process.env` win over `.env.*` values,
- `GENERICAGENT_ENV_PATH` can add one or more explicit env files,
- `MULTICA_ENV_PATH` is still supported as a legacy alias.

## 4. Start the App

```bash
npm run dev
```

Open `http://localhost:<PORT>`.

If `PORT` is not set, `server.mjs` falls back to `5175`.

## 5. Verify

```bash
npm run build
npm test
```

`npm run build` runs TypeScript and Vite production build checks.

`npm test` runs the Node test suite in `test/*.test.mjs`.

## 6. Optional Real PostgreSQL Setup

Use PostgreSQL when you need realistic persistence, webhook replay, production-like order state, or admin/analytics inspection.

Required variables:

```bash
MULTICA_POSTGRES_HOST=127.0.0.1
MULTICA_POSTGRES_DB=multica_dev
MULTICA_POSTGRES_USER=multica_dev
MULTICA_POSTGRES_PASSWORD=replace-with-dev-password
MULTICA_POSTGRES_PORT=5432
```

The app initializes and migrates its own tables in `server-lib/app-database.mjs`.

## First-Day Checklist

- `npm install` completed.
- `genericagent.config.json` exists or the legacy config is intentional.
- `.env.development` exists and has either memory or PostgreSQL settings.
- `npm run dev` starts and prints the actual local URL.
- The browser can load the app.
- `npm run build` passes.
- `npm test` passes.
