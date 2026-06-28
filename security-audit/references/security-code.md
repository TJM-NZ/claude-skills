# Code Security Patterns

## Scope

Race conditions | Logic flaws | Business logic bugs | Integer overflow | Resource exhaustion | Error handling | Insecure deserialization | Memory leaks | TOCTOU | Unsafe reflection | ReDoS | Input validation | Type confusion

## Vulnerabilities

### 1. Race Conditions

**Search**: `async|await|Promise|thread|lock|mutex|synchronized|concurrent|parallel|setTimeout|setInterval`

**Issue**: Concurrent access without locking | TOCTOU vulnerabilities | State changes between check and use | Non-atomic operations

**Severity**: High (data corruption, privilege escalation)

**Fix**: Use locks/mutexes | Atomic operations | Transactions | Avoid shared mutable state

**Evidence patterns**:
```
# Vulnerable - check then use
if (user.balance >= amount) {
  // Another request could change balance here!
  user.balance -= amount;
  await user.save();
}

# File TOCTOU
if (fs.existsSync(file)) {
  // File could be deleted/replaced here
  const content = fs.readFileSync(file);
}

# Concurrent counter
counter++;  // Not atomic!
```

**Secure patterns**:
```
# Database transaction
await db.transaction(async (trx) => {
  const user = await trx('users')
    .where({ id })
    .forUpdate()  // Row-level lock
    .first();

  if (user.balance >= amount) {
    await trx('users')
      .where({ id })
      .decrement('balance', amount);
  }
});

# Atomic operations
await User.findByIdAndUpdate(id, {
  $inc: { balance: -amount }  // Atomic increment
});

# Mutex for in-memory state
const mutex = new Mutex();
const release = await mutex.acquire();
try {
  counter++;
} finally {
  release();
}

# Avoid TOCTOU - do operation, handle error
try {
  const content = fs.readFileSync(file);
} catch (err) {
  // Handle missing file
}
```

**Ref**: CWE-362, CWE-367, A04:2021

---

### 2. Business Logic Flaws

**Search**: `price|discount|quantity|balance|credit|refund|coupon|promo|cart|checkout`

**Issue**: Price manipulation | Negative quantities | Discount stacking | Insufficient validation | Trust client-side calculations | Missing business rules

**Severity**: High (financial loss)

**Fix**: Server-side validation | Verify calculations | Enforce business rules | Never trust client

**Evidence patterns**:
```
# Vulnerable - client sets price
const order = {
  items: req.body.items,  // [{id: 1, price: 0.01}] - Client controls price!
  total: req.body.total
};

# Negative quantity
const quantity = parseInt(req.body.quantity);  // Could be -1!
user.balance += price * quantity;  // Adds money!

# No refund limit
if (req.body.action === 'refund') {
  user.balance += amount;  // Unlimited refunds!
}
```

**Secure patterns**:
```
# Server calculates price
const items = await Promise.all(
  req.body.items.map(async (item) => {
    const product = await Product.findById(item.id);
    return {
      id: product.id,
      price: product.price,  // Server price, not client!
      quantity: item.quantity
    };
  })
);
const total = items.reduce((sum, item) =>
  sum + (item.price * item.quantity), 0
);

# Validate quantity
const quantity = parseInt(req.body.quantity);
if (quantity < 1 || quantity > 100) {
  return res.status(400).json({ error: 'Invalid quantity' });
}

# Refund limits
const order = await Order.findById(orderId);
if (order.refunded) {
  return res.status(400).json({ error: 'Already refunded' });
}
order.refunded = true;
await order.save();
```

**Ref**: CWE-840, A04:2021

---

### 3. Insecure Deserialization

**Search**: `pickle|unserialize|yaml\.load|JSON\.parse|ObjectInputStream|readObject|deserialize|unmarshall`

**Issue**: Deserializing untrusted data | Code execution via gadget chains | YAML load without safe loader | pickle exploits

**Severity**: Critical (RCE)

**Fix**: Avoid deserializing untrusted data | Use safe loaders | Validate before deserialize | Sign/encrypt serialized data

**Evidence patterns**:
```
# Vulnerable - Python pickle
import pickle
data = pickle.loads(untrusted_data)  // RCE!

# YAML unsafe load
import yaml
config = yaml.load(user_input)  // Code execution!

# Java deserialization
ObjectInputStream ois = new ObjectInputStream(untrusted_stream);
Object obj = ois.readObject();  // Gadget chain RCE!

# PHP unserialize
$obj = unserialize($_COOKIE['data']);  // RCE!
```

**Secure patterns**:
```
# Use JSON instead of pickle
import json
data = json.loads(untrusted_data)

# YAML safe load
import yaml
config = yaml.safe_load(user_input)  // No code execution

# Validate before deserialize
if (validateSignature(data, signature)) {
  obj = deserialize(data);
}

# Use typed DTOs, not arbitrary objects
class UserDTO {
  @IsString() name: string;
  @IsEmail() email: string;
}
const user = plainToClass(UserDTO, JSON.parse(input));
validate(user);

# Avoid deserialization of untrusted data entirely
```

**Ref**: CWE-502, A08:2021

---

### 4. Regular Expression DoS (ReDoS)

**Search**: `new RegExp|regex|regexp|test\(|match\(|exec\(|replace\(.*\/.*\+.*\*|\.+.*\*`

**Issue**: Catastrophic backtracking | Nested quantifiers | ReDoS attack | Exponential time complexity

**Severity**: High (DoS)

**Fix**: Avoid nested quantifiers | Use non-backtracking regex | Timeout regex | Validate input length

**Evidence patterns**:
```
# Vulnerable - nested quantifiers
const regex = /^(a+)+$/;  // Catastrophic backtracking!
regex.test('aaaaaaaaaaaaaaaaaaaaX');  // Hangs!

const regex = /(a|a)*$/;  // Also vulnerable
const regex = /(a|ab)*$/;  // Vulnerable

# Email regex (common ReDoS)
/^([a-zA-Z0-9._%-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})*$/
```

**Secure patterns**:
```
# Avoid nested quantifiers
const regex = /^a+$/;  // No nesting

# Use atomic groups (if supported)
const regex = /^(?>a+)+$/;

# Validate length first
if (input.length > 1000) {
  return res.status(400).json({ error: 'Input too long' });
}

# Use specialized parsers
const validator = require('validator');
validator.isEmail(email);  // Not regex-based

# Timeout regex (Node.js)
const safeRegex = require('safe-regex');
if (!safeRegex(pattern)) {
  throw new Error('Unsafe regex');
}

# Test for ReDoS
npm install -g redos-detector
redos-detector "^(a+)+$"
```

**Ref**: CWE-1333, A05:2021

---

### 5. Integer Overflow / Underflow

**Search**: `parseInt|parseFloat|int\(|Integer\.|Long\.|BigInteger|Math\.|price|quantity|balance|total`

**Issue**: Arithmetic overflow | Negative wrap-around | Type coercion | Unbounded multiplication

**Severity**: Medium (logic bugs, DoS)

**Fix**: Validate ranges | Use BigInt for large numbers | Check before arithmetic | Type validation

**Evidence patterns**:
```
# Vulnerable - overflow
const price = 2147483647;  // Max int32
const quantity = 2;
const total = price * quantity;  // Overflows to negative!

# Underflow
const balance = 0;
const withdrawal = 1;
balance -= withdrawal;  // -1 (could bypass balance checks)

# Type coercion
const quantity = req.body.quantity;  // "1e100"
const total = price * quantity;  // Infinity!
```

**Secure patterns**:
```
# Validate ranges
const quantity = parseInt(req.body.quantity);
if (!Number.isInteger(quantity) || quantity < 1 || quantity > 1000) {
  return res.status(400).json({ error: 'Invalid quantity' });
}

# Check before arithmetic
const MAX_SAFE = Number.MAX_SAFE_INTEGER;
if (price > MAX_SAFE / quantity) {
  return res.status(400).json({ error: 'Value too large' });
}

# Use BigInt for large numbers
const price = BigInt(req.body.price);
const quantity = BigInt(req.body.quantity);
const total = price * quantity;

# Database constraints
ALTER TABLE orders ADD CONSTRAINT check_quantity
CHECK (quantity >= 0 AND quantity <= 1000);
```

**Ref**: CWE-190, CWE-191, A04:2021

---

### 6. Improper Error Handling

**Search**: `catch|except|try|error|exception|finally|throw|raise`

**Issue**: Empty catch blocks | Swallowed exceptions | Generic error handling | No logging | Silent failures

**Severity**: Medium (bugs hidden, security issues masked)

**Fix**: Log errors | Handle specifically | Fail securely | Never swallow exceptions

**Evidence patterns**:
```
# Vulnerable - empty catch
try {
  criticalOperation();
} catch (err) {
  // Silent failure!
}

# Generic catch-all
try {
  operation();
} catch {
  return 'Error';  // What error? No logging!
}

# Continue on error
files.forEach(file => {
  try {
    process(file);
  } catch (err) {
    // Skip this file, continue - could miss critical errors
  }
});
```

**Secure patterns**:
```
# Log and handle
try {
  criticalOperation();
} catch (err) {
  logger.error('Critical operation failed:', err);
  throw err;  // Or handle appropriately
}

# Specific error handling
try {
  operation();
} catch (err) {
  if (err instanceof ValidationError) {
    return res.status(400).json({ error: err.message });
  } else if (err instanceof AuthError) {
    return res.status(401).json({ error: 'Unauthorized' });
  } else {
    logger.error('Unexpected error:', err);
    return res.status(500).json({ error: 'Internal error' });
  }
}

# Fail securely
try {
  verifySignature(data, signature);
  process(data);
} catch (err) {
  logger.error('Signature verification failed:', err);
  return res.status(403).json({ error: 'Invalid signature' });
  // Don't process data on error!
}
```

**Ref**: CWE-390, CWE-755, A04:2021

---

### 7. Unsafe Reflection / Dynamic Code

**Search**: `eval\(|Function\(|require\(.*input|import\(.*req\.|getattr|setattr|__import__|exec\(|compile\(`

**Issue**: Dynamic require/import with user input | Unsafe getattr/setattr | eval alternatives | Code injection

**Severity**: Critical (RCE)

**Fix**: Whitelist allowed modules | Avoid dynamic code execution | Validate input strictly

**Evidence patterns**:
```
# Vulnerable - dynamic require
const module = require(req.query.module);  // RCE!

# Dynamic import
const lib = await import(`./plugins/${userInput}.js`);

# Python getattr
method = getattr(obj, user_input)  // Can access private methods!
method()

# Eval alternatives
const fn = new Function(userCode)();  // Still RCE!
```

**Secure patterns**:
```
# Whitelist modules
const ALLOWED_MODULES = ['lodash', 'moment', 'axios'];
const moduleName = req.query.module;
if (!ALLOWED_MODULES.includes(moduleName)) {
  return res.status(400).json({ error: 'Invalid module' });
}
const module = require(moduleName);

# Whitelist methods
const ALLOWED_METHODS = ['getData', 'setData'];
const method = req.body.method;
if (!ALLOWED_METHODS.includes(method)) {
  return res.status(400).json({ error: 'Invalid method' });
}
obj[method]();

# Avoid dynamic code entirely
# Use configuration objects instead
const handlers = {
  'getData': () => obj.getData(),
  'setData': () => obj.setData()
};
handlers[req.body.method]();
```

**Ref**: CWE-470, CWE-95, A03:2021

---

### 8. Resource Exhaustion

**Search**: `while|for|recursion|setTimeout|setInterval|map\(|filter\(|reduce\(|memory|cpu|disk`

**Issue**: Unbounded loops | Infinite recursion | Memory leaks | Large allocations | No timeouts | CPU-intensive operations

**Severity**: High (DoS)

**Fix**: Limit iterations | Timeout operations | Validate input size | Rate limiting | Async with limits

**Evidence patterns**:
```
# Vulnerable - unbounded loop
const items = req.body.items;  // Could be millions!
for (const item of items) {
  process(item);  // No limit!
}

# Infinite recursion
function factorial(n) {
  return n * factorial(n - 1);  // No base case!
}

# Large allocation
const buffer = Buffer.alloc(req.body.size);  // Could allocate GB!

# Regex backtracking (see ReDoS)
```

**Secure patterns**:
```
# Limit iterations
const MAX_ITEMS = 1000;
if (req.body.items.length > MAX_ITEMS) {
  return res.status(400).json({ error: 'Too many items' });
}

# Timeout operations
const timeoutPromise = (promise, ms) => {
  return Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
};
await timeoutPromise(longOperation(), 5000);

# Validate allocation size
const MAX_SIZE = 10 * 1024 * 1024;  // 10MB
if (req.body.size > MAX_SIZE) {
  return res.status(400).json({ error: 'Size too large' });
}

# Stream large data
const stream = fs.createReadStream(file);
stream.pipe(res);  // Don't load entire file in memory
```

**Ref**: CWE-400, CWE-770, A04:2021

---

### 9. Time-of-Check Time-of-Use (TOCTOU)

**Search**: `existsSync|access|stat|lstat|check.*file|verify.*then|if.*exists`

**Issue**: Check file existence then use | Race between check and use | Symlink attacks | File replaced between check/use

**Severity**: Medium (privilege escalation, data corruption)

**Fix**: Atomic operations | Open file directly | Avoid check-then-use pattern

**Evidence patterns**:
```
# Vulnerable
if (fs.existsSync('/tmp/file')) {
  // File could be replaced with symlink here!
  fs.writeFileSync('/tmp/file', data);
}

# Check permissions then use
if (canUserAccessFile(user, file)) {
  // Permissions could change here
  const data = fs.readFileSync(file);
}
```

**Secure patterns**:
```
# Don't check, just do (handle errors)
try {
  fs.writeFileSync(file, data, { flag: 'wx' });  // Fails if exists
} catch (err) {
  if (err.code === 'EEXIST') {
    // Handle existing file
  }
}

# Atomic operations
const fd = fs.openSync(file, 'wx');  // Atomic: open if not exists
fs.writeSync(fd, data);
fs.closeSync(fd);

# Use file descriptor, not path
const fd = fs.openSync(file, 'r');
const stat = fs.fstatSync(fd);  // fstat, not stat
if (stat.isFile()) {
  const data = fs.readSync(fd, buffer, 0, buffer.length);
}
fs.closeSync(fd);
```

**Ref**: CWE-367, CWE-362

---

### 10. Type Confusion / Type Juggling

**Search**: `==|!=|req\.body|req\.query|JSON\.parse|loose.*equal|abstract.*equal`

**Issue**: Loose equality (==) | Type coercion | String to number conversion | Array/object in comparisons

**Severity**: Medium (authentication bypass, logic bugs)

**Fix**: Strict equality (===) | Type validation | Explicit type conversion | Schema validation

**Evidence patterns**:
```
# Vulnerable - type coercion
if (req.body.userId == user.id) {  // "1" == 1 is true!
}

# Loose equality bypass
if (password == hash) {  // 0 == "0abc" is true!
}

# Array comparisons
if (userRoles == 'admin') {  // ['admin'] == 'admin' is true!
}

# JSON.parse type confusion
const isAdmin = JSON.parse(req.query.admin);  // "true" → true
```

**Secure patterns**:
```
# Strict equality
if (req.body.userId === user.id) {
}

# Type validation
const userId = parseInt(req.body.userId, 10);
if (!Number.isInteger(userId)) {
  return res.status(400).json({ error: 'Invalid ID' });
}
if (userId === user.id) {
}

# Schema validation
const schema = Joi.object({
  userId: Joi.number().integer().required(),
  admin: Joi.boolean()
});
const { error, value } = schema.validate(req.body);

# Never use ==
# eslint: "eqeqeq": ["error", "always"]
```

**Ref**: CWE-843, A04:2021

---

