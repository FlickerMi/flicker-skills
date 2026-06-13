---
name: project-mapper
description: Scan a codebase and generate two on-demand reference docs — a code-map (architecture/module-level map for orienting an AI agent fast) and a debug-playbook (scenario-based debugging guide). Use this whenever the user wants to map a project, document its structure for AI agents, create a code map / architecture overview, build a debug playbook, onboard onto an unfamiliar repo, or reduce token cost from agents scanning the whole project on every bug. Trigger even if the user only says "generate project docs", "help the AI understand my repo", "code map", or "debug guide" without naming this skill explicitly. Output language is configurable to Chinese or English (defaults to matching the repo's existing docs).
---

# Project Mapper

Generate two lean, on-demand reference documents for a codebase so an AI coding agent can **orient instantly** instead of scanning the whole project on every task.

## Why two separate docs (and not CLAUDE.md)

CLAUDE.md is auto-loaded into context every session — every line there is a fixed cost paid on every turn. These two docs are **loaded on demand** (the agent reads them with the Read tool only when needed), so they carry no idle cost.

- **code-map.md** — the architecture map. Answers "where does X live / how is this wired". Used for *orienting*: pointing the agent at the right subtree before it searches.
- **debug-playbook.md** — the debugging runbook. Answers "this symptom → look here → check this". Loaded only when debugging.

Division of labor: **code-map orients (rough direction), live grep/LSP locates (exact line).** Never document down to the function level — function-level detail drifts the moment code changes, becomes stale, and misleads the agent. Stop at the module/architecture level and let the agent's real-time search handle the rest.

## Workflow

### 1. Scan efficiently (don't read everything)

The scan itself must be token-cheap. Do NOT read every file. Build understanding in this order:

1. **Identify stack + entry points from manifests**, not source: `package.json`, `pom.xml`, `build.gradle`, `requirements.txt`/`pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, `docker-compose.yml`, `Makefile`, framework configs (`next.config.*`, `application.yml`, `vite.config.*`). These reveal language, framework, scripts, dependencies, and boot entry.
2. **List the directory tree** (2–3 levels, skip `node_modules`/`vendor`/`target`/`dist`/`.git`). Map top-level dirs to responsibilities from their names + a glance inside.
3. **Read only the orienting files**: the main/boot entry (`main.*`, `index.*`, `app.*`, `Application.java`), the routing/controller registry, central config, and ONE representative file per major module to confirm its role.
4. **Use grep/glob to enumerate, not read**: find all controllers/routes/services/models by pattern (e.g. `@RestController`, `@Service`, `export async function GET`, `router.(get|post)`) to list them without opening each.
5. **Trace the primary flow once** at a high level (e.g. request → controller → service → repository → DB; or page → API route → handler → DB).

If something can't be determined without a deep scan, write it as `TODO: verify` rather than guessing. Never fabricate structure.

### 2. Decide the doc language

Output language is **Chinese or English**, decided in this priority order:

1. **User-specified** — if the user asked for a language (e.g. "生成中文版" / "output in English" / "用英文"), use it. This always wins.
2. **Match repo convention** — otherwise, if `CLAUDE.md`/`AGENTS.md`/`README` exists, match its language (Chinese repo docs → Chinese; English → English).
3. **Fallback** — if neither applies, use the language the user is conversing in.

If the choice is ambiguous (e.g. the repo mixes both and the user didn't specify), ask once: "Chinese or English for the docs?" before writing. Apply the chosen language to **all prose** — section bodies, descriptions, table cells — while keeping code identifiers, paths, commands, and template section headers as-is.

### 3. Write the two docs

Read the two template files in `references/` and use them as the exact structure:
- `references/code-map-template.md`
- `references/debug-playbook-template.md`

Default output location: `docs/code-map.md` and `docs/debug-playbook.md`. If the repo already keeps docs elsewhere, match that. Fill every section with **real findings from the scan** — drop sections that genuinely don't apply rather than padding. Translate the prose into the chosen language (step 2) while keeping section headers, paths, and commands intact.

Tailor the debug-playbook's common-scenario list to the detected stack. Stack hints:
- **Spring Boot / Java**: bean injection failures, `@Transactional` not applying (self-invocation / non-public), MyBatis mapper-XML mismatch, lazy-load `LazyInitializationException`, datasource/connection-pool exhaustion, circular deps, profile/config not loaded.
- **Next.js / React / TS**: server vs client component boundary, hydration mismatch, API route handler errors, env var not exposed to client (`NEXT_PUBLIC_`), build vs runtime errors, stale cache / ISR, type errors hiding runtime bugs.
- **Python**: import path / circular import, virtualenv/dependency mismatch, async event-loop misuse, silent exception swallowing.
- **DB-heavy**: slow query / missing index, connection leak, transaction isolation, migration drift, dialect-specific SQL incompatibility.

Adapt to whatever stack is actually present.

### 4. Wire it into CLAUDE.md as a pointer (not an import)

If `CLAUDE.md` (or `AGENTS.md`) exists, add a **plain-text** pointer so the docs stay on-demand — do NOT use `@import` syntax, which would auto-inline them every session and defeat the purpose:

```markdown
## Reference docs (read on demand)
- Architecture / where things live → `docs/code-map.md`
- Debugging a specific symptom → `docs/debug-playbook.md`
```

Even better, if the project has a `/debug` slash command, suggest the user have it read `docs/debug-playbook.md` on invocation (trigger-time load, zero idle cost). If no CLAUDE.md exists, tell the user where the docs are and offer the pointer snippet.

### 5. Note maintenance cadence

Tell the user: regenerate when the **architecture** changes (new module, new integration, restructure) — not on every commit. Module-level docs are stable; that's the point.

---

## Skill structure

```
project-mapper/
├── SKILL.md                          # this file — workflow
├── README.md                         # human-facing overview (English)
├── README.zh-CN.md                   # human-facing overview (中文)
└── references/
    ├── code-map-template.md          # structure for docs/code-map.md
    └── debug-playbook-template.md    # structure for docs/debug-playbook.md
```

---

## Principles

- **Efficiency over completeness**: the goal is an agent that orients in one read, not an exhaustive wiki. Shorter and accurate beats long and stale.
- **No function-level detail**: it drifts and misleads. Module/architecture level only.
- **Honesty**: mark unknowns as `TODO: verify`; never invent structure to fill a template.
- **On-demand by design**: keep these out of auto-loaded context; reference them by plain path.
