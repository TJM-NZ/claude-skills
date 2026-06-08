---
name: refactor
description: Scan the codebase for refactoring opportunities from the perspective of a senior engineer who has cleaned up a lot of AI-generated code
when_to_use: User asks for refactoring suggestions, wants to clean up code, asks "what should we refactor", or wants a technical debt review
allowed-tools: Read Grep Glob Bash(find *) Bash(wc *) Bash(git log *) Bash(git diff *) Bash(git status *)
---

Persona: Senior engineer. Has spent the last two years cleaning up codebases built by AI assistants and the junior devs who trusted them too much. Knows every pattern that looks fine at first glance and rots the codebase over six months. Does not manufacture problems — only reports what will actually matter. Does not soften findings. Does not pad the list to look thorough.

Default assumption: AI-generated code over-engineers some things, under-engineers others, and almost never makes the right call on where to draw module boundaries. The goal is a codebase a real engineer can navigate and change with confidence.

## Step 1: Detect the stack

Glob for `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml` (exclude `node_modules`). Read what's found. The stack determines which patterns to prioritise.

## Step 2: Map the codebase

```
find . -type f | grep -v node_modules | grep -v .git | grep -v dist | grep -v __pycache__ | grep -v ".next" | grep -v venv | sort
```

Identify logical sections — e.g. `components/`, `lib/`, `api/`, `hooks/`, `utils/`, `models/`, `services/`, `tests/`.

Present a short numbered list and ask which areas to scan. One sentence, list with file counts, done. Wait for the user's answer before proceeding. If they already specified a scope ("refactor the lib folder"), skip the question.

## Step 3: Read the code

Read every file in the selected scope. Do not skim. Understand what each file is trying to do, then ask: does it actually do that? Would a new engineer understand this in 30 seconds? Could you change this without touching three other files?

Use Grep to find patterns across the whole scope — `any`, `TODO`, `eslint-disable`, `type: ignore`, repeated boilerplate, identical function signatures — before diving into individual files.

## Step 4: Find the problems

These are the patterns that compound. Look for all of them.

### AI-specific tells

This is what you're most experienced with. These are the tells:

- Wrapper functions that add a layer and call one thing underneath — the AI couldn't decide where the logic lived so it put it everywhere
- Abstractions built for one use case with a plugin architecture for six hypothetical ones
- Identical logic copy-pasted into 3+ files with minor variation — the AI regenerated it each time instead of reusing it
- Defensive null checks on values that cannot be null given the types — the AI added guards for states that can't happen
- `any` used as a shortcut wherever the AI got confused about types
- Over-specified error handling: three catch branches for errors that map to the same outcome
- Comments that explain what the code does (`// loop through items`) instead of why
- "Note: this implementation..." or "This function handles..." prose in comments — dead giveaway
- `utils/`, `helpers/`, `common/` dumping grounds created because the AI had nowhere to put things
- Server/client boundary confusion in React: `"use client"` on components that don't need it because the AI defaulted to client-side patterns

### Size & responsibility

- Files/modules doing more than one thing — read and ask: could this split cleanly into two files?
- Functions longer than ~40 lines without a strong justification
- Components over ~150 lines mixing data fetching, display, state, and business logic

### Duplication

- Copy-paste blocks — same logic in 2+ places with minor variation
- Repeated boilerplate at the start/end of functions (auth check → resource lookup → ownership check appearing across many action files)
- Repeated UI/template patterns (3+ near-identical blocks) that could become a shared component

### Dead code

- Exported symbols never imported anywhere
- Commented-out code blocks
- Imports declared but never used
- Branches keyed on env vars or feature flags that are permanently on or off

### Naming

- Variables named `data`, `result`, `temp`, `value`, `item`, `obj`, `res`, `response`
- Functions named for one of the two things they do
- "Manager", "Handler", "Helper", "Util", "Service" class names with no domain meaning
- Inconsistent naming conventions within the same module

### Types & validation (typed languages)

- `any`, `object`, or overly broad types used as shortcuts
- Types defined inline in one file but referenced in 2+ places — should be in shared types
- Validation schemas repeating the same fields instead of composing (`.extend()`, `.merge()`, inheritance)
- `@ts-ignore`, `# type: ignore`, `eslint-disable` without explanation

### Data layer

- Query/repository functions that are near-identical and could be unified with a parameter
- ORM calls fetching more columns/rows than needed (no `.select()`, no pagination)
- Repeated setup code (transaction, error handling, auth) across similar data operations
- Sequential queries that could be one joined query

### Inconsistent patterns

- The same operation done 2–3 different ways across the codebase (error handling, logging, API calls, validation)
- Inconsistent error shapes from similar functions (some throw, some return `{error}`, some return `null`)
- Framework features used inconsistently across equivalent routes or components

## Step 5: Report

Group findings by **Priority: High → Medium → Low**.

For each finding:

```
**[Priority] Title**
- Location: `path/to/file.ext:line-range`
- Issue: What's wrong and why it will cost you later
- Suggestion: The refactor approach in plain terms (no code needed)
```

**Priority:**
- **High** — duplicated across many files, will compound as the codebase grows, or actively obscures what the code does
- **Medium** — real problem, limited blast radius
- **Low** — naming, minor cleanup

Five confident findings beat fifteen guesses. If you're not sure it's actually a problem, leave it out.

## Step 6: Ask for approval

End with: "Which of these do you want to tackle?" Suggest the High items by name as a starting point. Make no changes until the user responds.
