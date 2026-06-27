# Async & Event Loop Patterns

Find blocking operations and inefficient async that waste CPU or stall the event loop.

## Synchronous I/O

Grep: `readFileSync|writeFileSync|execSync|spawnSync|appendFileSync`

- Pattern: sync I/O in request handlers or server startup paths
- Fix: async equivalents (`readFile`, `writeFile`, `exec`) or read once at startup into memory
- UX impact: none (same result, non-blocking)

## Sequential Awaits (Parallelisable)

Grep: `await.*\n.*await|const.*=.*await.*\n.*const.*=.*await`

- Pattern: multiple independent `await` calls in sequence — each waits before starting next
- Verify: are the calls truly independent (no data dependency)?
- Fix: `Promise.all([a(), b(), c()])` runs all concurrently
- UX impact: none — page loads faster or same speed

## CPU-Bound Work on Main Thread

Grep: `while\s*\(|for\s*\(.*;\s*i\s*[<>].*;\s*i[++]` in server-side files

- Pattern: heavy loops processing large datasets synchronously in request handlers
- Fix: offload to `worker_threads` (Node) or process in chunks with `setImmediate` between batches
- UX impact: none if chunked correctly; moderate if streaming is removed (assess per case)

## Unthrottled Event Listeners

Grep: `addEventListener\(|on\('data'|\.on\('message'`

- Pattern: event handlers that trigger heavy computation on every tick (scroll, resize, data stream)
- Fix: debounce/throttle the handler, or process in batches
- UX impact: minimal — debounce delay is imperceptible if set to ≤16ms for visual events

## Missing Streaming for Large Responses

Grep: `res\.json\(|res\.send\(` in files handling large datasets

- Read files: check if full dataset is collected then sent vs streamed
- Pattern: building entire response in memory before sending (delays first byte, spikes CPU)
- Fix: stream response using `res.write()` / ReadableStream / `json-stream-stringify`
- UX impact: minimal to positive — TTFB improves or stays same

## Polling Instead of Push

Grep: `setInterval\(.*fetch|setInterval\(.*axios|setInterval\(.*request`

- Pattern: repeated HTTP calls on interval to check for updates
- Fix: WebSocket, SSE, or webhook depending on stack
- UX impact: none — same data, lower CPU overhead
