# Kablo Takip Sistemi

Enerji sektöründe kullanılan kabloların proje bazında takip edilmesini sağlayan profesyonel SaaS uygulaması.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/kablo-takip run dev` — run the frontend (port 25442)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + Tailwind CSS v4 + shadcn/ui + wouter
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Auth: Clerk (Replit-managed)
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)
- Excel: xlsx library

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI contract (source of truth)
- `lib/db/src/schema/` — Drizzle schema: `users.ts`, `projects.ts`, `cables.ts`
- `artifacts/api-server/src/routes/` — Express route handlers
- `artifacts/kablo-takip/src/` — React frontend
- `artifacts/kablo-takip/src/pages/` — Page components
- `lib/api-client-react/src/generated/` — Generated React Query hooks

## Architecture decisions

- Contract-first API: OpenAPI spec gates codegen which gates the frontend. Re-run codegen after every spec change.
- Clerk auth with proxy: cookie-based for web. No manual token handling needed in browser.
- User JIT provisioning: users are created in our DB on first API call, seeded with `free` plan (3 projects, 200 cables).
- Auto-calculated cable length: `totalLength = endNo - startNo` (can be overridden manually).
- Numeric DB columns use `numeric` type (not float) to avoid precision issues; serialized as `parseFloat()` in API responses.

## Product

- **Landing page**: Public-facing hero page for unauthenticated users
- **Dashboard**: Stats overview (projects, cables, metraj), usage progress bars, recent projects
- **Projeler**: Full CRUD with search/sort; free plan blocked at 3 projects
- **Proje Detayı**: Cable table with add/edit/delete, Excel import/export
- **Paketler**: Free / Pro / Enterprise plan comparison
- **Raporlar**: Enterprise-only; shows upgrade prompt for Free/Pro

## SaaS Plans

| Plan | Projects | Cables | Reports |
|------|----------|--------|---------|
| Free | 3 | 200 | No |
| Pro | Unlimited | Unlimited | No |
| Enterprise | Unlimited | Unlimited | Yes |

## Gotchas

- Always re-run codegen after changing `lib/api-spec/openapi.yaml`
- Avoid query params on operations that also have path params — Orval creates a `{Op}Params` name collision
- The Clerk proxy 504s in dev console are expected (proxy only runs in production)
- DB schema changes: run `pnpm --filter @workspace/db run push` after editing schema files

## User preferences

- UI language: Turkish
- No emojis in the UI
