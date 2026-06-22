# Spectre — Data Model

**Last updated:** 2026-06-22

Schema is defined in code with Drizzle ORM (`src/lib/db/schema.ts`). This doc is the human-readable reference. Two broad groups: **Spectre data** (users, orgs, fleets) and **UEX cache** (`uex_*` tables mirroring synced game data).

---

## 1. Entity Overview

```
users ──< fleet_ships >── uex_vehicles
  │
  ├──< org_memberships >── orgs ──< org_events
  │                          │
  │                          └──< org_ranks
  │
  └── accounts / sessions (Auth.js)

uex_vehicles, uex_commodities, uex_commodity_prices,
uex_terminals, uex_locations, uex_refinery_yields, ...  (synced cache)

sync_log  (sync bookkeeping)
```

---

## 2. Spectre Core Tables

### `users`
| Column | Type | Notes |
|--------|------|-------|
| id | uuid PK | |
| discord_id | text unique | from OAuth |
| handle | text | display name (RSI handle, user-editable) |
| avatar_url | text | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

### `accounts`, `sessions`, `verification_tokens`
Managed by Auth.js (Drizzle adapter). Standard schema; see `07-AUTH.md`.

### `fleet_ships`
A user's owned/pledged ships.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid PK | |
| user_id | uuid FK → users | indexed |
| vehicle_id | int FK → uex_vehicles.uex_id | indexed |
| nickname | text | optional custom name |
| status | text | owned / pledged / loaner |
| acquired_at | date | optional |
| notes | text | optional |

### `orgs`
| Column | Type | Notes |
|--------|------|-------|
| id | uuid PK | |
| name | text | |
| tag | text | short org tag |
| owner_id | uuid FK → users | |
| description | text | |
| created_at | timestamptz | |

### `org_ranks`
Configurable rank ladder per org (supports custom themes, e.g. Norse).
| Column | Type | Notes |
|--------|------|-------|
| id | uuid PK | |
| org_id | uuid FK → orgs | indexed |
| name | text | e.g. "Jarl" |
| level | int | sort/authority order |
| permissions | jsonb | what the rank can do |

### `org_memberships`
| Column | Type | Notes |
|--------|------|-------|
| id | uuid PK | |
| org_id | uuid FK → orgs | indexed |
| user_id | uuid FK → users | indexed |
| rank_id | uuid FK → org_ranks | |
| division_tags | text[] | optional divisional tags |
| joined_at | timestamptz | |
| UNIQUE(org_id, user_id) | | one membership per org |

### `org_events`
| Column | Type | Notes |
|--------|------|-------|
| id | uuid PK | |
| org_id | uuid FK → orgs | indexed |
| title | text | |
| description | text | |
| starts_at | timestamptz | |
| created_by | uuid FK → users | |

---

## 3. UEX Cache Tables (`uex_*`)

Mirror UEX data so pages read locally. Each keeps the UEX primary id plus a `synced_at`. Exact columns mirror the UEX response (validated via Zod on sync). Representative set:

| Table | Source endpoint(s) | Key fields |
|-------|--------------------|-----------|
| `uex_vehicles` | `vehicles`, `vehicles_loaners` | uex_id, name, manufacturer, role, crew, cargo, specs |
| `uex_vehicle_prices` | `vehicles_purchases_prices`, `_rentals_prices` | vehicle_id, terminal_id, buy/rent price |
| `uex_commodities` | `commodities` | uex_id, name, code, is_raw |
| `uex_commodity_prices` | `commodities_prices_all` | commodity_id, terminal_id, buy, sell, scu |
| `uex_terminals` | `terminals` | uex_id, name, type, location refs |
| `uex_locations` | `star_systems`, `planets`, `moons`, `space_stations`, `cities`, `outposts`, `poi` | normalized location tree |
| `uex_routes` | `commodities_routes` | origin, destination, commodity, margin |
| `uex_refinery_yields` | `refineries_yields`, `_methods`, `_capacities` | material, method, yield, time |
| `uex_fuel_prices` | `fuel_prices_all` | terminal_id, type, price |
| `uex_game_versions` | `game_versions` | current patch string |

> Location modeling: UEX exposes systems → planets → moons → stations/cities/outposts/POIs separately. Spectre normalizes these into a `uex_locations` tree (with a `parent_id` and `kind`) so the UI can browse hierarchically and joins stay simple. Mapping logic lives in `src/lib/uex/`.

---

## 4. `sync_log`
| Column | Type | Notes |
|--------|------|-------|
| id | uuid PK | |
| resource | text | which endpoint group |
| status | text | ok / error / rate_limited |
| game_version | text | patch at sync time |
| rows | int | rows upserted |
| synced_at | timestamptz | indexed |
| message | text | error detail if any |

Powers "last synced Xh ago" and staleness warnings in the UI.

---

## 5. Indexing & Integrity Rules

- Index every foreign key and every common filter/sort column (price, margin, location).
- Use DB-level unique constraints for natural keys (`discord_id`, `(org_id, user_id)`, UEX ids).
- `uex_*` tables are **replaceable cache**: a sync can truncate-and-reload or upsert; no user data lives there, so they're safe to rebuild.
- User data tables use soft deletes only where history matters (e.g., org membership); otherwise hard delete is fine.

---

## 6. Migration Workflow

- Schema changes are made in `schema.ts`, then `drizzle-kit generate` produces a SQL migration in `drizzle/`.
- Migrations are committed and run on deploy.
- Never edit a shipped migration; add a new one.
