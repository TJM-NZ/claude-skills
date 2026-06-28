# Web Application Security

## Scope

XSS (reflected, stored, DOM) | CSRF protection | Security headers | Content Security Policy (CSP) | Clickjacking | Cookie security | HTML injection | Subresource Integrity (SRI) | Iframe security

## Vulnerabilities

### 1. Cross-Site Scripting (XSS)

**Search**: `innerHTML|dangerouslySetInnerHTML|\.html\(|document\.write|eval\(|v-html|ng-bind-html|\$\{.*\}.*<|\.render.*raw|safe.*filter`

**Issue**: Unescaped user input in HTML | Reflected XSS in URLs | Stored XSS in database | DOM XSS via JavaScript | Unsafe rendering

**Severity**: High (session theft, account takeover)

**Fix**: Auto-escape by default | Sanitize HTML input | Use CSP | Avoid innerHTML | Use textContent

**Evidence patterns**:
```
# Vulnerable - Reflected XSS
<h1>Search results for: ${req.query.q}</h1>
document.getElementById('result').innerHTML = userInput;

# Stored XSS
<div>{!! $comment !!}</div>  // Laravel unescaped
<div v-html="userComment"></div>  // Vue unescaped
dangerouslySetInnerHTML={{ __html: comment }}  // React

# DOM XSS
const hash = location.hash.substring(1);
document.write(hash);

# Template injection
{{ user_input | safe }}  // Jinja2 unescaped
```

**Secure patterns**:
```
# Auto-escape (default in most frameworks)
<h1>Search results for: {{ query }}</h1>  // Escaped
<div>{userComment}</div>  // React auto-escapes
<div>{{ comment }}</div>  // Vue auto-escapes

# Sanitize HTML if needed
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(dirtyHTML);

# Use textContent, not innerHTML
element.textContent = userInput;

# CSP to mitigate
Content-Security-Policy: default-src 'self'; script-src 'self'
```

**Ref**: CWE-79, A03:2021

---

### 2. Missing CSRF Protection

**Search**: `csrf|CSRF|@csrf|csrf_token|_token|X-CSRF-Token|SameSite|method.*POST|method.*PUT|method.*DELETE`

**Issue**: State-changing operations without CSRF tokens | No SameSite cookie attribute | GET requests for mutations

**Severity**: High (unauthorized actions)

**Fix**: CSRF tokens on forms | SameSite=Strict/Lax | Use POST/PUT/DELETE (not GET) for mutations | Verify Origin header

**Evidence patterns**:
```
# Vulnerable - no CSRF token
<form method="POST" action="/delete-account">
  <button>Delete</button>
</form>

# State change via GET
app.get('/api/delete/:id', deleteHandler)

# No SameSite
res.cookie('session', token)  // Missing SameSite
```

**Secure patterns**:
```
# CSRF token (Express)
const csrf = require('csurf');
app.use(csrf());
<form method="POST">
  <input type="hidden" name="_csrf" value="{{ csrfToken }}">
</form>

# SameSite cookies
res.cookie('session', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict'  // or 'lax'
});

# Verify Origin/Referer
if (req.headers.origin !== 'https://example.com') {
  return res.status(403).send('Invalid origin');
}

# POST for mutations (never GET)
app.post('/api/delete/:id', deleteHandler)
```

**Ref**: CWE-352, A01:2021

---

### 3. Missing Security Headers

**Search**: `helmet|express|app\.use|middleware|headers|X-Frame-Options|X-Content-Type-Options|Strict-Transport-Security|X-XSS-Protection`

**Issue**: No X-Frame-Options | No X-Content-Type-Options | No HSTS | Permissive CSP | Missing Referrer-Policy

**Severity**: Medium (various attacks enabled)

**Fix**: Set all security headers | Use helmet.js | Enable HSTS | Restrictive CSP

**Evidence patterns**:
```
# Missing headers (check responses)
curl -I https://example.com
# Look for absence of security headers
```

**Secure patterns**:
```
# Express + Helmet
const helmet = require('helmet');
app.use(helmet());

# Or manual headers
app.use((req, res, next) => {
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  next();
});

# Nginx
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
```

**Ref**: CWE-1021, A05:2021

---

### 4. Weak Content Security Policy

**Search**: `Content-Security-Policy|CSP|script-src|unsafe-inline|unsafe-eval|meta.*http-equiv.*Content-Security-Policy`

**Issue**: No CSP | unsafe-inline in script-src | unsafe-eval allowed | Overly permissive CSP | CSP in report-only mode forever

**Severity**: Medium (XSS mitigation weakened)

**Fix**: Strict CSP | No unsafe-inline/unsafe-eval | Nonce or hash-based CSP | Whitelist specific domains

**Evidence patterns**:
```
# Vulnerable
Content-Security-Policy: default-src *  // Too permissive!
script-src 'unsafe-inline' 'unsafe-eval'  // Defeats CSP purpose

# No CSP at all
curl -I https://example.com | grep -i content-security
```

**Secure patterns**:
```
# Strict CSP
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{random}';
  style-src 'self' 'nonce-{random}';
  img-src 'self' data: https:;
  font-src 'self';
  connect-src 'self';
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self'

# Nonce-based (recommended)
<script nonce="random123">...</script>
script-src 'nonce-random123'

# Hash-based
script-src 'sha256-{hash-of-script}'

# Express
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'"]
  }
}));
```

**Ref**: CWE-1021, A05:2021

---

### 5. Clickjacking

**Search**: `X-Frame-Options|frame-ancestors|iframe|frameset|embed`

**Issue**: No X-Frame-Options | No frame-ancestors in CSP | Site can be iframed | UI redressing attacks possible

**Severity**: Medium (phishing, unauthorized actions)

**Fix**: X-Frame-Options: DENY/SAMEORIGIN | CSP frame-ancestors 'none'/'self' | Check if site needs iframing

**Evidence patterns**:
```
# Test if site can be iframed
<iframe src="https://victim.com"></iframe>

# Missing protection
curl -I https://example.com | grep X-Frame-Options
# No header found
```

**Secure patterns**:
```
# Deny all framing
X-Frame-Options: DENY

# Allow same-origin only
X-Frame-Options: SAMEORIGIN

# CSP (more flexible)
Content-Security-Policy: frame-ancestors 'none'
Content-Security-Policy: frame-ancestors 'self'
Content-Security-Policy: frame-ancestors https://trusted.com

# Express
app.use(helmet.frameguard({ action: 'deny' }));

# Nginx
add_header X-Frame-Options "DENY";
```

**Ref**: CWE-1021, A05:2021

---

### 6. Insecure Cookie Configuration

**Search**: `Set-Cookie|res\.cookie|response\.cookies|document\.cookie|httpOnly|secure|sameSite`

**Issue**: Missing HttpOnly flag | Missing Secure flag | No SameSite attribute | Session ID in JavaScript-accessible cookie

**Severity**: High (session theft via XSS)

**Fix**: HttpOnly for session cookies | Secure flag (HTTPS only) | SameSite=Strict/Lax | Don't use document.cookie for sensitive data

**Evidence patterns**:
```
# Vulnerable
res.cookie('session', token)  // No flags!
document.cookie = "session=" + token;  // XSS can steal
Set-Cookie: session=abc123  // No HttpOnly, Secure, SameSite
```

**Secure patterns**:
```
# Secure cookies
res.cookie('session', token, {
  httpOnly: true,    // No JS access
  secure: true,      // HTTPS only
  sameSite: 'strict', // CSRF protection
  maxAge: 3600000,   // 1 hour
  path: '/',
  domain: 'example.com'
});

# Express session
app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 3600000
  }
}));

# Check existing cookies
curl -I https://example.com
# Verify Set-Cookie has HttpOnly; Secure; SameSite
```

**Ref**: CWE-1004, A05:2021

---

### 7. Missing Subresource Integrity (SRI)

**Search**: `<script.*src.*cdn|<link.*href.*cdn|integrity.*sha|crossorigin`

**Issue**: External scripts/styles without SRI | CDN compromise risk | No integrity verification

**Severity**: Medium (supply chain attack)

**Fix**: Add integrity attribute | Use SRI for all CDN resources | Verify hashes match

**Evidence patterns**:
```
# Vulnerable - no SRI
<script src="https://cdn.example.com/lib.js"></script>
<link href="https://cdn.example.com/style.css" rel="stylesheet">
```

**Secure patterns**:
```
# With SRI
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux..."
  crossorigin="anonymous">
</script>

<link
  href="https://cdn.example.com/style.css"
  rel="stylesheet"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux..."
  crossorigin="anonymous">

# Generate SRI hash
openssl dgst -sha384 -binary lib.js | openssl base64 -A
```

**Ref**: CWE-353, A08:2021

---

### 8. HTML Injection

**Search**: `innerHTML|insertAdjacentHTML|outerHTML|document\.write|createContextualFragment`

**Issue**: User input in HTML context | Not full XSS but HTML structure manipulation | Phishing, defacement

**Severity**: Medium

**Fix**: Use textContent | Sanitize HTML | CSP | Avoid innerHTML with user data

**Evidence patterns**:
```
# Vulnerable
element.innerHTML = "<div>" + userName + "</div>";
element.insertAdjacentHTML('beforeend', userContent);
```

**Secure patterns**:
```
# Use textContent
element.textContent = userName;

# Or createElement
const div = document.createElement('div');
div.textContent = userName;
element.appendChild(div);

# If HTML needed, sanitize
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userHTML);
```

**Ref**: CWE-79, A03:2021

---

### 9. Autocomplete on Sensitive Fields

**Search**: `input.*type.*password|input.*type.*text.*credit|autocomplete|sensitive.*input`

**Issue**: Autocomplete enabled on password/credit card fields | Browser saves sensitive data | Shared computers risk

**Severity**: Low (information disclosure on shared devices)

**Fix**: autocomplete="off" on sensitive fields | Use autocomplete="new-password" for password fields

**Evidence patterns**:
```
# Vulnerable
<input type="password" name="password">
<input type="text" name="credit_card">
<input type="text" name="ssn">
```

**Secure patterns**:
```
# Disable autocomplete
<input type="password" name="password" autocomplete="new-password">
<input type="text" name="credit_card" autocomplete="off">
<input type="text" name="ssn" autocomplete="off">

# For login (can use autocomplete)
<input type="password" name="password" autocomplete="current-password">
```

**Ref**: CWE-522

---

### 10. Referrer Leakage

**Search**: `Referrer-Policy|referrerpolicy|<meta.*referrer|rel.*noreferrer`

**Issue**: No Referrer-Policy header | Sensitive data in URLs leaked via Referer | External sites see internal URLs

**Severity**: Low (information disclosure)

**Fix**: Referrer-Policy header | Use strict-origin-when-cross-origin | rel="noreferrer" on external links

**Evidence patterns**:
```
# Missing Referrer-Policy
curl -I https://example.com | grep Referrer-Policy
# Not found

# URL with sensitive data
https://example.com/reset?token=secret123
# This token leaks in Referer to external sites
```

**Secure patterns**:
```
# Header
Referrer-Policy: strict-origin-when-cross-origin
# or
Referrer-Policy: no-referrer-when-downgrade

# Meta tag
<meta name="referrer" content="strict-origin-when-cross-origin">

# Per-link
<a href="https://external.com" rel="noreferrer noopener">Link</a>

# Express
app.use(helmet.referrerPolicy({
  policy: 'strict-origin-when-cross-origin'
}));
```

**Ref**: CWE-200

---

