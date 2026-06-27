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
