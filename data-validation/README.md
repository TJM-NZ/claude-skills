# Data Validation

Comprehensive data validation framework for ensuring data integrity, quality, and correctness. Specialized focus on finance (accuracy, compliance) and AI/ML (data quality, model reliability) systems.

## What It Does

Systematic validation framework that identifies missing validations, weak constraints, and data quality issues across:

- **Schema Validation** - Type checking, required fields, structure
- **Data Quality** - Completeness, accuracy, consistency, timeliness
- **Constraint Validation** - Range checks, length limits, enums
- **Format Validation** - Dates, emails, URLs, phone numbers, regex
- **Business Rules** - Domain-specific logic, cross-field validation
- **Statistical Validation** - Outliers, distributions, anomalies (AI/ML)
- **Input Sanitization** - XSS prevention, SQL injection, input cleaning
- **Referential Integrity** - Foreign keys, uniqueness constraints

## When to Use

- Building finance applications (transaction validation, compliance)
- Developing AI/ML systems (training data quality, feature validation)
- API input validation review
- Form/user input handling
- Data pipeline quality checks
- Compliance/audit requirements

## Domain-Specific Focus

### Finance
- Decimal precision (never float for money)
- Range validation (negative amounts, date ranges)
- Audit trails (who validated, when)
- Compliance rules (regulatory requirements)
- Idempotency (duplicate transaction detection)

### AI/ML
- Feature ranges match training data
- Missing value handling strategies
- Outlier detection thresholds
- Data type consistency (categorical vs continuous)
- Label validation
- Dataset balance checks

### General
- Type safety leveraging type systems
- Required vs optional fields
- Input sanitization (security)
- Business rule enforcement
- User-friendly error messages

## Approach

1. **Understand context** - Identify domain (finance/AI/general)
2. **Discover validation types** - Map to relevant validation frameworks
3. **Execute review** - Find gaps, anti-patterns, missing validations
4. **Assess risk** - Critical (data corruption) to Low (edge cases)
5. **Provide fixes** - Specific implementations with test strategies

## Output Format

```
**[SEVERITY] Validation Gap/Issue**
- Type: [schema/quality/constraint/format/business/statistical]
- Location: `file:line`
- Risk: [data corruption/security/compliance/model failure]
- Current: [code showing issue]
- Missing: [what validation is absent]
- Fix: [implementation with code]
- Test: [verification strategy]
```

## Example Usage

```bash
/data-validation
```

Options:
1. Full Validation Audit (all types)
2. Input Validation (schema + format + sanitization)
3. Data Quality Check (quality + statistical + integrity)
4. Finance Validation (precision, compliance, audit)
5. AI/ML Validation (quality + statistical + features)

## Validation Principles

1. **Validate early** - At system boundaries (API, file upload, forms)
2. **Fail fast** - Reject invalid data immediately with clear errors
3. **Be specific** - "Email must contain @ and domain" not "Invalid input"
4. **Trust nothing** - Validate internal data at critical points
5. **Use type systems** - Leverage static typing
6. **Test validation** - Unit tests for edge cases
7. **Log failures** - For monitoring and debugging

## Common Anti-Patterns Detected

- Validating too late (after processing)
- Swallowing validation errors
- Generic error messages
- Client-side only validation
- Trusting internal data without validation
- Missing decimal precision in financial calculations
- No outlier detection in ML pipelines

## For Employers

Demonstrates:
- Understanding of data integrity requirements in finance/AI
- Systematic approach to quality assurance
- Security awareness (input sanitization, injection prevention)
- Compliance mindset (audit trails, regulatory requirements)
- Knowledge of ML data quality impact on model performance
