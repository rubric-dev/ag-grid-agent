# AG Grid Skills & Agents for Claude Code

Production-ready [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills and agent for [AG Grid](https://www.ag-grid.com/) development. Built from insights across official documentation, community best practices, and real-world production patterns.

Covers AG Grid **v29 through v35+**, with emphasis on the modern module architecture (v33+).

## What's Included

### Skills (Slash Commands)

Structured workflows that benefit from a repeatable checklist approach.

| Skill | Command | Description |
|-------|---------|-------------|
| **Optimize** | `/ag-grid-optimize` | Audit AG Grid code for performance issues: bundle size, React re-renders, virtualization tuning, memory leaks. Checklist-based with severity levels and before/after fixes |
| **Migrate** | `/ag-grid-migrate` | Migrate between AG Grid versions. Complete breaking change database for v31→v35, package migration maps, codemod support, theming migration |
| **Review** | `/ag-grid-review` | Code review with 60+ checks across 16 categories: module registration, row identity, React performance, column defs, type safety, SSRM, editing, theming, accessibility |

### Agent

| Agent | Description |
|-------|-------------|
| **ag-grid-expert** | All-purpose AG Grid expert. Answers architecture questions, generates components/column defs/tests, troubleshoots issues, compares features. Deep knowledge of API, modules, performance patterns, and TypeScript types |

The expert agent absorbs what would otherwise be thin standalone skills — component scaffolding, column definition generation, test generation, and troubleshooting — because these work better as contextual conversations than rigid workflows.

## Installation

### Option 1: Copy into your project (recommended)

```bash
# Clone and copy the .claude directory into your project
git clone https://github.com/rubric-dev/ag-grid-agent.git /tmp/ag-grid-agent
cp -r /tmp/ag-grid-agent/.claude/skills/ your-project/.claude/skills/
cp -r /tmp/ag-grid-agent/.claude/agents/ your-project/.claude/agents/
```

### Option 2: Git submodule

```bash
git submodule add https://github.com/rubric-dev/ag-grid-agent.git .claude/ag-grid
```

Then reference in your project's `.claude/settings.json`:

```json
{
  "skills": [
    ".claude/ag-grid/.claude/skills/ag-grid-optimize.md",
    ".claude/ag-grid/.claude/skills/ag-grid-migrate.md",
    ".claude/ag-grid/.claude/skills/ag-grid-review.md"
  ],
  "agents": [
    ".claude/ag-grid/.claude/agents/ag-grid-expert.md"
  ]
}
```

### Option 3: CLAUDE.md only

For lightweight AG Grid context without skills/agents:

```bash
# Append to your existing CLAUDE.md
curl -s https://raw.githubusercontent.com/rubric-dev/ag-grid-agent/develop/CLAUDE.md >> your-project/CLAUDE.md
```

## Usage Examples

### Performance audit

```
/ag-grid-optimize

Our main dashboard grid is slow — 3 second load, laggy scrolling.
Please audit and fix.
```

The optimizer scans your codebase for all AG Grid instances, checks against a severity-ranked checklist (bundle size, memoization, virtualization, data handling), and proposes specific fixes.

### Version migration

```
/ag-grid-migrate

We're on AG Grid v31.2.0 and want to upgrade to v35.
Scan the codebase and handle the migration.
```

The migrator detects your current version, maps every breaking change across versions, runs the official codemod, and handles manual fixes the codemod misses (theming, module registration, API renames).

### Code review

```
/ag-grid-review

Review the AG Grid code in src/components/DataGrid.tsx
```

Reviews against 60+ checks: missing `getRowId`, unstable references, type safety gaps, SSRM error handling, accessibility issues, and version-specific deprecations.

### Ask the expert

For anything else — generating components, column definitions, tests, debugging, architecture decisions — just ask naturally:

```
Create a server-side row model grid for our trade blotter.
500K+ rows, grouped by trader and symbol, with Excel export.
```

```
My grid loses selection when new data comes in. The component is at
src/components/OrderGrid.tsx — can you diagnose and fix?
```

```
Generate Playwright E2E tests for the grid at src/pages/Dashboard.tsx
```

## What Knowledge Is Baked In

- **AG Grid official documentation** (v29-v35.2): Full API, module system, theming, server-side model, all grid options and events
- **Community insights**: Top 10 developer problems, Stack Overflow patterns, GitHub discussions
- **Performance patterns**: Selective module imports (43% bundle reduction), React re-render prevention, virtualization tuning, streaming data
- **Testing patterns**: RTL with AG Grid test IDs (v33+), jsdom polyfills, Playwright/Cypress helpers
- **Enterprise patterns**: Server-side row model, master/detail, clipboard, Excel export with styles, integrated charts
- **Migration knowledge**: Complete breaking change database v31→v35 with codemod support
- **TypeScript patterns**: Full generic typing for components, events, params, custom components
- **Cross-framework**: React, Angular, Vue, and vanilla JS awareness

## AG Grid Version Support

| Version | Support Level |
|---------|--------------|
| v35.x | Full (latest) |
| v34.x | Full |
| v33.x | Full (module architecture migration) |
| v32.x | Migration support (LTS available) |
| v31.x | Migration support |
| v29-v30 | Basic migration support |

## Contributing

Contributions welcome! If you find a pattern or gotcha not covered:

1. Fork this repo
2. Add to the relevant skill or agent markdown file
3. Open a PR with a description of the scenario

## License

MIT License — see [LICENSE](LICENSE) for details.
