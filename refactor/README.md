# Refactor

Scans codebase for refactoring opportunities from the perspective of a senior engineer who has cleaned up AI-generated code. Identifies technical debt, unnecessary complexity, and maintainability issues.

## What It Does

Identifies opportunities to simplify, improve maintainability, and reduce technical debt:

- **Unnecessary Abstraction** - Layers that add no value, premature generalization
- **Code Duplication** - Similar logic in multiple places, copy-paste code
- **Naming Issues** - Generic names, misleading names, inconsistent conventions
- **Structural Problems** - God objects, tight coupling, missing separation of concerns
- **Dead Code** - Unused imports, commented code, unreachable branches
- **Over-Engineering** - Complex solutions to simple problems
- **Under-Engineering** - Missing abstractions where duplication is rampant

## When to Use

- Before adding major features (clean foundation)
- After rapid prototyping phase
- Onboarding new team members (reduce confusion)
- Technical debt review
- Post-AI-assisted development cleanup
- Improving codebase maintainability

## Approach

1. **Map codebase structure** - Identify modules, layers, patterns
2. **Detect patterns** - Find duplication, abstraction issues, naming problems
3. **Assess complexity** - Identify over/under-engineered areas
4. **Prioritize refactoring** - High-value changes first
5. **Provide specific refactorings** - Concrete before/after examples

## Output Format

```
**[PRIORITY] Refactoring Opportunity**
- Type: [abstraction/duplication/naming/structure/dead-code/complexity]
- Location: `file:line` (or multiple files)
- Problem: [why current approach is problematic]
- Current: [existing code]
- Refactored: [improved version]
- Benefit: [maintainability/readability/testability improvement]
- Effort: [S/M/L - hours to implement]
```

## Priority Levels

- **High** - Blocking feature work, causing bugs, confusing all developers
- **Medium** - Slowing development, technical debt accumulating
- **Low** - Nice-to-have, minor improvements, polish

## Example Usage

```bash
/refactor
```

Options:
1. Full codebase refactoring scan
2. Specific directory/module focus
3. Quick wins only (high value, low effort)

## Common Refactoring Opportunities

### Abstraction Issues
- Functions with one line that just call another function
- Classes with single method (should be function)
- Interfaces for only one implementation
- Utils/helpers dumping ground

### Duplication
- Copy-pasted blocks with minor variations
- Similar validation logic across forms
- Repeated data transformations
- Duplicated constants/configuration

### Naming
- Variables named `data`, `result`, `temp`, `obj`
- Class names like `Manager`, `Handler`, `Helper` (no domain meaning)
- Inconsistent naming conventions
- Misleading names (function says one thing, does another)

### Structural
- God objects doing too much
- Tight coupling (changing A breaks B, C, D)
- Missing domain boundaries
- Business logic in UI components

### Dead Code
- Commented-out code blocks
- Unused imports and exports
- Unreachable if/else branches
- Functions called nowhere

### Complexity
- Nested ternaries (hard to read)
- Boolean parameters (should be two functions)
- Functions doing multiple things
- Config systems for single config value

## Philosophy

Not all code needs refactoring. Focus on:
- Code that's actively changing (high churn)
- Code that's hard to understand (blocks new devs)
- Code that's causing bugs (complexity breeds errors)

Don't refactor:
- Code that works and never changes (leave it alone)
- For "cleanliness" sake (focus on value)
- Before understanding why it exists (might have good reason)

## For Employers

Demonstrates:
- Understanding of technical debt and maintainability
- Ability to balance refactoring with feature delivery
- Knowledge of common code smells
- Experience with AI-generated code cleanup
- Pragmatic approach (refactor what matters, not everything)
- Systematic thinking about code quality
