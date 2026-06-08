# Database Security

## Scope

Database permissions | Row-level security (RLS) | Default credentials | Connection strings | Database configuration | Backup security | Privilege escalation | Unencrypted connections | Exposed ports | Query logging | Database roles

## Vulnerabilities

### 1. Overly Permissive Database User

**Search**: `GRANT|CREATE USER|database.*user|db.*user|connection.*string|DATABASE_URL|DB_USER|POSTGRES_USER|MYSQL_USER`

**Issue**: Application using admin/root user | GRANT ALL | Excessive permissions | No principle of least privilege | Single user for all operations

**Severity**: High (privilege escalation, data breach)

**Fix**: Dedicated users per service | Minimal permissions | Separate read/write users | No superuser for app

**Evidence patterns**:
```
# Vulnerable - using root/admin
DATABASE_URL=postgres://postgres:password@localhost/db
MYSQL_USER=root
MYSQL_PASSWORD=password

# App connects as superuser
const db = new Client({
  user: 'postgres',  // Superuser!
  password: 'admin'
});

# GRANT ALL
GRANT ALL PRIVILEGES ON *.* TO 'appuser'@'%';
```

**Secure patterns**:
```
# Dedicated user with minimal permissions
CREATE USER appuser WITH PASSWORD 'strong_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE users TO appuser;
GRANT USAGE ON SCHEMA public TO appuser;

# Read-only user for analytics
CREATE USER readonly WITH PASSWORD 'strong_password';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

# Connection string with limited user
DATABASE_URL=postgres://appuser:password@localhost/db

# Different users for different services
write_user: SELECT, INSERT, UPDATE, DELETE
read_user: SELECT only
backup_user: SELECT, pg_dump
migration_user: CREATE, ALTER, DROP (CI/CD only)
```

**Ref**: CWE-250, CWE-269, A01:2021

---

### 2. Missing Row-Level Security (RLS)

**Search**: `CREATE POLICY|ROW LEVEL SECURITY|RLS|ENABLE ROW LEVEL|SELECT.*WHERE.*user_id|multi.tenant`

**Issue**: No row-level policies | Users can access other users' data | Multi-tenant data leakage | Application-level filtering only

**Severity**: Critical (unauthorized data access)

**Fix**: Enable RLS | Create policies | Database-enforced isolation | Per-user/tenant policies

**Evidence patterns**:
```
# Vulnerable - app-level filtering only
SELECT * FROM documents WHERE user_id = ?  // App must remember WHERE clause!

# Multi-tenant without RLS
SELECT * FROM data WHERE tenant_id = ?  // Easy to forget!

# No RLS enabled
# Check: SELECT tablename, rowsecurity FROM pg_tables;
# rowsecurity = false
```

**Secure patterns**:
```
# Enable RLS (PostgreSQL)
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

# Policy: users see only their own rows
CREATE POLICY user_isolation ON documents
  FOR ALL
  USING (user_id = current_user_id());

# Multi-tenant policy
CREATE POLICY tenant_isolation ON data
  FOR ALL
  USING (tenant_id = current_setting('app.tenant_id')::int);

# Set tenant context per request
SET LOCAL app.tenant_id = 123;
SELECT * FROM data;  // Automatically filtered!

# Supabase (RLS built-in)
CREATE POLICY "Users can only access their own data"
  ON documents FOR ALL
  USING (auth.uid() = user_id);
```

**Ref**: CWE-639, A01:2021

---

### 3. Default Database Credentials

**Search**: `postgres:postgres|root:root|admin:admin|sa:sa|mysql.*password.*=|POSTGRES_PASSWORD|MYSQL_ROOT_PASSWORD`

**Issue**: Default passwords unchanged | Weak passwords | Same password across environments | Credentials in docker-compose.yml

**Severity**: Critical (unauthorized access)

**Fix**: Strong unique passwords | Rotate credentials | Use secrets management | Never commit passwords

**Evidence patterns**:
```
# Vulnerable
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres

MYSQL_ROOT_PASSWORD=password
MYSQL_USER=admin
MYSQL_PASSWORD=admin

# Default MongoDB (no auth enabled)
mongod --noauth

# SQL Server default SA account
sa / password123
```

**Secure patterns**:
```
# Strong random passwords
POSTGRES_PASSWORD=$(openssl rand -base64 32)
MYSQL_ROOT_PASSWORD=$(openssl rand -base64 32)

# Environment-specific
# .env.production
DB_PASSWORD=<strong-unique-production-password>

# .env.development (different password!)
DB_PASSWORD=<dev-password>

# Docker secrets
secrets:
  db_password:
    file: ./secrets/db_password.txt

environment:
  POSTGRES_PASSWORD_FILE: /run/secrets/db_password

# Disable default accounts
# PostgreSQL: no default postgres password needed with peer auth
# MySQL: DELETE FROM mysql.user WHERE User='root' AND Host='%';
# MongoDB: createUser with strong password
```

**Ref**: CWE-798, A07:2021

---

### 4. Connection String Exposure

**Search**: `DATABASE_URL|CONNECTION_STRING|mongodb:|postgres:|mysql:|jdbc:|Server=|password=|pwd=`

**Issue**: Connection strings in code | Passwords in URLs | Connection strings logged | Committed to git

**Severity**: Critical (credential exposure)

**Fix**: Environment variables | Secrets manager | Never log connection strings | Parse and redact

**Evidence patterns**:
```
# Vulnerable - hardcoded
const db = 'postgres://user:password@host/db';

# In code
DATABASE_URL=postgres://admin:secret123@db.example.com/prod

# Logged
console.log('Connecting to:', process.env.DATABASE_URL);
// Logs password!

# Committed
git log -p | grep DATABASE_URL
```

**Secure patterns**:
```
# Environment variable
DATABASE_URL=postgres://user:password@host/db
const db = process.env.DATABASE_URL;

# Parse and redact for logging
const url = new URL(process.env.DATABASE_URL);
const safe = `${url.protocol}//${url.username}:***@${url.host}${url.pathname}`;
console.log('Connecting to:', safe);

# Use connection config object (no URL)
const db = new Client({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});

# Secrets manager
const secret = await secretsManager.getSecretValue('db-password');
const password = JSON.parse(secret.SecretString).password;
```

**Ref**: CWE-312, CWE-522, A02:2021

---

### 5. Unencrypted Database Connections

**Search**: `sslmode|ssl.*=.*false|encrypt.*=.*false|useSSL|requireSSL|pg.*ssl|mysql.*ssl`

**Issue**: No SSL/TLS for database connections | Credentials in plaintext over network | MITM attacks

**Severity**: High (credential interception)

**Fix**: Require SSL/TLS | Verify certificates | Encrypted connections only

**Evidence patterns**:
```
# Vulnerable - no SSL
postgres://user:pass@host/db  // No sslmode
mysql://user:pass@host/db?useSSL=false

# Disabled SSL
const db = new Client({
  ssl: false
});

# MongoDB without TLS
mongodb://user:pass@host/db  // No tls=true
```

**Secure patterns**:
```
# PostgreSQL - require SSL
postgres://user:pass@host/db?sslmode=require
# or verify-full for cert verification
?sslmode=verify-full&sslrootcert=/path/to/ca.crt

const db = new Client({
  ssl: {
    rejectUnauthorized: true,
    ca: fs.readFileSync('/path/to/ca.crt')
  }
});

# MySQL
mysql://user:pass@host/db?useSSL=true

# MongoDB
mongodb://user:pass@host/db?tls=true&tlsCAFile=/path/to/ca.pem

# Force SSL in database config
# PostgreSQL postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'

# MySQL my.cnf
require_secure_transport = ON
```

**Ref**: CWE-319, A02:2021

---

### 6. Database Exposed to Internet

**Search**: `0\.0\.0\.0|bind.*address|listen.*addresses|port.*5432|port.*3306|port.*27017|port.*1433`

**Issue**: Database listening on 0.0.0.0 | Publicly accessible | No firewall | Default ports exposed

**Severity**: Critical (unauthorized access)

**Fix**: Bind to localhost/private IP | Firewall rules | VPN/private network | Non-default ports

**Evidence patterns**:
```
# Vulnerable - PostgreSQL
listen_addresses = '*'  // All interfaces!
port = 5432  // Default port

# MySQL
bind-address = 0.0.0.0
port = 3306

# MongoDB
bindIp: 0.0.0.0
port: 27017

# docker-compose.yml
ports:
  - "5432:5432"  // Exposed to host (and possibly internet!)
```

**Secure patterns**:
```
# Localhost only
# PostgreSQL
listen_addresses = 'localhost'

# MySQL
bind-address = 127.0.0.1

# MongoDB
bindIp: 127.0.0.1

# Private IP only (if multi-server)
listen_addresses = '10.0.1.5'
bind-address = 192.168.1.10

# Docker - no host port binding
# Don't expose port to host, use Docker networks
services:
  db:
    # No ports section
    networks:
      - internal

# Firewall (UFW)
ufw deny 5432
ufw allow from 192.168.0.0/24 to any port 5432

# Non-default ports (obscurity, not security)
port = 54320  // Not 5432
```

**Ref**: CWE-200, CWE-668, A05:2021

---

### 7. Weak Database Configuration

**Search**: `postgresql\.conf|my\.cnf|mongod\.conf|log_statement|general_log|audit|max_connections`

**Issue**: No query logging | Weak auth settings | No connection limits | Missing audit logs | Insecure defaults

**Severity**: Medium (makes attacks harder to detect)

**Fix**: Enable logging | Audit trails | Connection limits | Secure defaults

**Evidence patterns**:
```
# PostgreSQL - no logging
log_statement = 'none'

# MySQL - no general log
general_log = 0

# MongoDB - no auth
security:
  authorization: disabled

# No connection limits
max_connections = unlimited
```

**Secure patterns**:
```
# PostgreSQL logging
log_statement = 'all'  // Or 'ddl', 'mod'
log_connections = on
log_disconnections = on
log_duration = on
log_line_prefix = '%t [%p] %u@%d '

# Connection limits
max_connections = 100

# Statement timeout (prevent long-running queries)
statement_timeout = 30000  // 30 seconds

# MySQL
general_log = 1
general_log_file = /var/log/mysql/general.log
slow_query_log = 1
long_query_time = 2

# MongoDB
security:
  authorization: enabled
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log

# PostgreSQL pg_hba.conf - restrict access
host all all 127.0.0.1/32 scram-sha-256
host all all ::1/128 scram-sha-256
# No 'trust' auth in production!
```

**Ref**: CWE-778, A09:2021

---

### 8. Missing Database Backups / Insecure Backups

**Search**: `pg_dump|mysqldump|mongodump|backup|dump.*sql|\.sql$|\.bak$`

**Issue**: No backups | Backups unencrypted | Backups publicly accessible | No backup testing | Credentials in backup scripts

**Severity**: High (data loss, credential exposure)

**Fix**: Automated encrypted backups | Secure storage | Test restores | Credential management

**Evidence patterns**:
```
# Vulnerable
pg_dump -U postgres -W password db > backup.sql  // Password in command!
mysqldump --password=secret db > backup.sql  // Password visible!

# Backup in web root
cp db.sql /var/www/html/backup.sql  // Publicly downloadable!

# No encryption
pg_dump db > backup.sql  // Plaintext!
```

**Secure patterns**:
```
# Use .pgpass (no password in command)
# ~/.pgpass
hostname:port:database:username:password
chmod 600 ~/.pgpass

pg_dump -h hostname -U username database > backup.sql

# Encrypt backup
pg_dump database | gpg --encrypt --recipient backup@example.com > backup.sql.gpg
mysqldump database | openssl enc -aes-256-cbc -salt -pbkdf2 > backup.sql.enc

# Secure storage
aws s3 cp backup.sql.gpg s3://private-backups/ --sse AES256

# Automated backups (cron)
0 2 * * * /scripts/backup-db.sh

# Test restores regularly
pg_restore -d test_db backup.dump

# Retention policy
find /backups -name "*.sql.gpg" -mtime +30 -delete
```

**Ref**: CWE-312, CWE-522, A05:2021

---

### 9. SQL Injection in Dynamic Queries

**Search**: `EXECUTE|exec|sp_executesql|query.*\+|\.raw\(|execute.*format|string.*format.*SELECT`

**Issue**: Dynamic SQL with string concatenation | EXECUTE with user input | Format strings in queries | Server-side injection

**Severity**: Critical (data breach, RCE)

**Fix**: Parameterized queries always | Avoid dynamic SQL | Use ORM | Whitelist table/column names

**Evidence patterns**:
```
# Vulnerable - stored procedure
EXECUTE('SELECT * FROM users WHERE id = ' + @userId);

# Dynamic table name
query = "SELECT * FROM " + table_name + " WHERE id = ?";

# Format string
query = format("SELECT * FROM %s WHERE id = %s", table, id);
```

**Secure patterns**:
```
# Parameterized (SQL Server)
EXEC sp_executesql N'SELECT * FROM users WHERE id = @id',
  N'@id INT',
  @id = @userId;

# PostgreSQL prepared statements
PREPARE stmt (int) AS SELECT * FROM users WHERE id = $1;
EXECUTE stmt(user_id);

# Whitelist table names
const ALLOWED_TABLES = ['users', 'products', 'orders'];
if (!ALLOWED_TABLES.includes(table)) {
  throw new Error('Invalid table');
}

# Use identifier escaping (last resort)
const table = pgformat.ident(tableName);
const query = `SELECT * FROM ${table} WHERE id = $1`;
```

**Ref**: CWE-89, A03:2021

---

### 10. Database Function/Procedure Vulnerabilities

**Search**: `CREATE FUNCTION|CREATE PROCEDURE|SECURITY DEFINER|EXECUTE.*AS|xp_cmdshell|OPENROWSET`

**Issue**: Functions with SECURITY DEFINER | SQL injection in functions | Command execution functions enabled | Excessive function permissions

**Severity**: High (privilege escalation, RCE)

**Fix**: SECURITY INVOKER | Parameterize function queries | Disable dangerous functions | Minimal permissions

**Evidence patterns**:
```
# Vulnerable - SECURITY DEFINER (runs as owner)
CREATE FUNCTION get_user(user_id INT)
RETURNS TABLE(...)
SECURITY DEFINER  // Runs with elevated privileges!
AS $$
  SELECT * FROM users WHERE id = user_id;  // SQL injection if not careful!
$$ LANGUAGE SQL;

# SQL Server - xp_cmdshell enabled
EXEC sp_configure 'xp_cmdshell', 1;  // OS command execution!

# PostgreSQL - copy from program
COPY users FROM PROGRAM 'cat /etc/passwd';  // RCE if user has access!
```

**Secure patterns**:
```
# SECURITY INVOKER (runs as caller)
CREATE FUNCTION get_user(user_id INT)
RETURNS TABLE(...)
SECURITY INVOKER  // Uses caller's permissions
AS $$
  SELECT * FROM users WHERE id = user_id;
$$ LANGUAGE SQL;

# Parameterized function
CREATE FUNCTION search_users(search_term TEXT)
RETURNS TABLE(...)
AS $$
  SELECT * FROM users WHERE name ILIKE '%' || search_term || '%';
  -- Still need to sanitize search_term!
$$ LANGUAGE SQL;

# Disable dangerous functions
# SQL Server
EXEC sp_configure 'xp_cmdshell', 0;

# PostgreSQL
REVOKE EXECUTE ON FUNCTION pg_read_file FROM PUBLIC;
REVOKE EXECUTE ON FUNCTION pg_ls_dir FROM PUBLIC;

# Minimal function permissions
REVOKE ALL ON FUNCTION sensitive_function() FROM PUBLIC;
GRANT EXECUTE ON FUNCTION sensitive_function() TO admin_role;
```

**Ref**: CWE-250, CWE-89, A03:2021

---

## Review Process

1. **Permissions**: Check database users → verify minimal privileges, no superuser for app
2. **RLS**: Check for row-level security policies → verify enabled for multi-tenant
3. **Credentials**: Search for default passwords → verify strong unique passwords
4. **Connection strings**: Find DATABASE_URL → verify not hardcoded, not logged
5. **Encryption**: Check SSL settings → verify sslmode=require or equivalent
6. **Exposure**: Check listen addresses → verify localhost/private IP only
7. **Configuration**: Review postgresql.conf/my.cnf → verify logging enabled, secure defaults
8. **Backups**: Find backup scripts → verify encrypted, secure storage
9. **Dynamic SQL**: Search EXECUTE, .raw() → verify parameterized queries
10. **Functions**: Find SECURITY DEFINER functions → verify necessary, safe

## Report Format

```
**[CRITICAL] Database User Has Excessive Permissions**
- **Category**: Database Security
- **Location**: `config/database.js:12`
- **Issue**: Application connects as postgres superuser
- **Risk**: If application is compromised, attacker has full database control, can read all data, drop tables, create backdoors
- **Evidence**: `user: 'postgres', password: process.env.DB_PASSWORD`
- **Fix**: Create dedicated user with minimal permissions: `CREATE USER appuser; GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE users TO appuser;`
- **Reference**: CWE-250, A01:2021
```

## Common References

CWE-89 (SQL injection) | CWE-200 (information exposure) | CWE-250 (execution with unnecessary privileges) | CWE-269 (improper privilege management) | CWE-312 (cleartext storage) | CWE-319 (cleartext transmission) | CWE-522 (insufficiently protected credentials) | CWE-639 (insecure direct object reference) | CWE-668 (exposure of resource) | CWE-778 (insufficient logging) | CWE-798 (hardcoded credentials) | A01:2021 (Broken Access Control) | A02:2021 (Crypto Failures) | A03:2021 (Injection) | A05:2021 (Security Misconfiguration) | A07:2021 (Auth Failures) | A09:2021 (Logging Failures)
