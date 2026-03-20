---
description: Show test system health — memory stats, recent results, coverage
argument-hint:
---

# /test-status — System Health Dashboard

Display the current state of the AI Test Engine memory and test health.

## Behavior

1. **Check prerequisites**:
   - Is `.test-memory/` present? If not: "Run /test-init first"
   - Is `.test-config.json` present? If not: warn

2. **Read and display memory stats**:

### Selector Memory
- Read `.test-memory/selectors.json`
- Count: total elements, elements with confidence > 0.9, elements with confidence < 0.7
- Show any elements with `lastVerified` > 30 days as "stale"

### Pattern Memory
- Read `.test-memory/patterns.md`
- Count: verified patterns, known anti-patterns

### Product Map
- Read `.test-memory/product-map.json`
- Count: total pages, pages with testCoverage "full" / "partial" / "none"
- Count: total flows, flows with testCoverage

### Test History
- Read `.test-memory/test-history.json`
- Show: total runs, last run date, overall pass rate
- Show: flaky tests (if any, >20% fail rate in last 10 runs)
- Show: recent heal history (last 5 entries)

3. **Output format**:

```
🔬 AI Test Engine Status
========================
📊 Memory: 35 selectors (32 healthy, 3 stale) | 9 patterns | 12 pages mapped
📈 History: 15 runs | Last: 2026-03-20 | Pass rate: 97.2%
🩺 Health: 1 flaky test | 0 pending heals
📋 Coverage: 10/12 pages (83%) | 2/2 flows (100%)
```
