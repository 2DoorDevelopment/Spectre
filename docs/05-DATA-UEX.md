# Spectre — Data Layer (UEX API)

**Last updated:** 2026-06-22

Spectre's Star Citizen game data comes from the **UEX API 2.0**, a community-maintained dataset.

---

## 1. Reference Links

| Resource | URL |
|----------|-----|
| API documentation home | https://uexcorp.space/api/documentation/ |
| Create app / get token ("My Apps") | https://uexcorp.space/api/apps |
| Postman collection (typed reference) | https://www.postman.com/starcitizen/star-citizen/collection/xheji6p/uex-corp-api |
| Community tools (examples) | https://uexcorp.space/api/community_made |
| Webhooks | https://uexcorp.space/api/webhooks/ |
| UEX Discord (support) | https://discord.gg/bF4W6V4xVD |

---

## 2. Authentication & Limits

| Property | Value |
|----------|-------|
| Base URL | `https://api.uexcorp.uk/2.0/` |
| Auth method | **Bearer token** (`Authorization: Bearer <token>`) |
| Get token | Create an app on the **My Apps** page (linked above) |
| Daily quota | 172,800 requests/day (**120 requests/minute**) |
| Methods | GET, POST, DELETE |
| Format | `application/json` |
| Optional version lock | `X-Client-Version` header if you lock a client version on the key |

### Response shapes
| Case | Body |
|------|------|
| Success | `{ "status": "ok", "data": ... }` |
| Internal error | `{ "status": "error", "http_code": 500, "message": ... }` |
| Rate limited | `{ "status": "requests_limit_reached", "data": ... }` |

**Always** check `status` before using `data`. Treat `requests_limit_reached` as a signal to back off and serve cached data.

---

## 3. URL Patterns

```
https://api.uexcorp.uk/2.0/{resource}/
https://api.uexcorp.uk/2.0/{resource}/{param1}/{value1}/{param2}/{value2}/
https://api.uexcorp.uk/2.0/{resource}/?{param1}={value1}&{param2}={value2}
```

---

## 4. Endpoints Spectre Will Use

Grouped by feature phase. (GET unless noted.)

### Reference / sync (foundational — cached locally)
| Endpoint | Use |
|----------|-----|
| `game_versions` / `game_versions_all` | Current patch (shown in dashboard/home). |
| `star_systems`, `planets`, `moons`, `orbits`, `space_stations`, `cities`, `outposts`, `poi` | Location data for browsers & calculators. |
| `terminals` | Trade/shop terminals + locations. |
| `vehicles`, `vehicles_loaners` | Ship database + loaner matrix (Fleet). |
| `commodities` | Commodity master list. |
| `data_extract` | **Bulk static-data dump** — the "extracted data package." Use this to seed/refresh reference tables in fewer calls. |
| `data_parameters` | Valid parameters for queries. |

### Trade (Phase 4)
| Endpoint | Use |
|----------|-----|
| `commodities_prices` / `commodities_prices_all` | Live buy/sell prices per terminal. |
| `commodities_routes` | Best-route finder. |
| `commodities_averages`, `commodities_ranking` | Market context. |
| `fuel_prices` / `fuel_prices_all` | Fuel cost for route math. |

### Mining (Phase 5)
| Endpoint | Use |
|----------|-----|
| `commodities_raw_prices` / `_all`, `commodities_raw_averages` | Raw ore pricing. |
| `refineries_methods`, `refineries_yields`, `refineries_capacities`, `refineries_audits` | Refining calculator inputs. |

### Vehicles pricing (Fleet detail, Phase 3)
| Endpoint | Use |
|----------|-----|
| `vehicles_prices`, `vehicles_purchases_prices(_all)`, `vehicles_rentals_prices(_all)` | Buy/rent pricing for ships. |

### Org / user (optional, later)
| Endpoint | Use |
|----------|-----|
| `organizations` | Org lookup. |
| `user`, `user_trades`, `user_refineries_jobs`, `wallet_balance` | Only if a user supplies their own UEX secret key (opt-in). |

> Note: `user_*` and `wallet_*` endpoints act on a UEX *user's* own account and require that user's secret key. Spectre will not require this; it's an optional power-user feature, kept clearly separate from Spectre's own auth.

---

## 5. Sync Strategy (the key design)

**Principle:** pages read Spectre's Postgres, never UEX live. A background sync keeps Postgres fresh.

### Tiers by volatility
| Data | Volatility | Sync cadence |
|------|-----------|--------------|
| Systems, planets, terminals, ship specs | Rarely changes (per patch) | On deploy + weekly, or when `game_versions` changes. |
| Commodity & ore prices, routes, fuel | Changes constantly (community reports) | Nightly, plus optional on-demand refresh with caching. |

### Mechanism
1. A protected Route Handler `POST /api/sync` (guarded by `SYNC_SECRET`) runs the sync.
2. Railway cron (or any external scheduler) hits it on schedule.
3. The handler calls UEX (using `data_extract` for bulk reference data; targeted endpoints for prices), then **upserts** into `uex_*` tables.
4. Writes a `sync_log` row: `{ resource, status, game_version, synced_at, rows }`.
5. The UI reads `sync_log` to show "last synced Xh ago" and staleness warnings.

### Rate-limit discipline
- Stay well under 120 req/min: batch with the `_all` and `data_extract` endpoints; add a small delay between calls in a sync run.
- On `requests_limit_reached`, abort the run gracefully and retry later; never fail a user page because a background sync hit the cap.

---

## 6. Client Implementation Notes

A typed UEX client lives in `src/lib/uex/`:
- A single `uexFetch(resource, params)` wrapper that injects the Bearer token, sets headers, parses the `status`/`data` envelope, and throws typed errors on non-`ok` statuses.
- Zod schemas per endpoint to validate incoming shapes (UEX is crowdsourced; defensive parsing matters).
- Sync functions per resource group that map UEX rows → Drizzle upserts.

### Attribution
Where UEX data is displayed, credit UEX per their badge guidance (https://uexcorp.space/about/resources) and link back. This is both courteous and aligned with their terms.
