---
name: ag-grid-optimize
description: Audit AG Grid implementation for performance issues and apply fixes. Covers bundle size, rendering performance, memory usage, React re-render prevention, and large dataset handling.
user_invokable: true
---

# AG Grid Performance Optimizer

You are an AG Grid performance expert. Audit the codebase for AG Grid performance issues and apply targeted fixes.

## Instructions

1. **Scan the codebase** for all AG Grid usage:
   - Find all files importing from `ag-grid-community`, `ag-grid-enterprise`, `ag-grid-react`
   - Find all `<AgGridReact` component instances
   - Check module registration patterns
   - Check `package.json` for AG Grid version

2. **Run the audit checklist** on each grid instance:

### Bundle Size Audit

| Issue | Severity | Check |
|-------|----------|-------|
| Using `AllCommunityModule` or `AllEnterpriseModule` | HIGH | Replace with selective module imports |
| Importing unused enterprise modules | HIGH | Remove modules not referenced by any grid |
| Not tree-shaking (old `@ag-grid-community/*` packages) | MEDIUM | Migrate to `ag-grid-community` v33+ |
| Importing entire AG Charts for integrated charts | MEDIUM | Use `IntegratedChartsModule.with(AgChartsCommunityModule)` |

**Fix pattern — selective modules:**
```tsx
// BEFORE (bad): imports everything
import { AllCommunityModule } from 'ag-grid-community';
ModuleRegistry.registerModules([AllCommunityModule]);

// AFTER (good): only what's needed
import {
  ModuleRegistry,
  ClientSideRowModelModule,
  TextFilterModule,
  NumberFilterModule,
  PaginationModule,
  RowSelectionModule,
  ValidationModule,
} from 'ag-grid-community';

ModuleRegistry.registerModules([
  ClientSideRowModelModule,
  TextFilterModule,
  NumberFilterModule,
  PaginationModule,
  RowSelectionModule,
  ValidationModule,
]);
```

### React Re-render Prevention

| Issue | Severity | Check |
|-------|----------|-------|
| `columnDefs` not in `useMemo` | CRITICAL | Wrap in `useMemo` |
| `defaultColDef` not in `useMemo` | CRITICAL | Wrap in `useMemo` |
| Inline objects/arrays as props | HIGH | Extract to `useMemo` |
| Event handlers not in `useCallback` | MEDIUM | Wrap in `useCallback` |
| `rowData` recreated every render | HIGH | Stabilize reference |
| `context` object not memoized | MEDIUM | Wrap in `useMemo` |
| `sideBar` config not memoized | MEDIUM | Wrap in `useMemo` |
| `statusBar` config not memoized | MEDIUM | Wrap in `useMemo` |

**Fix pattern:**
```tsx
// BEFORE (bad): new reference every render
<AgGridReact
  columnDefs={[{ field: 'name' }, { field: 'age' }]}
  defaultColDef={{ sortable: true, filter: true }}
  onGridReady={(e) => e.api.sizeColumnsToFit()}
/>

// AFTER (good): stable references
const columnDefs = useMemo<ColDef<IRowData>[]>(() => [
  { field: 'name' },
  { field: 'age' },
], []);

const defaultColDef = useMemo<ColDef>(() => ({
  sortable: true,
  filter: true,
}), []);

const onGridReady = useCallback((event: GridReadyEvent) => {
  event.api.sizeColumnsToFit();
}, []);

<AgGridReact
  columnDefs={columnDefs}
  defaultColDef={defaultColDef}
  onGridReady={onGridReady}
/>
```

### Row Identity & Data Updates

| Issue | Severity | Check |
|-------|----------|-------|
| Missing `getRowId` | CRITICAL | Add `getRowId` — prevents full re-render on data change |
| Using `api.setRowData()` for updates | HIGH | Use `rowData` prop or `api.applyTransaction()` |
| Not using transactions for streaming data | HIGH | Use `api.applyTransactionAsync()` |
| Recreating entire rowData on single-row update | MEDIUM | Use transaction API for partial updates |

**Fix pattern:**
```tsx
// ALWAYS provide getRowId
const getRowId = useCallback((params: GetRowIdParams<IRowData>) => {
  return params.data.id;
}, []);

// For high-frequency updates (streaming, real-time)
const updateRows = useCallback((updates: IRowData[]) => {
  gridRef.current?.api.applyTransactionAsync({
    update: updates,
  });
}, []);
```

### Cell Rendering Performance

| Issue | Severity | Check |
|-------|----------|-------|
| Using `cellRenderer` for simple text formatting | HIGH | Replace with `valueFormatter` |
| React cell renderer for non-interactive content | MEDIUM | Consider vanilla JS renderer |
| Complex cell renderer without memoization | MEDIUM | Add `React.memo()` |
| `cellClassRules` with expensive functions | MEDIUM | Simplify or cache |
| `autoHeight` on all columns | HIGH | Use only where needed |
| `wrapText` without `autoHeight` | LOW | Add `autoHeight` or set fixed `rowHeight` |

**Fix pattern:**
```tsx
// BEFORE (bad): cell renderer for simple formatting
{ field: 'price', cellRenderer: (p) => <span>${p.value?.toLocaleString()}</span> }

// AFTER (good): value formatter (no DOM overhead)
{ field: 'price', valueFormatter: (p) => `$${p.value?.toLocaleString() ?? ''}` }
```

### Large Dataset Handling

| Issue | Severity | Check |
|-------|----------|-------|
| Client-side model with >50K rows | HIGH | Switch to server-side or infinite row model |
| `domLayout='autoHeight'` with >500 rows | CRITICAL | Use `domLayout='normal'` with fixed height |
| `suppressColumnVirtualisation={true}` with >20 cols | HIGH | Remove — let virtualization work |
| `suppressRowVirtualisation={true}` | CRITICAL | Remove unless accessibility requires it |
| High `rowBuffer` value (>20) | MEDIUM | Reduce to 5-10 |

**Fix pattern:**
```tsx
// Optimize for large datasets
<AgGridReact
  rowBuffer={5}
  animateRows={false}
  suppressColumnVirtualisation={false}
  debounceVerticalScrollbar={true}
  cacheBlockSize={100}
/>
```

### Memory Leaks

| Issue | Severity | Check |
|-------|----------|-------|
| Grid API stored in state/ref without cleanup | MEDIUM | Clear ref on unmount |
| Event listeners not removed | HIGH | Use grid event props, not manual `addEventListener` |
| Master/Detail without `keepDetailRowsCount` limit | MEDIUM | Set `keepDetailRowsCount` |
| Infinite cache without `maxBlocksInCache` | MEDIUM | Set reasonable limit |

### Quick Filter & Search

| Issue | Severity | Check |
|-------|----------|-------|
| `quickFilterText` without `cacheQuickFilter` | MEDIUM | Add `cacheQuickFilter={true}` |
| Custom filter calling `api.onFilterChanged()` excessively | MEDIUM | Debounce filter updates |

3. **Generate a report** with:
   - Issues found, categorized by severity (CRITICAL > HIGH > MEDIUM > LOW)
   - Specific file paths and line numbers
   - Before/after code for each fix
   - Estimated impact (bundle size reduction, render count reduction, etc.)

4. **Apply fixes** after confirming with the user, starting from CRITICAL issues.
