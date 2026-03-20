---
name: heal-selectors
description: Diagnose and repair broken selectors in Playwright test files. Attempts R1 (text fix) → R2 (alternative swap) → R3 (relocate) → R4 (needs-human).
---

# heal-selectors Skill

Repair broken selectors in Playwright test files using a tiered strategy.

## Prerequisites

- `.test-memory/selectors.json` must exist
- `.test-config.json` must exist (for `i18nPath` and `baseUrl`)
- Failed test information (test file path, error message, failing selector)

## Input

For each broken selector, you receive:
- `testFile` — path to the `.spec.ts` or `.spec.new.ts` file
- `errorMessage` — Playwright error output
- `failingSelector` — the selector string that failed (extracted from error)

## Execution Steps

### 1. Classify Failure

Extract the failing selector from the error message. Common patterns:
- `"waiting for locator('...')"` → extract the locator string
- `"locator resolved to N elements"` → strict mode violation, selector too broad
- `"locator.click: Target closed"` → element disappeared

### 2. Look Up Selector in Memory

Search `selectors.json` for the failing selector:
- Check all entries' `selector` and `alternatives` fields
- If found → you have the `elementId` and alternative selectors
- If not found → create an `elementId` using `{page-id}.{description}` format

### 3. Attempt R1 — Text Fix (i18n Change)

**When**: The failing selector contains a text string (e.g., `getByText('タグ管理')`)

1. Read the i18n file from `.test-config.json` → `i18nPath`
   - If `i18nPath` is missing or file doesn't exist → skip to R2
2. Search the i18n JSON for the old text value
3. If the key still exists but the value changed → the text was updated
4. Replace the old text with the new text in the selector
5. Update the test file with the new selector
6. Set confidence to 0.8

**If R1 succeeds → stop. Otherwise continue to R2.**

### 4. Attempt R2 — Alternative Swap

**When**: The element has alternatives in `selectors.json`

1. Read the `alternatives` array for this element
2. For each alternative, check if it would work:
   - Read the test file to understand the page context
   - Determine if the alternative selector pattern is still valid
3. If a viable alternative exists:
   - Swap: move current `selector` to `failHistory`, set alternative as new `selector`
   - Update the test file
   - Set confidence to 0.7

**If R2 succeeds → stop. Otherwise continue to R3.**

### 5. Attempt R3 — Relocate (Live Page)

**When**: No alternatives available or all failed

1. Extract the page URL from the test file's `page.goto()` call
2. If URL has parameters (e.g., `/bot/:id/settings`):
   - Look for a known entity ID in `selectors.json` or test fixtures
   - If no known entity → skip to R4
3. Navigate to the page using Playwright:
   - Use the same auth flow as tests (`auth.setup.ts` → stored auth state)
   - Run: `npx playwright test --project=chromium --headed` with a minimal test script
4. On the live page, identify the element by context:
   - Read surrounding selectors in the test to understand what element is expected
   - Use ARIA role, placeholder, text-content, or label to propose a new selector
5. Verify the proposed selector matches exactly 1 element
6. If verified:
   - Update the test file with the new selector
   - Add the new selector to `selectors.json` with confidence 0.7
   - Record old selector in `failHistory`

**If R3 succeeds → stop. Otherwise continue to R4.**

### 6. R4 — Needs Human

1. Do NOT modify the test file
2. Add failure to `selectors.json` → `failHistory` with today's date
3. Set confidence to 0.3
4. Append to `heal-history.json` with `status: "needs-human"`
5. Return a clear report of what was attempted and why it failed

## Memory Updates

After each repair (success or failure):

1. **selectors.json**: Update `selector`, `alternatives`, `failHistory`, `confidence`, `lastVerified`
2. **heal-history.json**: Append repair record:
   ```json
   {
     "date": "YYYY-MM-DD",
     "testFile": "tests/example.spec.ts",
     "elementId": "page.element-name",
     "failType": "SELECTOR_BROKEN",
     "repairLevel": "R1",
     "oldSelector": "getByText('old')",
     "newSelector": "getByText('new')",
     "status": "fixed"
   }
   ```
3. **changelog.md**: One-line entry: `- YYYY-MM-DD: heal: R1 fix for {elementId} in {testFile}`
4. **patterns.md** (optional): If the repair reveals a new generalizable pattern (e.g., "Mantine v8 changed Tab rendering"), append it to the Verified Patterns section

## Important Rules

- **Only modify selector strings** — never change test logic, assertions, or flow
- **Max 1 repair attempt per level** — don't retry the same level
- **Verify before accepting** — a new selector must be plausible (correct ARIA role, matching text)
- **Always record what happened** — even failed attempts go to heal-history.json
