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

1. **Scans efficiently** — reads manifests (`package.json`, `pom.xml`, etc.) and the directory tree to identify the stack and entry points, then uses grep/glob to enumerate routes/controllers/services instead of reading every file.
2. **Generates `code-map.md`** — overview, architecture diagram, directory map, entry/boot flow, core flows, key modules, integrations, conventions, and a "where to look" symptom→area index.
3. **Generates `debug-playbook.md`** — run/build/test commands, where logs go, stack-tailored common failure scenarios, a diagnostic toolbox, an investigation workflow, and project-specific gotchas.
4. **Wires a pointer into `CLAUDE.md`** — plain-text reference (not `@import`), so the docs stay on-demand.

## Language

Output language is **Chinese or English**, decided by:

1. **What you ask for** — e.g. "generate the English version" / "用中文生成". This always wins.
2. **The repo's existing docs** — matches `CLAUDE.md`/`AGENTS.md`/`README` language if you don't specify.
3. **Fallback** — the language of the conversation.

## Install

Install the `project-mapper.skill` file in Claude (Claude Code / Claude.ai skills). Once installed it triggers automatically.

## Usage

Just ask, in any project:

- "Generate a code map and debug playbook for this project"
- "用中文生成 code map 和 debug playbook"
- "Map this repo so the AI understands it"

If the repo mixes languages and you didn't specify, the skill asks once which language to use.

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

## Maintenance

Regenerate when the **architecture** changes (new module, new integration, restructure) — not on every commit. Module-level docs are stable by design.
