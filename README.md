# AG Grid Skills & Agents for Claude Code

Production-ready [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills and agents for [AG Grid](https://www.ag-grid.com/) development. Built from insights across official documentation, community best practices, and real-world production patterns.

Covers AG Grid **v29 through v35+**, with emphasis on the modern module architecture (v33+).

## What's Included

### Skills (Slash Commands)

| Skill | Command | Description |
|-------|---------|-------------|
| **Generate** | `/ag-grid-generate` | Scaffold a fully typed AG Grid component with module registration, proper React hooks, and best practices |
| **Column Defs** | `/ag-grid-coldef` | Generate typed column definitions from TypeScript interfaces or JSON data. Auto-detects currency, date, status fields |
| **Optimize** | `/ag-grid-optimize` | Audit your AG Grid implementation for performance issues: bundle size, re-renders, virtualization, memory leaks |
| **Migrate** | `/ag-grid-migrate` | Migrate between AG Grid versions. Detects breaking changes, deprecated APIs, module system updates |
| **Test** | `/ag-grid-test` | Generate tests for AG Grid components. Supports RTL, Playwright, Cypress, with virtualization-aware patterns |
| **Troubleshoot** | `/ag-grid-troubleshoot` | Diagnose common AG Grid issues: blank grids, state loss, module errors, performance problems, styling conflicts |

### Agents (Autonomous Specialists)

| Agent | Description |
|-------|-------------|
| **ag-grid-expert** | Deep AG Grid knowledge agent for architecture advice, API reference, feature comparison, and implementation guidance |
| **ag-grid-review** | AG Grid code review agent with 60+ checks across performance, type safety, accessibility, and common bugs |

## Installation

### Option 1: Clone into your project (recommended)

```bash
# From your project root
git clone https://github.com/rubric-dev/ag-grid-agent.git .claude/ag-grid

# Or add as a git submodule
git submodule add https://github.com/rubric-dev/ag-grid-agent.git .claude/ag-grid
```

Then reference in your project's `.claude/settings.json`:

```json
{
  "skills": [
    ".claude/ag-grid/.claude/skills/ag-grid-generate.md",
    ".claude/ag-grid/.claude/skills/ag-grid-coldef.md",
    ".claude/ag-grid/.claude/skills/ag-grid-optimize.md",
    ".claude/ag-grid/.claude/skills/ag-grid-migrate.md",
    ".claude/ag-grid/.claude/skills/ag-grid-test.md",
    ".claude/ag-grid/.claude/skills/ag-grid-troubleshoot.md"
  ],
  "agents": [
    ".claude/ag-grid/.claude/agents/ag-grid-expert.md",
    ".claude/ag-grid/.claude/agents/ag-grid-review.md"
  ]
}
```

### Option 2: Copy individual files

Copy the files you need from `.claude/skills/` and `.claude/agents/` into your project's `.claude/` directory.

### Option 3: Reference the CLAUDE.md

Copy the `CLAUDE.md` file to your project root to give Claude Code AG Grid context without installing skills/agents:

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/rubric-dev/ag-grid-agent/develop/CLAUDE.md
```

## Usage Examples

### Generate a new grid component

```
/ag-grid-generate

Create a data grid for our user management page.
- TypeScript interface: User { id, name, email, role, department, createdAt, active }
- Server-side row model (we have 500K+ users)
- Row grouping by department
- Excel export
- Set filter on role and department columns
```

### Generate column definitions

```
/ag-grid-coldef

Generate column definitions for this interface:
interface Trade {
  tradeId: string;
  symbol: string;
  quantity: number;
  price: number;
  totalValue: number;
  tradeDate: string;
  status: 'pending' | 'executed' | 'cancelled';
  trader: string;
}
```

### Optimize an existing grid

```
/ag-grid-optimize

Our main dashboard grid is slow. The page takes 3 seconds to load
and scrolling is laggy. Please audit and fix.
```

### Migrate to the latest version

```
/ag-grid-migrate

We're on AG Grid v31.2.0 and want to upgrade to v35.
Please scan the codebase and handle the migration.
```

### Generate tests

```
/ag-grid-test

Generate tests for the TradeGrid component at src/components/TradeGrid.tsx.
We use React Testing Library and Playwright.
```

### Troubleshoot an issue

```
/ag-grid-troubleshoot

Our grid loses the selected rows whenever new data comes in from the API.
The grid is at src/components/OrderGrid.tsx.
```

## What Knowledge Is Baked In

These skills and agents encode knowledge from:

- **AG Grid official documentation** (v29-v35.2): API reference, module system, theming, server-side model, all grid options and events
- **Community insights**: Top 10 most common problems, Stack Overflow patterns, GitHub discussions, blog posts
- **Performance patterns**: Bundle optimization (43% reduction with selective modules), React re-render prevention, virtualization tuning, streaming data handling
- **Testing patterns**: RTL with AG Grid test IDs (v33+), jsdom polyfills, Playwright helpers, virtualization-aware assertions
- **Enterprise patterns**: Server-side row model, master/detail, clipboard, Excel export with styles, integrated charts
- **Migration knowledge**: Complete breaking change database for v31, v32, v33, v34, v35 including codemod support
- **TypeScript patterns**: Full generic typing for components, events, params, and custom components
- **Cross-framework awareness**: React, Angular, Vue, and vanilla JS patterns

## AG Grid Version Support

| Version | Support Level |
|---------|--------------|
| v35.x | Full support (latest) |
| v34.x | Full support |
| v33.x | Full support (module architecture migration) |
| v32.x | Migration support (LTS available) |
| v31.x | Migration support |
| v29-v30 | Basic migration support |

## Contributing

Contributions welcome! If you find a pattern or gotcha not covered:

1. Fork this repo
2. Add to the relevant skill or agent file
3. Open a PR with a description of the scenario

## License

MIT License - see [LICENSE](LICENSE) for details.
