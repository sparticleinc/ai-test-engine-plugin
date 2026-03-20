---
description: Generate E2E tests for uncovered pages using product knowledge and learned patterns
argument-hint: [page-id | module | --all]
---

# /test-gen — Generate Tests

Generate Playwright E2E test files from the product map and learned patterns.

## Usage

- `/test-gen` — Generate tests for all uncovered pages
- `/test-gen team` — Generate tests for the team page
- `/test-gen bot-*` — Generate tests for all bot sub-pages
- `/test-gen --refresh` — Re-scan codebase first, then generate

## Behavior

1. **Check product-map freshness**:
   - If product-map.json has fewer than 15 pages → suggest running Product Expert first
   - If `--refresh` flag → invoke Product Expert agent to re-scan codebase

2. **Invoke Test Generator agent** with the specified target

3. **Show generated files** and suggest verification:
   - "Generated 3 test files as `.spec.new.ts`"
   - "Run `/test-run team.spec.new.ts` to verify"
   - "When satisfied, rename: `mv tests/team.spec.new.ts tests/team.spec.ts`"

## Prerequisites

- Run `/test-init` first if `.test-memory/` doesn't exist
- Product Expert should have scanned the codebase (product-map.json is enriched)
