# Injection Vulnerabilities

## Scope

SQL injection | Command injection | NoSQL injection | LDAP injection | XML/XXE injection | Template injection | Code injection | Expression language injection | ORM injection

## Vulnerabilities

### 1. SQL Injection

**Search**: `execute\(.*\+.*\)|\.query\(.*\+|\.raw\(.*\+|SELECT.*\+.*FROM|WHERE.*\+|cursor\.execute\(.*%|f"SELECT|f'SELECT|\$\{.*\}.*SELECT`

**Issue**: User input concatenated into SQL queries | String formatting in queries | f-strings in SQL

**Severity**: Critical

**Fix**: Parameterized queries | ORM methods | Prepared statements

**Evidence patterns**:
```
# Vulnerable
query = "SELECT * FROM users WHERE id = " + user_id
db.execute(f"SELECT * FROM users WHERE name = '{name}'")
cursor.execute("SELECT * FROM users WHERE email = '%s'" % email)
```

**Secure patterns**:
```
# Parameterized (Python)
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))

# ORM (safe by default)
User.objects.filter(email=email)
db.query(User).filter(User.email == email)

# Prepared (Node.js)
db.query("SELECT * FROM users WHERE id = ?", [userId])
```

**Ref**: CWE-89, A03:2021

---

### 2. Command Injection

**Search**: `exec\(|eval\(|system\(|shell_exec|subprocess\.|os\.system|os\.popen|Runtime\.exec|Process\.|child_process|shell.*true`

**Issue**: User input passed to shell commands | shell=True with user input | Unescaped command arguments

**Severity**: Critical (RCE)

**Fix**: Avoid shell execution | Use array args | Whitelist input | Use safe APIs

**Evidence patterns**:
```
# Vulnerable
os.system("ping " + ip_address)
subprocess.call("ls " + directory, shell=True)
exec("import " + module_name)
child_process.exec(`convert ${filename} output.jpg`)
```

**Secure patterns**:
```
# Array args (no shell)
subprocess.run(["ping", "-c", "1", ip_address])
subprocess.run(["ls", directory], shell=False)

# Whitelist
allowed = ['user', 'admin', 'guest']
if role not in allowed:
    raise ValueError()
subprocess.run(["script.sh", role])

# Avoid exec/eval entirely
```

**Ref**: CWE-77, CWE-78, A03:2021

---

### 3. NoSQL Injection

**Search**: `db\.\w+\.find\(.*\$|collection\.find\(.*req\.|\.where\(.*JSON\.parse|MongoDB.*\$where|\$ne.*password`

**Issue**: User input in MongoDB queries | $where operator with user data | JSON.parse on user input | Unvalidated operators

**Severity**: Critical

**Fix**: Validate input types | Sanitize operators | Use parameterized queries | Avoid $where

**Evidence patterns**:
```
# Vulnerable
db.users.find({ username: req.body.username })  // Can inject {"$ne": null}
db.users.find(JSON.parse(req.body.query))
db.users.find({ $where: `this.username == '${username}'` })
```

**Secure patterns**:
```
# Type validation
const username = String(req.body.username)
db.users.findOne({ username })

# Sanitize operators
const sanitize = (obj) => {
  for (let key in obj) {
    if (key.startsWith('$')) delete obj[key]
  }
  return obj
}

# Use mongoose (validates by schema)
User.findOne({ username: req.body.username })
```

**Ref**: CWE-943, A03:2021

---

### 4. Template Injection (SSTI)

**Search**: `render_template_string|Jinja2.*fromstring|\.render\(.*request\.|{{.*}}.*user|template\.Template\(.*input|ejs\.render\(.*req\.|eval.*template`

**Issue**: User input in template strings | Server-side template with user data | Dynamic template compilation

**Severity**: Critical (RCE)

**Fix**: Never use user input in template strings | Use template files | Auto-escape enabled | Sandbox templates

**Evidence patterns**:
```
# Vulnerable (Python/Jinja2)
render_template_string(request.args.get('template'))
Template(user_input).render()

# Vulnerable (Node.js/EJS)
ejs.render(req.body.template, data)

# Vulnerable (Any)
{{user_input}}  // If user_input contains {{7*7}} → RCE
```

**Secure patterns**:
```
# Use template files, not strings
render_template('template.html', name=user_name)

# Auto-escape (enabled by default in Flask/Django)
{{ user_input }}  // Auto-escaped

# If dynamic templates needed, sandbox them
from jinja2.sandbox import SandboxedEnvironment
env = SandboxedEnvironment()
```

**Ref**: CWE-94, CWE-1336, A03:2021

---

### 5. LDAP Injection

**Search**: `ldap\.|LdapContext|ldapsearch|cn=.*\+.*|filter.*\+|DirectorySearcher`

**Issue**: User input concatenated in LDAP filters | Unescaped special chars

**Severity**: High

**Fix**: Escape LDAP special chars: `* ( ) \ NUL` | Use parameterized filters

**Evidence patterns**:
```
# Vulnerable
filter = "(cn=" + username + ")"
filter = f"(uid={user_id})"
```

**Secure patterns**:
```
# Python
import ldap
username = ldap.filter.escape_filter_chars(username)
filter = f"(cn={username})"

# Java
String filter = "cn=" + escapeForLDAP(username);
```

**Ref**: CWE-90, A03:2021

---

### 6. XML/XXE Injection

**Search**: `etree\.parse|xml\.etree|parseXML|DocumentBuilder|XMLReader|ENTITY|DOCTYPE.*ENTITY|lxml|xmltodict`

**Issue**: External entity processing enabled | DTD processing enabled | Unsafe XML parser config

**Severity**: High (file disclosure, SSRF, DoS)

**Fix**: Disable external entities | Disable DTD processing | Use safe parser config

**Evidence patterns**:
```
# Vulnerable (Python)
import xml.etree.ElementTree as ET
tree = ET.parse(user_file)  // XXE vulnerable

# Vulnerable XML
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<data>&xxe;</data>
```

**Secure patterns**:
```
# Python (lxml - safe)
from lxml import etree
parser = etree.XMLParser(resolve_entities=False, no_network=True)
tree = etree.parse(user_file, parser)

# Python (defusedxml)
import defusedxml.ElementTree as ET
tree = ET.parse(user_file)

# Java
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)
factory.setFeature("http://xml.org/sax/features/external-general-entities", false)
```

**Ref**: CWE-611, A05:2021

---

### 7. Code Injection

**Search**: `eval\(|exec\(|Function\(.*req\.|execfile\(|__import__\(|compile\(.*input`

**Issue**: eval() with user input | exec() with user data | Dynamic code compilation | Dynamic imports

**Severity**: Critical (RCE)

**Fix**: Never use eval/exec with user input | Use safe alternatives | Whitelist approach

**Evidence patterns**:
```
# Vulnerable
eval(request.body.expression)
exec("result = " + user_code)
new Function(req.body.code)()
```

**Secure patterns**:
```
# Don't use eval/exec
# For math expressions, use safe library
import ast
import operator

def safe_eval(expr):
    ops = {ast.Add: operator.add, ast.Sub: operator.sub}
    # Parse and validate AST

# For config, use JSON
config = json.loads(user_input)
```

**Ref**: CWE-94, CWE-95, A03:2021

---

### 8. ORM Injection

**Search**: `\.raw\(|\.extra\(|RawSQL|execute_sql|query\(.*\+|order_by\(.*request\.|\.annotate\(.*RawSQL`

**Issue**: Raw SQL in ORM with user input | User-controlled order_by | Dynamic table/column names

**Severity**: High

**Fix**: Use ORM methods | Whitelist column names | Avoid raw SQL with user input

**Evidence patterns**:
```
# Vulnerable (Django)
User.objects.raw("SELECT * FROM users WHERE id = " + user_id)
User.objects.order_by(request.GET.get('sort'))  // Can inject: "-id; DROP TABLE--"
User.objects.extra(where=[f"status = '{status}'"])

# Vulnerable (SQLAlchemy)
session.execute(f"SELECT * FROM users WHERE name = '{name}'")
```

**Secure patterns**:
```
# Use ORM methods
User.objects.filter(id=user_id)

# Whitelist columns
ALLOWED_SORT = ['name', 'email', 'created_at']
sort = request.GET.get('sort')
if sort in ALLOWED_SORT:
    User.objects.order_by(sort)

# Parameterize raw queries
User.objects.raw("SELECT * FROM users WHERE id = %s", [user_id])
```

**Ref**: CWE-89, A03:2021

---

### 9. Path Traversal

**Search**: `open\(.*request\.|readFile\(.*req\.|\.\.\/|path.*\+.*input|os\.path\.join\(.*input|File\(.*request`

**Issue**: User input in file paths | No path validation | Allows ../ sequences

**Severity**: High (file disclosure)

**Fix**: Whitelist files | Validate no ../ | Use safe path joining | Check resolved path in allowed dir

**Evidence patterns**:
```
# Vulnerable
open("/var/data/" + filename)
fs.readFile("./uploads/" + req.body.file)
os.path.join(base_dir, user_path)  // Still vulnerable!
```

**Secure patterns**:
```
# Python
import os
from pathlib import Path

base = Path("/var/data")
user_file = base / filename
if not user_file.resolve().is_relative_to(base):
    raise ValueError("Invalid path")

# Node.js
const path = require('path');
const safeJoin = require('path').join;
const filepath = path.join(__dirname, 'uploads', req.body.file);
if (!filepath.startsWith(path.join(__dirname, 'uploads'))) {
    throw new Error('Invalid path');
}

# Whitelist
ALLOWED_FILES = ['report.pdf', 'data.csv']
if filename not in ALLOWED_FILES:
    raise ValueError()
```

**Ref**: CWE-22, A01:2021

---

## Review Process

1. **SQL**: Grep for query builders → check for string concat | Look for raw(), execute() with user input
2. **Command**: Grep for exec, system, subprocess → verify no shell=True | Check for array args
3. **NoSQL**: Find MongoDB queries → check for $operators from user input | Look for JSON.parse
4. **Template**: Find template rendering → verify no user input in template strings
5. **LDAP**: Search ldap usage → check for filter escaping
6. **XML**: Find XML parsers → verify external entities disabled
7. **Code**: Search eval/exec → verify never used with user input
8. **ORM**: Find .raw() calls → check parameterization | Check order_by() for whitelisting
9. **Path**: Find file operations → verify path validation | Check for ../ prevention

## Report Format

```
**[SEVERITY] SQL Injection in User Search**
- **Category**: Injection
- **Location**: `api/users.py:45`
- **Issue**: User input concatenated into SQL query
- **Risk**: Attacker can extract entire database, modify data, or bypass authentication
- **Evidence**: `query = "SELECT * FROM users WHERE name = '" + name + "'"`
- **Fix**: Use parameterized query: `cursor.execute("SELECT * FROM users WHERE name = ?", (name,))`
- **Reference**: CWE-89, A03:2021
```

## Common References

CWE-22 (path traversal) | CWE-77 (command injection) | CWE-78 (OS command injection) | CWE-89 (SQL injection) | CWE-90 (LDAP injection) | CWE-94 (code injection) | CWE-95 (eval injection) | CWE-611 (XXE) | CWE-943 (NoSQL injection) | CWE-1336 (template injection) | A03:2021 (Injection)
