# AG Grid Skills & Agents for Claude Code

This repository provides Claude Code skills and an expert agent specialized for AG Grid development. They encode deep knowledge of AG Grid's API (v33-v35+), module system, performance patterns, and common pitfalls.

## Repository Structure

```
.claude/
  skills/       # User-invokable slash commands
    ag-grid-optimize.md   # /ag-grid-optimize - Performance audit & fixes
    ag-grid-migrate.md    # /ag-grid-migrate - Version migration helper
    ag-grid-review.md     # /ag-grid-review - AG Grid code review (60+ checks)
  agents/       # Specialized autonomous agents
    ag-grid-expert.md     # Deep AG Grid knowledge agent
```

## AG Grid Version Context

- Current latest: v35.2.x (April 2026)
- v33 introduced the modular architecture (tree-shakable single bundles)
- v34 added cell editor validation, batch editing, tree data DnD
- v35 added formulas, row group dragging, BigInt support
- ColumnApi was deprecated in v31 (all methods moved to GridApi)
- Packages consolidated from 25+ to 2: `ag-grid-community` and `ag-grid-enterprise`

## Key Conventions

- Always use module registration (`ModuleRegistry.registerModules()`) for optimal bundle size
- Always provide `getRowId` for stable row identity
- Stabilize references with `useMemo`/`useCallback` in React
- Prefer `valueFormatter` over `cellRenderer` for simple display formatting
- Use `ValidationModule` during development to catch missing module errors
- TypeScript generics: `AgGridReact<TData>`, `ColDef<TData>`, event types like `CellClickedEvent<TData>`
