# Spectre — Conventions

**Last updated:** 2026-06-22

Keep the codebase consistent, readable, and easy to grow.

---

## 1. Code Style

- **TypeScript strict mode** on. No `any` without a written reason.
- **Server Components by default.** Add `"use client"` only when interactivity requires it, and keep client components small and leaf-level.
- **Validation at boundaries:** every external input (forms, UEX responses, route params) is validated with **Zod** before use.
- **Errors:** never swallow silently. Throw typed errors; handle them at the page/route level with user-facing fallbacks (cached data, empty states).
- **Comments:** explain *why* for non-obvious logic only. Self-documenting names over narration.
- **Formatting/linting:** Prettier + ESLint, enforced; no debate over style in review.

---

## 2. Naming

| Thing | Convention | Example |
|-------|------------|---------|
| Files (components) | PascalCase | `FleetTable.tsx` |
| Files (utils/lib) | kebab-case | `uex-client.ts` |
| React components | PascalCase | `function StatPanel()` |
| Variables/functions | camelCase | `getFleetSummary()` |
| DB tables/columns | snake_case | `fleet_ships`, `synced_at` |
| Env vars | SCREAMING_SNAKE | `UEX_API_TOKEN` |
| Routes | kebab-case | `/trade/price-finder` |

---

## 3. Folder Rules

- `components/ui/` = generic design-system primitives, no business logic.
- `components/features/` = composite, feature-aware components.
- `lib/` = non-React logic (db, uex, auth, utils). No JSX here.
- Co-locate a page's private components in its route folder if not reused.
- One source of truth for design tokens (Tailwind config + CSS vars).

---

## 4. Data Access

- All DB access goes through `lib/db`; pages/components never write raw SQL inline.
- All UEX access goes through `lib/uex`; never call UEX directly from a page.
- Mutations happen in **Server Actions** (or Route Handlers), always after a session + permission check.

---

## 5. Git Workflow

- `main` is always deployable.
- Feature branches per page/feature: `feat/home-page`, `feat/fleet`, `fix/...`.
- Conventional-commit style messages: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`.
- PRs (even solo) for non-trivial changes, so there's a reviewable history.
- Migrations committed alongside the schema change that needs them.

---

## 6. Definition of Done (per page)

Restated from the roadmap so it's enforceable in review:
1. Works correctly with real data.
2. Responsive (desktop + mobile).
3. Matches the design system.
4. Has loading + empty states.
5. Has error handling (UEX/DB failure → graceful fallback).
6. Passes a quick performance check (no needless client JS, indexed queries).

---

## 7. Performance Guardrails (continuous)

- Default to Server Components; justify every `"use client"`.
- No N+1 queries; batch and index.
- No blocking external calls in the request path (read cache).
- `next/image` + `next/font`; no unoptimized media.
- Respect `prefers-reduced-motion` for all animation.
