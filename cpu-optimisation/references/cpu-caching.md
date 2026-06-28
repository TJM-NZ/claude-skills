# Caching & Memoization

Find repeated computation on identical inputs that should be cached.

## Expensive Computations Without Memoization

Grep: `useMemo\|useCallback` — check for their *absence* in components with heavy logic

- Pattern: expensive derivations (`filter` + `sort` + `map` on large arrays) recalculated every render
- Fix: `useMemo(() => expensiveOp(data), [data])` — recomputes only when deps change

## Repeated DB / API Calls for Same Data

Grep: look for identical query calls across multiple handlers or in loops

- Pattern: same data fetched multiple times per request cycle without caching
- Fix: in-memory cache (`Map` with TTL), Redis, or request-scoped dataloader pattern

## Missing HTTP Response Caching

Read: `next.config.js`, route handlers, Express middleware, or equivalent

- Pattern: no `Cache-Control` headers on responses for data that changes infrequently
- Fix: set `Cache-Control: s-maxage=X, stale-while-revalidate` or equivalent
- UX impact: positive (faster repeat loads) — first load unchanged

## Function Results Not Memoized

Grep: `function.*\(|const.*=.*\(.*\).*=>` — look for pure functions called repeatedly with same args

- Pattern: pure functions called in loops or frequently with identical arguments
- Fix: `memoize()` (lodash or hand-rolled with `Map`) for functions with expensive bodies

## Re-Computing Derived State

Grep (React): `const.*=.*props\.|const.*=.*state\.` inside render/component body

- Pattern: derived values computed from props/state on every render without useMemo
- Fix: `useMemo` for derived values, move static derivations outside the component

## Server-Side: No Caching on Stable Data

Read: API route handlers and data-fetching functions

- Pattern: fully dynamic responses for data that rarely changes (config, reference data, public content)
- Fix: cache at server level (in-process Map, Redis, CDN headers)
- UX impact: none to positive
