# Workflow Audit

App workflow analysis — finds tangled user flows, data waterfalls, stale state, logic in the wrong layer, and dead/redundant paths.

## What It Does

Reviews how users and data move through the app, not code quality or performance. Finds:

- **Routing** — multi-hop redirects, dead routes, guard logic duplicated across pages, forms losing state
- **Data Flow** — waterfall fetches, siblings re-fetching the same data, client-filtering what the server could return, mutations with no cache invalidation
- **State** — global state for local concerns, multiple sources of truth, stale derived state
- **Layering** — business logic in UI components, fetch calls in event handlers, duplicate transform pipelines
- **Redundancy** — two implementations coexisting, orphaned feature flags, duplicate data paths

## When to Use

- "This flow feels clunky" — routing or navigation complexity
- "Why is this page slow to load?" — data fetching patterns
- "Something feels redundant" — duplicate paths or logic
- Before refactoring a feature area
- After the app has grown and no one's looked at the big picture

## Severities

- `TANGLE` — unnecessarily complex; will trap future changes
- `WATERFALL` — sequential async with no dependency reason
- `STALE` — data that can drift with no recovery path
- `LEAK` — logic in the wrong layer
- `DEAD` — unreachable route, orphaned state, unused flow

## Output

```
[SEVERITY] file:line — what's wrong. what compounds. exactly what to do.
```

Blunt. No hedging. Names the route, quotes the fetch, shows the fix.
