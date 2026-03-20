---
description: Manually trigger selector repair on failing tests using the latest test report
argument-hint: [test-file.spec.ts | --all]
---

# /test-heal — Repair Broken Selectors

Manually invoke the self-healing system on tests that have failed due to broken selectors.

## Usage

- `/test-heal` — Repair all SELECTOR_BROKEN failures from the latest test run
- `/test-heal bot-tag.spec.new.ts` — Repair selectors in a specific test file
- `/test-heal --all` — Attempt repair on all failures (including non-selector issues)

## Behavior

1. **Load latest test results**: Read `test-results/.last-run.json` or parse `playwright-report/` output
2. **Filter failures**: Extract tests with SELECTOR_BROKEN classification (unless `--all`)
3. **Invoke heal-selectors skill** for each broken selector
4. **Report results**:
   - Fixed: list of repaired selectors with old → new
   - Needs-human: list of selectors that could not be auto-repaired
5. **Suggest re-run**: "Run `/test-run` to verify repairs"

## Prerequisites

- Tests must have been run at least once (test results must exist)
- `.test-memory/` must be initialized
