# Spectre — Authentication

**Last updated:** 2026-06-22

Auth is handled by **Auth.js (NextAuth v5)** with the **Discord OAuth** provider. Email/password is deferred but the design leaves room for it.

---

## 1. Decision Summary

- **v1: Discord OAuth only.** No passwords to manage, no password-reset flows, no credential storage burden, and most Star Citizen communities already live on Discord. It also smooths the future Discord-bot integration (same identity).
- **The mockup's "Login with RSI" is not buildable:** RSI has no public OAuth. Discord is the realistic, standard choice and the button copy becomes "Sign in with Discord."
- **Future: email/password** can be added as a second Auth.js provider (Credentials) without disrupting existing Discord sessions. Tracked in the roadmap, not built yet.

---

## 2. Flow

```
User clicks "Sign in with Discord"
        │
        ▼
Discord OAuth consent  ──►  redirect back with code
        │
        ▼
Auth.js exchanges code for tokens, fetches Discord profile
        │
        ▼
Drizzle adapter upserts user (by discord_id) + creates session
        │
        ▼
Session cookie set  ──►  user lands on /dashboard
```

### Scopes
Request the minimum: `identify` (handle, id, avatar). Add `guilds` later only if/when the Discord integration needs to read the user's servers. Do not request more than a feature actively uses.

---

## 3. Configuration

`src/lib/auth/` holds the Auth.js config:
- Discord provider with `AUTH_DISCORD_ID` / `AUTH_DISCORD_SECRET`.
- Drizzle adapter pointed at the `users`/`accounts`/`sessions` tables.
- `AUTH_SECRET` signs/encrypts session tokens.
- Session strategy: database sessions (so we can revoke and so the bot can read session-linked data later).

Discord application setup (one-time, in the Discord Developer Portal):
1. Create an application → OAuth2 → add redirect URL `https://<app-url>/api/auth/callback/discord`.
2. Copy Client ID/Secret into env.
3. (Later) configure a bot user under the same application for Phase 7.

---

## 4. Authorization (who can do what)

- **App-level:** logged-in vs. anonymous. Marketing pages are public; everything under `(app)/` requires a session.
- **Org-level:** permissions come from `org_ranks.permissions` (jsonb) on the user's membership. Checked in Server Actions before any org mutation (e.g., only ranks with `manage_members` can change roles).
- Never trust the client for authorization; every mutation re-checks session + permission server-side.

---

## 5. Security Notes

- Secrets only in env vars, never in the repo (`.env.example` documents names, not values).
- CSRF handled by Auth.js for auth routes; Server Actions are origin-checked by Next.js.
- Rate-limit auth and sensitive endpoints at the edge when traffic grows.
- The optional UEX *secret key* (for power-user `user_*` endpoints) is stored encrypted at rest and never exposed to the client; treated as sensitive.

---

## 6. Development Login (testing without Discord)

You can test the full authenticated app locally without going through Discord OAuth each time. This is a standard dev affordance, not a bypass of the database.

### How it works
- A **Credentials provider** is added to Auth.js, **hard-gated** so it only exists when `NODE_ENV === "development"`. It is not merely hidden in production; it is absent from the production build, so it can never be used to sign in on the live site.
- It signs you in as a **seeded test user** — a real row in the `users` table. Sessions, org memberships, fleet data, and permission checks all behave exactly as they would for a real Discord login, because the underlying data is real DB data.
- A small dev **user-switcher** lets you jump between seeded users (e.g. an org owner vs. a regular member) to test permission-gated features without juggling real accounts.

### Seed data
A seed script (`src/lib/db/seed.ts`, run via a package script) populates:
- A handful of test users.
- A sample org with a rank ladder and memberships at different ranks.
- Sample fleet ships per user.

This gives every page realistic populated data immediately, while empty states are still testable by signing in as a fresh/empty user.

### Guardrails
- The dev provider is excluded at build time in production; even if the env were misconfigured, the code path does not ship.
- Seed scripts only run against the local/dev database, never automatically against production.
- This is documented and reviewed so it can't silently leak into a release.

---

## 7. Future: Email/Password

When added:
- Add Auth.js Credentials provider + a `password_hash` column (Argon2id) on `users`.
- Add verification + reset flows (email provider needed).
- Allow account linking (a user can attach both Discord and email/password to one account).
- None of this changes the existing Discord path.
