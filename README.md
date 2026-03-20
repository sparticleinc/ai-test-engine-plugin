# AI Test Engine — Claude Code Plugin

Intelligent self-healing E2E test system with persistent memory. Understands your product, generates Playwright tests, automatically repairs broken tests, and learns from every debug session.

## Installation

```bash
claude plugin add sparticleinc/ai-test-engine-plugin
```

## Commands

| Command | Description |
|---------|-------------|
| `/test-init` | Initialize `.test-memory/` in your project |
| `/test-run` | Run tests (all, by tag, by file) |
| `/test-run @smoke` | Run smoke tests only |
| `/test-status` | Show memory stats and test health |
| `/test-gen` | Generate tests from codebase (Phase 1) |
| `/test-heal` | Auto-fix failing tests (Phase 2) |
| `/test-review` | Quality review + recommendations (Phase 3) |

## How It Works

The plugin maintains a `.test-memory/` directory in your project with three layers of knowledge:

1. **Selectors** (`selectors.json`) — Best Playwright selector for each UI element, with fallbacks and failure history
2. **Patterns** (`patterns.md`) — Reusable test patterns and anti-patterns learned from debugging
3. **Product Map** (`product-map.json`) — Page structure, routes, components, and test coverage

Every test run, heal, and review enriches this memory. The more you use it, the smarter it gets.

## License

MIT
