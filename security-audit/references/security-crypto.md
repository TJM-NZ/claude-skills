# Cryptography

## Scope

Encryption algorithms | Key management | SSL/TLS config | Certificate validation | Random number generation | Hash functions | Cipher modes | Key derivation | Crypto library usage

## Vulnerabilities

### 1. Weak Encryption Algorithms

**Search**: `DES|3DES|RC4|Blowfish|cipher.*des|cipher.*rc4|algorithm.*des|AES.*ECB|MODE_ECB`

**Issue**: Deprecated algorithms (DES, 3DES, RC4, Blowfish) | ECB mode (pattern leakage)

**Severity**: High

**Fix**: Use AES-256-GCM | ChaCha20-Poly1305 | Use authenticated encryption (AEAD)

**Evidence patterns**:
```
# Vulnerable
Cipher.getInstance("DES")
Cipher.getInstance("AES/ECB/PKCS5Padding")  // ECB mode!
crypto.createCipheriv('des-ede3', key, iv)
openssl_encrypt($data, 'DES-CBC', $key)
```

**Secure patterns**:
```
# AES-256-GCM (recommended)
crypto.createCipheriv('aes-256-gcm', key, iv)
Cipher.getInstance("AES/GCM/NoPadding")
AES.new(key, AES.MODE_GCM)

# ChaCha20-Poly1305
crypto.createCipheriv('chacha20-poly1305', key, nonce)
```

**Ref**: CWE-327, A02:2021

---

### 2. Weak Key Management

**Search**: `key.*=.*['"][^'"]{8,16}['"]|AES.*key.*=|SECRET_KEY.*=.*['"]|hardcoded.*key|private.*key.*=.*['"]|\b[A-Fa-f0-9]{32,64}\b.*key`

**Issue**: Hardcoded keys | Short keys | Keys in source code | Keys in version control | Reused keys/IVs

**Severity**: Critical

**Fix**: Generate keys securely | Store in secure key management (KMS, Vault) | Rotate keys | Never commit keys

**Evidence patterns**:
```
# Vulnerable
const key = "mySecretKey123";
private static final String KEY = "1234567890123456";
SECRET_KEY = "abc123"
iv = b'0000000000000000'  // Static IV!
```

**Secure patterns**:
```
# Generate secure keys
key = crypto.randomBytes(32)  // 256 bits
key = os.urandom(32)
key = SecureRandom.getInstanceStrong()

# Load from environment/KMS
key = process.env.ENCRYPTION_KEY
key = kms_client.decrypt(encrypted_key)

# Generate random IV per encryption
iv = crypto.randomBytes(16)
```

**Ref**: CWE-321, CWE-798, A02:2021

---

### 3. Insecure SSL/TLS Configuration

**Search**: `SSLv2|SSLv3|TLSv1\.|TLS_RSA|VERIFY.*false|verify.*false|rejectUnauthorized.*false|verify_mode.*NONE|check.*hostname.*false`

**Issue**: Old TLS versions (SSLv3, TLS 1.0, TLS 1.1) | Weak cipher suites | Certificate validation disabled | Hostname verification disabled

**Severity**: High

**Fix**: TLS 1.2+ only | Strong cipher suites | Enable certificate validation | Enable hostname verification

**Evidence patterns**:
```
# Vulnerable
ssl_version: SSLv3
tls_version: TLSv1.0
rejectUnauthorized: false  // Node.js
verify: false  // Python requests
CURLOPT_SSL_VERIFYPEER => false
ctx.check_hostname = False
```

**Secure patterns**:
```
# Node.js
https.request({
  minVersion: 'TLSv1.2',
  rejectUnauthorized: true
})

# Python
import ssl
context = ssl.create_default_context()
context.minimum_version = ssl.TLSVersion.TLSv1_2
# Never set verify=False or check_hostname=False

# Nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
```

**Ref**: CWE-295, CWE-326, A02:2021

---

### 4. Weak Random Number Generation

**Search**: `Math\.random|rand\(\)|random\.random\(\)|Random\(\)|time\(\).*seed|predictable.*random|mt_rand|srand\(`

**Issue**: Non-cryptographic PRNG for security (Math.random, rand()) | Predictable seeds | Weak random for tokens/keys/IVs

**Severity**: High (token prediction) | Critical (key generation)

**Fix**: Use crypto.randomBytes | secrets module | SecureRandom | /dev/urandom

**Evidence patterns**:
```
# Vulnerable
token = Math.random().toString(36)
token = str(random.randint(100000, 999999))
sessionId = rand(1000, 9999)
key = md5(time())  // Predictable!
```

**Secure patterns**:
```
# Node.js
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex');

# Python
import secrets
token = secrets.token_urlsafe(32)
token = secrets.token_hex(32)

# Java
SecureRandom sr = SecureRandom.getInstanceStrong();
byte[] token = new byte[32];
sr.nextBytes(token);

# PHP
$token = bin2hex(random_bytes(32));
```

**Ref**: CWE-330, CWE-338, A02:2021

---

### 5. Weak Hash Functions

**Search**: `md5\(|MD5|sha1\(|SHA1|hashlib\.md5|hashlib\.sha1|MessageDigest\.getInstance.*MD5|MessageDigest\.getInstance.*SHA-1|hash.*md5|hash.*sha1`

**Issue**: MD5 or SHA1 for integrity | Collision attacks possible | Fast hashing for passwords

**Severity**: Medium (integrity) | High (passwords - see auth review)

**Fix**: SHA-256+ for integrity | HMAC for auth | bcrypt/argon2 for passwords (see auth)

**Evidence patterns**:
```
# Vulnerable
hash = md5(file_content)
hash = hashlib.md5(data).hexdigest()
digest = MessageDigest.getInstance("MD5")
hash('md5', $data)
```

**Secure patterns**:
```
# SHA-256 for integrity
hash = crypto.createHash('sha256').update(data).digest('hex')
hash = hashlib.sha256(data).hexdigest()

# HMAC for authentication
hmac = crypto.createHmac('sha256', secret).update(data).digest('hex')
hmac = hmac.new(secret, data, hashlib.sha256).hexdigest()

# For file integrity
hash = hashlib.sha256()
for chunk in iter(lambda: f.read(4096), b''):
    hash.update(chunk)
```

**Ref**: CWE-328, CWE-916, A02:2021

---

### 6. Missing HMAC/Authentication

**Search**: `encrypt\(|cipher\.|AES\.|encrypt|ciphertext.*=.*(?!.*hmac|.*tag|.*auth)`

**Issue**: Encryption without authentication (no HMAC/MAC) | Malleable ciphertext | No integrity check

**Severity**: High (tampering)

**Fix**: Use AEAD modes (GCM, ChaCha20-Poly1305) | Or encrypt-then-MAC

**Evidence patterns**:
```
# Vulnerable
ciphertext = cipher.encrypt(plaintext)
return ciphertext  // No authentication!

# AES-CBC without HMAC
cipher = AES.new(key, AES.MODE_CBC, iv)
ciphertext = cipher.encrypt(padded_data)
```

**Secure patterns**:
```
# Use AEAD (GCM - recommended)
cipher = crypto.createCipheriv('aes-256-gcm', key, iv)
ciphertext = cipher.update(plaintext)
cipher.final()
tag = cipher.getAuthTag()  // Authentication tag
return { ciphertext, iv, tag }

# Or Encrypt-then-MAC
ciphertext = cipher.encrypt(plaintext)
mac = hmac_sha256(key_mac, ciphertext)
return { ciphertext, mac }
```

**Ref**: CWE-353, A02:2021

---

### 7. Insufficient Key Length

**Search**: `AES.*128|keysize.*128|key.*length.*128|DH.*512|DH.*1024|RSA.*512|RSA.*1024`

**Issue**: AES-128 (consider 256) | RSA < 2048 | DH < 2048 | Elliptic curve < 256

**Severity**: Medium (future-proofing)

**Fix**: AES-256 | RSA-2048+ (prefer 3072/4096) | DH-2048+ | EC-256+

**Evidence patterns**:
```
# Weak
Cipher.getInstance("AES_128")
rsa = RSA.generate(1024)
dh_params = generate_dh_params(1024)
```

**Secure patterns**:
```
# Strong
Cipher.getInstance("AES_256")
rsa = RSA.generate(3072)  // or 4096
dh_params = generate_dh_params(2048)
ec = ec.generate_private_key(ec.SECP256R1)
```

**Ref**: CWE-326, A02:2021

---

### 8. Improper Certificate Validation

**Search**: `verify.*=.*false|CERT_NONE|SSL_VERIFY_NONE|InsecureRequestWarning|disable.*warnings|curl.*-k|wget.*--no-check`

**Issue**: Certificate validation disabled | Self-signed certs accepted | Hostname not verified

**Severity**: High (MITM)

**Fix**: Always validate certificates | Pin certificates for high-security | Verify hostname

**Evidence patterns**:
```
# Vulnerable
requests.get(url, verify=False)
urllib3.disable_warnings()
ssl._create_unverified_context()
curl -k https://example.com
```

**Secure patterns**:
```
# Always verify
requests.get(url, verify=True)  // Default
response = urllib.request.urlopen(url)  // Default context validates

# Certificate pinning (high-security)
import certifi
requests.get(url, verify=certifi.where())

# Custom CA bundle
requests.get(url, verify='/path/to/ca-bundle.crt')
```

**Ref**: CWE-295, A02:2021

---

### 9. Null/Empty Cipher

**Search**: `cipher.*null|NULL.*cipher|eNULL|aNULL|EXPORT|algorithm.*none|cipher.*NONE`

**Issue**: Null cipher accepted | Anonymous cipher suites | Export-grade crypto

**Severity**: Critical

**Fix**: Disable null/anon/export ciphers | Use strong cipher suites only

**Evidence patterns**:
```
# Vulnerable (OpenSSL/Nginx)
ssl_ciphers ALL:!aNULL:!eNULL;  // Better but still weak
ssl_ciphers EXPORT;  // Very bad!
```

**Secure patterns**:
```
# Strong cipher suites (Nginx)
ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
ssl_prefer_server_ciphers on;

# Mozilla modern config
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
```

**Ref**: CWE-327, A02:2021

---

### 10. Padding Oracle Vulnerabilities

**Search**: `PKCS.*Padding|AES.*CBC|MODE_CBC|PaddingException|padding.*error|invalid.*padding`

**Issue**: CBC mode with unpadding errors revealed | Timing attacks on padding

**Severity**: High (plaintext recovery)

**Fix**: Use GCM mode (no padding) | Don't reveal padding errors | Constant-time comparison

**Evidence patterns**:
```
# Vulnerable
try:
    plaintext = cipher.decrypt(ciphertext)
except PaddingException as e:
    return "Padding error"  // Reveals info!
```

**Secure patterns**:
```
# Use GCM (no padding)
cipher = AES.new(key, AES.MODE_GCM, nonce)

# If CBC required, don't reveal padding errors
try:
    plaintext = cipher.decrypt(ciphertext)
except:
    return "Decryption failed"  // Generic error
```

**Ref**: CWE-696, A02:2021

---

## Review Process

1. **Algorithms**: Grep for DES, RC4, MD5, SHA1 → check if used for security
2. **Keys**: Search for hardcoded keys → verify key generation is secure
3. **SSL/TLS**: Find TLS config → verify TLS 1.2+, strong ciphers, validation enabled
4. **Random**: Search Math.random, rand() → verify crypto-secure RNG for security
5. **Hashing**: Find md5/sha1 → check context (integrity vs passwords)
6. **AEAD**: Find encryption → verify authenticated encryption (GCM) or MAC
7. **Key length**: Search AES/RSA/DH → verify sufficient lengths
8. **Cert validation**: Search verify=false → ensure never disabled
9. **Cipher suites**: Check SSL config → verify no null/anon/export ciphers
10. **Padding**: Find CBC mode → check error handling doesn't leak info

## Report Format

```
**[SEVERITY] Weak Encryption Algorithm**
- **Category**: Cryptography
- **Location**: `crypto/encryption.py:23`
- **Issue**: Using DES encryption (deprecated, 56-bit key)
- **Risk**: DES can be brute-forced in hours; attacker can decrypt all data
- **Evidence**: `cipher = DES.new(key, DES.MODE_ECB)`
- **Fix**: Use AES-256-GCM: `cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)`
- **Reference**: CWE-327, A02:2021
```

## Common References

CWE-295 (cert validation) | CWE-321 (hardcoded key) | CWE-326 (weak crypto strength) | CWE-327 (broken crypto) | CWE-328 (weak hash) | CWE-330 (weak random) | CWE-338 (weak PRNG) | CWE-353 (missing integrity) | CWE-696 (incorrect behavior) | CWE-798 (hardcoded creds) | CWE-916 (weak password hash) | A02:2021 (Crypto Failures)
