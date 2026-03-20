# Mantine v8 — Playwright Selector Guide

## Overview

Mantine v8 (8.x) is a React component library. This guide documents reliable Playwright selector strategies for common Mantine components. These strategies were verified against the GBase Admin staging environment.

**Key difference from v7**: Mantine v8 uses CSS modules by default, making class-based selectors unreliable. Always prefer ARIA-based selectors.

## Component Selector Strategies

### Tabs (`@mantine/core` Tabs)
```typescript
// ✅ CORRECT — use full tab label text
page.getByRole('tab', { name: '一般設定' })

// ❌ WRONG — partial text doesn't match
page.getByText('一般', { exact: true })
```
- Mantine Tab renders the full label text (e.g., "一般設定" not "一般")
- Tab panels: `getByRole('tabpanel')`

### TextInput
```typescript
// ✅ Best — by placeholder (unique per field)
page.getByPlaceholder('チャットボットの名前を入力してください')

// ✅ Good — by label
page.getByLabel('名前')

// ❌ WRONG — positional (may hit readonly fields)
page.getByRole('textbox').first()
```

### Button
```typescript
// ✅ Standard button
page.getByRole('button', { name: 'OK' })

// ✅ Duplicate buttons — use .last() for dialog confirm
page.getByRole('button', { name: '削除' }).last()
```

### Modal / Dialog
```typescript
// ✅ By dialog title
page.getByRole('dialog', { name: 'AIアシスタントを作成しています' })
```

### Menu (ActionIcon with dropdown)
```typescript
// ✅ Menu items have role="menuitem"
page.getByRole('menuitem', { name: '削除' })

// ⚠️ Menu trigger buttons often have NO accessible name
// Use evaluate() for DOM traversal from nearby text
```

### Select / Dropdown
```typescript
// Open the select
page.getByRole('combobox', { name: 'ラベル' }).click()
// Then select option
page.getByRole('option', { name: 'オプション値' }).click()
```

### Card (with action buttons)
```typescript
// ⚠️ Card action buttons (three-dot menu) often lack accessible names
// Reliable approach: find card by text, then traverse DOM
const card = page.getByText('BDD_Test_Agent', { exact: true });
await card.evaluate(el => {
  let c = el.parentElement;
  while (c) {
    const btn = c.querySelector('button');
    if (btn) { btn.click(); return; }
    c = c.parentElement;
  }
});
```

### Navbar / Navigation
```typescript
// ✅ Links with text
page.getByRole('link', { name: 'マイボット' })
page.getByRole('link', { name: 'ナレッジベース' })
```

### Table
```typescript
// ✅ Table headers
page.getByRole('columnheader', { name: 'ヘッダー名' })
// ✅ Table rows — locate by cell content
page.getByRole('row').filter({ hasText: '行のテキスト' })
```

### Switch / Checkbox
```typescript
// ✅ By label
page.getByRole('switch', { name: 'ラベル' })
page.getByRole('checkbox', { name: 'ラベル' })
```

## General Rules

1. **Always prefer ARIA selectors** — Mantine v8 CSS classes are generated and unstable
2. **Japanese text must be exact** — no spaces before particles (を、が、は)
3. **Use `{ exact: true }` with getByText** — avoid substring matching
4. **Wait for networkidle after navigation** — Mantine pages load lazily
5. **Cards without accessible button names** — use `evaluate()` for DOM traversal
