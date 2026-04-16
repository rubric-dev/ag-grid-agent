---
name: ag-grid-coldef
description: Generate typed AG Grid column definitions from a TypeScript interface, JSON data sample, or API response shape. Outputs production-ready ColDef arrays with proper typing, formatters, filters, and editors.
user_invokable: true
---

# AG Grid Column Definition Generator

You are an AG Grid column definition expert. Generate production-ready, fully typed column definitions.

## Instructions

1. **Determine the data source** — the user may provide:
   - A TypeScript interface or type
   - A JSON data sample
   - A file path to read the data shape from
   - An API endpoint response description
   - An existing AG Grid component to enhance

2. **Search the codebase** for context:
   - Find the TypeScript interface for the row data if not provided
   - Check existing column definitions for patterns/conventions used in the project
   - Look for existing value formatters, cell renderers, or custom components

3. **Generate column definitions** following these rules:

### Type Inference Rules

Map data types to appropriate AG Grid column configurations:

| TypeScript Type | Filter | Editor | Formatter |
|----------------|--------|--------|-----------|
| `string` | `'agTextColumnFilter'` | `'agTextCellEditor'` | — |
| `number` | `'agNumberColumnFilter'` | `'agNumberCellEditor'` | `toLocaleString()` |
| `boolean` | `'agTextColumnFilter'` | `'agCheckboxCellEditor'` | — (use `cellRenderer` checkbox) |
| `Date \| string` (date) | `'agDateColumnFilter'` | `'agDateCellEditor'` | date format |
| `enum \| union` | `'agSetColumnFilter'` (enterprise) | `'agSelectCellEditor'` or `'agRichSelectCellEditor'` | — |
| `number` (currency) | `'agNumberColumnFilter'` | `'agNumberCellEditor'` | currency format |
| `number` (percent) | `'agNumberColumnFilter'` | `'agNumberCellEditor'` | percent format |

### Semantic Detection

Detect column semantics from field names and apply appropriate formatting:

- `*price*`, `*cost*`, `*amount*`, `*total*`, `*salary*`, `*revenue*` → currency formatting
- `*date*`, `*created*`, `*updated*`, `*timestamp*`, `*At` → date formatting
- `*percent*`, `*rate*`, `*ratio*` → percentage formatting
- `*email*` → email link renderer
- `*url*`, `*link*`, `*href*` → link renderer
- `*phone*` → phone formatting
- `*status*`, `*state*`, `*type*`, `*category*` → badge/chip renderer with set filter
- `*name*`, `*title*` → wider column, text filter
- `*id*` → narrower column, hide by default or pin left
- `*description*`, `*notes*`, `*comment*` → wider column, wrapText, autoHeight
- `*image*`, `*avatar*`, `*photo*`, `*thumbnail*` → image renderer
- `*active*`, `*enabled*`, `*visible*`, `*is*` (boolean) → checkbox renderer

### Output Format

```tsx
import type { ColDef, ValueFormatterParams, ICellRendererParams } from 'ag-grid-community';

interface IRowData {
  id: string;
  name: string;
  email: string;
  salary: number;
  startDate: string;
  department: string;
  active: boolean;
}

export const columnDefs: ColDef<IRowData>[] = [
  {
    field: 'id',
    headerName: 'ID',
    width: 90,
    pinned: 'left',
    filter: 'agTextColumnFilter',
    suppressMenu: true,
  },
  {
    field: 'name',
    headerName: 'Name',
    flex: 2,
    minWidth: 150,
    filter: 'agTextColumnFilter',
    filterParams: {
      filterOptions: ['contains', 'startsWith'],
      defaultOption: 'contains',
    },
  },
  {
    field: 'email',
    headerName: 'Email',
    flex: 2,
    minWidth: 200,
    filter: 'agTextColumnFilter',
    cellRenderer: (params: ICellRendererParams<IRowData>) => {
      if (!params.value) return null;
      return `<a href="mailto:${params.value}">${params.value}</a>`;
    },
  },
  {
    field: 'salary',
    headerName: 'Salary',
    flex: 1,
    minWidth: 120,
    filter: 'agNumberColumnFilter',
    valueFormatter: (params: ValueFormatterParams<IRowData, number>) => {
      if (params.value == null) return '';
      return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',
        minimumFractionDigits: 0,
      }).format(params.value);
    },
    cellStyle: { textAlign: 'right' },
  },
  {
    field: 'startDate',
    headerName: 'Start Date',
    flex: 1,
    minWidth: 120,
    filter: 'agDateColumnFilter',
    valueFormatter: (params: ValueFormatterParams<IRowData, string>) => {
      if (!params.value) return '';
      return new Date(params.value).toLocaleDateString();
    },
  },
  {
    field: 'department',
    headerName: 'Department',
    flex: 1,
    minWidth: 130,
    filter: 'agSetColumnFilter', // Enterprise — fallback to agTextColumnFilter
    filterParams: {
      values: ['Engineering', 'Sales', 'Marketing', 'HR', 'Finance'],
    },
  },
  {
    field: 'active',
    headerName: 'Active',
    width: 100,
    cellRenderer: 'agCheckboxCellRenderer',
    cellEditor: 'agCheckboxCellEditor',
    editable: true,
  },
];
```

### Column Groups

When the data has logical groupings, use `ColGroupDef`:

```tsx
const columnDefs: (ColDef<IRowData> | ColGroupDef<IRowData>)[] = [
  {
    headerName: 'Personal Info',
    children: [
      { field: 'firstName' },
      { field: 'lastName' },
      { field: 'email' },
    ],
  },
  {
    headerName: 'Employment',
    children: [
      { field: 'department' },
      { field: 'salary' },
      { field: 'startDate' },
    ],
  },
];
```

### Column Types for Reuse

When multiple columns share config, define column types:

```tsx
const columnTypes: { [key: string]: ColDef } = {
  currencyColumn: {
    filter: 'agNumberColumnFilter',
    cellStyle: { textAlign: 'right' },
    valueFormatter: (params) => {
      if (params.value == null) return '';
      return new Intl.NumberFormat('en-US', {
        style: 'currency', currency: 'USD', minimumFractionDigits: 0,
      }).format(params.value);
    },
  },
  dateColumn: {
    filter: 'agDateColumnFilter',
    valueFormatter: (params) => {
      if (!params.value) return '';
      return new Date(params.value).toLocaleDateString();
    },
  },
};

// Usage: { field: 'salary', type: 'currencyColumn' }
```

### Critical Rules

- **ALWAYS** use `ColDef<TData>` generic for type safety
- **ALWAYS** include `minWidth` when using `flex` to prevent columns from becoming too narrow
- **ALWAYS** check for null/undefined in `valueFormatter` and `cellRenderer`
- **ALWAYS** use `valueFormatter` for display-only formatting (not `cellRenderer`)
- Only use `cellRenderer` when you need HTML elements or interactivity
- Set `headerName` explicitly — don't rely on AG Grid's auto-generation from `field`
- Include `filterParams` when the default filter behavior isn't optimal
- For enterprise features (set filter, etc.), note when community fallbacks are needed

4. **After generating**, offer to:
   - Create reusable column types if patterns repeat
   - Add editable configuration
   - Add row grouping / aggregation setup
   - Add export-specific formatting (Excel styles differ from display)
