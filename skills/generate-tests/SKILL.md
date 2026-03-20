---
name: generate-tests
description: Generate Playwright .spec.new.ts test files from product-map + patterns + selectors memory
---

# generate-tests Skill

Generate Playwright E2E test files for pages that lack test coverage.

## Prerequisites

- `.test-memory/product-map.json` must exist and be enriched (run Product Expert first)
- `.test-memory/patterns.md` must exist
- `.test-memory/selectors.json` must exist

## Input

- `target` — which pages to generate tests for:
  - `all-uncovered` — all pages with `testCoverage: "none"`
  - `{page-id}` — specific page (e.g., `team`)
  - `{module}` — all pages in a module (e.g., `bot-*`)

## Execution Steps

### 1. Load Memory

Read all three memory files:
- `product-map.json` → page structure, components, i18n keys, keySelectors
- `patterns.md` → test patterns to follow, anti-patterns to avoid
- `selectors.json` → existing verified selectors (reuse where possible)

### 2. Select Target Pages

Based on `target`, filter pages from product-map.json where `testCoverage` is `"none"` or `"partial"`.

### 3. Determine Test Level per Page

Apply the spec's level assignment rules:

| Condition | Test Level | Description |
|-----------|-----------|-------------|
| Any page | @L1 | Reachability — page loads, key elements visible |
| Page has forms | @L3 | Validation — form submit, error messages |
| Page has CRUD | @L2 | Operations — create, read, update, delete |
| Page has table/list | @L2 | Data display — table renders, sort, filter |
| Multi-step workflow | @L4 | Workflow — end-to-end flow |
| Edge cases | @L5 | Edge — boundary values, error states |

### 4. Generate Test File

For each target page, generate a `.spec.new.ts` file:

**File naming**: `tests/{page-id}.spec.new.ts`

**Template structure**:

```typescript
import { test, expect } from '@playwright/test';
import { login, navigateToBot } from './helpers/auth';

test.describe('{PageTitle} @{tag}', () => {
  test.beforeEach(async ({ page }) => {
    await login(page);
    // Navigate to the page under test
    {navigationCode}
  });

  // @L1 — Reachability test (always generated)
  test('{pageTitle}ページ表示 @L1 @{priority}', async ({ page }) => {
    // Verify page loaded
    await expect(page).toHaveURL(/{urlPattern}/);

    // Verify key elements are visible
    {keyElementAssertions}
  });

  // @L2 — CRUD test (if page has CRUD)
  // @L3 — Form validation test (if page has forms)
  // etc.
});
```

**Selector strategy** (in priority order):
1. Reuse verified selector from `selectors.json` (confidence ≥ 0.9)
2. Use keySelector from `product-map.json` (confidence ≥ 0.7)
3. Generate new selector following `patterns.md` rules:
   - `getByRole()` + Japanese label from i18n (preferred)
   - `getByPlaceholder()` for form inputs
   - `getByText({ exact: true })` for text elements
   - `evaluate()` for inaccessible elements (last resort)

**Must follow these patterns from patterns.md:**
- `search-before-click` — use search box before clicking list items
- `networkidle-for-navigation` — `waitForLoadState('networkidle')` after page load
- `timestamp-test-data` — `E2E_{Module}_${Date.now()}` for test data names
- `mantine-tab-full-text` — use full tab label text
- `placeholder-over-position` — use placeholder, not positional index
- `no-space-in-japanese-particles` — no space before を、が、は

**Must avoid these anti-patterns from patterns.md:**
- `waitForTimeout-overuse` — use explicit waits instead
- `catch-false-pattern` — use `expect().toBeVisible()` for required elements

### 5. Add Generated Selectors to selectors.json

For each new selector used in generated tests:
- Add to `selectors.json` with confidence 0.7 (unverified)
- Set `lastVerified` to empty string (never verified)
- Note `"source": "generated"` in metadata

### 6. Determine Priority

Assign test priority based on page importance:
- **@P0**: Core user journey pages (bot list, chat, settings, datasets)
- **@P1**: Supporting features (team, account, API keys, dictionary)
- **@P2**: Administrative/secondary features (admin console, org collaboration)

### 7. Output

Save generated files as `tests/{page-id}.spec.new.ts` (NOT `.spec.ts`).

Report:
```
📝 Tests Generated
==================
✅ team.spec.new.ts — 3 tests (@L1 reachability, @L2 member list, @L3 invite form)
✅ account.spec.new.ts — 2 tests (@L1 reachability, @L3 profile edit)
✅ bot-tools.spec.new.ts — 2 tests (@L1 reachability, @L2 tool list)

⚠️ Low-confidence selectors (need manual verification):
  - team: getByRole('button', { name: '招待' }) — confidence 0.7
  - account: getByPlaceholder('メールアドレス') — confidence 0.7

Next: Run /test-run on generated tests to verify, then rename .spec.new.ts → .spec.ts
```
