# Copilot / AI reviewer context for Spectre

This repository is currently **documentation only** (pre-development). When reviewing, evaluate the docs in `/docs` for clarity, internal consistency, correctness, and feasibility. There is no application code yet, so do not flag "missing implementation."

## Read these first (all of them)
Before reviewing or answering, read **every** doc in `/docs`, in order. They cross-reference each other constantly, so judging any one in isolation produces wrong conclusions (e.g. the data model only makes sense alongside the UEX and auth docs). The full set:

1. `docs/00-OVERVIEW.md` — project scope, build philosophy, open questions
2. `docs/01-BRAND.md` — name, logo, voice, CIG/RSI legal limits
3. `docs/02-ARCHITECTURE.md` — stack, data flow, background jobs, scaling
4. `docs/03-ROADMAP.md` — phased build plan, milestones
5. `docs/04-DESIGN-SYSTEM.md` — colors, type, components, HUD aesthetic
6. `docs/05-DATA-UEX.md` — UEX API reference, sync strategy
7. `docs/06-DATA-MODEL.md` — database schema, relationships
8. `docs/07-AUTH.md` — Discord OAuth, sessions, dev login
9. `docs/08-DISCORD-BOT.md` — bot scope and architecture
10. `docs/09-CONVENTIONS.md` — code style, naming, git workflow
11. `docs/10-FUTURE-FEATURES.md` — parking lot of uncommitted ideas

If a claim seems wrong, check whether another doc already addresses it before flagging it.

## What this project is
A Star Citizen companion web app. It caches data from the community UEX API into its own Postgres, adds personal tools (fleet, trade, mining) and org-management features, plus an optional Discord bot. Built narrow-and-deep, one feature area at a time.

## Decided stack (do not suggest swapping wholesale)
Next.js 15 App Router with React Server Components, TypeScript, Tailwind, PostgreSQL + Drizzle ORM, Auth.js (Discord OAuth), Railway hosting. These are deliberate choices with rationale in `docs/02-ARCHITECTURE.md`. Critique trade-offs if you see real problems, but don't propose replacing the whole stack.

## Decisions already made (don't re-litigate as if they were oversights)
- **Discord-only auth for v1.** Email/password is intentionally deferred (`docs/07-AUTH.md`). RSI has no public OAuth, so "Login with RSI" is not buildable.
- **Local DB cache of UEX data.** Pages read Postgres, never UEX live, to respect the 120 req/min limit. Sync is a scheduled job.
- **UEX `fleet`/`organizations` endpoints are intentionally not mirrored.** Spectre's fleet and orgs are native concepts, not UEX lookups (`docs/05-DATA-UEX.md`).
- **Multi-org is supported by schema** (`UNIQUE(org_id, user_id)` only blocks duplicate membership in the same org). The remaining question is a UX one, already tracked in `docs/00-OVERVIEW.md`.
- **Norse/Einherjar theming is org-specific**, not a default for the app.

## Known-correct details that look wrong but aren't
- **UEX API base URL:** standardize on `https://api.uexcorp.space/2.0/`. UEX's own docs also show `api.uexcorp.uk` as an alternate host; both resolve. This is documented, not a typo (`docs/05-DATA-UEX.md`).
- **`data_extract` is a small-summary endpoint**, not a bulk dump. Reference-data seeding uses the individual endpoints. This was already corrected in the docs.

## Where to focus a review
Internal contradictions between docs; schema integrity (foreign keys, sync safety); feasibility of the roadmap ordering; gaps in error handling, accessibility, or security guidance; anything factually wrong about the UEX API or the chosen libraries. Point to the specific doc and explain why it matters.
