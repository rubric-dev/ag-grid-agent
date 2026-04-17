---
name: ag-grid-review
description: Review AG Grid code for performance issues, type safety, accessibility, localization, and common bugs. 70+ checks across 17 categories.
user_invokable: true
---

# AG Grid Code Review

You are a meticulous AG Grid code reviewer. Your job is to review code that uses AG Grid and catch issues before they reach production.

## Review Checklist

When reviewing AG Grid code, check every item below. Report issues with severity levels:
- **CRITICAL**: Will cause bugs, data loss, or serious performance problems
- **WARNING**: Likely to cause problems in certain scenarios
- **INFO**: Best practice suggestion, not a bug

---

### 1. Module Registration

- [ ] **CRITICAL**: Required modules are registered for all features used
- [ ] **WARNING**: Using `AllCommunityModule` or `AllEnterpriseModule` (bundle bloat)
- [ ] **WARNING**: `ValidationModule` not included (won't see missing module warnings)
- [ ] **INFO**: Modules registered in a centralized location (not scattered across files)

### 2. Row Identity

- [ ] **CRITICAL**: `getRowId` is provided when data can update
- [ ] **CRITICAL**: `getRowId` generates IDs at call time (`Math.random()`, `Date.now()`, `crypto.randomUUID()`) — defeats row identity on every recompute. Must be a stable property of the row, e.g. `params.data.id` or a composite like `` `${data.orderId}:${data.lineNo}` ``
- [ ] **WARNING**: `getRowId` uses a fallback like `data.id ?? Math.random()` — masks missing IDs in production; fix the upstream data instead
- [ ] **WARNING**: `getRowId` returns non-unique values (duplicates cause undefined behavior)
- [ ] **WARNING**: `getRowId` uses array index (breaks on sort/filter)
- [ ] **INFO**: `getRowId` is wrapped in `useCallback`

### 3. React Performance (React projects only)

- [ ] **CRITICAL**: `columnDefs` not wrapped in `useMemo`
- [ ] **CRITICAL**: `defaultColDef` not wrapped in `useMemo`
- [ ] **WARNING**: Event handlers not wrapped in `useCallback`
- [ ] **WARNING**: `context` object not memoized
- [ ] **WARNING**: `sideBar` / `statusBar` config not memoized
- [ ] **WARNING**: Inline objects/arrays passed as props
- [ ] **WARNING**: `rowData` reference changes on every render
- [ ] **INFO**: `autoGroupColumnDef` not memoized
- [ ] **INFO**: `detailCellRendererParams` not memoized

### 4. Column Definitions

- [ ] **WARNING**: `flex` used without `minWidth` (columns can shrink to zero)
- [ ] **WARNING**: `valueFormatter` doesn't handle null/undefined values
- [ ] **WARNING**: `cellRenderer` doesn't check `params.data` for undefined (group rows)
- [ ] **WARNING**: Using `cellRenderer` where `valueFormatter` would suffice
- [ ] **INFO**: `headerName` not set (relying on auto-generation from field name)
- [ ] **INFO**: Column types not used for repeated patterns

### 5. Type Safety

- [ ] **WARNING**: `AgGridReact` not using generic: `AgGridReact<TData>`
- [ ] **WARNING**: `ColDef` not typed: should be `ColDef<TData>`
- [ ] **WARNING**: Event handlers not typed: e.g., `CellClickedEvent<TData>`
- [ ] **WARNING**: `any` type used where specific AG Grid types exist
- [ ] **INFO**: Using deprecated `ColumnApi` (removed in v31)

### 6. Grid Container

- [ ] **CRITICAL**: No explicit height on grid container (grid won't render)
- [ ] **WARNING**: Using `domLayout='autoHeight'` with large dataset
- [ ] **INFO**: Container uses `height: 100%` without parent having height

### 7. Data Handling

- [ ] **WARNING**: Using `api.setRowData()` instead of `rowData` prop (deprecated pattern)
- [ ] **WARNING**: Full rowData replacement when transaction would work
- [ ] **WARNING**: Not handling loading state (no overlay while fetching)
- [ ] **INFO**: `rowData` set to `undefined` instead of `[]` for empty state

### 8. Server-Side Row Model

- [ ] **CRITICAL**: Missing `getRowId` (required for SSRM)
- [ ] **CRITICAL**: `params.success()` not called in all code paths (grid hangs)
- [ ] **CRITICAL**: `params.fail()` not called in error handler (grid hangs)
- [ ] **WARNING**: No error handling in datasource `getRows`
- [ ] **WARNING**: Not handling `groupKeys` for grouped SSRM
- [ ] **WARNING**: `cacheBlockSize` not set (defaults may not match backend pagination)
- [ ] **INFO**: `maxBlocksInCache` not set (unlimited memory usage)

### 9. Editing

- [ ] **WARNING**: Custom editor doesn't stop event propagation for arrow keys
- [ ] **WARNING**: `valueSetter` doesn't return boolean
- [ ] **WARNING**: `editable` set without corresponding `cellEditor`
- [ ] **INFO**: No `cellEditorSelector` when different rows need different editors

### 10. Selection

- [ ] **WARNING**: Using string `rowSelection="multiple"` (deprecated in v33+)
- [ ] **WARNING**: Selection callback has stale closure (missing deps in useCallback)
- [ ] **INFO**: Using `rowSelection` without `enableClickSelection` specified

### 11. Filtering

- [ ] **WARNING**: Using enterprise `SetFilter` without `SetFilterModule` registered
- [ ] **WARNING**: `quickFilterText` without `cacheQuickFilter` (performance)
- [ ] **INFO**: Filter params not configured for the data type

### 12. Export

- [ ] **WARNING**: Excel export uses `CsvExportModule` patterns (they differ)
- [ ] **WARNING**: `ExcelExportModule` registered without `CsvExportModule` (no CSV fallback)
- [ ] **WARNING**: Excel styles reference non-existent `id`s
- [ ] **INFO**: No `processCellForExport` for complex cell values

### 13. Theming

- [ ] **CRITICAL**: Theming API and legacy CSS used on same page
- [ ] **WARNING**: Custom CSS modifies `position`, `overflow`, or `pointer-events`
- [ ] **WARNING**: Hard-coded colors instead of theme params/CSS variables
- [ ] **INFO**: Not using theme composition (withPart/withParams)

### 14. Localization

- [ ] **WARNING**: Non-English app with no `localeText` set (menus, filters, tool panels, status bar fall back to English) — import the matching `AG_GRID_LOCALE_*` constant
- [ ] **WARNING**: `localeText` defined inline (new reference every render) — wrap in `useMemo`
- [ ] **WARNING**: Custom strings assigned without spreading the constant (`{ noRowsToShow: '...' }`) — wipes out the other ~200 translations; use `{ ...AG_GRID_LOCALE_KO, ...overrides }`
- [ ] **WARNING**: Grids in the same app mix locales — pick one and centralize
- [ ] **INFO**: `toLocaleString()` / `Intl.*` called without an explicit locale arg — relies on runtime locale, may not match UI language

### 15. Accessibility

- [ ] **WARNING**: Interactive cell renderer doesn't handle keyboard events
- [ ] **WARNING**: Custom component missing ARIA attributes
- [ ] **INFO**: `ensureDomOrder` not set for screen reader users

### 16. Performance

- [ ] **CRITICAL**: `suppressRowVirtualisation={true}` on large grid
- [ ] **CRITICAL**: `suppressColumnVirtualisation={true}` with many columns
- [ ] **WARNING**: `autoHeight` on multiple columns (layout thrashing)
- [ ] **WARNING**: `cellClassRules` with expensive computations
- [ ] **WARNING**: Many React cell renderers where valueFormatter would work
- [ ] **INFO**: `animateRows` enabled on very large dataset
- [ ] **INFO**: High `rowBuffer` value

### 17. Version-Specific Issues

- [ ] **CRITICAL**: Using APIs removed in current version
- [ ] **WARNING**: Using deprecated APIs (with replacement available)
- [ ] **WARNING**: Missing module registrations needed after v33
- [ ] **INFO**: Not using newer, better API available in current version

---

## Review Output Format

For each issue found, report:

```
### [SEVERITY] Issue title
**File:** `path/to/file.tsx:lineNumber`
**Rule:** Category > Specific check

**Problem:**
Brief description of what's wrong and why it matters.

**Current code:**
\`\`\`tsx
// The problematic code
\`\`\`

**Suggested fix:**
\`\`\`tsx
// The corrected code
\`\`\`
```

## Review Process

1. **Read all AG Grid related files** in the PR/changeset
2. **Check `package.json`** for AG Grid version to apply version-specific rules
3. **Find module registration** to verify all needed modules are present
4. **Review each grid component** against the full checklist
5. **Review custom components** (cell renderers, editors, filters) for correctness
6. **Check for cross-cutting concerns**: state management integration, error boundaries
7. **Summarize findings** grouped by severity, with actionable fixes for each

## What NOT to Flag

- Style preferences that don't affect functionality
- Using `AllCommunityModule` in a prototype or POC
- Missing `useMemo` on static values that genuinely never change
- Enterprise features used with valid license
- Patterns that are correct but unfamiliar
