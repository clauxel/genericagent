# Frontend Surfaces

The frontend is a product site, checkout flow, workspace console launcher, account surface, and admin tool in one React/Vite app.

## Main Entrypoints

| Path | Role |
| --- | --- |
| `src/main.tsx` | React mount. |
| `src/App.tsx` | App shell, route rendering, page state, checkout and console views. |
| `src/styles.css` | Global design system and responsive layout. |
| `src/lib/api.ts` | Browser API client. |
| `src/lib/api-base.ts` | API base URL resolution. |
| `src/lib/routing.ts` | Route helpers. |
| `src/lib/seo.ts` | Metadata helpers. |
| `src/content/site-content.tsx` | Page copy and content blocks. |
| `src/content/catalog.ts` | Frontend catalog presentation helpers. |
| `src/components/logos.tsx` | Brand/logo components. |

## Public Site

The public site includes marketing, solution, comparison, FAQ, pricing, legal, and SEO pages.

When changing public copy:

1. update `src/content/site-content.tsx`,
2. check route rendering in `src/App.tsx`,
3. check metadata behavior in `src/lib/seo.ts`,
4. run `npm run build`.

## Checkout UI

Checkout depends on shared catalog data from:

```text
shared/catalog.mjs
```

The frontend should not invent prices or model IDs that the backend cannot validate.

API calls flow through `src/lib/api.ts` to:

- `/api/catalog`
- `/api/launch-orders`
- `/api/launch-checkout`
- `/api/orders/:id/checkout-session`
- `/api/orders/:id/creem-confirm`
- `/api/orders/:id/paypal-capture`

## Console Surface

The console page reads visible orders and instances from:

```text
GET /api/console-data
```

Opening a workspace console uses:

```text
POST /api/orders/:id/multica-console
```

The backend returns a session URL built from deployment metadata and guest/auth context.

## Admin Surface

Admin data comes from:

```text
GET  /api/admin/users
POST /api/admin/users
PATCH /api/admin/users/:id
GET  /api/admin/analytics/summary
GET  /api/admin/analytics/sessions
```

Admin access is configured through `ADMIN_ALLOWED_EMAILS` and user role helpers.

## Analytics

Client analytics uses:

```text
src/lib/analytics.ts
POST /api/analytics/events
```

Backend helpers live in:

```text
server-lib/analytics-helpers.mjs
server-lib/api/analytics-routes.mjs
```

## API Base URL

Same-origin production should leave `VITE_API_BASE_URL` unset.

Split frontend/API deployment can set:

```bash
VITE_API_BASE_URL=https://api.genericagent.example.com
```

Example:

```text
deploy/vercel-frontend.env.example
```

## Frontend Verification

Use:

```bash
npm run build
```

For visual or flow-sensitive changes, also run the dev server and manually check:

- homepage first viewport,
- pricing and model selection,
- checkout submit behavior,
- console order list,
- admin-only sections,
- mobile layout.
