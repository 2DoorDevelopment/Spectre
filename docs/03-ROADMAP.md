# Spectre — Roadmap

**Last updated:** 2026-06-22

The build follows the **page-by-page** philosophy: stand up the foundation once, then build each page end-to-end before moving to the next.

---

## Phase 0 — Foundation (one-time setup)

Built before any feature page. Everything downstream depends on it.

- [ ] Initialize Next.js 15 + TypeScript + Tailwind project.
- [ ] Install and configure the design system (tokens, fonts, base components: Panel, Stat, Button, Nav).
- [ ] Set up Postgres + Drizzle, run initial migration (users, sessions, sync_log).
- [ ] Configure Auth.js with Discord OAuth (login/logout works end to end).
- [ ] Add a **dev-only credentials provider** (hard-gated to `NODE_ENV === "development"`, absent from production builds) for one-click login as a seeded test user. See `07-AUTH.md`.
- [ ] Write a **seed script**: a few test users, a sample org with ranks, and sample fleet ships, so pages have realistic data to render against. Include a dev user-switcher.
- [ ] Build the **app shell**: dark HUD layout, sidebar, top nav, footer with the CIG/RSI disclaimer.
- [ ] Stand up the UEX client + a first manual sync of core reference data (ships, commodities, locations).
- [ ] Deploy a "hello world" to Railway to confirm the pipeline.

**Exit criteria:** A logged-in user (real Discord or dev test user) sees an empty but styled dashboard shell; seed + UEX data is in the DB; the app is deployed.

---

## Phase 1 — Dashboard (authenticated home)  ← FIRST PAGE

The post-login core, the first page you'll actually use day to day. Starts empty-but-styled and fills in as features land. Built first because it's the heart of the app; the public landing page is polished afterward, once there's a real app behind it to showcase.

- [ ] Welcome panel (citizen name, org, current patch — patch pulled from UEX `game_versions`).
- [ ] Quick stats (ships owned, locations visited, etc. — placeholders until Fleet/other features exist).
- [ ] Quick actions grid (links to feature pages, some disabled until built).
- [ ] Recent activity feed (empty state until there's activity).
- [ ] System status (UEX reachability, last sync time from `sync_log`).
- [ ] Responsive down to mobile.

**Exit criteria:** A logged-in user lands on a polished dashboard shell with real (seeded/synced) data where available and clean empty states elsewhere; responsive; fast.

> Feature contents below are a working plan, not locked. Pages and items can be added or cut at any time — ideally before they're built.

---

## Phase 2 — Home / Landing Page

The public face, polished once the dashboard exists so it can showcase the real thing. Mostly presentational. Matches the supplied mockup's intent.

- [ ] Hero: "Your Star Citizen Command Center," tagline, CTA (Sign in with Discord).
- [ ] Feature highlights (Fleet, Trade, Mining, Org).
- [ ] Live demo/preview visual of the dashboard.
- [ ] System-status / "built by the community" trust strip.
- [ ] Footer with disclaimer + UEX credit.
- [ ] Responsive down to mobile.
- [ ] Performance pass (static generation, optimized hero asset).

**Exit criteria:** Home page is visually polished, responsive, fast (Lighthouse 95+), and the Discord sign-in works from it.

> After Phase 2, we choose the next feature area together. Recommended order below, but it is flexible.

---

## Phase 3 — Fleet

- [ ] Browse the full ship database (from UEX `vehicles`).
- [ ] Add ships to a personal hangar.
- [ ] Ship detail view (specs, components, loaner matrix).
- [ ] Fleet summary stats.

---

## Phase 4 — Trade Tools

- [ ] Commodity browser + live prices (UEX `commodities_prices`).
- [ ] Best-route finder (UEX `commodities_routes`).
- [ ] Price finder by commodity/location.
- [ ] Personal trade log (optional, user-entered).

---

## Phase 5 — Mining Tools

- [ ] Refinery yields + methods (UEX `refineries_*`).
- [ ] Mining location browser.
- [ ] Rock/refining calculator (the cryon-style calculator goal).
- [ ] Mineral pricing reference.

---

## Phase 6 — Organization Tools

- [ ] Create/join an org within Spectre.
- [ ] Member roster with ranks/roles.
- [ ] Ops scheduling / events.
- [ ] Contribution tracking (optional).

---

## Phase 7 — Discord Integration

Built after the web data model is stable so the bot can share it. See `08-DISCORD-BOT.md`.

- [ ] Discord bot service (slash commands for prices, ship lookup, route finder).
- [ ] Link a Discord account/server to a Spectre org.
- [ ] Push notifications/alerts to Discord channels.

---

## Phase 8+ — Polish & Future

- [ ] Email/password auth (if still wanted).
- [ ] User settings & preferences.
- [ ] Public Spectre API (if there's demand).
- [ ] Mobile-specific optimizations / PWA.

---

## Milestone Definition of "Done" (per page)

A page is "done enough to move on" when:
1. It works correctly with real data.
2. It is responsive (desktop + mobile).
3. It matches the design system.
4. It has loading and empty states.
5. It has basic error handling (e.g., UEX unreachable → cached/fallback).
6. It passes a quick performance check.

Pages can be revisited for enhancement later; this bar just means "stable and usable."
