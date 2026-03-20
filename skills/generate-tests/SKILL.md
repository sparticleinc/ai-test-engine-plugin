---
name: generate-tests
description: Generate Playwright .spec.new.ts test files with configurable depth (display/interaction/crud) from product-map + patterns + selectors memory
---

# generate-tests Skill

Generate Playwright E2E test files for pages based on coverage depth.

## Prerequisites

- `.test-memory/product-map.json` must exist and be enriched (run Product Expert first)
- `.test-memory/patterns.md` must exist
- `.test-memory/selectors.json` must exist

## Input

- `target` — which pages to generate tests for:
  - `all-uncovered` — all pages with `testCoverage: "none"`
  - `{page-id}` — specific page (e.g., `team`)
  - `{module}` — all pages in a module (e.g., `bot-*`)

- `depth` — what level of tests to generate (default: `display`):
  - `display` — L1 reachability tests (page loads, key elements visible)
  - `interaction` — L1/L2 interaction tests (tab switching, search, sort, modals)
  - `crud` — L2/L3 CRUD flow tests (create → verify → edit → delete)

- `page` — generate for a specific page regardless of priority

## Execution Steps

### 1. Load Memory

Read all memory files:
- `product-map.json` → page structure, components, i18n keys, keySelectors, priority
- `patterns.md` → test patterns to follow, anti-patterns to avoid
- `selectors.json` → existing verified selectors (reuse where possible)
- `references/test-levels.md` → component-to-test mapping

### 2. Select Target Pages

**If `--page` specified:** Use that page only, any depth applies.

**If `--depth` specified without `--page`:**
- `display` → all pages with `testCoverage: "none"`
- `interaction` → P0 + P1 pages (filter by `priority` field)
- `crud` → P0 pages only

**Default (no flags):** Same as `display` for all uncovered pages.

### 3. Determine Tests per Page (Depth-Aware)

#### Depth: `display` (default)

For each page, generate:
- 1 reachability test tagged `@L1`: page loads, URL correct, key elements visible

#### Depth: `interaction`

Inspect the page's `components` array in product-map.json. For each component type found, generate a test using the mapping in `references/test-levels.md`:

| Component | Test |
|-----------|------|
| `Tabs` | Click each tab, verify content changes (`@L1`) |
| `TextInput`/`SearchInput` | Type query, verify filtering (`@L2`) |
| `Table` | Verify columns visible; test sort if applicable (`@L2`) |
| `Select`/`Menu` | Open, select option, verify (`@L2`) |
| `Pagination` | Navigate pages (`@L2`) |
| `Modal` | Open/close modal (`@L1`) |
| `Switch`/`Checkbox` | Toggle and verify (`@L2`) |

Skip components that don't exist on the page. A page with `Tabs` + `Table` generates 2 interaction tests.

#### Depth: `crud`

1. Check if the page has a `flows` entry in product-map.json or has CRUD-capable components
2. Read an existing CRUD test as template (e.g., `bot-crud.spec.ts`, `dictionary.spec.ts`)
3. Generate a CRUD flow test:
   - Create: fill form, submit, verify creation
   - Read: verify item appears in list
   - Update: edit a field, save, verify change
   - Delete: delete item, confirm dialog, verify removal
4. Include cleanup in `test.afterEach`
5. Use `timestamp-test-data` pattern: `E2E_{Module}_${Date.now()}`
6. Tag with `@L2` (CRUD) and `@L3` (validation)

### 4. Generate Test File

For each target page, generate a `.spec.new.ts` file:

**File naming**: `tests/{page-id}.spec.new.ts` (for display depth) or `tests/{page-id}-interaction.spec.new.ts` / `tests/{page-id}-crud.spec.new.ts` (for deeper depths, if a display-level file already exists)

**Selector strategy** (in priority order):
1. Reuse verified selector from `selectors.json` (confidence ≥ 0.9)
2. Use keySelector from `product-map.json` (confidence ≥ 0.7)
3. Generate new selector following `patterns.md` rules:
   - `getByRole()` + Japanese label from i18n (preferred)
   - `getByPlaceholder()` for form inputs
   - `getByText({ exact: true })` for text elements
   - `evaluate()` for inaccessible elements (last resort)

**Must follow patterns from patterns.md:**
- `search-before-click` — use search box before clicking list items
- `networkidle-for-navigation` — `waitForLoadState('networkidle')` after page load
- `timestamp-test-data` — `E2E_{Module}_${Date.now()}` for test data names
- `mantine-tab-full-text` — use full tab label text
- `placeholder-over-position` — use placeholder, not positional index
- `no-space-in-japanese-particles` — no space before を、が、は

**Must avoid anti-patterns from patterns.md:**
- `waitForTimeout-overuse` — use explicit waits instead
- `catch-false-pattern` — use `expect().toBeVisible()` for required elements

### 5. Add Generated Selectors to selectors.json

For each new selector used in generated tests:
- Add to `selectors.json` with confidence 0.7 (unverified)
- Set `lastVerified` to empty string
- Note `"source": "generated"` in metadata

### 6. Update testCoverage in product-map.json

After generating tests, update each page's `testCoverage`:
- Only display tests → `"partial"`
- Display + interaction → `"partial"`
- Display + interaction + crud → `"full"`

### 7. Output

Save generated files as `tests/{page-id}[-depth].spec.new.ts`.

Report:
```
📝 Tests Generated (depth: interaction)
========================================
✅ bot-settings-interaction.spec.new.ts — 3 tests (tab switch @L1, search @L2, modal @L1)
✅ dataset-list-interaction.spec.new.ts — 2 tests (search @L2, sort @L2)

⚠️ Low-confidence selectors (need verification):
  - bot-settings: getByRole('tab', { name: '外部連携' }) — confidence 0.7

Next: Run /test-run to verify, then rename .spec.new.ts → .spec.ts
```
