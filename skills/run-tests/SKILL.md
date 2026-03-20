---
name: run-tests
description: Execute Playwright tests with auto-healing. Diagnoses failures, repairs broken selectors, re-runs healed tests, and updates memory.
---

# run-tests Skill

Execute Playwright tests with automatic self-healing for broken selectors.

## Prerequisites

- `.test-memory/` must exist (run `/test-init` first)
- `.test-config.json` must exist with `testCommand` field
- Playwright must be installed (`npx playwright --version`)

## Execution Steps

### 1. Read Config

Load `.test-config.json` to get `testCommand`, `baseUrl`, `i18nPath`.

### 2. Build Command

Based on arguments:
- No args: `{testCommand}` (run all)
- `@tag`: `{testCommand} --grep "@tag"` (filter by tag)
- `filename`: `{testCommand} tests/{filename}` (specific file)
- `--headed`: append `--headed` flag

Always add `--reporter=json --reporter=list` for both machine-readable and human-readable output.

### 3. Run Tests (First Pass)

Execute the command via Bash tool. Capture:
- Exit code (0 = all passed, 1 = failures)
- JSON reporter output (structured results)
- List reporter output (console display)

### 4. If All Passed → Update Confidence & Done

For every selector in `selectors.json` that was exercised by a passing test:
- Increment `confidence` by 0.02 (cap at 0.99)
- Update `lastVerified` to today's date

Update `test-history.json`:
- Append new run to `runs[]`
- Prune if over 30 entries

Report: `✅ {passed}/{total} passed ({duration})`

**Stop here if all tests passed.**

### 5. If Failures → Parse & Classify

From JSON reporter output, extract each failure:
- Test name and file path
- Error message
- Error type classification:

| Error Pattern | Classification |
|--------------|---------------|
| `"waiting for locator"` / `"locator resolved to"` / `"strict mode violation"` | `SELECTOR_BROKEN` |
| `"expect(received)"` / `"Expected:"` / `"Received:"` | `ASSERTION_FAILED` |
| `"ERR_CONNECTION"` / `"Timeout exceeded"` / `"net::ERR"` | `NAVIGATION_ERROR` |
| Everything else | `UNKNOWN` |

### 6. Handle Each Failure Type

**SELECTOR_BROKEN:**
→ Invoke the `heal-selectors` skill with the test file, error message, and failing selector.
→ The skill attempts R1 → R2 → R3 → R4 repair cascade.
→ Track which tests were healed.

**NAVIGATION_ERROR:**
→ Retry the specific test once (staging server may be temporarily down).
→ If still fails → record as environment issue in report.

**ASSERTION_FAILED:**
→ Record in report. May indicate a real UI change — not auto-fixable.

**UNKNOWN:**
→ Record in report.

### 7. Re-run Healed Tests (Max 1 Round)

If any selectors were healed in step 6:
1. Collect the list of healed test files
2. Re-run ONLY those tests: `{testCommand} tests/{file1} tests/{file2} --reporter=json --reporter=list`
3. Parse results
4. If healed tests now pass → healing succeeded, update confidence to healed level (0.7 or 0.8)
5. If healed tests still fail → mark as `needs-human`, revert selector change

**Do NOT re-run a second time.** Max 1 healing round per invocation.

### 8. Update Memory

- `test-history.json`: Append run record (including heal attempts)
- `selectors.json`: Updated by heal-selectors skill
- `heal-history.json`: Updated by heal-selectors skill
- `changelog.md`: Append healing events

### 9. Generate Report

If healing activity occurred, write `test-results/heal-report.md`:

```
# Heal Report — {date}

## Summary
- Tests run: {total}
- Passed: {passed} | Failed: {failed}
- Selectors healed: {healed_count}
- Needs human: {needs_human_count}

## Healed Selectors
| Test File | Element | Old Selector | New Selector | Level |
|-----------|---------|-------------|-------------|-------|
| bot-tag.spec.new.ts | bot-tag.title | getByText('タグ管理') | getByText('タグ一覧') | R1 |

## Needs Human
| Test File | Element | Error | Attempts |
|-----------|---------|-------|----------|
| (none) | | | |

## Other Failures
| Test File | Type | Error |
|-----------|------|-------|
| (none) | | |
```

### 10. Report to User

Display summary:
- If all passed (with or without healing): `✅ {passed}/{total} passed ({duration})`
  - If healing occurred: `🔧 {healed} selectors auto-repaired`
- If failures remain: `❌ {failed}/{total} failed`
  - List each failure with classification
  - If needs-human items: `⚠️ {count} selectors need manual attention — see test-results/heal-report.md`

### 11. Return Exit Code

- 0 if all tests passed (including after healing)
- 1 if any tests still failing after healing attempt
