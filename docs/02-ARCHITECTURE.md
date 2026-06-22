# Spectre — Architecture

**Last updated:** 2026-06-22

---

## 1. Stack Decisions (and why)

| Layer | Choice | Why |
|-------|--------|-----|
| Framework | **Next.js 15 (App Router)** | Server Components render pages on the server and ship minimal JS, directly serving the "fast" pillar. One codebase for UI + backend logic. |
| Language | **TypeScript** | Type safety across the data model and UEX response shapes; fewer runtime surprises as the app grows. |
| Styling | **Tailwind CSS** | Fast iteration, tiny production CSS, easy to encode the design system as tokens. |
| Database | **PostgreSQL** | Data is relational (users own fleets; orgs have ranked members). Reliable, upgradeable, the correct model. |
| ORM | **Drizzle ORM** | Lightweight, type-safe SQL, fast, no heavy runtime. Schema lives in code (`06-DATA-MODEL.md`). |
| Auth | **Auth.js (NextAuth v5)** | First-class Discord OAuth provider; session handling built in; extensible to email/password later. |
| Hosting | **Railway** (start) | Managed Postgres + app hosting in one place, cheap to start, scales up, and uses standard tech so we are never locked in. |
| Data source | **UEX API 2.0** | Community-maintained live SC data. See `05-DATA-UEX.md`. |

### Why Next.js over a split frontend/backend
Spectre's pages are predominantly "render data from the database behind a login." React Server Components handle exactly that with minimal shipped JavaScript and no separate API layer to maintain. A split architecture (standalone React SPA + separate API server) would mean two deployments, two auth surfaces, and hand-written endpoints for what Server Components and Server Actions do natively. The split only pays off if we later need a separate native mobile client or a public API, and both can be added **alongside** Next.js without a rewrite (expose Route Handlers as a versioned API when that day comes).

### Why a local DB cache of UEX data
UEX allows 120 requests/minute. Reading UEX live on every page load would be slow and would burn the quota under any real traffic. Instead, Spectre runs a **scheduled bulk sync** (nightly + on-demand) that pulls UEX data into Spectre's own Postgres. Pages then read from the local DB, which is fast and quota-free. See `05-DATA-UEX.md` for the sync design.

---

## 2. Data Flow

### Read path (typical page load)
1. Browser requests a page (e.g. `/trade/routes`).
2. Next.js Server Component runs on the server, queries **Postgres** (local cached UEX data + user data).
3. Fully rendered HTML is sent to the browser with minimal JS.
4. Interactive bits (filters, sort) hydrate as small client components.

### Write path (user mutation)
1. User submits (e.g. "add ship to fleet").
2. A **Server Action** (or Route Handler) validates input (Zod), checks the session, writes to Postgres.
3. The affected page revalidates and re-renders.

### Sync path (UEX → Postgres)
1. A scheduled job (cron on Railway, or a Route Handler hit by an external scheduler) calls UEX's `data_extract` / bulk endpoints.
2. Data is upserted into the `uex_*` cache tables in Postgres.
3. A `sync_log` row records timestamp + game version for staleness display.

---

## 3. Project Structure

```
spectre/
├── docs/                      # All documentation (this folder)
├── public/                    # Static assets, logo SVGs, favicons
├── src/
│   ├── app/                   # Next.js App Router
│   │   ├── (marketing)/       # Public pages (home/landing)
│   │   │   └── page.tsx        # Home page
│   │   ├── (app)/             # Authenticated app
│   │   │   ├── dashboard/
│   │   │   ├── fleet/
│   │   │   ├── trade/
│   │   │   ├── mining/
│   │   │   ├── org/
│   │   │   └── settings/
│   │   ├── api/               # Route Handlers (auth, sync, future public API)
│   │   ├── layout.tsx         # Root layout (fonts, theme)
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                # Design-system primitives (Panel, Stat, Button…)
│   │   └── features/          # Feature-specific composite components
│   ├── lib/
│   │   ├── db/                # Drizzle schema + client
│   │   ├── uex/               # UEX API client + sync jobs
│   │   ├── auth/              # Auth.js config
│   │   └── utils/
│   └── styles/                # Design tokens
├── drizzle/                   # DB migrations
├── .env.example               # Documented env vars (never commit real .env)
├── package.json
└── README.md
```

### Route groups
- `(marketing)` — unauthenticated, SEO-friendly public pages (home, about, legal).
- `(app)` — everything behind login. Shares the HUD dashboard shell (sidebar + top nav).

---

## 4. Environment Variables

Documented in `.env.example`; real values never committed.

```
# Database
DATABASE_URL=postgresql://...

# Auth.js
AUTH_SECRET=...                # generated, signs sessions
AUTH_DISCORD_ID=...            # Discord application client ID
AUTH_DISCORD_SECRET=...        # Discord application client secret

# UEX
UEX_API_TOKEN=...              # Bearer token from uexcorp.space My Apps
UEX_API_BASE=https://api.uexcorp.uk/2.0

# App
NEXT_PUBLIC_APP_URL=https://...
SYNC_SECRET=...                # protects the sync route handler from public calls
```

---

## 5. Performance Strategy (the "optimize it" requirement)

Concrete, non-degrading optimizations baked in from day one:

1. **Server Components by default.** Client components only where interactivity demands it. This is the single biggest JS-payload win.
2. **Local DB reads, not live API.** UEX data is cached; pages never block on an external call.
3. **Streaming + Suspense.** Slow sections stream in so the shell paints immediately.
4. **Indexed queries.** Every foreign key and common filter column is indexed (see `06-DATA-MODEL.md`).
5. **Static where possible.** Marketing pages are statically generated; app pages use request-time rendering with caching where data allows.
6. **Image discipline.** `next/image`, modern formats, lazy loading.
7. **Font subsetting.** Orbitron/Rajdhani loaded via `next/font` with only needed weights.

These are applied continuously, not as a later "optimization pass," because retrofitting performance is far more expensive than building it in.

---

## 6. Scaling Path (how it grows)

| Stage | Users | What changes |
|-------|-------|--------------|
| Launch | 1–500 | Railway hobby tier, single Postgres, single app instance. |
| Growth | 500–5k | Bump Railway plan, add connection pooling (PgBouncer), add a CDN for assets, cache hot queries. |
| Scale | 5k+ | Horizontal app scaling (stateless app makes this easy), read replicas for Postgres, move sync to a dedicated worker, consider Redis for sessions/cache. |

Nothing in the codebase assumes Railway specifically, so a move to a VPS, Fly.io, or a cloud provider is a deployment change, not a rewrite.
