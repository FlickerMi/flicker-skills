# Project Mapper

**[English](./README.md) | [中文](./README.zh-CN.md)**

A Claude skill that scans a codebase and generates two lean, **on-demand** reference docs so an AI coding agent can orient instantly instead of scanning the whole project on every task.

## Why

`CLAUDE.md` is auto-loaded into context every session — every line there is a fixed cost paid on every turn. The two docs this skill produces are **loaded on demand** (the agent reads them only when needed), so they carry no idle cost.

| Doc | Answers | Used for |
|-----|---------|----------|
| `docs/code-map.md` | "where does X live / how is this wired" | **Orienting** — point the agent at the right subtree before it searches |
| `docs/debug-playbook.md` | "this symptom → look here → check this" | **Debugging** — loaded only when chasing a bug |

Division of labor: **code-map orients (rough direction), live grep/LSP locates (exact line).** Docs stay at the module/architecture level — never function level, which drifts and misleads.

## What it does

1. **Scans efficiently** — reads manifests (`package.json`, `pom.xml`, etc.), the directory tree, and key entry files; uses grep/glob to enumerate routes/controllers/services instead of reading every file. For unfamiliar projects, also pulls signal from `git log --oneline -100`, `CHANGELOG.md`, and `.github/ISSUE_TEMPLATE/`.
2. **Generates `docs/code-map.md`** (target 150–250 lines) — directory map (tagged `active` / `legacy`), entry/boot flow, core flows, key modules, integrations, conventions, and most importantly a **symptom→area "Where to look" table** (target 15–25 rows mixing business surfaces × error surfaces).
3. **Generates `docs/debug-playbook.md`** (target 100–200 lines) — run/build/test commands, where logs go, **evidence-backed** common failure scenarios (grouped by startup / request-time / background-job phase), diagnostic toolbox, project-specific gotchas.
4. **Writes a trigger contract into `CLAUDE.md`** (plain text, not `@import`) — tells future agents **when they must read these docs**; otherwise they'll still grep the whole project by default.

## Monorepo

If the repo root contains multiple sub-projects (`frontend/` + `backend/`, `apps/*` + `packages/*`, etc.), the skill generates **a single pair of files at the git repo root** covering all sub-projects — never inside a sub-project directory, even if that sub-project already has its own `docs/` folder (writing there would hide the docs from agents working at the root).

## Install

Install the `project-mapper.skill` file in Claude (Claude Code / Claude.ai skills). Once installed it triggers automatically.

## Recommended usage

### 1. Generate the docs

In any project root, just ask Claude Code:

- "Generate a code map and debug playbook for this project"
- "用中文生成 code map 和 debug playbook"
- "Map this repo so the AI understands it"

The skill scans → generates the docs → suggests how to wire them into CLAUDE.md. If the repo mixes Chinese and English docs and you didn't specify, it asks once before writing.

### 2. Hard-code the trigger into `/debug` (strongest recommendation)

Generating the docs isn't enough — the agent doesn't always read them on its own. **The most reliable path is to bake "read the playbook" into your `/debug` command itself**: it fires at command-invocation time and removes any reliance on agent self-direction.

Create `.claude/commands/debug.md` (project-level) or `~/.claude/commands/debug.md` (global):

```markdown
---
description: Investigate a bug in this project
---

Follow this order for any bug report — do not skip steps:

1. **First, read `docs/debug-playbook.md`.** Match the user's described symptom against the playbook's "Common scenarios" and check whether it hits a known pattern.
2. If it hits: follow the playbook's diagnose steps; once narrowed to a directory, jump to `docs/code-map.md` for the exact path.
3. If it doesn't hit: look up the closest business or error surface in `docs/code-map.md`'s "Where to look" table, narrow to a subtree, then grep only within that subtree. **Do not grep the whole project first.**
4. Once you've isolated a suspect file, Read it and propose a fix.

Bug report: $ARGUMENTS
```

Then `/debug "401 immediately after login"` forces step 1 (read the playbook) before anything else.

### 3. Write a trigger contract into CLAUDE.md / AGENTS.md

For when the user describes a bug without using `/debug`. Add this block to `CLAUDE.md` or `AGENTS.md` (**do NOT use `@import`** — that would auto-inline both docs every session, defeating the on-demand design):

```markdown
## Reference docs (read on demand)

These are on-demand. Read with the Read tool only when the trigger condition matches — they are not auto-loaded.

- **`docs/code-map.md`** — architecture / where things live.
  **Read BEFORE exploring any unfamiliar area.** Do not grep the codebase blindly first; use the "Where to look" table to narrow to a subtree, then grep within that subtree only.

- **`docs/debug-playbook.md`** — symptom → area → diagnosis.
  **Read FIRST when investigating any bug, test failure, or unexpected behavior** — before scanning logs or code. Match the user's symptom against the playbook's scenarios before forming hypotheses or running broad searches.
```

### 4. When to regenerate

Regenerate when **architecture** changes — not on every commit:

- A module is added / split / merged
- A new integration or database is wired in
- A large refactor or directory restructure
- The repo is split into a monorepo, or the reverse

Module-level docs are stable by design; that's the point. Day-to-day work (bug fixes, field additions, small features) doesn't make them inaccurate.

## Language

Output language is **Chinese or English**, decided by:

1. **What you ask for** — e.g. "generate the English version" / "用中文生成". This always wins.
2. **The repo's existing docs** — matches `CLAUDE.md` / `AGENTS.md` / `README` language if you don't specify.
3. **Fallback** — the language of the conversation.

## Structure

```
project-mapper/
├── SKILL.md                          # workflow (read by the agent)
├── README.md                         # this file (English)
├── README.zh-CN.md                   # 中文说明
└── references/
    ├── code-map-template.md          # structure for docs/code-map.md
    └── debug-playbook-template.md    # structure for docs/debug-playbook.md
```
