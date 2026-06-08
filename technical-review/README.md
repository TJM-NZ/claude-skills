# Technical Review

Comprehensive technical code review skill that identifies design flaws, code quality issues, error handling gaps, and architectural problems.

## What It Does

Performs systematic code review from a senior engineering perspective, finding real problems that would cause issues in production. Goes beyond style checking to identify:

- **Structure/Design Issues** - Unnecessary abstractions, premature generalization, poor naming
- **Code Quality** - Generic variable names, duplication, inconsistent conventions
- **Error Handling** - Silent failures, missing edge cases, weak retry logic
- **Testing Gaps** - Mock-heavy tests, missing error paths, meaningless assertions
- **AI-Generated Code Tells** - Defensive null checks, verbose comments, type system bypasses

## When to Use

- Before shipping a feature or codebase
- PR/code review requests
- Identifying technical debt
- Validating AI-assisted code quality
- Learning what "production-ready" means

## Approach

1. **Map the codebase** - Understand structure before diving in
2. **Read thoroughly** - No skimming, reads every file in scope
3. **Pattern detection** - Uses grep to find anti-patterns across codebase
4. **Severity classification** - BUG, DESIGN, SLOP, WEAK, STYLE
5. **Actionable fixes** - Specific file:line references with exact corrections

## Output Format

```
[SEVERITY] file.ext:line — What's wrong. Impact. Exact fix.
```

**Severities:**
- `BUG` - Causes incorrect behavior, data loss, or crashes
- `DESIGN` - Architectural mistakes that compound over time
- `SLOP` - AI-generated filler with no value
- `WEAK` - Handles easy cases, ignores hard ones
- `STYLE` - Inconsistencies that reduce readability

## Example Usage

```bash
/technical-review
```

Then select scope:
1. Specific directory (e.g., `src/components/`)
2. Full codebase review
3. Recent changes (git diff)

## Key Checks

- Functions longer than ~40 lines without justification
- Variables named `data`, `result`, `temp`, `value`
- Empty catch blocks or swallowed errors
- Tests that only assert mocks were called
- Comments that restate the code
- `any`, `@ts-ignore`, `eslint-disable` without explanation
- Copy-paste duplication across files

## Philosophy

Code must earn a pass, not the other way around. Reviews are blunt and precise - finds what's actually wrong and states exactly what to do instead. No hedging, no softening, just actionable technical feedback.

## For Employers

Demonstrates:
- Deep understanding of code quality and maintainability
- Ability to identify production risks before they ship
- Knowledge of common pitfalls in AI-assisted development
- Systematic approach to technical debt
