# Debug Playbook

> Read this when debugging. Pairs with `code-map.md`: this says *what symptom → which area*, the code-map says *where that area is*.

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
[Per scenario — tailored to the detected stack:]

### [Symptom, e.g. "500 on an API route"]
- **Likely area**: [point into code-map, e.g. "service layer, see `path/`"]
- **Diagnose**: [concrete steps — what to grep, what log line to find, what to check first]
- **Common causes / fixes**: [the usual culprits for this stack]

[Repeat for the top 5–10 symptoms realistic for this project.]

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
