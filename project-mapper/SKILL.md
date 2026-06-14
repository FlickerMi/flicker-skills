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

**Before anything else, anchor to the repo root.** Run `git rev-parse --show-toplevel` to find the git repo root and treat that as the base directory for both the scan and the output. If the cwd is not a git repo, use the cwd. **Do not anchor to a sub-project**, even if the user invoked the skill from inside one — the map describes the whole repo.

If the root contains multiple sub-projects (e.g. `frontend/` + `backend/`, `apps/*` + `packages/*`, `client/` + `server/`), the docs cover **all of them in a single pair of files** at the root. Do not pick one and ignore the others, and do not generate separate doc sets per sub-project.

The scan itself must be token-cheap. Do NOT read every file. Build understanding in this order:

1. **Identify stack + entry points from manifests**, not source: `package.json`, `pom.xml`, `build.gradle`, `requirements.txt`/`pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, `docker-compose.yml`, `Makefile`, framework configs (`next.config.*`, `application.yml`, `vite.config.*`). These reveal language, framework, scripts, dependencies, and boot entry. In a monorepo, repeat this for **each** sub-project so the map reflects every half of the stack.
2. **List the directory tree** (2–3 levels, skip `node_modules`/`vendor`/`target`/`dist`/`.git`). Map top-level dirs to responsibilities from their names + a glance inside.
3. **Read only the orienting files**: the main/boot entry (`main.*`, `index.*`, `app.*`, `Application.java`), the routing/controller registry, central config, and ONE representative file per major module to confirm its role.
4. **Use grep/glob to enumerate, not read**: find all controllers/routes/services/models by pattern (e.g. `@RestController`, `@Service`, `export async function GET`, `router.(get|post)`) to list them without opening each.
5. **Trace the primary flow once** at a high level (e.g. request → controller → service → repository → DB; or page → API route → handler → DB).

**For unfamiliar projects** (the user didn't write this code, no maintainer to ask — the primary use case for this skill): pull extra signal from project history before mapping, because business context is otherwise unrecoverable.
- `git log --oneline -100` — what areas have been active recently? `fix:`-prefixed commits reveal real bug patterns worth seeding the playbook with.
- `CHANGELOG.md` / `RELEASES.md` — recent module changes, breaking changes, the actual "what's been touched lately" story when README is stale.
- `.github/ISSUE_TEMPLATE/` and recently closed issues/PRs — the team's own taxonomy of bug categories beats any generic stack-hint list.
- Logger/error patterns: a quick `rg 'logger\.(error|warn)|throw new|console\.error'` count per directory reveals where pain concentrates.

If something can't be determined without a deep scan, write it as `TODO: verify` rather than guessing. The **why** of a design choice often isn't recoverable from code or history — write `TODO: verify with maintainer` instead of inventing a rationale. Silent fabrication is worse than an explicit gap.

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

**Output location**: `<repo-root>/docs/code-map.md` and `<repo-root>/docs/debug-playbook.md`, where `<repo-root>` is the path resolved in step 1 (`git rev-parse --show-toplevel`, falling back to cwd). **Always write to the repo root, never into a sub-project** — even if a sub-project (e.g. `frontend/`) already has its own `docs/` folder, do NOT put the project-mapper output there; these docs describe the whole repo and belong at the root. The only exception: the repo root itself uses a non-`docs/` convention for its top-level documentation (e.g. `documentation/`, `.docs/`) — then match that folder, but still at the root.

Fill every section with **real findings from the scan** — drop sections that genuinely don't apply rather than padding. Translate the prose into the chosen language (step 2) while keeping section headers, paths, and commands intact.

**Target size**: code-map ≈ 150–250 lines, debug-playbook ≈ 100–200 lines. These are budgets, not minimums — every line costs tokens on every on-demand read, and a bloated doc undermines the whole point of this skill. If you're over budget, compress descriptive prose first; never compress the "Where to look" table or the symptom→diagnosis mappings — those carry the most orienting value per byte.

**"Where to look" (code-map's symptom→area table) is the single highest-leverage section** — it's what lets a future agent skip whole-project grep, which is the user's core pain point. Aim for **15–25 rows** mixing two axes:
- **Business surfaces**: auth, login, payment, search, dashboard, admin, public API, internal API, file upload, notifications, jobs/cron, settings, etc.
- **Error surfaces**: 401/403, 500 on POST, validation failures, timeouts, cache misses, hydration errors, migration drift, etc.

Discover symptoms from the project itself, not your imagination: `rg 'logger\.(error|warn)|throw new|console\.error'`, `catch` blocks, test files named `*should-fail*` / `*error*` / `*throws*`, and the last 100 `fix:` commits in git log.

**Common scenarios in debug-playbook must be evidence-based, not stack-hint copy-paste.** The list below is a *candidate pool*, not a checklist — only include a scenario if you have project-specific evidence it actually applies (a matching grep hit, a relevant `fix:` commit, a test exercising it, a config that enables the feature). A Next.js project that never calls `useEffect` doesn't need a hydration-mismatch entry; listing it anyway dilutes the playbook and makes the agent doubt the rest. Better to ship 5 verified scenarios than 15 speculative ones.

Stack-hint candidate pool (verify before including):
- **Spring Boot / Java**: bean injection failures, `@Transactional` not applying (self-invocation / non-public), MyBatis mapper-XML mismatch, lazy-load `LazyInitializationException`, datasource/connection-pool exhaustion, circular deps, profile/config not loaded.
- **Next.js / React / TS**: server vs client component boundary, hydration mismatch, API route handler errors, env var not exposed to client (`NEXT_PUBLIC_`), build vs runtime errors, stale cache / ISR, type errors hiding runtime bugs.
- **Python**: import path / circular import, virtualenv/dependency mismatch, async event-loop misuse, silent exception swallowing.
- **DB-heavy**: slow query / missing index, connection leak, transaction isolation, migration drift, dialect-specific SQL incompatibility.

Adapt to whatever stack is actually present. Group scenarios by phase (startup failures, request-time failures, background-job failures) if the project has all three — it makes the playbook easier to navigate under stress.

### 4. Wire it into CLAUDE.md as a trigger contract (not a passive note)

The docs are useless if a future agent doesn't actually read them when relevant. A bare file path in CLAUDE.md ("docs are here") doesn't change agent behavior — the agent will still default to grep-the-whole-project, which is exactly the cost this skill exists to eliminate. The pointer needs to tell the agent **when to read** and **what NOT to do until it has**.

If `CLAUDE.md` (or `AGENTS.md`) exists, add this **plain-text** block (do NOT use `@import` — that would auto-inline the docs every session and defeat the on-demand design):

```markdown
## Reference docs (read on demand)

These are on-demand. Read with the Read tool only when the trigger condition matches — they are not auto-loaded.

- **`docs/code-map.md`** — architecture / where things live.
  **Read BEFORE exploring any unfamiliar area.** Do not grep the codebase blindly first; use this map's "Where to look" table to narrow to a subtree, then grep within that subtree only.

- **`docs/debug-playbook.md`** — symptom → area → diagnosis.
  **Read FIRST when investigating any bug, test failure, or unexpected behavior** — before scanning logs or code. Match the user's symptom against this playbook's scenarios before forming hypotheses or running broad searches.
```

If the project has a `/debug` slash command, also hard-code the playbook read into the command itself (step 1 of `/debug`: "Read `docs/debug-playbook.md`") — that's the strongest trigger possible because it fires at command-invocation time, no inference required.

If no CLAUDE.md/AGENTS.md exists, offer to create one containing just the block above, or hand the user the exact snippet to paste into whatever session-loaded instructions file their setup uses.

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
