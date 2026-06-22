# Spectre

> A Star Citizen command center: fleet, org, trade, and mining tools in one fast, clean web app, with optional Discord integration.

**Status:** Pre-development (documentation + design phase). No application code yet; this repository currently holds the project's design and planning documentation.

Spectre is a web-first companion app for Star Citizen players. It caches live game data from the community-maintained UEX API into its own database, layers on personal tools (fleet tracking, trade and mining calculators, route planning), and adds organization-management features for groups. A Discord integration extends key features into Discord servers.

It is built **narrow and deep**: one feature area is completed before the next begins, and the foundation (skeleton, database, auth, design system) is stood up once at the start.

---

## Documentation

All planning and design docs live in [`/docs`](./docs). Read them in order; `00-OVERVIEW.md` is the entry point.

| Doc | Purpose |
|-----|---------|
| [`00-OVERVIEW.md`](./docs/00-OVERVIEW.md) | The 30,000-ft view, build philosophy, open questions. |
| [`01-BRAND.md`](./docs/01-BRAND.md) | Name, logo, voice, CIG/RSI legal constraints. |
| [`02-ARCHITECTURE.md`](./docs/02-ARCHITECTURE.md) | Tech stack, data flow, background jobs, hosting, structure. |
| [`03-ROADMAP.md`](./docs/03-ROADMAP.md) | Phased build plan, page-by-page order, milestones. |
| [`04-DESIGN-SYSTEM.md`](./docs/04-DESIGN-SYSTEM.md) | Colors, type, components, spacing, the HUD aesthetic. |
| [`05-DATA-UEX.md`](./docs/05-DATA-UEX.md) | UEX API reference, auth, endpoints, sync strategy. |
| [`06-DATA-MODEL.md`](./docs/06-DATA-MODEL.md) | Database schema, entities, relationships. |
| [`07-AUTH.md`](./docs/07-AUTH.md) | Discord OAuth, sessions, dev login, local-account future path. |
| [`08-DISCORD-BOT.md`](./docs/08-DISCORD-BOT.md) | Bot scope, architecture, how it shares data. |
| [`09-CONVENTIONS.md`](./docs/09-CONVENTIONS.md) | Code style, naming, folder rules, git workflow. |
| [`10-FUTURE-FEATURES.md`](./docs/10-FUTURE-FEATURES.md) | Parking lot: uncommitted ideas kept out of the roadmap. |

---

## Intended stack (planned, not yet built)

Next.js 15 (App Router, React Server Components), TypeScript, Tailwind CSS, PostgreSQL with Drizzle ORM, Auth.js (Discord OAuth), hosted on Railway. Game data from the UEX API 2.0, cached locally. Full rationale in [`02-ARCHITECTURE.md`](./docs/02-ARCHITECTURE.md).

---

## Legal

Spectre is an unofficial, community-made tool and is not affiliated with Cloud Imperium Games or Roberts Space Industries. All game content and materials are property of their respective owners. Game data is provided by [UEX](https://uexcorp.space) and is community-maintained.
