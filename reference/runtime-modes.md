# Runtime Modes

The repository is operated through `npm` scripts and direct Node entrypoints.

## Development Server

```bash
npm run dev
```

Runs:

```bash
node server.mjs
```

Mode defaults to `development`, which loads `.env.development`.

## Production Server

```bash
npm run start
```

Runs:

```bash
node server.mjs --mode production
```

Production mode loads `.env.production` unless explicit env path variables add more files.

## Build

```bash
npm run build
```

Runs TypeScript build and Vite production build.

Use this before shipping frontend, route, catalog, or type changes.

## Preview

```bash
npm run preview
```

Runs Vite preview for built frontend assets. It is not the main production control plane.

## Test

```bash
npm test
```

Runs:

```bash
node --test test/*.test.mjs
```

The suite covers checkout, deployment flow, analytics, env loading, model proxy helpers, router behavior, console visibility, and related helpers.

## Router

```bash
npm run router:start
```

Runs:

```bash
node server-router.mjs
```

Use this when `MULTICA_ROUTER_BASE_URL` points console traffic through the instance router.

## PostgreSQL Setup

```bash
npm run postgres:setup
```

Runs:

```bash
node scripts/setup-postgres-server.mjs
```

Use for preparing PostgreSQL server-side settings.

## Package Remote Runtime

```bash
npm run genericagent:package
```

Runs:

```bash
node scripts/package-remote-multica-instance.mjs
```

Use this to package a deployed workspace template/archive.

## Development Deployment Helpers

```bash
npm run dev:deploy
npm run dev:tunnel:up
npm run dev:tunnel:down
npm run dev:tunnel:status
```

These wrap shell scripts under `scripts/`.

On Windows, run them from Git Bash or WSL if the default shell cannot execute Bash scripts.

## Direct Script Entrypoints

| Script | Use |
| --- | --- |
| `node scripts/deploy-order-to-real-server.mjs [orderId]` | Deploy a paid order to a real server. |
| `node scripts/reset-dev-environment.mjs` | Clear development database records and clean tracked instances. |
| `node scripts/bootstrap-remote-multica-postgres.mjs` | Bootstrap remote dev/prod PostgreSQL databases. |
| `bash scripts/deploy-production.sh` | Production deployment helper. |
| `bash scripts/deploy-development.sh` | Development deployment helper. |

## Which Mode To Use

| Situation | Use |
| --- | --- |
| Normal local development | `npm run dev` |
| Production app process | `npm run start` |
| Static/frontend build check | `npm run build` |
| Test suite | `npm test` |
| Router process only | `npm run router:start` |
| Real order provisioning from CLI | `node scripts/deploy-order-to-real-server.mjs` |
| Runtime template packaging | `npm run genericagent:package` |
