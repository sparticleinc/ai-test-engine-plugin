---
name: run-tests
description: Execute Playwright tests and parse results. Updates test-history.json after each run.
---

# run-tests Skill

Execute Playwright tests for the current project and record results in memory.

## Prerequisites

- `.test-memory/` must exist (run `/test-init` first)
- `.test-config.json` must exist with `testCommand` field
- Playwright must be installed (`npx playwright --version`)

## Execution Steps

1. **Read config**: Load `.test-config.json` to get `testCommand`

2. **Build command**: Based on arguments:
   - No args: `{testCommand}` (run all)
   - `@tag`: `{testCommand} --grep "@tag"` (filter by tag)
   - `filename`: `{testCommand} tests/{filename}` (specific file)
   - `--headed`: append `--headed` flag

3. **Run tests**: Execute the command via Bash tool
   - Use `--reporter=json` to get machine-readable output
   - Also use `--reporter=list` for human-readable console output
   - Capture both stdout and exit code

4. **Parse results**: From JSON reporter output, extract:
   - Total tests, passed, failed, flaky, skipped
   - Per-test: name, status, duration, error message (if failed)
   - Failed test artifacts: screenshot path, trace path, error-context.md path

5. **Update test-history.json**:
   - Append new run to `runs[]` array
   - If `runs[]` exceeds 30 entries, summarize oldest into `summary` and prune
   - Compute per-test pass rates in `summary.perTest`

6. **Report results**:
   - Summary line: "✅ 33/33 passed (20.4s)" or "❌ 2/33 failed"
   - For failures: show test name, error message, artifact paths
   - If flaky tests detected (>20% fail rate in last 10 runs): warn

7. **Return exit code**: 0 if all passed, 1 if any failed
