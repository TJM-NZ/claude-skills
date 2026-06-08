# Security Audit

Comprehensive security audit framework detecting vulnerabilities across authentication, injection attacks, cryptography, APIs, infrastructure, web security, data protection, and dependencies.

## What It Does

Systematic security review covering:

- **Authentication & Authorization** - Session management, password policies, MFA, OAuth, JWT, RBAC
- **Injection Attacks** - SQL injection, XSS, command injection, LDAP, XXE
- **Cryptography** - Algorithm selection, key management, TLS/SSL, password hashing
- **API Security** - Rate limiting, input validation, CORS, authentication
- **Database Security** - Prepared statements, least privilege, encryption at rest
- **Infrastructure** - Secrets management, container security, network config
- **Web Application Security** - CSRF, clickjacking, cookie security, CSP
- **Data Protection** - PII handling, encryption, data retention, GDPR compliance
- **Code Security** - Unsafe deserialization, path traversal, race conditions
- **Dependency Security** - Vulnerable packages, supply chain risks
- **OS/Server Security** - File permissions, process isolation, updates

## When to Use

- Pre-deployment security review
- Compliance requirements (SOC2, GDPR, PCI-DSS)
- Post-incident security assessment
- Regular security posture checks
- Third-party integration review
- Open source project security validation

## Approach

1. **Detect environment** - Identify frameworks, languages, dependencies
2. **Discover audit modules** - Map relevant security domains
3. **Determine scope** - Full audit, quick scan, or custom selection
4. **Load reference frameworks** - Read security best practices per domain
5. **Execute security review** - Search for vulnerabilities systematically
6. **Report findings** - Severity-based (Critical → Info) with CVE references

## Output Format

```
**[SEVERITY] Vulnerability Title**
- Category: [auth/injection/crypto/api/db/infra/web/data/code/deps/os]
- Location: `file:line`
- Risk: [exploitation scenario]
- Evidence: [vulnerable code]
- Impact: [data breach/RCE/privilege escalation/data loss]
- Fix: [secure implementation]
- Reference: [OWASP/CVE/CWE link]
```

## Severity Ratings

- **Critical** - Remote code execution, SQL injection, auth bypass, exposed secrets
- **High** - XSS, CSRF, insecure crypto, privilege escalation, sensitive data exposure
- **Medium** - Weak session management, missing rate limits, verbose errors, outdated deps
- **Low** - Missing security headers, weak password policy, incomplete logging
- **Info** - Security best practices, defense-in-depth recommendations

## Example Usage

```bash
/security-audit
```

Options:
1. Full Security Audit (all domains)
2. Quick Scan (auth + injection + crypto + API)
3. Compliance Focus (OWASP Top 10, GDPR, PCI-DSS)
4. Custom (user selects specific security domains)

## Key Checks

### Authentication
- Password policies, secure storage (bcrypt/Argon2)
- Session fixation, token expiration
- OAuth/SAML implementation flaws
- MFA enforcement

### Injection
- SQL injection via ORM misuse
- XSS in user-generated content
- Command injection in system calls
- Template injection

### Cryptography
- Weak algorithms (MD5, SHA1, DES)
- Hardcoded secrets in code
- Insecure random number generation
- Missing TLS/certificate validation

### API Security
- Missing authentication/authorization
- No rate limiting (DoS risk)
- CORS misconfiguration
- Excessive data exposure

## For Employers

Demonstrates:
- Security-first development mindset
- Knowledge of OWASP Top 10 and common CVEs
- Understanding of compliance requirements
- Ability to identify vulnerabilities before exploitation
- Systematic approach to security posture assessment
- Awareness of supply chain and infrastructure security
