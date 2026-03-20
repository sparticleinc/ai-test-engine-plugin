# Test Depth Definitions & Generation Rules

## Terminology

Two separate concepts exist in the test engine:

### Difficulty Tags (@L1-@L5) — Phase 1

Used as Playwright `@tag` annotations. Unchanged from Phase 1.

| Tag | Meaning | Example |
|-----|---------|---------|
| `@L1` | Reachability — page loads, key elements visible | Page display test |
| `@L2` | CRUD operations — create, read, update, delete | Bot create/delete test |
| `@L3` | Form validation — submit, error messages | Name empty error test |
| `@L4` | Workflow — multi-step end-to-end flow | Dataset add source flow |
| `@L5` | Edge cases — boundary values, error states | Name 100-char limit test |

### Test Depth (display/interaction/crud) — Phase 2

Describes coverage depth for the `--depth` parameter of `/test-gen`.

| Depth | What it Tests | Maps to Tags |
|-------|--------------|-------------|
| `display` | Page reachable, key elements visible, empty states | `@L1` |
| `interaction` | Tab switch, search/filter, sort, pagination, modal open/close | `@L1` or `@L2` |
| `crud` | Full create → verify → edit → delete cycle with cleanup | `@L2` + `@L3` |

## Component-to-Test Mapping

When generating `interaction` depth tests, inspect `product-map.json` → `components`:

| Component Found | Test to Generate | Tag |
|----------------|-----------------|-----|
| `Tabs` | Click each tab, verify active state and content change | `@L1` |
| `TextInput` / `SearchInput` | Type query, verify list filters or results update | `@L2` |
| `Table` | Verify column headers visible; if sortable, click header and verify order | `@L2` |
| `Select` / `Menu` | Open dropdown, select option, verify selection applied | `@L2` |
| `Pagination` | Click next/prev, verify page content changes | `@L2` |
| `Modal` | Open modal (click trigger), verify modal visible, close modal | `@L1` |
| `Switch` / `Checkbox` | Toggle state, verify visual change | `@L2` |

## Priority-Based Scope

When `--depth` is used without `--page`:

| Depth | Default Scope |
|-------|--------------|
| `display` | All uncovered pages |
| `interaction` | P0 + P1 pages |
| `crud` | P0 pages only |

When `--page` is specified, depth applies regardless of page priority.

## Page Priority Assignment

Priority is stored in `product-map.json` → `pages[].priority`:

| Priority | Criteria |
|----------|----------|
| `P0` | Main navigation + pages with defined `flows` |
| `P1` | Pages with forms, tables, or CRUD capability |
| `P2` | Display-only, low-frequency, or stub pages |
