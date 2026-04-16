---
name: ag-grid-troubleshoot
description: Diagnose and fix common AG Grid issues. Covers rendering problems, state loss, performance degradation, module errors, styling conflicts, and data update issues.
user_invokable: true
---

# AG Grid Troubleshooter

You are an AG Grid troubleshooting expert. Diagnose the user's issue and provide targeted fixes.

## Instructions

1. **Understand the problem** — ask clarifying questions if needed:
   - What is the expected behavior?
   - What is the actual behavior?
   - When did it start happening? (after upgrade, after adding feature, etc.)
   - Is there an error in the console?

2. **Search the codebase** for the relevant AG Grid code:
   - Find the component experiencing the issue
   - Check module registration
   - Check AG Grid version in `package.json`

3. **Match against known issues database:**

---

## Known Issue Database

### RENDERING ISSUES

#### Grid shows blank / no rows
**Symptoms:** Grid container renders but no data appears.
**Causes & Fixes:**
1. **Missing container height** — AG Grid needs explicit height
   ```tsx
   // FIX: Ensure parent has height
   <div style={{ height: 500 }}> {/* or height: '100%' with parent height */}
     <AgGridReact ... />
   </div>
   ```
2. **Missing row model module** — No `ClientSideRowModelModule` registered
   ```tsx
   // FIX: Register the module
   ModuleRegistry.registerModules([ClientSideRowModelModule]);
   ```
3. **rowData is undefined** — Data not loaded yet
   ```tsx
   // FIX: Pass empty array while loading, or use loading overlay
   <AgGridReact rowData={data ?? []} />
   ```
4. **Theming API + legacy CSS conflict** — Cannot mix on same page
   ```tsx
   // FIX: Choose one approach
   <AgGridReact theme={themeQuartz} /> // Theming API
   // OR
   <AgGridReact theme="legacy" />     // Legacy CSS
   ```

#### Grid renders but columns are missing
**Causes:**
1. `columnDefs` is empty or undefined
2. `field` names don't match row data property names (case-sensitive!)
3. `hide: true` on columns
4. Column `width: 0` or `maxWidth: 0`

#### Cells show `[object Object]`
**Cause:** Field points to an object, not a primitive.
**Fix:** Use `valueGetter` or `valueFormatter`:
```tsx
{
  headerName: 'Address',
  valueGetter: (params) => params.data?.address?.street,
  // OR
  field: 'address',
  valueFormatter: (params) => params.value?.street ?? '',
}
```

#### Cell renderer not showing / flickering
**Causes:**
1. Cell renderer returns undefined instead of null/empty string
2. Cell renderer doesn't handle `params.data` being undefined (group rows)
3. React cell renderer not memoized, causing constant re-mounts
**Fix:**
```tsx
const MyCellRenderer = React.memo((params: ICellRendererParams<IRowData>) => {
  if (!params.data) return null; // Handle group rows
  return <span>{params.value}</span>;
});
```

---

### STATE LOSS ISSUES

#### Selection/scroll/expansion lost on data update
**Cause:** Missing `getRowId` — grid can't track row identity.
**Fix:**
```tsx
<AgGridReact
  getRowId={(params) => params.data.id}
  rowData={rowData}
/>
```

#### Column state (width/sort/filter/order) resets on re-render
**Cause:** `columnDefs` reference changes every render.
**Fix:**
```tsx
// BAD
<AgGridReact columnDefs={[{ field: 'name' }]} />

// GOOD
const columnDefs = useMemo(() => [{ field: 'name' }], []);
<AgGridReact columnDefs={columnDefs} />
```

#### Filter model not persisting
**Cause:** Grid re-initializes due to key change or columnDefs instability.
**Fix:** Stabilize all object references. Use `initialState` for persistence:
```tsx
const savedState = JSON.parse(localStorage.getItem('gridState') ?? 'null');
const onStateUpdated = useCallback((e: StateUpdatedEvent) => {
  localStorage.setItem('gridState', JSON.stringify(e.api.getState()));
}, []);

<AgGridReact
  initialState={savedState}
  onStateUpdated={onStateUpdated}
/>
```

---

### MODULE / IMPORT ERRORS

#### "Module not registered" console warning
**Cause:** Using a feature without registering its module.
**Fix:** Register the required module. Add `ValidationModule` to see clear warnings:
```tsx
import { ValidationModule } from 'ag-grid-community';
ModuleRegistry.registerModules([ValidationModule, /* missing module */]);
```

Common missing modules:
| Feature | Required Module |
|---------|----------------|
| Sorting/filtering | Included in row model module |
| Pagination | `PaginationModule` |
| CSV export | `CsvExportModule` |
| Row selection checkboxes | `RowSelectionModule` |
| Text editing | `TextEditorModule` |
| Set filter | `SetFilterModule` (enterprise) |
| Excel export | `ExcelExportModule` (enterprise) |
| Row grouping | `RowGroupingModule` (enterprise) |
| Clipboard | `ClipboardModule` (enterprise) |
| Context menu | `ContextMenuModule` (enterprise) |
| Column menu | `ColumnMenuModule` (enterprise) |
| Side bar | `ColumnsToolPanelModule` / `FiltersToolPanelModule` (enterprise) |
| Master detail | `MasterDetailModule` (enterprise) |
| Range/cell selection | `CellSelectionModule` (enterprise) |

#### "ag-grid-enterprise is not a module" / import errors after upgrade
**Cause:** Mixing old scoped packages (`@ag-grid-community/*`) with new unified packages.
**Fix:** Remove all `@ag-grid-community/*` and `@ag-grid-enterprise/*` packages:
```bash
npm uninstall @ag-grid-community/core @ag-grid-community/react @ag-grid-enterprise/all-modules
npm install ag-grid-community ag-grid-react ag-grid-enterprise
```

---

### PERFORMANCE ISSUES

#### Grid is slow to render initially
**Diagnose:** Check for:
1. `AllCommunityModule` / `AllEnterpriseModule` (large bundle)
2. Many custom React cell renderers (DOM overhead)
3. `domLayout='autoHeight'` with many rows
4. Missing `getRowId` causing full re-diff

#### Grid is slow when scrolling
**Diagnose:** Check for:
1. Complex React cell renderers (replace with `valueFormatter` where possible)
2. `autoHeight: true` on many columns
3. `cellClassRules` with expensive calculations
4. High `rowBuffer` value
5. `suppressColumnVirtualisation={true}`

**Fix:**
```tsx
<AgGridReact
  rowBuffer={5}
  animateRows={false}
  debounceVerticalScrollbar={true}
/>
```

#### Grid is slow with large dataset (>10K rows)
**Fix:**
1. Switch to server-side row model for >50K rows
2. Use pagination: `pagination={true} paginationPageSize={100}`
3. Use `api.applyTransactionAsync()` for streaming updates
4. Reduce columns or use column virtualization

---

### EDITING ISSUES

#### Editor doesn't open
**Causes:**
1. `editable` not set on column or `defaultColDef`
2. Missing `TextEditorModule` registration
3. `suppressCellFocus` is enabled
4. Cell has `cellRenderer` that captures click events

#### Editor opens but value doesn't save
**Causes:**
1. Custom `valueSetter` returns `false` or doesn't return `true`
2. `stopEditingWhenCellsLoseFocus` is `false` and user clicks away
3. Custom editor doesn't call `onValueChange`

#### Arrow keys navigate grid instead of moving cursor in editor
**Fix:** Stop propagation in editor:
```tsx
const onKeyDown = (event: React.KeyboardEvent) => {
  if (['ArrowLeft', 'ArrowRight'].includes(event.key)) {
    event.stopPropagation();
  }
};
```

---

### STYLING ISSUES

#### Custom CSS breaks grid layout
**Cause:** Modifying `position`, `overflow`, or `pointer-events` on AG Grid elements.
**Safe to modify:** colors, fonts, padding, margins, borders, backgrounds, box-shadow.
**Dangerous to modify:** position, display, overflow, pointer-events, z-index on internal elements.

#### Theme not applying
**Causes:**
1. Theming API and legacy CSS used on same page
2. CSS import order wrong (AG Grid CSS must load before custom CSS)
3. Theme parameter names are wrong (they changed in v33)

**v33+ Theming API:**
```tsx
import { themeQuartz } from 'ag-grid-community';
const myTheme = themeQuartz.withParams({
  accentColor: '#3B82F6',
  backgroundColor: '#FFFFFF',
  headerBackgroundColor: '#F1F5F9',
});
<AgGridReact theme={myTheme} />
```

#### Dark mode not working
**Fix with Theming API:**
```tsx
import { themeQuartz, colorSchemeDark } from 'ag-grid-community';
const darkTheme = themeQuartz.withPart(colorSchemeDark);
```

---

### SERVER-SIDE ROW MODEL ISSUES

#### SSRM loads data but shows "Loading..." forever
**Cause:** `params.success()` not called, or called with wrong shape.
**Fix:** Ensure `params.success({ rowData: [...], rowCount: total })` is called.

#### SSRM grouping doesn't work
**Cause:** Server doesn't handle `groupKeys` and `rowGroupCols` from the request.
**Fix:** Server must detect group request and return group data:
```tsx
// params.request.groupKeys.length > 0 means a group is being expanded
// params.request.rowGroupCols defines which columns are grouped
```

#### SSRM cache not refreshing
**Fix:**
```tsx
api.refreshServerSide({ route: [], purge: true }); // Refresh all
api.refreshServerSide({ route: ['US'], purge: false }); // Refresh specific group
```

---

### TYPESCRIPT ERRORS

#### "Type 'X' is not assignable to type 'ColDef'"
**Common causes:**
1. Using old API names (`cellRendererFramework` → `cellRenderer`)
2. Wrong generic type (`ColDef` should be `ColDef<TData>`)
3. `filterParams` type mismatch

#### Event handler type errors
**Fix:** Use proper generic event types:
```tsx
const onCellClicked = useCallback((event: CellClickedEvent<IRowData>) => {
  // event.data is typed as IRowData | undefined
}, []);
```

---

4. **Apply the fix** and verify:
   - Fix compiles without TypeScript errors
   - Fix doesn't introduce new console warnings
   - Original functionality is preserved
   - If performance issue, measure before/after (suggest Chrome DevTools Performance tab)
