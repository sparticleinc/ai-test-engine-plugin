---
description: Run E2E tests with optional filtering by tag, file, or pattern
argument-hint: [@smoke | @P0 | filename.spec.ts | --headed | --all]
---

# /test-run — Execute Tests

Run Playwright E2E tests for the current project.

## Usage

- `/test-run` — Run all tests
- `/test-run @smoke` — Run smoke tests only
- `/test-run @P0` — Run critical path tests
- `/test-run @L1` — Run level 1 (reachability) tests
- `/test-run bot-crud` — Run specific test file
- `/test-run --headed` — Run with browser visible (for debugging)

## Behavior

1. Invoke the `run-tests` skill with the provided arguments
2. Display results in a clear summary format
3. If tests fail, suggest running `/test-heal` to auto-fix
4. Update `.test-memory/test-history.json` with run results

## Prerequisites

Run `/test-init` first if `.test-memory/` doesn't exist.
