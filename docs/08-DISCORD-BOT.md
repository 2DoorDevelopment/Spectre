# Spectre — Discord Integration

**Last updated:** 2026-06-22
**Status:** Planned (Phase 7) — built after the web data model is stable.

The Discord integration extends Spectre's most useful features into Discord servers. It is intentionally built **after** the web app so it can reuse the same database and data model rather than duplicating logic.

---

## 1. Scope (v1 of the bot)

Lightweight, read-focused, high-value commands:

| Slash command | Does |
|---------------|------|
| `/price <commodity>` | Best buy/sell locations + prices (from cached UEX data). |
| `/ship <name>` | Ship spec card (specs, crew, cargo, loaners). |
| `/route <from> <to>` | Best trade route / margin. |
| `/refine <ore>` | Refining yield/method summary. |
| `/status` | UEX data freshness (last sync). |

Later additions: org-linked features (roster lookups, event announcements, contribution updates) once org tools exist on the web side.

---

## 2. Architecture

The bot is a **separate service** from the Next.js app, but shares the same Postgres database (or calls a small internal Spectre API). This keeps the web app's request path clean and lets the bot run as a long-lived process.

```
Discord  ◄──gateway──►  Spectre Bot (Node, discord.js)
                              │
                              ├── reads cached UEX data  ──►  Postgres
                              │   (same DB as the web app)
                              │
                              └── (later) reads org data for linked servers
```

### Why a shared DB rather than the bot calling UEX directly
The web app already caches UEX data on a sync schedule. The bot reading that same cache means: one sync, consistent data everywhere, and the bot never touches UEX's rate limit. The bot becomes a thin presentation layer over data Spectre already has.

### Hosting
Runs as its own Railway service (or any always-on host). Stateless aside from the DB connection, so it scales/restarts cleanly.

---

## 3. Linking Discord ↔ Spectre

Two link levels:
1. **User link (already implicit):** since web auth is Discord OAuth, a user's Discord identity is already known to Spectre. The bot can match a Discord user to their Spectre account by `discord_id`.
2. **Server/org link (later):** an org admin links a Discord server (guild) to a Spectre org, enabling roster commands and channel announcements. Requires the `guilds` scope and a stored guild→org mapping.

---

## 4. Bot-Specific Considerations

- **Permissions:** request only the Discord permissions the commands need (slash commands, send messages, embeds). No admin perms.
- **Rate limits:** respect Discord's gateway/REST limits; discord.js handles most of this.
- **Secrets:** bot token in env, never committed.
- **Failure isolation:** if the bot is down, the web app is unaffected (separate service). If UEX sync is stale, the bot shows the same staleness notice the web app does.

---

## 5. Decision: third-party bots vs. custom

For **org server management** (moderation, roles, scheduling), established bots (Apollo for events, Carl-bot/AutoMod for moderation) are better than reinventing those wheels. Spectre's bot focuses only on **Spectre-specific data** (prices, ships, routes, org data from Spectre's DB) that no off-the-shelf bot can provide. This keeps the bot's scope tight and maintainable.
