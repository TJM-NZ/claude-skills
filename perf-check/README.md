# Performance Check

Code performance analysis that identifies bottlenecks, N+1 queries, missing database indexes, inefficient algorithms, bundle bloat, poor caching, and unnecessary re-renders.

## What It Does

Systematic performance review focusing on real bottlenecks that impact production:

- **Database Performance** - N+1 queries, missing indexes, unoptimized queries, connection pooling
- **Algorithm Efficiency** - O(n²) where O(n) exists, unnecessary iterations, inefficient data structures
- **Caching Issues** - Missing cache layers, cache invalidation problems, over-caching
- **Bundle Size** - Unused dependencies, large libraries, missing code splitting, unoptimized images
- **React/UI Performance** - Unnecessary re-renders, missing memoization, virtual list needs
- **API/Network** - Waterfall requests, missing pagination, over-fetching, missing compression
- **Memory Leaks** - Event listener cleanup, closure issues, growing collections
- **Async/Concurrency** - Serial where parallel possible, promise anti-patterns, blocking operations

## When to Use

- Application feels slow or unresponsive
- Database queries timing out
- High server costs/resource usage
- Large bundle sizes affecting load time
- UI lag or jank during interactions
- Scaling issues as data grows
- Before optimization (establish baseline)

## Approach

1. **Identify performance context** - Web app, API, data processing, ML inference
2. **Profile current state** - Understand what's actually slow (not guessing)
3. **Find bottlenecks** - Database, compute, network, rendering
4. **Measure impact** - Quantify performance cost (ms, queries, MB, renders)
5. **Provide optimizations** - Specific fixes with expected improvements

## Output Format

```
**[SEVERITY] Performance Issue**
- Category: [db/algorithm/cache/bundle/ui/network/memory/async]
- Location: `file:line`
- Impact: [+X ms per request / +Y MB bundle / Z extra queries]
- Evidence: [current implementation]
- Bottleneck: [specific problem]
- Fix: [optimized code]
- Expected gain: [performance improvement estimate]
```

## Severity Ratings

- **Critical** - 10x+ performance impact, blocks scaling, causes timeouts
- **High** - 3-10x impact, noticeable user slowdown, high resource cost
- **Medium** - 1.5-3x impact, minor delays, optimization opportunity
- **Low** - <1.5x impact, edge case optimization, future-proofing

## Example Usage

```bash
/perf-check
```

Then specify scope:
1. Full application performance audit
2. Database/API focus
3. Frontend/bundle size focus
4. Specific component or feature

## Common Performance Issues Detected

### Database
- N+1 query patterns (loading relations in loops)
- Missing indexes on frequently queried columns
- SELECT * instead of specific columns
- Missing query result pagination
- Connection pool exhaustion

### Algorithms
- Nested loops creating O(n²) complexity
- Sorting inside loops
- Repeated calculations that could be cached
- Wrong data structure choice (array vs Set/Map)

### React/UI
- Missing React.memo for expensive components
- useEffect dependencies causing infinite loops
- Massive lists without virtualization
- Unnecessary context re-renders
- Large state objects causing re-renders

### Bundle/Network
- Importing entire libraries for one function
- Missing lazy loading/code splitting
- Uncompressed assets
- No CDN for static assets
- Missing HTTP/2 multiplexing

## Not Included

This is performance-focused, not general refactoring:
- Code style issues (use /technical-review)
- Security vulnerabilities (use /security-audit)
- Best practices that don't affect performance

## For Employers

Demonstrates:
- Understanding of performance bottlenecks across stack
- Database optimization knowledge (indexes, N+1 prevention)
- Algorithm complexity analysis (Big O)
- Frontend performance optimization (bundle size, rendering)
- Data-driven optimization approach (measure, don't guess)
- Cost optimization awareness (resource usage impacts budget)
