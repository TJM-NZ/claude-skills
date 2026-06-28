---
name: data-validation
description: Comprehensive data validation framework - schema validation, data quality checks, constraint validation, input sanitization, statistical validation, and business rules enforcement. Critical for finance and AI systems.
when_to_use: User mentions "data validation", "validate data", "data quality", "input validation", "schema validation", "data integrity", "validate input", "check data quality"
allowed-tools: Read Grep Glob Bash(git rev-parse *) Bash(git status *)
---

Data validation specialist — ensures data integrity, quality, and correctness across systems. Critical for finance (accuracy, compliance) and AI (model reliability, data quality).

## Approach

Systematic validation framework | Type-safe | Fail-fast | Actionable errors | Context-aware (finance vs AI vs general)

## Step 1: Understand Context

**Identify domain:**
- Finance: accuracy, compliance, audit trails, decimal precision
- AI/ML: data quality, outliers, feature validation, training data integrity
- General: type safety, business rules, input sanitization

**Read existing code:**
- Validation logic (if any)
- Data models/schemas
- Input sources (API, forms, file uploads, database)
- Error handling patterns

## Step 2: Discover Validation Types

Glob `validation-*.md` in the skill's `references/` directory → build validation checklist

## Step 3: Determine Scope

If user specified validation type → load that reference. Else offer:
1. **Full Validation Audit** - All validation types across codebase
2. **Input Validation** - Schema + format + sanitization
3. **Data Quality Check** - Quality + statistical + integrity
4. **Finance Validation** - Precision, compliance, audit requirements
5. **AI/ML Validation** - Quality + statistical + feature validation
6. **Custom** - User picks specific types

## Step 4: Load References

**Full audit**: Glob + Read all `validation-*.md` from references/
**Scoped**: Read specific references based on domain/scope
**Custom**: Map user request → filenames → Read

## Step 5: Execute Validation Review

Per reference file:
1. Read validation framework completely
2. Search codebase for validation logic (Grep/Glob/Read)
3. Identify gaps, anti-patterns, missing validations
4. Check error handling and user feedback
5. Verify fail-fast vs fail-safe approach matches domain needs

**Context-specific checks:**

**Finance:**
- Decimal precision (never use float for money)
- Range validation (negative amounts, date ranges)
- Audit trail (who validated, when)
- Compliance rules (regulatory requirements)
- Idempotency (duplicate transaction detection)

**AI/ML:**
- Feature ranges match training data
- Missing value handling
- Outlier detection thresholds
- Data type consistency (categorical vs continuous)
- Label validation
- Dataset balance checks

**General:**
- Type safety (leverage type system)
- Required vs optional fields
- Input sanitization (XSS, injection)
- Business rule enforcement
- User-friendly error messages

## Step 6: Report Findings

### Executive Summary
```
Data Validation Review | Generated: [date] | Domain: [Finance/AI/General]
Scope: [validation types reviewed]
Findings: Critical X, High Y, Medium Z, Low W
Risk Level: [High/Medium/Low]
```

### Findings Format
```
**[SEVERITY] Validation Gap/Issue**
- Type: [schema/quality/constraint/format/business/statistical/sanitization/integrity]
- Location: `file:line`
- Risk: [data corruption/security/compliance/model failure/user error]
- Current: ```code showing issue```
- Missing: [what validation is absent]
- Fix: [implementation] ```corrected code```
- Test: [how to verify]
```

Group by severity (Critical first), then by validation type

### Critical Issues
Missing validations that could cause:
- Data corruption or loss
- Security vulnerabilities
- Compliance violations
- Model failures (AI)
- Financial errors

### Validation Gaps
Areas with no validation or insufficient validation

### Anti-patterns
- Validating too late (after processing)
- Swallowing validation errors
- Generic error messages
- Client-side only validation (no server-side)
- Trusting internal data without validation

### Recommendations
**Quick wins:** Easy validations with high impact
**Strategic:** Validation framework/library adoption
**Process:** Where to add validation in data flow

## Step 7: Offer Implementation

Ask if user wants to:
- Implement missing validations
- Add validation framework/library
- Create validation utilities
- Add comprehensive tests
- Improve error messages

## Severity Ratings

- **Critical**: Missing validation that could cause data corruption, security breach, compliance violation, or financial loss
- **High**: Insufficient validation causing user errors, data quality issues, or model failures
- **Medium**: Weak validation that reduces reliability or user experience
- **Low**: Missing edge case handling, suboptimal error messages

## Validation Principles

1. **Validate early** - At system boundaries (API input, file upload, user forms)
2. **Fail fast** - Reject invalid data immediately with clear errors
3. **Be specific** - "Email must contain @ and domain" not "Invalid input"
4. **Trust nothing** - Validate internal data at critical points (finance, ML inference)
5. **Use type systems** - Leverage static typing where available
6. **Test validation** - Unit tests for validation logic, especially edge cases
7. **Log validation failures** - For monitoring and debugging
8. **Context matters** - Finance needs precision, AI needs quality, web needs sanitization

## Skip

- Over-validation (checking invariants maintained by type system)
- Validation that duplicates database constraints without adding value
- Suggestions for validation where data is already guaranteed valid
