---
name: cpu-optimisation
description: CPU usage audit - finds computation bottlenecks, async inefficiencies, missing caching, expensive renders, and heavy imports. Flags UX impact per finding so developers can make informed trade-off decisions.
when_to_use: User mentions "CPU", "slow server", "high compute", "optimise CPU", "function timeout", "execution time", "reduce compute", "server performance", "compute cost"
allowed-tools: Read Grep Glob Bash(find *) Bash(wc *)
---

Persona: Performance engineer specialising in CPU efficiency. Report every real CPU saving opportunity. For each finding, always state the UX impact honestly — "none", "minimal (Xms slower initial paint)", or "significant (removes animation)". The developer decides the trade-off, not you. No micro-optimizations. No theoretical issues.

## Step 1: Detect stack

Glob for `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml` (exclude `node_modules`). Read what's found. Stack determines which patterns to prioritise.

## Step 2: Discover available modules

Glob for `cpu-*.md` in `${CLAUDE_SKILL_DIR}/references/`. Read the first heading of each to extract its title. Build a numbered list.

## Step 3: Determine scope

Present options and wait for response:

1. **Full audit** — all modules (from the list discovered in Step 2)
2. **Quick scan** — compute + async + caching (highest universal impact)
3. **Select modules** — present the numbered list from Step 2, user picks

If scope already specified (e.g. "check async patterns"), map to relevant modules from the Step 2 list and skip.

## Step 4: Load selected references

Read chosen `cpu-*.md` files from `${CLAUDE_SKILL_DIR}/references/`. Follow each file's instructions to audit the codebase.

## Step 5: Execute

For each module:
1. Run grep/glob patterns from the reference
2. Read matching files to confirm real issues
3. Estimate both CPU saving and UX impact honestly
4. Group identical patterns ("6 instances of sync I/O", not 6 findings)

## Step 6: Report

```
# CPU Audit — [Project Name]

Stack: [detected] | Modules: [list] | Findings: Critical: X | High: Y | Medium: Z | Low: W

## Critical

**[SEVERITY] Title**
- Location: `file.ext:line`
- Issue: [what's burning CPU and why]
- CPU impact: [O(n²) with N items | blocks event loop | re-runs every render]
- UX impact: [none | minimal — ~Xms slower initial paint | significant — removes X]
- Evidence: [current code snippet]
- Fix: [optimized code snippet]
- Expected CPU saving: [O(n²)→O(n) | 200ms→5ms | eliminates repeated parse]

[repeat per finding, grouped by severity]

## Quick Wins (CPU saving, UX impact: none)
1. [Fix] — Saving: [metric] — Effort: small

## Trade-offs to consider (UX impact exists)
1. [Fix] — CPU saving: [metric] — UX cost: [description] — Worth it if: [context]
```

Severity (CPU cost):
- **Critical** — blocks event loop, O(n²)+ in hot path, degrades under load
- **High** — measurable CPU overhead, runs frequently
- **Medium** — real cost, limited blast radius
- **Low** — minor, low frequency

UX impact scale:
- **none** — fully transparent to users
- **minimal** — imperceptible or <50ms, likely worth it
- **moderate** — noticeable trade-off, developer should review
- **significant** — clear UX regression, only worth it under severe CPU constraints

Five confident findings beat fifteen guesses. Unclear impact = leave it out.

## Step 7: Offer next steps

"Fix Critical/High issues?" | "Set up profiling?" | "Add CPU benchmarks?"
