# API Security

## Scope

REST/GraphQL endpoints | Authentication/authorization | Rate limiting | CORS | Input validation | Mass assignment | Excessive data exposure | SSRF | Open redirects | API keys | Versioning

## Vulnerabilities

### 1. Missing Rate Limiting

**Search**: `@app\.route|router\.(get|post)|express\(\)|app\.use|rateLimit|rate.limit|express-rate-limit|@throttle|SlowDown`

**Issue**: No rate limiting on API endpoints | Unlimited requests | No throttling

**Severity**: High (DoS, brute force, scraping)

**Fix**: Implement rate limiting per IP/user | Different limits for auth vs public | Lower limits on expensive operations

**Evidence patterns**:
```
# Vulnerable - no rate limiting
app.post('/api/login', loginHandler)
@app.route('/api/search')
router.post('/api/upload', uploadHandler)
```

**Secure patterns**:
```
# Express.js
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 min
  max: 100,  // 100 requests per window
  standardHeaders: true
});
app.use('/api/', limiter);

# Different limits per endpoint
const strictLimiter = rateLimit({ windowMs: 15*60*1000, max: 5 });
app.post('/api/login', strictLimiter, loginHandler);

# Django
from django_ratelimit.decorators import ratelimit
@ratelimit(key='ip', rate='100/h')
```

**Ref**: CWE-770, A04:2021

---

### 2. CORS Misconfiguration

**Search**: `Access-Control-Allow-Origin|cors\(|CORS|allowedOrigins|origin.*\*|credentials.*true.*origin.*\*`

**Issue**: Wildcard origin (*) with credentials | Reflected origin without validation | Overly permissive origins

**Severity**: High (credential theft, data exposure)

**Fix**: Whitelist specific origins | Never use * with credentials | Validate origin before reflection

**Evidence patterns**:
```
# Vulnerable
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true  // With wildcard!

# Reflected origin (no validation)
origin = request.headers.get('Origin')
response.headers['Access-Control-Allow-Origin'] = origin

# Express - too permissive
app.use(cors({ origin: '*', credentials: true }))
```

**Secure patterns**:
```
# Whitelist origins
const allowedOrigins = ['https://app.example.com', 'https://www.example.com'];
app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true
}));

# Django
CORS_ALLOWED_ORIGINS = ['https://app.example.com']
CORS_ALLOW_CREDENTIALS = True
```

**Ref**: CWE-942, A05:2021

---

### 3. Excessive Data Exposure

**Search**: `\.json\(\)|jsonify\(|to_json|serialize|ModelSerializer|\.all\(\)|find\(\)\.toJSON`

**Issue**: Returning full objects without filtering | Exposing internal IDs | Leaking sensitive fields | No field selection

**Severity**: Medium (information disclosure)

**Fix**: Return only needed fields | Use DTOs/serializers | Exclude sensitive fields | Support field selection

**Evidence patterns**:
```
# Vulnerable
res.json(user)  // Includes password hash, email, etc.
return jsonify(user.__dict__)
return User.objects.all()  // All fields!
```

**Secure patterns**:
```
# Explicit fields
const { password, resetToken, ...safeUser } = user;
res.json({ id: user.id, name: user.name, email: user.email });

# Serializer (Django)
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']  // Explicit
        exclude = ['password', 'reset_token']

# GraphQL field selection (built-in)
query { user(id: 1) { id name } }  // Only requested fields
```

**Ref**: CWE-213, A01:2021

---

### 4. Missing Input Validation

**Search**: `request\.body|request\.query|request\.params|req\.body|req\.query|req\.params|\$_GET|\$_POST|params\[`

**Issue**: No validation on API inputs | Type coercion issues | No length limits | Accepting unexpected fields

**Severity**: High (injection, DoS, logic bugs)

**Fix**: Validate all inputs | Use schemas (Joi, Zod, Pydantic) | Whitelist allowed fields | Enforce types and lengths

**Evidence patterns**:
```
# Vulnerable
const age = req.body.age;  // No validation!
const email = request.json.get('email')
User.create(req.body)  // Mass assignment!
```

**Secure patterns**:
```
# Express + Joi
const Joi = require('joi');
const schema = Joi.object({
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(0).max(120),
  name: Joi.string().max(100)
});
const { error, value } = schema.validate(req.body);

# FastAPI + Pydantic
from pydantic import BaseModel, EmailStr, Field
class UserCreate(BaseModel):
    email: EmailStr
    age: int = Field(ge=0, le=120)
    name: str = Field(max_length=100)

# Validate before use
@app.post('/users')
def create_user(user: UserCreate):
    # user is validated automatically
```

**Ref**: CWE-20, A03:2021

---

### 5. Mass Assignment

**Search**: `create\(req\.body\)|update\(.*request\.|new.*\(req\.body\)|\.save\(request\.|Model\(.*params`

**Issue**: Directly using user input in model creation | No field whitelisting | Can set admin/role fields

**Severity**: High (privilege escalation)

**Fix**: Whitelist allowed fields | Use DTOs | Never trust client for protected fields

**Evidence patterns**:
```
# Vulnerable
User.create(req.body)  // Can set isAdmin: true!
user.update(request.json)  // Can modify role!
new User(req.body).save()
```

**Secure patterns**:
```
# Whitelist fields
const allowedFields = ['name', 'email', 'bio'];
const userData = _.pick(req.body, allowedFields);
User.create(userData);

# Explicit assignment
User.create({
  name: req.body.name,
  email: req.body.email,
  role: 'user'  // Set explicitly, not from input!
});

# Django - protect fields
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['name', 'email']  // Only these can be set
        read_only_fields = ['id', 'is_admin', 'created_at']
```

**Ref**: CWE-915, A01:2021

---

### 6. API Key Exposure

**Search**: `api.?key.*=|API_KEY.*=|x-api-key|authorization.*bearer.*[A-Za-z0-9]{20,}|apikey|api_key`

**Issue**: API keys in client-side code | Keys in URLs | Keys logged | Keys in git | Insufficient key permissions

**Severity**: Critical (unauthorized access)

**Fix**: Keys server-side only | Never in URLs | Rotate keys | Least privilege | Monitor usage

**Evidence patterns**:
```
# Vulnerable
const apiKey = "sk_live_abc123xyz789";  // Hardcoded!
fetch(`https://api.example.com?apikey=${key}`)  // In URL!
console.log('API Key:', apiKey)  // Logged!
```

**Secure patterns**:
```
# Server-side only
const apiKey = process.env.API_KEY;

# Header, not URL
fetch('https://api.example.com', {
  headers: { 'X-API-Key': apiKey }
});

# Rotate keys regularly
# Use scoped keys (read-only, limited endpoints)
# Monitor for unusual usage patterns
```

**Ref**: CWE-798, A02:2021

---

### 7. Broken Object Level Authorization (BOLA)

**Search**: `findById|find.*params\.|get.*req\.params|User\.find.*id.*req|Document\.findOne.*id`

**Issue**: No ownership check before data access | Can access other users' objects via ID manipulation

**Severity**: Critical (unauthorized data access)

**Fix**: Always verify user owns resource | Check permissions before returning data

**Evidence patterns**:
```
# Vulnerable
app.get('/api/documents/:id', async (req, res) => {
  const doc = await Document.findById(req.params.id);
  res.json(doc);  // No ownership check!
});
```

**Secure patterns**:
```
# Verify ownership
app.get('/api/documents/:id', requireAuth, async (req, res) => {
  const doc = await Document.findById(req.params.id);
  if (!doc) return res.status(404).json({ error: 'Not found' });
  if (doc.userId !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  res.json(doc);
});

# Query with user filter
const doc = await Document.findOne({
  _id: req.params.id,
  userId: req.user.id  // Implicit ownership check
});
if (!doc) return res.status(404).json({ error: 'Not found' });
```

**Ref**: CWE-639, A01:2021

---

### 8. GraphQL Specific Issues

**Search**: `graphql|GraphQL|type.*Query|type.*Mutation|resolver|@Field|buildSchema`

**Issue**: No query depth limiting | No complexity analysis | Introspection enabled in prod | Missing auth on resolvers | Batching attacks

**Severity**: High (DoS, data exposure)

**Fix**: Depth limiting | Complexity analysis | Disable introspection in prod | Auth on resolvers | Query cost analysis

**Evidence patterns**:
```
# Vulnerable query (infinite depth)
query {
  user {
    friends {
      friends {
        friends {
          # ... can nest very deep
        }
      }
    }
  }
}

# No auth on resolver
const resolvers = {
  Query: {
    users: () => User.findAll()  // No auth check!
  }
}
```

**Secure patterns**:
```
# Depth limiting
const depthLimit = require('graphql-depth-limit');
app.use('/graphql', graphqlHTTP({
  schema,
  validationRules: [depthLimit(5)]
}));

# Complexity analysis
const { createComplexityLimitRule } = require('graphql-validation-complexity');
validationRules: [createComplexityLimitRule(1000)]

# Auth on resolvers
const resolvers = {
  Query: {
    users: (parent, args, context) => {
      if (!context.user?.isAdmin) {
        throw new Error('Unauthorized');
      }
      return User.findAll();
    }
  }
}

# Disable introspection in prod
if (process.env.NODE_ENV === 'production') {
  validationRules: [NoIntrospection]
}
```

**Ref**: CWE-400, A04:2021

---

### 9. Server-Side Request Forgery (SSRF)

**Search**: `fetch\(.*req\.|axios\(.*request\.|requests\.get\(.*input|curl.*\$|wget.*\$|HttpClient.*user|url.*=.*request`

**Issue**: User-controlled URLs in server requests | No URL validation | Access to internal IPs | Cloud metadata access

**Severity**: Critical (internal network access, credential theft)

**Fix**: Whitelist URLs/domains | Block internal IPs | Validate URL scheme | Use DNS rebinding protection

**Evidence patterns**:
```
# Vulnerable
app.post('/api/fetch', (req, res) => {
  const url = req.body.url;
  fetch(url).then(r => r.text()).then(data => res.send(data));
  // Can access http://169.254.169.254/latest/meta-data/
});
```

**Secure patterns**:
```
# Whitelist domains
const ALLOWED_DOMAINS = ['api.example.com', 'cdn.example.com'];
const url = new URL(req.body.url);
if (!ALLOWED_DOMAINS.includes(url.hostname)) {
  return res.status(400).json({ error: 'Invalid domain' });
}

# Block internal IPs
const dns = require('dns').promises;
const { address } = await dns.lookup(url.hostname);
const isPrivate = /^(10\.|172\.(1[6-9]|2[0-9]|3[01])\.|192\.168\.|127\.|169\.254\.)/.test(address);
if (isPrivate) {
  return res.status(400).json({ error: 'Invalid host' });
}

# Only allow HTTPS
if (url.protocol !== 'https:') {
  return res.status(400).json({ error: 'Only HTTPS allowed' });
}
```

**Ref**: CWE-918, A10:2021

---

### 10. Open Redirect

**Search**: `redirect\(.*request\.|Location:.*req\.|\.redirect\(.*query|returnUrl|next=|url=|redirect.*=`

**Issue**: User-controlled redirect target | No URL validation | Open redirector abused for phishing

**Severity**: Medium (phishing, token theft)

**Fix**: Whitelist redirect URLs | Validate domain | Use relative URLs only | Warn on external redirects

**Evidence patterns**:
```
# Vulnerable
res.redirect(req.query.next)
return redirect(request.args.get('returnUrl'))
header('Location: ' . $_GET['url']);
```

**Secure patterns**:
```
# Whitelist
const ALLOWED_REDIRECTS = ['/dashboard', '/profile', '/settings'];
if (!ALLOWED_REDIRECTS.includes(req.query.next)) {
  return res.redirect('/dashboard');  // Safe default
}

# Validate same domain
const url = new URL(req.query.next, req.headers.origin);
if (url.origin !== req.headers.origin) {
  return res.status(400).json({ error: 'Invalid redirect' });
}

# Relative URLs only
if (redirectUrl.startsWith('http://') || redirectUrl.startsWith('https://')) {
  return res.status(400).json({ error: 'External redirects not allowed' });
}
```

**Ref**: CWE-601, A01:2021

---

## Review Process

1. **Rate limiting**: Search route definitions → check for rate limit middleware
2. **CORS**: Find CORS config → verify no wildcard with credentials, validate origins
3. **Data exposure**: Find response handlers → check for field filtering
4. **Input validation**: Search request.body usage → verify validation before use
5. **Mass assignment**: Find model.create() → check for field whitelisting
6. **API keys**: Search for API_KEY → verify not hardcoded, not in URLs
7. **BOLA**: Find ID-based queries → verify ownership checks
8. **GraphQL**: If GraphQL, check depth limits, introspection, auth on resolvers
9. **SSRF**: Find fetch/requests with user input → verify URL validation
10. **Redirects**: Search redirect() → verify URL whitelisting

## Report Format

```
**[SEVERITY] Missing Rate Limiting**
- **Category**: API Security
- **Location**: `routes/api.js:45`
- **Issue**: No rate limiting on login endpoint
- **Risk**: Attacker can brute force credentials, perform credential stuffing
- **Evidence**: `app.post('/api/login', loginHandler)  // No rate limiter`
- **Fix**: Add rate limiting: `app.post('/api/login', loginLimiter, loginHandler)`
- **Reference**: CWE-770, A04:2021
```

## Common References

CWE-20 (input validation) | CWE-213 (data exposure) | CWE-400 (resource exhaustion) | CWE-601 (open redirect) | CWE-639 (insecure direct object ref) | CWE-770 (no throttling) | CWE-915 (mass assignment) | CWE-918 (SSRF) | CWE-942 (CORS) | A01:2021 (Broken Access Control) | A03:2021 (Injection) | A04:2021 (Insecure Design) | A05:2021 (Security Misconfiguration)
