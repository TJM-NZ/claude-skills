# Route Static/Dynamic Audit (Next.js App Router)

Identify routes silently forced into dynamic SSR that should be static or ISR. Every dynamic render costs CPU on every request — static/ISR routes pay once.

Argument hint: `<app/ or routes/ directory path>` (default: `./app`)

## force-dynamic Declarations

Grep: `export const dynamic.*=.*['"]force-dynamic['"]`

- Pattern: explicit opt-in to dynamic rendering — confirm it's actually needed
- Verify: does the route read per-request data (user session, live prices)? If not, remove the declaration
- Fix: remove `export const dynamic` and let Next.js infer, or set `'force-static'` / `revalidate`
- UX impact: none if data is not per-user; stale risk if it was dynamic for a reason

## cookies() / headers() in Page Components

Grep: `cookies\(\)|headers\(\)` in `page.tsx`, `page.js`, `layout.tsx`, `layout.js`

- Pattern: calling `cookies()` or `headers()` anywhere in a page/layout opts the entire route into dynamic rendering, even if only one field is read
- Verify: is the value actually used to personalise the rendered output, or just for an auth check?
- Fix (auth-only): move the auth check to middleware (`middleware.ts`) and redirect there; keep page static
- Fix (partial personalisation): fetch personalised data client-side after a static shell renders
- UX impact: minimal — static shell renders instantly; personalised content hydrates after

## searchParams Usage in Page Components

Grep: `searchParams` in `page.tsx`, `page.js` (prop destructuring or `props.searchParams`)

- Pattern: accessing `searchParams` in a Server Component forces the route dynamic
- Verify: are params used for filtering/pagination that genuinely varies per request?
- Fix (static filters): if the set of param values is finite and known, use `generateStaticParams` to pre-render each combination
- Fix (dynamic filters): keep dynamic but add `export const revalidate = 60` if underlying data changes infrequently
- UX impact: none for generateStaticParams path; minimal stale risk for ISR path

## noStore() Calls

Grep: `noStore\(\)` in page, layout, or data-fetching files

- Pattern: `unstable_noStore()` / `noStore()` opts the enclosing route out of the cache entirely
- Verify: is real-time data actually required, or was this added to fix a stale-data bug during development?
- Fix: remove `noStore()` and add explicit `revalidate` to the fetch or route instead
- UX impact: slight staleness risk (document the TTL); significant CPU saving under load

## fetch() Without Cache Config

Grep: `fetch\(` in `page.tsx`, `page.js`, `layout.tsx`, `layout.js`, `actions.ts` — exclude `cache:`, `next:` options

- Pattern: bare `fetch()` in App Router defaults to `no-store` in some Next.js versions, forcing dynamic; in others defaults to full cache with no revalidation — both are footguns
- Verify: read the fetch call context; check Next.js version (`package.json`) for default behaviour
- Fix: always set explicit cache intent — `fetch(url, { next: { revalidate: 3600 } })` for ISR or `fetch(url, { cache: 'force-cache' })` for static
- UX impact: none (makes existing implicit behaviour explicit)

## Auth / Session Reads at Page Level

Grep: `getSession\(|getServerSession\(|auth\(\)|verifyToken\(|jwt\.verify\(` in `page.tsx`, `page.js`

- Pattern: reading session/token in every page component forces dynamic rendering and duplicates auth work per route
- Fix: centralise auth in `middleware.ts` (redirect unauthenticated) + one root `layout.tsx` session fetch passed via context; individual pages stay static
- UX impact: none — auth behaviour identical; CPU saving proportional to number of protected routes

## Missing generateStaticParams

Grep: `\[.*\]` in directory names under app/ (dynamic segments) — then read each `page.tsx` to check for `generateStaticParams` export

- Pattern: dynamic route segments (`[id]`, `[slug]`) with a finite, known set of values that are never pre-rendered
- Verify: is the full set of params knowable at build time (product slugs, blog posts, user-facing IDs from a CMS)?
- Fix: export `generateStaticParams` returning all known param combinations; set `dynamicParams = false` if exhaustive
- UX impact: positive — pre-rendered pages serve from CDN with zero SSR CPU cost
