---
description: Generate E2E tests for uncovered pages with configurable depth (display/interaction/crud)
argument-hint: [--depth display|interaction|crud] [--page page-id] [--refresh]
---

# /test-gen — Generate Tests

Generate Playwright E2E test files from the product map and learned patterns.

## Usage

- `/test-gen` — Generate display tests for all uncovered pages
- `/test-gen --depth interaction` — Generate interaction tests for P0/P1 pages
- `/test-gen --depth crud` — Generate CRUD tests for P0 pages
- `/test-gen --page bot-settings` — Generate for a specific page (all depths)
- `/test-gen --depth crud --page bot-tag` — Generate CRUD tests for a specific page
- `/test-gen --refresh` — Re-scan codebase first, then generate

## Depth Levels

| Depth | What it generates | Default scope |
|-------|------------------|---------------|
| `display` (default) | Page load + key elements visible | All uncovered pages |
| `interaction` | Tab switch, search, sort, modal, toggle | P0 + P1 pages |
| `crud` | Create → verify → edit → delete flows | P0 pages only |

## Behavior

1. **Check product-map freshness**:
   - If product-map.json has fewer than 15 pages → suggest running Product Expert first
   - If `--refresh` flag → invoke Product Expert agent to re-scan codebase

2. **Invoke Test Generator agent** with the specified target and depth

3. **Show generated files** and suggest verification:
   - "Generated 3 test files as `.spec.new.ts`"
   - "Run `/test-run` to verify"
   - "When satisfied, rename `.spec.new.ts` → `.spec.ts`"

## Prerequisites

- Run `/test-init` first if `.test-memory/` doesn't exist
- Product Expert should have scanned the codebase (product-map.json is enriched)
