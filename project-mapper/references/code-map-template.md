# Code Map

> Architecture-level map for orienting. For exact lines, the agent should grep/use LSP from the area this map points to. Module-level only — no function-level detail (it drifts).

## Overview
[1–2 sentences: what this project does.] Stack: [languages, frameworks, key libs, DB, infra].

## Architecture
[A compact text or mermaid diagram of the major components and how they connect. Module-level boxes + arrows for the main flow. Keep it to the real shape of the system.]

## Directory map
| Path | Responsibility | Key files / entry |
|------|----------------|-------------------|
| `src/...` | [what lives here] | [main file(s)] |
[One row per significant top-level/module directory. Skip build artifacts and vendored deps.]

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
| If you're touching / seeing... | Start in |
|--------------------------------|----------|
| [auth / login] | `path/` |
| [a specific API / page] | `path/` |
| [DB / data layer] | `path/` |
[This index is the highest-value section for cutting scan cost — make it concrete.]
