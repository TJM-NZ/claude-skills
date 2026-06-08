---
name: perf-check
description: Code performance analysis - finds bottlenecks, N+1 queries, missing indexes, inefficient algorithms, bundle bloat, poor caching, unnecessary re-renders. Not general refactoring.
when_to_use: User mentions "slow", "performance", "optimize", "bottleneck", "speed up", "runs slow", "performance improvement"
allowed-tools: Read Grep Glob Bash(git rev-parse *) Bash(git status *) Bash(npm run *)
---

Performance optimization expert. Find real bottlenecks with measurable impact, not micro-optimizations.

## Workflow

1. **Detect stack**: Glob for `package.json`, `requirements.txt`, `go.mod`. Read to identify framework/ORM
2. **Ask scope** if unclear: Full analysis | Quick scan (DB + frontend + bundle) | Targeted (specific area)
3. **Analyze** using patterns below
4. **Report** findings with severity, evidence, fixes, expected improvement

## Search Patterns

### Critical (causes major slowdown, poor scaling)

**N+1 Queries** (highest priority):
- Grep: `for.*await.*find|map.*await.*query|\.map\(.*=>.*\.findMany|\.map\(.*=>.*\.query`
- Pattern: Loops containing DB queries
- Fix: Single query with JOIN or `whereIn`

**Missing DB Indexes**:
- Read schema files for foreign keys, common WHERE/ORDER BY columns
- Pattern: `@relation`, `references:`, `foreignKey` without corresponding index
- Fix: Add indexes

**Nested Loops (O(n²)+)**:
- Grep: `for.*for|\.map\(.*\.map\(|\.map\(.*\.filter\(|\.map\(.*\.find\(`
- Pattern: Nested iterations in hot paths
- Fix: Use Map/Set for O(1) lookups, combine operations

### High Impact

**Frontend - Unnecessary Re-renders**:
- Grep: `onClick=\{.*=>|onChange=\{.*=>|useState.*\[\]|useContext` (in large components)
- Pattern: Inline functions in props, missing memo/useMemo/useCallback
- Fix: Memoization, extract handlers

**Bundle Size**:
- Read `package.json` for: `moment`, `lodash`, `date-fns` (full imports)
- Grep: `import \* as|import.*lodash|import.*moment|import.*antd[^/]`
- Fix: Tree-shaking imports, lighter alternatives, dynamic imports

**Missing Caching**:
- Grep: Expensive operations without cache (repeated calculations, API calls)
- Pattern: Same operation in loops or frequent code paths
- Fix: useMemo, React Query, Redis, memoization

**Client vs Server Components** (Next.js):
- Grep: `"use client"` in files that don't need interactivity
- Pattern: Client components for static content or data fetching
- Fix: Remove "use client", use Server Components

**Blocking Operations**:
- Grep: `readFileSync|writeFileSync|JSON\.parse\([^)]{100,}`
- Pattern: Sync I/O or CPU-intensive work in request handlers
- Fix: Async equivalents, worker threads

### Medium Impact

**Inefficient Queries**:
- Grep: `SELECT \*|findMany\(\{[^}]*\}\)|\.find\(\)`
- Pattern: Fetching all columns/rows without select/limit
- Fix: Select specific columns, add pagination

**Unoptimized Images** (Next.js):
- Grep: `<img` in `.tsx?` files
- Fix: Use `next/image` with optimization

**Array Lookups Instead of Maps**:
- Pattern: `array.find(x => x.id === id)` in loops or frequent code
- Fix: Convert to Map for O(1) lookup

**Missing Lazy Loading**:
- Check for heavy components loaded upfront without dynamic imports
- Fix: `const Heavy = dynamic(() => import('./Heavy'))`

**Sequential Async Operations**:
- Pattern: Multiple `await fetch()` in sequence that could be parallel
- Fix: `Promise.all()` for independent operations

### Database/ORM Specific

**Drizzle/Prisma**:
- Missing `.select()` → selecting unnecessary columns
- N+1 from `.include()` or relations in loops
- Missing connection pooling config

**Schema**:
- Foreign keys without indexes
- Columns in WHERE/ORDER BY without indexes
- Over-indexing (too many indexes on small tables)

## Severity Rating

- **Critical**: >1s delay, O(n²) in hot path, N+1 on growing data
- **High**: 100-1000ms impact, noticeable to users, will scale poorly
- **Medium**: 10-100ms impact, poor practice
- **Low**: <10ms, micro-optimization

## Report Format

```
# Performance Analysis - [Project Name]

**Stack**: [Framework, ORM, DB]
**Findings**: Critical: X | High: Y | Medium: Z | Low: W

## Critical Issues

**[SEVERITY] Title**
- Location: `file.ts:123`
- Issue: [what's slow]
- Impact: [user-facing: "2s page load" or scaling: "O(n²) with 1000+ items"]
- Evidence:
  ```ts
  // Current slow code
  ```
- Fix:
  ```ts
  // Optimized code
  ```
- Expected: [O(n²)→O(n) | 500KB→50KB | 200ms→20ms]

[Repeat for each finding, grouped by severity]

## Quick Wins
1. [Fix] - Impact: [metric] - Effort: small
2. ...

## Roadmap
Priority 1 (Immediate): [list]
Priority 2 (Next): [list]
Priority 3 (Future): [list]
```

## Key Principles

- Focus on hot paths (frequent code, user-facing)
- Database queries are usually #1 bottleneck
- Measure/estimate impact, not theoretical issues
- No findings in test files unless they indicate production patterns
- Provide code examples for every fix
- Group similar issues (e.g., "5 N+1 queries" not 5 separate findings)

## After Report

Offer: "Fix Critical/High issues?" | "Set up bundle analyzer?" | "Add performance monitoring?"
