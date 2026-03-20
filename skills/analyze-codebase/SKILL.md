---
name: analyze-codebase
description: Scan frontend codebase to extract routes, components, i18n keys → enrich product-map.json
---

# analyze-codebase Skill

Scan the frontend React codebase and produce a structured product map.

## Prerequisites

- `.test-config.json` must exist with a non-empty `frontendPath`
- The frontend codebase must be accessible at that path

## Execution Steps

### 1. Read Configuration

Load `.test-config.json` to get:
- `frontendPath` — root of the frontend app (e.g., `/path/to/apps/admin`)
- `i18nPath` — absolute path to Japanese translation JSON file
- `routerConfigPath` — relative path to routes file (relative to `frontendPath`)

### 2. Parse Routes

Read the router config file (e.g., `src/routes/index.tsx`):

- Extract all `<Route path="..." element={<Component />} />` definitions
- Track which route modules exist (e.g., BotRoutes, MainRoutes, DatasetsRoutes)
- For nested routes (e.g., `/bot/:id/*`), combine parent + child paths
- Record the component name for each route (e.g., `LazyChatPage` → `Bot/Chat`)
- Resolve lazy imports to find the actual page component file path

**Output per route:**
```json
{
  "id": "bot-chat",
  "path": "/bot/:id/chat",
  "componentPath": "src/pages/Bot/Chat/index.tsx",
  "componentName": "ChatPage"
}
```

### 3. Scan Page Components

For each page component file:

1. Read the file and its imports
2. Identify key interactive elements:
   - Forms (look for `useForm`, `<form>`, `<TextInput>`, `<Select>`, `<Textarea>`)
   - Buttons (look for `<Button>`, `<ActionIcon>`, `onClick` handlers)
   - Dialogs/Modals (look for `useDisclosure`, `<Modal>`, `modals.open`)
   - Tables (look for `<Table>`, `<DataTable>`, column definitions)
   - Tabs (look for `<Tabs>`, `<Tabs.Tab>`)
3. Extract component names referenced in the page
4. Identify CRUD operations (create/update/delete patterns)
5. Look for `useTranslation()` / `t('key')` calls → collect i18n keys

**Output per page:**
```json
{
  "components": ["ChatInput", "MessageList", "TypingIndicator"],
  "interactiveElements": ["form:messageInput", "button:send", "button:share"],
  "i18nKeys": ["chat.placeholder", "chat.send", "chat.share"],
  "hasCRUD": false,
  "hasForm": true,
  "hasTabs": false,
  "hasTable": false,
  "hasModal": false
}
```

### 4. Parse i18n File

Read the Japanese translation JSON file:

1. Build a flat key→value map (e.g., `"bot.settings.general"` → `"一般設定"`)
2. For each page's collected i18n keys, resolve to Japanese text
3. Store resolved text in `keySelectors` for selector generation

### 5. Determine Test Coverage

Compare discovered pages against existing test files in `tests/`:

- For each route, check if a `*.spec.ts` file tests that page
- Assign coverage: `"full"` (dedicated test file), `"partial"` (tested as part of flow), `"none"` (no tests)

### 6. Update product-map.json

Merge discoveries into existing `.test-memory/product-map.json`:

- Add new pages (don't remove existing ones — they may have manual enrichments)
- Update existing pages with new component/i18n data
- Set `testCoverage` based on analysis
- Preserve existing `keySelectors` entries
- Update `lastUpdated` timestamp
- Add any new multi-step flows discovered (e.g., account settings flow)

### 7. Update changelog.md

Append entry: `- YYYY-MM-DD: analyze: Scanned frontend codebase, discovered N new pages, M i18n keys`

### 8. Report

Output a summary:
```
🔍 Codebase Analysis Complete
==============================
📄 Routes discovered: 45
📦 Pages scanned: 28
🆕 New pages added to map: 16
🌐 i18n keys collected: 234
📊 Coverage: 12 tested / 28 total (43%)

Uncovered pages (candidates for /test-gen):
  - /team (Team management)
  - /account (Account settings)
  - /bot/:id/tools (Bot tools config)
  - ...
```
