# Mr. Knight Journal

A full-stack mobile trading journal PWA for professional forex traders — log trades, review performance, and trade with discipline.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/mr-knight-journal run dev` — run frontend (port 26099)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` (Replit native Postgres), `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY`, `VITE_CLERK_PUBLISHABLE_KEY`, `SESSION_SECRET`

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, Tailwind CSS v4, shadcn/ui, Recharts, framer-motion, wouter
- Backend: Express 5, Clerk auth (`@clerk/express`)
- DB: PostgreSQL (Replit native) + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec → React Query hooks + Zod schemas)
- Build: esbuild (CJS bundle for api-server)

## Where things live

- `lib/api-spec/openapi.yaml` — source of truth for API contract
- `lib/api-spec/orval.config.ts` — codegen config
- `lib/api-client-react/src/generated/api.ts` — generated React Query hooks
- `lib/api-zod/src/generated/api/api.ts` — generated Zod schemas
- `lib/db/src/schema/` — Drizzle table definitions (accounts, trades, checklists, reviews)
- `artifacts/api-server/src/routes/` — Express route handlers (accounts, trades, checklists, reviews, dashboard)
- `artifacts/mr-knight-journal/src/pages/` — Frontend pages
- `artifacts/mr-knight-journal/src/components/` — Shared UI components

## Architecture decisions

- Contract-first: OpenAPI spec → Orval codegen → React Query hooks + Zod schemas used everywhere
- Clerk auth proxied through `/api/__clerk` for custom domain support; `clerkMiddleware` on every API request
- All routes require `getAuth(req)` userId extraction — no public data endpoints except `/api/healthz`
- Account balance updated in real-time on trade create/update/delete (denormalized for performance)
- Dark-only design — CSS vars set for both `:root` and `.dark` (same values), `dark` class forced on `<html>`

## Product

- **Dashboard**: Equity curve (AreaChart), key stats (win rate, profit factor, PnL, streaks), pair breakdown, recent trades
- **Trade Log**: Full CRUD trade journal with pair, direction, session, RR, confluences, star rating, emotion score
- **Pre-Trade Checklist**: 20-minute countdown timer, emotion slider, 10 discipline checkpoints
- **Calendar**: Monthly view with green/red day tiles based on daily PnL
- **Weekly Review**: Structured reflection — what worked, what didn't, improvements, market conditions
- **Multi-Account Manager**: Add/switch accounts with broker, currency, and balance tracking

## User preferences

- Dark theme only: #0a0c10 background, #00e5a0 green accent
- No emojis in the UI (except text-based emotion slider representations)
- Mobile-first (max-width 430px), bottom nav bar navigation
- Professional, information-dense aesthetic — "Bloomberg terminal meets pilot cockpit"

## Gotchas

- Do NOT run `pnpm dev` at workspace root — use restart_workflow or workflow-specific commands
- After changing OpenAPI spec, always run `pnpm --filter @workspace/api-spec run codegen`
- After changing DB schema, run `pnpm --filter @workspace/db run push` (dev) — prod schema managed by Publish flow
- `lib/api-zod/src/generated/index.ts` is orval-generated — must only export `./api/api` (not `./api/api.schemas`)
- API server must build before starting: dev script runs `build && start`

## Pointers

- See `.local/skills/pnpm-workspace/references/openapi.md` for codegen conventions
- See `.local/skills/pnpm-workspace/references/server.md` for Express route patterns
- See `.local/skills/clerk-auth/references/setup-and-customization.md` for Clerk proxy setup
