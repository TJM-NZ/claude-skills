# Authentication & Authorization

## Scope

Password storage | Session management | JWT/tokens | OAuth/OIDC | RBAC | MFA | Password reset | Brute force protection | Account enumeration

## Vulnerabilities

### 1. Weak Password Storage

**Search**: `md5(.*password|sha1(.*password|hashlib\.(md5|sha1)|createHash\(['"]md5|password.*=.*request\.|INSERT.*password`

**Issue**: Plaintext or weak hashing (MD5, SHA1, SHA256)

**Severity**: Critical (plaintext) | High (weak hash)

**Fix**: bcrypt (12+ rounds) | argon2 | scrypt | PBKDF2

**Ref**: CWE-259, CWE-916, A07:2021

---

### 2. Insecure Session Management

**Search**: `set-cookie|res\.cookie|httpOnly.*false|secure.*false|sameSite|maxAge|SESSION_COOKIE|express-session`

**Issue**: Missing httpOnly, Secure, SameSite | No expiration | Sessions not destroyed on logout

**Severity**: High

**Fix**: httpOnly: true, secure: true, sameSite: 'strict', maxAge: 3600000

**Ref**: CWE-384, CWE-613, A07:2021

---

### 3. JWT Issues

**Search**: `jsonwebtoken|jwt\.(sign|verify|decode)|algorithm.*none|verify.*false|localStorage.*token|expiresIn`

**Issue**: Algorithm confusion | "none" accepted | No expiration | Stored in localStorage (XSS vulnerable) | Sensitive data in payload

**Severity**: Critical (bypass) | High (theft)

**Fix**: Explicit algorithm whitelist | Require exp | Store in httpOnly cookie | No secrets in payload

**Ref**: CWE-287, CWE-327, A07:2021

---

### 4. Missing Auth Checks

**Search**: `@app\.route|router\.(get|post|put|delete)|@login_required|requireAuth|isAuthenticated|@admin_required|hasRole`

**Issue**: Endpoints without auth middleware | Admin routes without role check | No ownership verification

**Severity**: Critical

**Fix**: Apply auth middleware to all protected routes | Check user.role for admin | Verify ownership before access

**Ref**: CWE-306, CWE-862, A01:2021

---

### 5. OAuth/OIDC Flaws

**Search**: `oauth|authorize|authorization_code|state.*=|code_challenge|redirect_uri|callback`

**Issue**: Missing state (CSRF) | Missing PKCE | No redirect URI validation | Code reuse

**Severity**: High

**Fix**: Generate random state | PKCE for public clients | Whitelist redirect URIs | One-time code usage

**Ref**: CWE-352, A07:2021

---

### 6. Credential Exposure

**Search**: `password.*=.*['"][^'"]{8,}|api.?key.*=.*['"]|AWS_SECRET|PRIVATE_KEY.*=|console\.log.*password|git ls-files .env`

**Issue**: Hardcoded credentials | .env in git | Credentials in logs/errors/URLs

**Severity**: Critical

**Fix**: Use env vars | Add .env* to .gitignore | Never log sensitive fields | No credentials in URLs

**Ref**: CWE-798, CWE-312, CWE-522, A07:2021

---

### 7. Insecure Password Reset

**Search**: `reset.?password|forgot.?password|generateResetToken|Math\.random|crypto\.randomBytes|resetTokenExpiry`

**Issue**: Predictable tokens | No expiration | Reusable tokens | User enumeration

**Severity**: High

**Fix**: crypto.randomBytes(32) | Hash token in DB | 1hr expiration | Single-use | Generic error messages

**Ref**: CWE-640, CWE-640, A07:2021

---

### 8. Account Enumeration

**Search**: `invalid.*username|user.*not.*found|incorrect.*password|email.*already.*exists`

**Issue**: Different errors reveal user existence

**Severity**: Medium

**Fix**: Generic message: "Invalid email or password" | Same response time

**Ref**: CWE-203, CWE-204

---

### 9. No Brute Force Protection

**Search**: `rateLimit|rate.limit|express-rate-limit|loginAttempts|failedAttempts|MAX_LOGIN_ATTEMPTS`

**Issue**: Unlimited login attempts | No rate limiting | No account lockout

**Severity**: High

**Fix**: Rate limit: 5 attempts/15min | Account lockout after 5 fails | CAPTCHA (optional)

**Ref**: CWE-307, A07:2021

---
