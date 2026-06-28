# Rendering CPU Patterns

Find expensive render cycles and wrong rendering strategy choices.

## Unnecessary Re-Renders (React)

Grep: `onClick=\{.*=>|onChange=\{.*=>|onSubmit=\{.*=>`

- Pattern: inline arrow functions in JSX props — new function reference every render triggers child re-render
- Fix: extract to named handlers or `useCallback`

## Missing React.memo on Stable Components

Grep: `export default function|export const.*=.*\(` in component files

- Pattern: components that receive stable props but re-render on every parent render
- Verify: check if props are actually stable (primitives or memoized objects)
- Fix: wrap with `React.memo()`

## Heavy Computation in Render Body

Grep: `\.sort\(|\.filter\(|\.reduce\(|\.map\(` inside React component functions (not in event handlers)

- Pattern: expensive array operations recalculated every render
- Fix: `useMemo` with correct dependencies

## Wrong Rendering Strategy (Next.js / SSR frameworks)

Read: page/route files — check for `getServerSideProps`, dynamic rendering config

- Pattern: pages using SSR (CPU on every request) for data that rarely changes
- Fix: switch to ISR (`revalidate`) or SSG for stable content; use SSR only for per-user dynamic data
- UX impact: minimal — first load may use slightly stale data (document TTL clearly); subsequent loads faster

## Over-Serialisation

Grep: `JSON\.stringify\(|JSON\.parse\(` in SSR/RSC data passing

- Pattern: large objects serialised to pass from server to client, including fields never used client-side
- Fix: select only required fields before serialising; use typed DTOs
- UX impact: positive (smaller HTML, faster hydration)

## Waterfall Data Fetching in Components

Grep (React): `useEffect\(.*fetch|useEffect\(.*axios` nested inside components that themselves wait on data

- Pattern: child component initiates fetch only after parent resolves — serial waterfall
- Fix: lift fetches to parent or use parallel data-loading patterns (`Promise.all`, Suspense with parallel loaders)
- UX impact: positive — content appears sooner

## Unvirtualised Long Lists

Grep: `\.map\(` rendering lists — check if result is a long list of DOM nodes

- Pattern: rendering 100+ items into DOM when only ~10 are visible
- Fix: virtualise with `react-virtual`, `@tanstack/virtual`, or `react-window`
- UX impact: moderate — scroll behaviour changes slightly; test before shipping
