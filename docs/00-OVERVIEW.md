# Spectre — Project Overview

> A Star Citizen command center: fleet, org, trade, and mining tools in one fast, clean web app, with optional Discord integration.

**Status:** Pre-development (documentation + design phase)
**Last updated:** 2026-06-22

---

## 1. What Spectre Is

Spectre is a web-first companion application for Star Citizen players. It pulls live game data (commodity prices, ship specs, locations, mining yields) from the community-maintained UEX API, layers on personal tools (fleet tracking, trade/mining calculators, route planning), and provides organization-management features for groups. A Discord integration extends key features into Discord servers.

The product is built **broad in scope but delivered narrow and deep**: one feature area is built to completion (or to a stable, usable state) before the next begins. The home page is the first deliverable.

### Design pillars (in priority order)

1. **Function first.** Every feature must work correctly and reliably before it is considered done.
2. **Fast.** Pages should ship minimal client-side JavaScript and read from a local database, not hammer external APIs on every request. Performance is a feature, not an afterthought.
3. **Distinctive look.** The HUD/sci-fi aesthetic (see `04-DESIGN-SYSTEM.md`) should feel unique, not templated.
4. **Scalable foundation.** Architecture choices must allow growth from a handful of users to a large community without a rewrite.

### Explicit non-goals (for now)

- No native mobile app (the web app will be responsive; a mobile-specific build is a future consideration).
- No public Spectre API for third parties (future consideration).
- No real-money transactions, no grey-market facilitation.
- Not affiliated with Cloud Imperium Games (CIG) or Roberts Space Industries (RSI). This must be stated in the footer of every page.

---

## 2. The Page-by-Page Build Philosophy

Rather than building all pages as shells and filling them in, Spectre is built **one page end-to-end at a time**. Each page goes: layout → data wiring → interactivity → polish → "done enough," then we move on.

The foundation (project skeleton, database, authentication, design system) is stood up **once** at the start, because nearly every page depends on it. After that, pages are built sequentially.

See `03-ROADMAP.md` for the full page order and phase breakdown.

---

## 3. High-Level Architecture (summary)

```
┌─────────────────────────────────────────────────────────┐
│  Browser (Next.js frontend, mostly server-rendered)      │
└───────────────┬─────────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────────┐
│  Next.js App (App Router, React Server Components)        │
│  - Pages render on the server, ship minimal JS           │
│  - Server Actions + Route Handlers for mutations          │
│  - Auth.js handles Discord OAuth + sessions               │
└───────┬───────────────────────────────┬─────────────────┘
        │                               │
┌───────▼──────────┐         ┌──────────▼──────────────────┐
│  Postgres (DB)   │         │  UEX API (external)          │
│  - users, orgs   │◄────────│  Nightly bulk sync via       │
│  - fleets, etc.  │  sync   │  data_extract endpoint       │
│  - cached UEX    │         │  (so pages read local DB)    │
│    game data     │         └─────────────────────────────┘
└──────────────────┘
        ▲
        │ (future)
┌───────┴──────────┐
│  Discord Bot     │  Reads/writes the same DB or calls
│  (separate svc)  │  Spectre's internal API
└──────────────────┘
```

Full detail in `02-ARCHITECTURE.md`.

---

## 4. Document Index

| Doc | Purpose |
|-----|---------|
| `00-OVERVIEW.md` | This file. The 30,000-ft view. |
| `01-BRAND.md` | Name, logo, voice, naming constraints (CIG/RSI). |
| `02-ARCHITECTURE.md` | Tech stack, data flow, hosting, project structure. |
| `03-ROADMAP.md` | Phased build plan, page-by-page order, milestones. |
| `04-DESIGN-SYSTEM.md` | Colors, type, components, spacing, the HUD aesthetic. |
| `05-DATA-UEX.md` | UEX API reference, auth, endpoints, sync strategy. |
| `06-DATA-MODEL.md` | Database schema, entities, relationships. |
| `07-AUTH.md` | Discord OAuth, sessions, local-account future path. |
| `08-DISCORD-BOT.md` | Bot scope, architecture, how it shares data. |
| `09-CONVENTIONS.md` | Code style, naming, folder rules, git workflow. |

---

## 5. Open Questions / Decisions Pending

> Nothing in these docs is locked. The roadmap, page contents, and feature lists are a working plan; items can be added or cut at any time, ideally before they're built. This section just tracks the live decisions.

These are tracked here and resolved as the project progresses.

- [ ] Final domain name (Spectre.com likely taken; need a variant).
- [ ] Which feature area follows the dashboard + landing page (Fleet vs. Tools vs. Org)? — decide together once those are built.
- [ ] Whether to add email/password auth in v1 or defer (currently: defer, Discord-only, with a dev-login for local testing).
- [ ] Discord bot timing (built after the web app has a stable data model).
