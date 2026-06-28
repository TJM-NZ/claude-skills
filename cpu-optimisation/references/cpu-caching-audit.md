# HTTP & CDN Caching Audit

Identify missing or misconfigured caching that causes unnecessary function invocations. Every cache miss that could have been a hit wastes CPU and adds latency.

Argument hint: `<api routes directory path>` (default: `./app/api`)

## API Routes Without Cache-Control Headers

Grep: `export.*GET|export.*POST|export default.*handler` in route handler files — then read each to check for `Cache-Control` header set on the response

- Pattern: route handlers returning dynamic-looking but actually stable data with no cache directive — every client request hits the function
- Verify: is the response truly per-user/per-request, or is it the same for all callers?
- Fix (public stable data): `response.headers.set('Cache-Control', 's-maxage=3600, stale-while-revalidate=86400')`
- Fix (Next.js route handler): `return NextResponse.json(data, { headers: { 'Cache-Control': 's-maxage=3600' } })`
- UX impact: positive — CDN serves subsequent requests with zero function invocation

## Unnecessary cache: 'no-store' on fetch()

Grep: `cache.*:.*['"]no-store['"]` in page, layout, and data-fetching files

- Pattern: `no-store` added to fix a stale-data bug during development and never revisited; disables all caching including CDN
- Verify: does this data genuinely change between requests for the same user, or between users?
- Fix: replace with `{ next: { revalidate: N } }` where N matches actual data volatility; use `no-store` only for truly per-request personalised data
- UX impact: none if revalidate window matches data change frequency

## ISR Revalidate Values Set Too Low

Grep: `revalidate.*=.*[0-9]+` in page files and `next: { revalidate: [0-9]+ }` in fetch calls

- Pattern: `revalidate: 1` or similarly low values set during debugging and never tuned — causes near-constant revalidation, defeating ISR
- Verify: how often does the underlying data actually change? Check CMS webhooks, DB write frequency, or API update cadence
- Fix: set revalidate to match real data volatility (e.g. blog posts: 3600, product prices: 300, live scores: 30); use on-demand revalidation (`revalidatePath`, `revalidateTag`) for event-driven freshness instead
- UX impact: slight staleness risk within the window — document the TTL

## CDN Cache Headers Absent on Public Non-Personalised Responses

Grep: `export.*GET` in api route files — read each to check if response varies per user (auth token, session, user ID used in query)

- Pattern: public endpoints (product listings, blog content, static config) served with default no-cache behaviour because headers were never set
- Verify: confirm no `Authorization`, cookie-based personalisation, or user-specific query params affect the response
- Fix: add `Cache-Control: public, s-maxage=3600, stale-while-revalidate=86400` and `Vary: Accept-Encoding` (not `Vary: Cookie` or `Vary: Authorization`)
- UX impact: positive globally — CDN cache hit rate increases, function invocations drop

## stale-while-revalidate Opportunities

Grep: `s-maxage|max-age` in route handlers and next.config — check for absence of `stale-while-revalidate`

- Pattern: cache headers set with `s-maxage` only — when cache expires, all concurrent requests miss simultaneously (thundering herd) and each triggers a function invocation
- Fix: always pair with `stale-while-revalidate`: `Cache-Control: s-maxage=60, stale-while-revalidate=3600` — serves stale instantly while one background revalidation runs
- UX impact: positive — eliminates cache-expiry latency spikes for users

## x-vercel-cache: MISS Patterns (Vercel deployments)

Read: `next.config.js` — check for `headers()` config. Grep: `x-vercel-cache` in any logging/monitoring setup

- Pattern: routes expected to be cached are consistently missing (`x-vercel-cache: MISS` or `BYPASS`) due to misconfigured headers, dynamic rendering, or cookies triggering personalisation
- Verify: add `console.log` or check Vercel dashboard → Functions tab for invocation volume on routes that should be static
- Common causes: `Set-Cookie` on a cacheable response bypasses CDN; `Authorization` header present; route opted into dynamic rendering (see `cpu-route-audit.md`)
- Fix: strip `Set-Cookie` from cacheable responses; move auth to middleware; confirm `x-vercel-cache: HIT` in response headers after fix
- UX impact: none — cache behaviour becomes what was intended
