# Data Protection & Secrets Management

## Scope

Secrets in code | Hardcoded credentials | API keys | Sensitive data in logs | PII handling | Error messages | Version control | Backup security | Data retention | Encryption at rest | Debug info | Sensitive URLs

## Vulnerabilities

### 1. Hardcoded Secrets in Code

**Search**: `password.*=.*['"][^'"]{8,}|api.?key.*=.*['"]|secret.*=.*['"]|token.*=.*['"]|private.?key.*=|AWS_ACCESS|GITHUB_TOKEN|DATABASE_URL.*:|sk_live|pk_live`

**Issue**: Passwords, API keys, tokens in source code | Committed to version control | Visible to all developers

**Severity**: Critical (credential exposure)

**Fix**: Use environment variables | External secret managers | Never commit secrets | Rotate exposed secrets

**Evidence patterns**:
```
# Vulnerable
const apiKey = "sk_live_abc123xyz789";
const dbPassword = "myP@ssw0rd123";
STRIPE_SECRET_KEY = "sk_live_..."
private_key = "-----BEGIN PRIVATE KEY-----..."
DATABASE_URL = "postgres://user:password@host/db"
```

**Secure patterns**:
```
# Environment variables
const apiKey = process.env.STRIPE_SECRET_KEY;
const dbPassword = process.env.DB_PASSWORD;

# .env file (gitignored!)
STRIPE_SECRET_KEY=sk_live_...
DB_PASSWORD=secret123

# .gitignore
.env
.env.*
!.env.example
*.key
*.pem
credentials.json
secrets.yml

# Check git history for leaked secrets
git log -p | grep -i "password\|secret\|key"
```

**Ref**: CWE-798, CWE-259, A02:2021

---

### 2. Secrets in Version Control

**Search**: `.env|credentials|secrets|*.key|*.pem|config/database.yml|application.properties`

**Issue**: .env files committed | Private keys in git | Database credentials in config files | Git history contains secrets

**Severity**: Critical (credential exposure, cannot be fully removed from git history)

**Fix**: .gitignore secrets before first commit | Use git-secrets | Scan repos | Rotate if leaked

**Evidence patterns**:
```
# Check what's committed
git ls-files | grep -E '\.(env|key|pem)$'
git ls-files | grep -i secret
git ls-files | grep -i credential

# Check git history
git log --all --full-history -- .env
git log -p | grep -i "password"
```

**Secure patterns**:
```
# .gitignore (add before first commit!)
.env
.env.local
.env.production
*.key
*.pem
credentials.json
secrets.yml
config/database.yml

# .env.example (template only)
STRIPE_SECRET_KEY=your_key_here
DB_PASSWORD=your_password_here

# If already committed, remove from history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch .env' \
  --prune-empty --tag-name-filter cat -- --all

# Then rotate all secrets!
```

**Ref**: CWE-312, CWE-522, A02:2021

---

### 3. Sensitive Data in Logs

**Search**: `console\.log|logger\.|print\(|log\(|debug\(|error\(|info\(|warn\(|\.log|logging|winston|bunyan`

**Issue**: Passwords in logs | Credit cards logged | Tokens in error logs | PII in debug logs | Logs accessible to unauthorized users

**Severity**: High (credential/PII exposure)

**Fix**: Sanitize before logging | Redact sensitive fields | Secure log access | Log levels in production

**Evidence patterns**:
```
# Vulnerable
console.log('User:', user);  // Contains password!
logger.info('Login attempt:', req.body);  // Password in body!
console.log('Credit card:', creditCard);
logger.debug('Full request:', req);  // May contain tokens
print(f"API Key: {api_key}")
```

**Secure patterns**:
```
# Sanitize before logging
const { password, creditCard, ...safeUser } = user;
console.log('User:', safeUser);

# Redact sensitive fields
const redactedBody = { ...req.body, password: '[REDACTED]' };
logger.info('Login attempt:', redactedBody);

# Use structured logging with field filtering
winston.format.combine(
  winston.format.json(),
  winston.format((info) => {
    delete info.password;
    delete info.creditCard;
    return info;
  })()
)

# Different log levels
if (process.env.NODE_ENV === 'production') {
  logger.level = 'info';  // No debug logs in prod
}
```

**Ref**: CWE-532, CWE-532, A04:2021

---

### 4. Sensitive Data in Error Messages

**Search**: `throw.*Error|except|catch|error.*message|\.stack|traceback|printStackTrace`

**Issue**: Stack traces to users | Database errors exposed | File paths revealed | Internal info in errors

**Severity**: Medium (information disclosure aids attacks)

**Fix**: Generic error messages to users | Log details server-side | Disable stack traces in production

**Evidence patterns**:
```
# Vulnerable
catch (err) {
  res.status(500).json({ error: err.message, stack: err.stack });
}

# Database errors to user
# "Error: duplicate key value violates unique constraint users_email_key"
# Reveals database schema!

# File paths
# "ENOENT: no such file or directory, open '/var/www/app/config/secrets.json'"
```

**Secure patterns**:
```
# Generic errors to users
catch (err) {
  logger.error('Database error:', err);  // Log details
  res.status(500).json({
    error: 'An error occurred. Please try again.'  // Generic message
  });
}

# Express error handler
app.use((err, req, res, next) => {
  logger.error(err);
  res.status(500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal server error'
      : err.message  // Details only in dev
  });
});

# Never send stack traces to users
if (process.env.NODE_ENV !== 'production') {
  app.use(errorHandler());  // Detailed errors in dev only
}
```

**Ref**: CWE-209, CWE-497, A04:2021

---

### 5. PII Exposure

**Search**: `email|phone|address|ssn|social.?security|credit.?card|passport|drivers.?license|date.?of.?birth|medical`

**Issue**: PII in responses | PII in URLs | PII logged | PII in analytics | No data minimization | GDPR violations

**Severity**: High (privacy violation, regulatory fines)

**Fix**: Return minimal data | Redact PII | Encrypt PII at rest | Data retention policies | User consent

**Evidence patterns**:
```
# Vulnerable
res.json(users);  // All user data including email, phone!
/api/users?email=user@example.com  // PII in URL (logs!)
analytics.track({ email, phone, ssn });  // PII to analytics
SELECT * FROM users;  // Fetching unnecessary PII
```

**Secure patterns**:
```
# Return only needed fields
res.json(users.map(u => ({ id: u.id, name: u.name })));

# Redact PII
const redacted = {
  ...user,
  email: user.email.replace(/(.{2}).*(@.*)/, '$1***$2'),
  phone: user.phone.replace(/.(?=.{4})/g, '*')
};

# Use IDs, not PII in URLs
/api/users/123  // Not /api/users?email=...

# Don't send PII to analytics
analytics.track({ userId: user.id });  // ID only, not PII

# Encrypt PII at rest
const crypto = require('crypto');
const encrypted = encrypt(user.ssn, encryptionKey);
```

**Ref**: CWE-359, A01:2021, GDPR

---

### 6. Sensitive Data in URLs

**Search**: `password=|token=|api.?key=|secret=|access.?token=|reset.*token|href.*password|window\.location.*password`

**Issue**: Secrets in query parameters | Tokens in URLs | URLs logged in server logs, browser history, analytics

**Severity**: High (credential exposure via logs)

**Fix**: Use POST body or headers | Use cookies for tokens | Avoid sensitive data in URLs

**Evidence patterns**:
```
# Vulnerable
/reset-password?token=abc123xyz  // Token in URL!
/api/user?api_key=secret123  // API key in URL!
https://api.example.com?password=secret  // Password in URL!
window.location = '/login?redirect=' + encodeURI(window.location.href + '?token=' + token);
```

**Secure patterns**:
```
# POST body (not URL)
fetch('/api/user', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`  // Header, not URL
  },
  body: JSON.stringify({ data })
});

# Cookies for tokens
res.cookie('resetToken', token, {
  httpOnly: true,
  secure: true,
  maxAge: 3600000
});

# For password reset, use POST
<form method="POST" action="/reset-password">
  <input type="hidden" name="token" value="{{ token }}">
  <input type="password" name="newPassword">
</form>
```

**Ref**: CWE-598, A04:2021

---

### 7. Unencrypted Sensitive Data at Rest

**Search**: `fs\.write|writeFile|\.save\(|INSERT INTO|to_sql|backup|dump|export`

**Issue**: PII stored in plaintext | Passwords not hashed | Credit cards unencrypted | Backups unencrypted | Database columns in cleartext

**Severity**: High (data breach if storage compromised)

**Fix**: Encrypt sensitive fields | Hash passwords (bcrypt) | Encrypt database | Encrypt backups | Full disk encryption

**Evidence patterns**:
```
# Vulnerable
fs.writeFileSync('backup.json', JSON.stringify(users));  // PII in plaintext!
INSERT INTO users VALUES (1, 'user@example.com', 'password123');  // Plaintext password!
db.users.insert({ ssn: '123-45-6789' });  // Unencrypted SSN
```

**Secure patterns**:
```
# Encrypt sensitive fields
const crypto = require('crypto');
function encrypt(text, key) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  const encrypted = Buffer.concat([cipher.update(text, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  return { encrypted, iv, tag };
}

# Hash passwords
const bcrypt = require('bcrypt');
const hashedPassword = await bcrypt.hash(password, 12);

# Encrypt database
# Enable encryption at rest in database settings
# PostgreSQL: pgcrypto extension
# MySQL: InnoDB encryption
# MongoDB: encrypted storage engine

# Encrypt backups
gpg --encrypt --recipient admin@example.com backup.sql
```

**Ref**: CWE-311, CWE-312, A02:2021

---

### 8. Insecure Backups

**Search**: `backup|dump|export|mysqldump|pg_dump|mongodump`

**Issue**: Backups unencrypted | Backups publicly accessible | Backup credentials in scripts | Old backups not deleted

**Severity**: High (full database exposure)

**Fix**: Encrypt backups | Secure storage | Rotate backups | Access controls

**Evidence patterns**:
```
# Vulnerable
mysqldump -u root -pPassword123 db > backup.sql  // Password in script!
aws s3 cp backup.sql s3://bucket --acl public-read  // Public backup!
pg_dump database > /var/www/html/backup.sql  // Web-accessible!
```

**Secure patterns**:
```
# Encrypted backups
pg_dump database | gpg --encrypt --recipient backup@example.com > backup.sql.gpg
mysqldump database | openssl enc -aes-256-cbc -salt -pbkdf2 > backup.sql.enc

# Secure storage (private S3)
aws s3 cp backup.sql.gpg s3://private-backups/ --sse AES256

# Use .pgpass or .my.cnf (not password in script)
# ~/.pgpass
hostname:port:database:username:password

# Automated with proper permissions
chmod 600 ~/.pgpass

# Rotate old backups
find /backups -name "*.sql.gpg" -mtime +30 -delete
```

**Ref**: CWE-312, CWE-522, A02:2021

---

### 9. Debug Information Disclosure

**Search**: `DEBUG|debug.*true|app\.debug|FLASK_DEBUG|NODE_ENV.*development|error_reporting.*E_ALL|display_errors.*On`

**Issue**: Debug mode in production | Verbose errors | Source code in errors | Framework debug panels accessible

**Severity**: Medium (information disclosure aids attacks)

**Fix**: Disable debug in production | Generic error pages | No stack traces to users

**Evidence patterns**:
```
# Vulnerable
DEBUG=true
NODE_ENV=development  // In production!
app.debug = True  // Flask
error_reporting(E_ALL);
display_errors = On

# Laravel debug page showing env vars
# Django debug page showing settings
```

**Secure patterns**:
```
# Production settings
NODE_ENV=production
DEBUG=false
FLASK_DEBUG=0
display_errors = Off
error_reporting = 0

# Environment-specific
if (process.env.NODE_ENV === 'production') {
  app.set('env', 'production');
  // Generic error handler
}

# Django
DEBUG = False
ALLOWED_HOSTS = ['example.com']

# Never commit debug settings
# Use environment variables
```

**Ref**: CWE-11, CWE-215, A05:2021

---

### 10. Insufficient Data Retention Controls

**Search**: `delete|DELETE|remove|expiry|ttl|retention|GDPR|data.*retention`

**Issue**: Data kept indefinitely | No deletion mechanism | GDPR violations | Soft deletes only | Old data not purged

**Severity**: Medium (privacy violation, compliance)

**Fix**: Data retention policies | Automated deletion | Hard deletes for PII | User data export/deletion

**Evidence patterns**:
```
# Vulnerable
UPDATE users SET deleted=true WHERE id=?;  // Soft delete only!
# No expiration on user data
# No mechanism to purge old data
# No GDPR delete endpoint
```

**Secure patterns**:
```
# Hard delete PII
DELETE FROM users WHERE id = ?;
DELETE FROM user_data WHERE user_id = ?;

# Automated retention
# Delete inactive accounts after 2 years
DELETE FROM users
WHERE last_login < NOW() - INTERVAL 2 YEAR
AND deleted = true;

# GDPR delete endpoint
app.delete('/api/user/me', requireAuth, async (req, res) => {
  await db.users.destroy({ where: { id: req.user.id }});
  await db.userData.destroy({ where: { userId: req.user.id }});
  res.json({ message: 'Account deleted' });
});

# Scheduled cleanup (cron)
0 2 * * * /scripts/cleanup-old-data.sh

# TTL indexes (MongoDB)
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }
);
```

**Ref**: CWE-359, GDPR Art. 17

---

