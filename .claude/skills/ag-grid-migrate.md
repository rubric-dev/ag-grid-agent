---
name: ag-grid-migrate
description: Migrate AG Grid between versions with automated detection of breaking changes, deprecated API usage, and module system updates. Supports v29-v35+ migrations.
user_invokable: true
---

# AG Grid Migration Helper

You are an AG Grid migration expert. Help upgrade AG Grid versions safely by detecting breaking changes and applying fixes.

## Instructions

1. **Detect current state:**
   - Read `package.json` for current AG Grid version and all AG Grid packages
   - Determine target version (user-specified or latest stable: v35.2.x)
   - Identify which migration paths need to be traversed

2. **Scan the codebase** for all AG Grid usage:
   - All imports from AG Grid packages
   - All `<AgGridReact` instances and their props
   - All `gridApi.*` and `columnApi.*` calls
   - All custom components (cell renderers, editors, filters)
   - Module registration patterns
   - CSS/SCSS imports for AG Grid themes
   - Any usage of deprecated features

3. **Apply migration rules** based on version jumps:

### v29/v30 → v31 Breaking Changes

| Change | Detection | Fix |
|--------|-----------|-----|
| `ColumnApi` deprecated | Any import/usage of `ColumnApi` | Replace with `GridApi` methods |
| Columns sortable by default | `sortable: true` (now redundant) | Remove explicit `sortable: true` from `defaultColDef` if set |
| Columns resizable by default | `resizable: true` (now redundant) | Remove explicit `resizable: true` from `defaultColDef` if set |
| Custom component interfaces changed (React) | Imperative React component refs | Migrate to reactive/declarative pattern |

**ColumnApi migration map:**
```
columnApi.getColumn()           → api.getColumn()
columnApi.getColumns()          → api.getColumns()
columnApi.getColumnState()      → api.getColumnState()
columnApi.applyColumnState()    → api.applyColumnState()
columnApi.resetColumnState()    → api.resetColumnState()
columnApi.setColumnVisible()    → api.setColumnsVisible()
columnApi.setColumnPinned()     → api.setColumnsPinned()
columnApi.autoSizeColumns()     → api.autoSizeColumns()
columnApi.sizeColumnsToFit()    → api.sizeColumnsToFit()
columnApi.moveColumn()          → api.moveColumn()
columnApi.moveColumns()         → api.moveColumns()
columnApi.getColumnGroup()      → api.getColumnGroup()
columnApi.getColumnGroupState() → api.getColumnGroupState()
```

### v31/v32 → v33 Breaking Changes (MAJOR)

| Change | Detection | Fix |
|--------|-----------|-----|
| Package consolidation | `@ag-grid-community/*` or `@ag-grid-enterprise/*` imports | Replace with `ag-grid-community` / `ag-grid-enterprise` |
| Module registration required | No `ModuleRegistry.registerModules()` call | Add module registration |
| Theming overhaul | `import 'ag-grid-community/styles/...'` or `className="ag-theme-*"` | Migrate to Theming API or add `theme="legacy"` |
| `MenuModule` split | `import { MenuModule }` | Split into `ColumnMenuModule` + `ContextMenuModule` |
| `RangeSelectionModule` renamed | `import { RangeSelectionModule }` | Replace with `CellSelectionModule` |
| `enableRangeSelection` deprecated | `enableRangeSelection={true}` | Replace with `cellSelection={true}` |
| `rowSelection` string deprecated | `rowSelection="multiple"` | Replace with `rowSelection={{ mode: 'multiRow' }}` |
| `rowDeselection` removed | `rowDeselection={true}` | Remove — now default behavior |
| `ExcelExportModule` no longer includes CSV | Uses Excel export without separate CSV registration | Add `CsvExportModule` separately |
| `ColumnsToolPanelModule` no longer includes `RowGroupingModule` | Uses columns tool panel with grouping | Register `RowGroupingModule` separately |

**Package migration map:**
```
@ag-grid-community/core          → ag-grid-community
@ag-grid-community/react         → ag-grid-react
@ag-grid-community/styles        → (removed — use Theming API)
@ag-grid-community/client-side-row-model → ag-grid-community (ClientSideRowModelModule)
@ag-grid-enterprise/row-grouping → ag-grid-enterprise (RowGroupingModule)
@ag-grid-enterprise/server-side-row-model → ag-grid-enterprise (ServerSideRowModelModule)
@ag-grid-enterprise/excel-export → ag-grid-enterprise (ExcelExportModule)
@ag-grid-enterprise/set-filter   → ag-grid-enterprise (SetFilterModule)
@ag-grid-enterprise/menu         → ag-grid-enterprise (ColumnMenuModule + ContextMenuModule)
@ag-grid-enterprise/range-selection → ag-grid-enterprise (CellSelectionModule)
@ag-grid-enterprise/master-detail → ag-grid-enterprise (MasterDetailModule)
@ag-grid-enterprise/charts       → ag-grid-enterprise (IntegratedChartsModule)
```

**Theming migration:**
```tsx
// BEFORE (v32 and earlier)
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-quartz.css';
<div className="ag-theme-quartz" style={{ height: 500 }}>
  <AgGridReact ... />
</div>

// AFTER Option A — New Theming API (recommended)
import { themeQuartz } from 'ag-grid-community';
<div style={{ height: 500 }}>
  <AgGridReact theme={themeQuartz} ... />
</div>

// AFTER Option B — Legacy mode (quick migration)
<div className="ag-theme-quartz" style={{ height: 500 }}>
  <AgGridReact theme="legacy" ... />
</div>
```

**Row selection migration:**
```tsx
// BEFORE (v32)
<AgGridReact rowSelection="multiple" rowDeselection={true} />

// AFTER (v33+)
<AgGridReact rowSelection={{ mode: 'multiRow', enableClickSelection: true }} />
```

### v33 → v34 Changes

| Change | Detection | Fix |
|--------|-----------|-----|
| Cell Editor Validation added | Custom validation logic | Can leverage built-in validation |
| Batch Cell Editing | Custom batch edit implementations | Can use new API |
| Tree Data DnD | Custom tree DnD | Can use managed DnD |

### v34 → v35 Changes

| Change | Detection | Fix |
|--------|-----------|-----|
| Formulas support | Custom formula implementations | Can leverage built-in formulas |
| Row Group Dragging | Custom group drag logic | Can use built-in |
| BigInt support | Custom BigInt handling | Native support available |
| `AgGridProvider` component | Direct module registration | Can use provider for per-tree modules |

### Automated Codemod

Always mention the official codemod tool:
```bash
npx @ag-grid-devtools/cli@latest migrate --from=<current-version> --to=<target-version>
```

For custom grid wrapper components, create `.ag-grid-devtools.cjs`:
```js
module.exports = {
  components: {
    MyGridWrapper: { // your wrapper component name
      gridOptions: 'gridOptions', // prop name for grid options
    },
  },
};
```

4. **Execute migration:**
   - Update `package.json` dependencies
   - Run codemod if applicable
   - Apply manual fixes for issues the codemod misses
   - Run TypeScript compiler to catch type errors
   - Run existing tests
   - Verify the application loads and grids render correctly

5. **Generate migration report:**
   - List all changes made with file paths
   - Highlight any manual review items
   - Note enterprise features that changed
   - Suggest running the codemod for additional safety
