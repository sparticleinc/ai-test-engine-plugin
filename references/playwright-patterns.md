# Playwright Best Practices for AI Test Generation

## Selector Priority (most stable → least stable)

1. `getByRole('role', { name: 'accessible name' })` — ARIA roles, most stable
2. `getByLabel('label text')` — form fields with labels
3. `getByPlaceholder('placeholder text')` — input fields
4. `getByText('text', { exact: true })` — visible text (always use exact:true)
5. `getByTestId('test-id')` — data-testid attributes
6. `locator('css selector')` — CSS selectors (avoid)
7. `evaluate()` — DOM traversal (last resort, but reliable for inaccessible elements)

## Wait Strategies (most reliable → least reliable)

1. `expect(locator).toBeVisible({ timeout: N })` — wait for element
2. `page.waitForURL(pattern)` — wait for navigation
3. `page.waitForLoadState('networkidle')` — wait for network quiet
4. `page.waitForTimeout(N)` — fixed delay (avoid except after search/filter)

## Common Gotchas

### Mantine UI
- Tab components: use `getByRole('tab', { name: 'Full Tab Text' })` — text includes full label
- Select/dropdown: use `getByRole('option')` after opening
- Dialog: use `getByRole('dialog', { name: 'dialog title' })`
- Cards with action buttons: buttons may lack accessible names → use `evaluate()` for DOM traversal

### i18n (Japanese)
- No space before particles (を、が、は): `${name}を削除` not `${name} を削除`
- Always verify exact text against i18n JSON files or DOM snapshot
- Some text may be split across elements — use `getByText` on the parent

### List Pages
- Always use search/filter before clicking items in long lists
- After filtering: `waitForTimeout(1000)` for server-side search
- Use `{ exact: true }` to avoid matching substrings (e.g., "ISMS" vs "ISMS-copy(2)")

### Test Data Management
- Name pattern: `E2E_{Module}_{Date.now()}`
- Always include cleanup in test (create → verify → delete)
- Provide a cleanup utility for orphaned test data from failed runs
