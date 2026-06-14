# Debug Playbook

> **Read this FIRST when investigating any bug, test failure, or unexpected behavior — before scanning logs or grep'ing code.** This playbook tells you **how** to diagnose a symptom; once narrowed to an area, jump to `code-map.md` for the exact path. Skipping this and grep'ing blindly is exactly the token-burning behavior this doc exists to prevent.

## Run / build / test
- **Run**: [command]
- **Build**: [command]
- **Test**: [command, how to run a single test]
- **Lint / typecheck**: [command]

## Observe (logs & signals)
- **Where logs go**: [path / stdout / service]
- **Enable verbose/debug logging**: [how]
- **Health / metrics endpoints**: [if any]

## Common scenarios

**Evidence-based only.** Each scenario below must have project-specific evidence — a grep hit, a `fix:` commit, a test exercising it, or a config that enables the feature. Do not include generic stack-hints that don't apply to this codebase; 5 verified scenarios beat 15 speculative ones.

If the project has all three phases, group scenarios accordingly — it makes the playbook easier to navigate under stress:
- **Startup failures** (port in use, env missing, DI graph broken, schema mismatch on boot)
- **Request-time failures** (5xx, 4xx, slow responses, broken validation)
- **Background-job failures** (cron not firing, job stuck, retry storm, queue backed up)

### [Symptom, e.g. "500 on POST /api/orders"]
- **Likely area**: [point into code-map, e.g. "service layer, see `src/services/order/`"]
- **Diagnose**: [concrete steps — what to grep, what log line to find, what to check first]
- **Common causes / fixes**: [the actual culprits seen in this project, not generic ones]

[Repeat for 5–10 symptoms backed by real evidence in this project.]

## Diagnostic toolbox
- **Useful greps**: [e.g. `rg "TODO|FIXME"`, error-code patterns, specific patterns for this stack]
- **DB queries**: [common inspection queries if DB-heavy]
- **Quick checks**: [env vars, config flags, connection tests]

## Investigation workflow
1. Reproduce — [how, minimal path]
2. Locate — narrow to an area via this playbook + code-map's "Where to look"
3. Confirm — grep/LSP to the exact symbol; read only that subtree
4. Fix & verify — [re-run the relevant test/command above]

## Gotchas & traps
[Non-obvious pitfalls specific to this codebase: footguns, stale-cache surprises, env-specific behavior, known tech debt that bites.]
