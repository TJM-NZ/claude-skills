# Compute Patterns

Find CPU-intensive computation that scales poorly or runs in hot paths.

## Nested Iterations (O(n²)+)

Grep: `\.map\(.*\.map\(|\.map\(.*\.filter\(|\.map\(.*\.find\(|for.*for`

- Pattern: nested loops/maps over same or related datasets
- Worst case: `array.map(x => other.find(y => y.id === x.id))` — O(n²)
- Fix: build a `Map` from one array first, then O(1) lookup in the other
- UX impact: none (same output, faster)

## Regex in Hot Paths

Grep: `new RegExp\(|\.test\(|\.match\(|\.replace\(/`

- Pattern: `RegExp` constructed or `.test()`/`.replace()` called inside loops or render functions
- Fix: hoist regex to module scope as a `const` — compiled once, reused
- UX impact: none

## Heavy Parsing in Loops

Grep: `JSON\.parse\(|JSON\.stringify\(|parseInt\(|parseFloat\(`

- Pattern: `JSON.parse` / `JSON.stringify` inside `.map()`, `.forEach()`, or request handlers called frequently
- Fix: parse once, store result; avoid re-serialising data that hasn't changed
- UX impact: none

## Synchronous Crypto / Hashing

Grep: `crypto\.createHash|bcrypt\.hashSync|scryptSync|pbkdf2Sync`

- Pattern: sync hash/encrypt in request handlers
- Fix: use async variants (`bcrypt.hash`, `scrypt`, `pbkdf2`) or offload to worker thread
- UX impact: none (same result, non-blocking)

## Unnecessary Array Copies

Grep: `\[\.\.\.|\bArray\.from\(|\.slice\(\)`

- Pattern: spreading/copying large arrays when mutation isn't a risk
- Check: is the copy actually needed for immutability, or just defensive?
- Fix: operate on original when safe; use index access instead of copy
- UX impact: none

## Sorting Large Datasets

Grep: `\.sort\(`

- Pattern: `.sort()` inside render functions or called repeatedly on unchanged data
- Fix: sort once, memoize result; sort server-side before sending to client
- UX impact: none (unless sort order is dynamic — verify first)

## Large Payload JSON Parsing

Grep: `JSON\.parse\(` in request handlers and middleware — read context to estimate payload size (look for body/buffer reads, file reads, or external API responses)

- Pattern: `JSON.parse` on payloads likely exceeding 10KB (full DB rows, API responses, uploaded files) blocks the event loop for the full parse duration
- Verify: check what's being parsed — small config objects are fine; multi-KB arrays or nested documents are not
- Fix: stream-parse with a library (`stream-json`) for very large payloads; for API responses, select only required fields at the source rather than fetching and parsing everything
- UX impact: none (same data, non-blocking)

## Catastrophic Regex Backtracking

Grep: `new RegExp\(|\/.*[\+\*].*[\+\*]|\/.*\(.*\+\)` — look for nested quantifiers or alternation on unbounded input

- Pattern: regexes like `/(a+)+$/`, `/(x|y|z)*/`, or alternation with overlapping branches applied to user-supplied or large strings — exponential worst-case time
- Verify: test candidate regexes against a long string of near-matches (e.g. `'aaaaaaaaab'`) and measure time
- Fix: rewrite with possessive quantifiers or atomic groups; replace complex regexes with a parser; validate input length before matching
- UX impact: none for legitimate input; prevents request-path hangs on adversarial input

## Image Processing in the Request Path

Grep: `sharp\(|Jimp\.|Canvas\.|createCanvas\|\.resize\(|\.toBuffer\(|\.toFile\(` in route handlers and middleware

- Pattern: resizing, converting, or watermarking images synchronously in a request handler — CPU-intensive and blocks the event loop
- Fix: offload to a worker thread (`worker_threads`), a job queue (BullMQ, Inngest), or a dedicated image service (Cloudinary, Imgix, Vercel Image Optimization); cache the result
- UX impact: moderate if moved to async — response changes from synchronous to a job-poll or webhook pattern; worth it above trivial traffic

## Heavy String Manipulation Per Request

Grep: `\.replace\(.*\/g|\.split\(.*\)\.join\(|\.repeat\(|mustache|handlebars|ejs\.render\|nunjucks\.render` in request handlers

- Pattern: template rendering, repeated global string replacements, or large string assembly on every request for output that rarely changes
- Fix: render once and cache the result (in-memory Map, Redis, or CDN); use `stale-while-revalidate` if the template data changes periodically
- UX impact: none — cached output is identical

## Date / Intl / Locale Objects Reconstructed Per Request

Grep: `new Date\(|new Intl\.|Intl\.DateTimeFormat\(|Intl\.NumberFormat\(` inside request handlers and render functions (not at module scope)

- Pattern: `Intl.DateTimeFormat` and `Intl.NumberFormat` construction is expensive — locale data is loaded and compiled each time; placing them inside a handler means this cost is paid on every request
- Fix: hoist to module scope as a `const` and reuse: `const fmt = new Intl.DateTimeFormat('en-NZ', { ... })`; for per-locale needs, cache by locale key in a `Map`
- UX impact: none
