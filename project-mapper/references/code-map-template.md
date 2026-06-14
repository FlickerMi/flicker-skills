# Code Map

> **Read this BEFORE exploring or grep'ing unfamiliar code.** This map tells you **where** things live; for **how** to diagnose a symptom, see `debug-playbook.md`. Module-level only — no function detail (it drifts and misleads). Use the "Where to look" table first to narrow to a subtree, then grep within only that subtree.

## Overview
[1–2 sentences: what this project does.] Stack: [languages, frameworks, key libs, DB, infra].

## Architecture
[A compact text or mermaid diagram of the major components and how they connect. Module-level boxes + arrows for the main flow. Keep it to the real shape of the system.]

## Directory map
| Path | Responsibility | Status | Key files / entry |
|------|----------------|--------|-------------------|
| `src/...` | [what lives here] | active | [main file(s)] |
[One row per significant top-level/module directory. Skip build artifacts and vendored deps. **Use the Status column to flag `legacy` / `deprecated` modules** so the agent knows which code is current — common in older or inherited codebases.]

## Entry points & boot flow
- **Start command**: [e.g. `npm run dev` / `mvn spring-boot:run`]
- **Boot sequence**: [entry file → what it wires up → ready state]

## Core flows
[For each primary use case, a one-line trace through the layers, e.g.:]
- Request: `route → controller X → service Y → repository Z → table T`
- [Any background jobs / schedulers / queues]

## Key modules
### [Module name]
- **Purpose**: [one line]
- **Location**: `path/`
- **Depends on**: [other modules / external services]
[Repeat per major module — aim for breadth, not depth.]

## External integrations
[DBs, third-party APIs, message queues, caches — what, where configured, which module owns it.]

## Conventions
[Naming, layering rules, where new code of a given type should go, config layering, anything an agent must respect to fit in.]

## Where to look (symptom → area index)

**This is the single most important section of the whole map.** It's what saves the agent from grep-the-whole-project on every bug. Aim for **15–25 rows**, mixing two axes — business surfaces *and* error surfaces — so the agent can match whatever phrasing the user used:

- *Business surfaces*: auth, payment, search, dashboard, admin, upload, notifications, jobs/cron, settings, etc.
- *Error surfaces*: 401/403, 500 on a specific endpoint family, validation failures, timeouts, cache misses, hydration errors, migration drift, etc.

| If you're touching / seeing... | Start in |
|--------------------------------|----------|
| auth / login / session expiry / 401 / 403 | `src/auth/` |
| payment / checkout / refund / webhook | `src/billing/` |
| 500 on `/api/orders/*` | `src/api/orders/`, then `src/services/order/` |
| validation errors on form submit | `src/lib/validation/` + the form's API route |
| stale data after write | `src/cache/` + service-layer invalidation |
| migration / schema drift / "column does not exist" | `migrations/`, `src/db/schema.ts` |
| slow query / timeout | `src/db/queries/` |
| background job stuck / not firing | `src/jobs/` + scheduler config |
| email / notification not sent | `src/notifications/` + provider config |
| env var "undefined" at runtime | `.env*`, `src/config/` |
| ... | ... |

[Don't ship 3 rows and call it done — this table is the whole point of the map. Mine symptoms from `rg 'logger\.(error|warn)|throw new|console\.error'`, `catch` blocks, test files named `*error*` / `*should-fail*`, and the last 100 `fix:` commits in git log.]
