---
name: security-audit
description: Comprehensive security audit framework - detects vulnerabilities across authentication, injection, cryptography, APIs, infrastructure, web security, data protection, dependencies, and more. Use when user requests security review, vulnerability scan, pen test, or security assessment.
when_to_use: User mentions "security audit", "security review", "vulnerability scan", "security check", "pen test", "find security issues", "security assessment", "check for vulnerabilities"
allowed-tools: Read Grep Glob Bash(git rev-parse *) Bash(git status *) Bash(git diff *)
---

Professional security auditor with expertise in OWASP Top 10, CWE, and defensive security practices.

## Approach

Evidence-based (file:line refs) | Risk-focused (Critical/High/Med/Low/Info) | Educational (WHY + attack scenario) | Actionable (specific fixes) | Context-aware (dev vs prod) | Standards-based (OWASP/CWE)

## Step 1: Detect Environment

**Glob**: `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `docker-compose.yml`, `composer.json`, `.env*`
**Read**: Identify framework + check for git repo (`git rev-parse --git-dir`)

## Step 2: Discover Scans

Glob `security-*.md` in `${CLAUDE_SKILL_DIR}/references/` → Read first heading of each → build scan list

## Step 3: Determine Scope

User-specified → proceed. Else offer:
1. **Full Audit** — all scans
2. **Quick Scan** — auth + injection + crypto + api
3. **List scans** — show discovered files, user picks

## Step 4: Load References

**Full**: Glob `security-*.md` in `${CLAUDE_SKILL_DIR}/references/` → Read all
**Quick**: Read `security-auth.md`, `security-injection.md`, `security-crypto.md`, `security-api.md` from `${CLAUDE_SKILL_DIR}/references/`
**Custom**: Map request → filenames → Read

Follow each file's instructions to perform the review.

## Step 5: Execute Reviews

Per reference file:
1. Read completely
2. Search codebase (Grep/Glob/Read per guidance)
3. Verify actual exploitability — check for mitigations elsewhere before reporting
4. Dev credentials acceptable if documented | Focus on `.env.production` / live configs | Skip unused stack components
5. Document findings (Step 6)

## Step 6: Report

### Executive Summary

```
Security Audit Report
Generated: [date] | Scans: [list]
Findings: Critical X | High Y | Medium Z | Low W | Info V
Assessment: [1-2 sentence summary]
```

### Detailed Findings

Group by severity (Critical first), then category:

```
**[SEVERITY] Vulnerability Title**
- **Category**: [auth/injection/crypto/api/etc.]
- **Location**: `file:line`
- **Issue**: [what's vulnerable]
- **Risk**: [attack scenario and impact]
- **Evidence**: ```lang
  [vulnerable code]
  ```
- **Fix**: [remediation] ```lang
  [corrected code]
  ```
- **Reference**: [OWASP A01:2021, CWE-89, etc.]
```

### Remediation Roadmap

```
Priority 1 — Immediate (Critical + High): [issue] — Effort: S/M/L
Priority 2 — Next Sprint (Medium): [issue] — Effort: S/M/L
Priority 3 — Future (Low + Info): [issue] — Effort: S/M/L
```

### Recommendations

Quick wins | Long-term improvements | Security practices | Monitoring

## Step 7: Offer Assistance

Fix Critical/High? | Create security docs? | Configure automated scanning (npm audit, Dependabot)?

## Severity Ratings

- **Critical**: Immediate remote exploitation, complete compromise/breach/RCE
- **High**: Exploitable with moderate effort OR critical impact with auth required
- **Medium**: Requires specific conditions OR moderate impact
- **Low**: Minor concern, limited impact, significant user interaction needed
- **Info**: Security best practice, not a direct vulnerability

## Quality Standards

- `file:line` for every finding
- Critical/High must include attack scenario
- Every finding must have a concrete fix, not just "improve security"
- Group duplicate instances
- Only report findings relevant to this specific project

## Skip

Theoretical issues with no attack vector | Test-file issues unless exposing production secrets | Unused dependency paths | Missing features irrelevant to this app | Style issues that don't cause security bugs
