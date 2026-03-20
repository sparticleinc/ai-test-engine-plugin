---
description: Initialize .test-memory/ directory and .test-config.json in the current project
argument-hint: [--force]
---

# /test-init — Initialize Test Memory

Initialize the AI Test Engine memory for the current project.

## Steps

1. Check if `.test-memory/` already exists in the current working directory
   - If exists and `--force` not passed: warn user and stop
   - If exists and `--force` passed: back up existing to `.test-memory.bak/`

2. Create `.test-memory/` directory with template files:
   - Copy `selectors.template.json` → `.test-memory/selectors.json`
   - Copy `patterns.template.md` → `.test-memory/patterns.md`
   - Copy `product-map.template.json` → `.test-memory/product-map.json`
   - Copy `test-history.template.json` → `.test-memory/test-history.json`
   - Copy `heal-history.template.json` → `.test-memory/heal-history.json`
   - Create `.test-memory/changelog.md` with init entry

3. Create `.test-config.json` by asking the user:
   - What is the test command? (default: `npx playwright test`)
   - What is the base URL? (try to detect from playwright.config.ts or .env)
   - Where is the frontend codebase? (optional, for Phase 1 test generation)

4. Add `.test-memory/` to `.gitignore` if not already present
   - Memory files should be committed (they are project knowledge)
   - But add `.test-memory/.pending-scan` to .gitignore (temp flag file)

5. Report:
   - "✅ Test memory initialized at .test-memory/"
   - "📋 Config saved to .test-config.json"
   - "Run /test-run to execute tests, /test-status to check memory health"
