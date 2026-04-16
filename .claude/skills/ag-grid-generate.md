---
name: ag-grid-generate
description: Scaffold a fully typed AG Grid React component with proper module registration, column definitions, and best practices baked in. Supports client-side, server-side, infinite scroll, and master/detail patterns.
user_invokable: true
---

# AG Grid Component Generator

You are an AG Grid expert. Generate a production-ready AG Grid React component based on the user's requirements.

## Instructions

1. **Analyze the request** to determine:
   - Row model type: `clientSide` (default), `serverSide`, `infinite`
   - Whether master/detail is needed
   - Whether enterprise features are required (row grouping, pivot, set filter, excel export, etc.)
   - Framework: React (default), Angular, Vue, or vanilla JS
   - Data shape / TypeScript interface

2. **Search the codebase** for existing AG Grid usage patterns:
   - Check `package.json` for installed AG Grid version and packages
   - Look for existing module registration files
   - Find existing AG Grid components to match coding style
   - Check for existing TypeScript interfaces that match the data model

3. **Generate the component** following these rules:

### Module Registration

Always use selective module imports for tree shaking. Never use `AllCommunityModule` or `AllEnterpriseModule` in production code.

```tsx
// Register only what you need
import { ModuleRegistry, ClientSideRowModelModule, ValidationModule } from 'ag-grid-community';
// Add enterprise modules only if needed
import { RowGroupingModule, SetFilterModule } from 'ag-grid-enterprise';

ModuleRegistry.registerModules([
  ClientSideRowModelModule,
  ValidationModule, // dev-time warnings for missing modules
  // ... only the modules actually used
]);
```

If the project already has a module registration file, add new modules there instead of duplicating.

### Component Structure

```tsx
import { AgGridReact } from 'ag-grid-react';
import type { ColDef, GridReadyEvent, GridApi } from 'ag-grid-community';
import { useRef, useMemo, useCallback, useState } from 'react';

interface IRowData {
  // Typed from user's data shape
}

export const MyGrid = () => {
  const gridRef = useRef<AgGridReact<IRowData>>(null);

  const columnDefs = useMemo<ColDef<IRowData>[]>(() => [
    // Generated column definitions
  ], []);

  const defaultColDef = useMemo<ColDef<IRowData>>(() => ({
    sortable: true,
    filter: true,
    resizable: true,
    flex: 1,
    minWidth: 100,
  }), []);

  const getRowId = useCallback((params: GetRowIdParams<IRowData>) => {
    return params.data.id; // ALWAYS provide getRowId
  }, []);

  const onGridReady = useCallback((event: GridReadyEvent<IRowData>) => {
    // Initial setup
  }, []);

  return (
    <div style={{ height: '100%', width: '100%' }}>
      <AgGridReact<IRowData>
        ref={gridRef}
        columnDefs={columnDefs}
        defaultColDef={defaultColDef}
        getRowId={getRowId}
        onGridReady={onGridReady}
        rowData={rowData}
      />
    </div>
  );
};
```

### Critical Rules

- **ALWAYS** use `useMemo` for `columnDefs`, `defaultColDef`, and any object/array props
- **ALWAYS** use `useCallback` for event handlers with proper dependency arrays
- **ALWAYS** provide `getRowId` — this prevents state loss on data updates
- **ALWAYS** use TypeScript generics: `AgGridReact<TData>`, `ColDef<TData>`, etc.
- **NEVER** pass inline objects/arrays as props (causes re-renders)
- **NEVER** use deprecated `ColumnApi` — all methods are on `GridApi` since v31
- Prefer `valueFormatter` over `cellRenderer` for simple display formatting
- Use `cellRendererSelector` for conditional rendering instead of complex ternaries
- Check `params.data` for undefined in cell renderers (group rows have no data)

### Server-Side Row Model Pattern

When `serverSide` is requested:
```tsx
const datasource: IServerSideDatasource = useMemo(() => ({
  getRows: (params) => {
    const { startRow, endRow, sortModel, filterModel, groupKeys, rowGroupCols } = params.request;
    fetchData({ startRow, endRow, sortModel, filterModel, groupKeys, rowGroupCols })
      .then(response => {
        params.success({ rowData: response.rows, rowCount: response.totalCount });
      })
      .catch(() => params.fail());
  },
}), []);
```

### Master/Detail Pattern

When master/detail is requested:
```tsx
const detailCellRendererParams = useMemo(() => ({
  detailGridOptions: {
    columnDefs: [/* detail columns */],
    defaultColDef: { flex: 1 },
  },
  getDetailRowData: (params) => {
    params.successCallback(params.data.details);
  },
}), []);

<AgGridReact
  masterDetail={true}
  detailCellRendererParams={detailCellRendererParams}
/>
```

### Module Reference

Community modules: `ClientSideRowModelModule`, `InfiniteRowModelModule`, `CsvExportModule`, `PaginationModule`, `RowSelectionModule`, `TextEditorModule`, `TextFilterModule`, `NumberFilterModule`, `DateFilterModule`, `ValidationModule`, `RowDragModule`

Enterprise modules: `ServerSideRowModelModule`, `RowGroupingModule`, `ExcelExportModule`, `ColumnMenuModule`, `ContextMenuModule`, `CellSelectionModule`, `MasterDetailModule`, `ColumnsToolPanelModule`, `FiltersToolPanelModule`, `SetFilterModule`, `MultiFilterModule`, `SparklinesModule`, `IntegratedChartsModule`, `PivotModule`, `ClipboardModule`, `StatusBarModule`

### Theming (v33+)

```tsx
import { themeQuartz } from 'ag-grid-community';

// Basic
<AgGridReact theme={themeQuartz} />

// Customized
const myTheme = themeQuartz.withParams({
  accentColor: '#3B82F6',
  headerBackgroundColor: '#F8FAFC',
  oddRowBackgroundColor: '#F8FAFC',
  spacing: 8,
});
```

4. **After generating**, verify:
   - All imported modules are registered
   - No `any` types where specific types are available
   - All React hooks have correct dependency arrays
   - Container div has explicit height (AG Grid needs it)
   - `getRowId` is provided
