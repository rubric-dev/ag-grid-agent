---
name: ag-grid-test
description: Generate comprehensive tests for AG Grid components. Supports React Testing Library, Playwright, Cypress, and Jest. Handles virtualization, async data loading, and grid API assertions.
user_invokable: true
---

# AG Grid Test Generator

You are an AG Grid testing expert. Generate comprehensive tests for AG Grid components.

## Instructions

1. **Analyze the target component:**
   - Read the AG Grid component to be tested
   - Identify the row model type (client-side, server-side, infinite)
   - List all custom components (cell renderers, editors, filters)
   - Identify event handlers and API interactions
   - Check what testing framework the project uses (check `package.json`, existing test files)

2. **Determine the testing approach:**

### Testing Strategy by Complexity

| Scenario | Recommended Approach |
|----------|---------------------|
| Simple grid with static data | Unit test with React Testing Library |
| Grid with custom renderers | Unit test + RTL queries |
| Grid with editing | Unit test with fireEvent/userEvent |
| Grid with server-side data | Unit test with mocked datasource |
| Grid with complex interactions (DnD, clipboard) | E2E test with Playwright/Cypress |
| Grid with scroll-dependent behavior | E2E test (jsdom lacks layout) |
| Grid with virtualization edge cases | E2E test |

3. **Generate tests** following these patterns:

### Setup: AG Grid Test Utilities (v33+)

```typescript
// test-setup.ts
import { ModuleRegistry, ValidationModule } from 'ag-grid-community';
// Import modules needed by the component under test
import { ClientSideRowModelModule } from 'ag-grid-community';

// Register modules for tests
ModuleRegistry.registerModules([
  ClientSideRowModelModule,
  ValidationModule,
]);

// jsdom polyfill for innerText (AG Grid uses it)
if (typeof Element.prototype.innerText === 'undefined') {
  Object.defineProperty(Element.prototype, 'innerText', {
    get() { return this.textContent; },
    set(value) { this.textContent = value; },
  });
}
```

### Pattern 1: React Testing Library (Unit Tests)

```typescript
import { render, screen, waitFor, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { AgGridReact } from 'ag-grid-react';

// Helper to wait for AG Grid to fully render
const waitForGridReady = () =>
  waitFor(() => {
    expect(document.querySelector('.ag-root-wrapper')).toBeInTheDocument();
    expect(document.querySelectorAll('.ag-row').length).toBeGreaterThan(0);
  });

// Helper to get cell content
const getCellContent = (rowIndex: number, colId: string): string => {
  const row = document.querySelectorAll('.ag-row')[rowIndex];
  const cell = row?.querySelector(`[col-id="${colId}"]`);
  return cell?.textContent ?? '';
};

// Helper to get all row data displayed
const getDisplayedRows = (colId: string): string[] => {
  const cells = document.querySelectorAll(`[col-id="${colId}"]`);
  return Array.from(cells)
    .filter(cell => cell.closest('.ag-row') && !cell.closest('.ag-header'))
    .map(cell => cell.textContent ?? '');
};

describe('MyGrid', () => {
  const mockData = [
    { id: '1', name: 'Alice', age: 30, department: 'Engineering' },
    { id: '2', name: 'Bob', age: 25, department: 'Sales' },
    { id: '3', name: 'Charlie', age: 35, department: 'Engineering' },
  ];

  it('renders grid with data', async () => {
    render(<MyGrid data={mockData} />);
    await waitForGridReady();

    expect(getCellContent(0, 'name')).toBe('Alice');
    expect(getCellContent(1, 'name')).toBe('Bob');
  });

  it('sorts by column when header clicked', async () => {
    render(<MyGrid data={mockData} />);
    await waitForGridReady();

    const nameHeader = document.querySelector('[col-id="name"] .ag-header-cell-label');
    await userEvent.click(nameHeader!);

    await waitFor(() => {
      const names = getDisplayedRows('name');
      expect(names).toEqual(['Alice', 'Bob', 'Charlie']);
    });
  });

  it('filters data', async () => {
    render(<MyGrid data={mockData} />);
    await waitForGridReady();

    // Use Grid API via ref or context
    // Programmatic filter is more reliable than UI interaction in jsdom
  });
});
```

### Pattern 2: Using AG Grid Test IDs (v33+)

```typescript
import { setupAgTestIds, agTestIdFor } from 'ag-grid-community';
import { getByTestId, queryByTestId } from '@testing-library/react';

// Call once in test setup
setupAgTestIds();

describe('MyGrid with Test IDs', () => {
  it('displays correct cell values', async () => {
    const { container } = render(<MyGrid data={mockData} />);
    await waitForGridReady();

    // agTestIdFor.cell(rowId, colId) generates test IDs
    const cell = getByTestId(container, agTestIdFor.cell('1', 'name'));
    expect(cell).toHaveTextContent('Alice');
  });

  it('renders header correctly', async () => {
    const { container } = render(<MyGrid data={mockData} />);
    await waitForGridReady();

    const header = getByTestId(container, agTestIdFor.header('name'));
    expect(header).toHaveTextContent('Name');
  });
});
```

### Pattern 3: Grid API Testing

```typescript
describe('Grid API interactions', () => {
  it('selects rows programmatically', async () => {
    let gridApi: GridApi;
    const onGridReady = (event: GridReadyEvent) => {
      gridApi = event.api;
    };

    render(<MyGrid data={mockData} onGridReady={onGridReady} />);
    await waitForGridReady();

    gridApi!.selectAll();

    await waitFor(() => {
      const selectedRows = gridApi!.getSelectedRows();
      expect(selectedRows).toHaveLength(3);
    });
  });

  it('applies filter model correctly', async () => {
    let gridApi: GridApi;
    const onGridReady = (event: GridReadyEvent) => {
      gridApi = event.api;
    };

    render(<MyGrid data={mockData} onGridReady={onGridReady} />);
    await waitForGridReady();

    gridApi!.setFilterModel({
      department: { filterType: 'text', type: 'equals', filter: 'Engineering' },
    });

    await waitFor(() => {
      expect(gridApi!.getDisplayedRowCount()).toBe(2);
    });
  });

  it('applies transactions correctly', async () => {
    let gridApi: GridApi;
    const onGridReady = (event: GridReadyEvent) => {
      gridApi = event.api;
    };

    render(<MyGrid data={mockData} onGridReady={onGridReady} />);
    await waitForGridReady();

    gridApi!.applyTransaction({
      add: [{ id: '4', name: 'Diana', age: 28, department: 'HR' }],
    });

    await waitFor(() => {
      expect(gridApi!.getDisplayedRowCount()).toBe(4);
    });
  });
});
```

### Pattern 4: Server-Side Row Model Tests

```typescript
describe('Server-Side Grid', () => {
  const createMockDatasource = (data: IRowData[]): IServerSideDatasource => ({
    getRows: (params) => {
      const { startRow, endRow } = params.request;
      const slice = data.slice(startRow, endRow);
      params.success({
        rowData: slice,
        rowCount: data.length,
      });
    },
  });

  it('loads initial data from datasource', async () => {
    const datasource = createMockDatasource(mockData);
    render(<MyServerGrid datasource={datasource} />);
    await waitForGridReady();

    expect(getCellContent(0, 'name')).toBe('Alice');
  });

  it('handles datasource failure gracefully', async () => {
    const failingDatasource: IServerSideDatasource = {
      getRows: (params) => params.fail(),
    };

    render(<MyServerGrid datasource={failingDatasource} />);
    await waitForGridReady();

    // Grid should show overlay or handle error
    await waitFor(() => {
      expect(document.querySelector('.ag-overlay-no-rows-wrapper')).toBeInTheDocument();
    });
  });
});
```

### Pattern 5: Custom Component Tests

```typescript
describe('Custom Cell Renderer', () => {
  it('renders custom content', async () => {
    render(<MyGrid data={mockData} />);
    await waitForGridReady();

    // For custom renderers that add specific elements
    const statusBadge = document.querySelector('[col-id="status"] .status-badge');
    expect(statusBadge).toBeInTheDocument();
    expect(statusBadge).toHaveClass('status-active');
  });
});

describe('Custom Cell Editor', () => {
  it('opens editor on double-click', async () => {
    render(<MyGrid data={mockData} />);
    await waitForGridReady();

    const cell = document.querySelector('[col-id="name"][row-index="0"]');
    await userEvent.dblClick(cell!);

    await waitFor(() => {
      const editor = document.querySelector('.ag-cell-editor input');
      expect(editor).toBeInTheDocument();
    });
  });
});
```

### Pattern 6: Playwright E2E Tests

```typescript
import { test, expect, Page } from '@playwright/test';

// Playwright helper
const getAgGridCell = (page: Page, rowIndex: number, colId: string) =>
  page.locator(`.ag-row[row-index="${rowIndex}"] [col-id="${colId}"]`);

const waitForAgGrid = (page: Page) =>
  page.waitForSelector('.ag-row', { state: 'visible' });

test.describe('AG Grid E2E', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/grid-page');
    await waitForAgGrid(page);
  });

  test('displays data correctly', async ({ page }) => {
    const firstCell = getAgGridCell(page, 0, 'name');
    await expect(firstCell).toHaveText('Alice');
  });

  test('sorts on header click', async ({ page }) => {
    await page.click('.ag-header-cell[col-id="name"]');
    await page.waitForTimeout(300); // wait for sort animation

    const cells = page.locator('[col-id="name"].ag-cell');
    const texts = await cells.allTextContents();
    expect(texts).toEqual([...texts].sort());
  });

  test('inline editing works', async ({ page }) => {
    const cell = getAgGridCell(page, 0, 'name');
    await cell.dblclick();

    const input = page.locator('.ag-cell-editor input');
    await input.fill('Updated Name');
    await input.press('Enter');

    await expect(cell).toHaveText('Updated Name');
  });

  test('row selection works', async ({ page }) => {
    const checkbox = page.locator('.ag-row[row-index="0"] .ag-selection-checkbox');
    await checkbox.click();

    const selectedRow = page.locator('.ag-row-selected');
    await expect(selectedRow).toHaveCount(1);
  });

  test('context menu appears', async ({ page }) => {
    const cell = getAgGridCell(page, 0, 'name');
    await cell.click({ button: 'right' });

    const menu = page.locator('.ag-menu');
    await expect(menu).toBeVisible();
  });

  test('exports to CSV', async ({ page }) => {
    const [download] = await Promise.all([
      page.waitForEvent('download'),
      page.evaluate(() => {
        // Trigger export via grid API
        (window as any).gridApi.exportDataAsCsv();
      }),
    ]);

    expect(download.suggestedFilename()).toContain('.csv');
  });
});
```

### Testing Gotchas

- **jsdom has no CSS layout**: `getBoundingClientRect()` returns zeros. Scroll-dependent tests need E2E.
- **Virtualization**: Only visible rows exist in DOM. For unit tests, keep test data small (<50 rows) or disable virtualization.
- **Async rendering**: AG Grid renders asynchronously. Always use `waitFor` or `waitForGridReady`.
- **`innerText` polyfill**: jsdom doesn't support `innerText`. Add polyfill in setup.
- **Column IDs**: Use `col-id` attribute for reliable cell selection, not CSS class or index.
- **Row IDs**: Use `row-index` attribute for row selection. With `getRowId`, use `row-id` for stable selection.
- **Enterprise modules in tests**: Register the same modules the component uses, or tests will fail silently.

4. **After generating tests**, verify:
   - All necessary modules are registered in test setup
   - Mock data matches the component's expected TypeScript interface
   - Tests don't depend on specific row count due to virtualization
   - Async operations are properly awaited
   - Test IDs are used where available (v33+)
