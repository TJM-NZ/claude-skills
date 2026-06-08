# Infrastructure & Configuration

## Scope

Docker/containers | Firewall rules | Port exposure | Service hardening | Network segmentation | File permissions | Default configs | Unnecessary services | Cloud metadata | Process isolation

## Vulnerabilities

### 1. Privileged Docker Containers

**Search**: `privileged.*true|--privileged|security_opt.*seccomp.*unconfined|cap_add.*SYS_ADMIN|cap_add.*ALL`

**Issue**: Containers running with --privileged | Full host access | Disabled security features | Excessive capabilities

**Severity**: Critical (container escape, host compromise)

**Fix**: Never use --privileged | Drop unnecessary capabilities | Use security profiles | Minimal privileges

**Evidence patterns**:
```
# Vulnerable (docker-compose.yml)
privileged: true
security_opt:
  - seccomp:unconfined
cap_add:
  - ALL
  - SYS_ADMIN
```

**Secure patterns**:
```
# Minimal privileges
privileged: false
security_opt:
  - no-new-privileges:true
  - seccomp=default.json
cap_drop:
  - ALL
cap_add:
  - NET_BIND_SERVICE  # Only what's needed
read_only: true
```

**Ref**: CWE-250, CWE-269

---

### 2. Containers Running as Root

**Search**: `USER root|user.*:.*0:|dockerfile|FROM.*alpine|FROM.*ubuntu`

**Issue**: Containers running as root user | No USER directive in Dockerfile | uid 0 processes

**Severity**: High (privilege escalation if container compromised)

**Fix**: Create non-root user | Set USER in Dockerfile | Use --user flag | Check running processes

**Evidence patterns**:
```
# Vulnerable Dockerfile
FROM node:18
COPY . /app
WORKDIR /app
RUN npm install
CMD ["node", "server.js"]  # Runs as root!
```

**Secure patterns**:
```
# Create non-root user
FROM node:18
RUN groupadd -r appuser && useradd -r -g appuser appuser
COPY --chown=appuser:appuser . /app
WORKDIR /app
RUN npm install
USER appuser  # Switch to non-root
CMD ["node", "server.js"]

# docker-compose.yml
user: "1000:1000"
```

**Ref**: CWE-250

---

### 3. Exposed Unnecessary Ports

**Search**: `ports:|expose:|EXPOSE|-p\s|publish|0\.0\.0\.0|ufw|iptables|firewall-cmd`

**Issue**: Services bound to 0.0.0.0 | Ports exposed to internet | No firewall rules | Debug ports open

**Severity**: High (unauthorized access)

**Fix**: Bind to 127.0.0.1 for internal services | Use firewall | Close debug ports | Minimal exposure

**Evidence patterns**:
```
# Vulnerable - exposed to internet
ports:
  - "5432:5432"  # PostgreSQL on 0.0.0.0!
  - "6379:6379"  # Redis on 0.0.0.0!
  - "9000:9000"  # Debug port!

# Application binding
app.listen(3000, '0.0.0.0')  # Exposed!
```

**Secure patterns**:
```
# Bind to localhost only
ports:
  - "127.0.0.1:5432:5432"  # PostgreSQL localhost only
  - "127.0.0.1:6379:6379"  # Redis localhost only

# No ports for internal services
# Use Docker networks instead
networks:
  - internal

# Application
app.listen(3000, '127.0.0.1')  # Localhost only

# Firewall (UFW)
ufw default deny incoming
ufw allow from 192.168.0.0/24 to any port 5432  # LAN only
```

**Ref**: CWE-269, CWE-668

---

### 4. Secrets in Environment Variables

**Search**: `environment:|env:|ENV\s|\.env|-e\s|--env|PASSWORD|SECRET|API_KEY|TOKEN`

**Issue**: Secrets in docker-compose.yml | .env committed to git | Secrets in Dockerfile ENV | Visible in docker inspect

**Severity**: Critical (credential exposure)

**Fix**: Use Docker secrets | External secret managers | Mount secrets as files | Never commit .env

**Evidence patterns**:
```
# Vulnerable
environment:
  - DB_PASSWORD=secret123  # In compose file!
  - API_KEY=abc123xyz

# Dockerfile
ENV SECRET_KEY=hardcoded  # Baked into image!

# Committed .env
git ls-files .env
```

**Secure patterns**:
```
# Docker secrets (Swarm)
secrets:
  - db_password
environment:
  - DB_PASSWORD_FILE=/run/secrets/db_password

# External secrets
env_file:
  - .env  # Not committed to git!

# .gitignore
.env
.env.*
!.env.example

# Mount secrets
volumes:
  - ./secrets:/run/secrets:ro

# Read from file
const password = fs.readFileSync('/run/secrets/db_password', 'utf8').trim();
```

**Ref**: CWE-214, CWE-312, CWE-522

---

### 5. Insecure Docker Images

**Search**: `FROM.*latest|FROM.*:latest|image:.*latest|pull.*latest`

**Issue**: Using :latest tag | Outdated base images | Images from untrusted registries | No image scanning

**Severity**: High (vulnerable dependencies, supply chain)

**Fix**: Pin specific versions | Use minimal base images | Scan images | Use official images | Verify signatures

**Evidence patterns**:
```
# Vulnerable
FROM node:latest  # Unpredictable!
FROM ubuntu  # Defaults to latest
image: postgres:latest
```

**Secure patterns**:
```
# Pin versions
FROM node:18.20.2-alpine  # Specific version + minimal
FROM postgres:15.3-alpine

# Scan images
docker scan myimage:tag
trivy image myimage:tag

# Use minimal images
FROM gcr.io/distroless/nodejs18  # Minimal attack surface
FROM scratch  # For static binaries

# Multi-stage to reduce size
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM node:18-alpine  # Smaller runtime image
COPY --from=builder /app/dist /app
```

**Ref**: CWE-1104

---

### 6. Writable Container Filesystem

**Search**: `read_only|readOnlyRootFilesystem|tmpfs|volume`

**Issue**: Container filesystem writable | No read-only root | Malware can persist | No tmpfs for temp data

**Severity**: Medium (malware persistence)

**Fix**: Read-only root filesystem | tmpfs for /tmp | Volume mounts for data

**Evidence patterns**:
```
# Vulnerable - fully writable
services:
  app:
    image: myapp
    # No read_only flag
```

**Secure patterns**:
```
# Read-only filesystem
services:
  app:
    image: myapp
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
    volumes:
      - app-data:/data  # Only data dir writable

# Kubernetes
securityContext:
  readOnlyRootFilesystem: true
```

**Ref**: CWE-732

---

### 7. Missing Resource Limits

**Search**: `mem_limit|memory:|cpus:|cpu_shares|resources:|limits:|requests:|deploy:`

**Issue**: No memory limits | No CPU limits | Container can consume all resources | DoS via resource exhaustion

**Severity**: Medium (DoS)

**Fix**: Set memory/CPU limits | Use cgroups | Monitor resource usage

**Evidence patterns**:
```
# Vulnerable - no limits
services:
  app:
    image: myapp
    # No resource constraints
```

**Secure patterns**:
```
# Docker Compose
services:
  app:
    image: myapp
    mem_limit: 512m
    mem_reservation: 256m
    cpus: 0.5
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

# Kubernetes
resources:
  limits:
    cpu: "1"
    memory: "512Mi"
  requests:
    cpu: "0.5"
    memory: "256Mi"
```

**Ref**: CWE-400, CWE-770

---

### 8. Overly Permissive File Permissions

**Search**: `chmod 777|chmod.*777|0777|permission.*777|world.writable|/tmp.*777`

**Issue**: World-writable files/directories | 777 permissions | Sensitive files readable by all | No permission restrictions

**Severity**: High (unauthorized access, privilege escalation)

**Fix**: Least privilege permissions | 644 for files, 755 for dirs | Restrict sensitive files to owner only

**Evidence patterns**:
```
# Vulnerable
chmod 777 /app
chmod 666 /etc/passwd
RUN chmod -R 777 /var/www

# Check for 777 files
find / -type f -perm 0777
find / -type d -perm 0777
```

**Secure patterns**:
```
# Proper permissions
chmod 644 config.json  # rw-r--r--
chmod 600 secret.key   # rw------- (owner only)
chmod 755 /app         # rwxr-xr-x
chmod 700 ~/.ssh       # rwx------ (owner only)

# Dockerfile
COPY --chmod=644 config.json /app/
RUN chmod 600 /run/secrets/*
```

**Ref**: CWE-732

---

### 9. Default Credentials

**Search**: `admin:admin|root:root|password.*password|default.*password|user.*=.*admin.*password.*=|postgres.*postgres`

**Issue**: Default usernames/passwords | Admin accounts with weak passwords | No password change enforced

**Severity**: Critical (unauthorized access)

**Fix**: Change all defaults | Strong password policy | No default accounts in production

**Evidence patterns**:
```
# Vulnerable
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres

MONGODB_USERNAME=admin
MONGODB_PASSWORD=admin

# Code
if (username === 'admin' && password === 'admin') { ... }
```

**Secure patterns**:
```
# Strong, unique credentials
POSTGRES_USER=app_user
POSTGRES_PASSWORD=${RANDOM_PASSWORD_FROM_VAULT}

# Generate random passwords
openssl rand -base64 32

# Require password change on first login
# Disable default accounts
# Use password complexity requirements
```

**Ref**: CWE-798

---

### 10. Unnecessary Services Running

**Search**: `systemctl.*status|service.*status|ps aux|netstat -tulpn|ss -tulpn|docker ps`

**Issue**: Debug services in production | Unused services running | FTP, Telnet enabled | Unnecessary attack surface

**Severity**: Medium (increased attack surface)

**Fix**: Disable unused services | Minimal installations | Remove debug tools from production | Principle of least functionality

**Evidence patterns**:
```
# Check running services
systemctl list-units --type=service --state=running
docker ps  # Check all containers

# Risky services
telnet, ftp, rsh, rlogin
phpmyadmin, adminer in production
```

**Secure patterns**:
```
# Disable unnecessary services
systemctl disable telnet
systemctl stop ftp
apt remove --purge telnetd vsftpd

# Minimal Docker images (no shells, no debug tools)
FROM gcr.io/distroless/nodejs18

# Multi-stage builds (no build tools in final image)
FROM node:18 AS builder
RUN npm ci && npm run build

FROM node:18-alpine
COPY --from=builder /app/dist /app
# Build tools not in final image
```

**Ref**: CWE-1188

---

### 11. Cloud Metadata Accessible

**Search**: `169\.254\.169\.254|metadata\.google|169\.254\.170\.2|100\.100\.100\.200`

**Issue**: EC2/GCP metadata accessible from containers | SSRF to metadata endpoint | Container can fetch credentials

**Severity**: High (credential theft, privilege escalation)

**Fix**: Block metadata endpoint | Use IMDSv2 (AWS) | Network policies | Least privilege IAM

**Evidence patterns**:
```
# Test from container
curl http://169.254.169.254/latest/meta-data/
curl http://metadata.google.internal/computeMetadata/v1/

# No network restrictions
docker run --network=host  # Dangerous!
```

**Secure patterns**:
```
# AWS - use IMDSv2 (session-based)
aws ec2 modify-instance-metadata-options \
  --http-tokens required \
  --http-put-response-hop-limit 1

# Block metadata via firewall
iptables -A OUTPUT -d 169.254.169.254 -j DROP

# Docker - restrict network access
docker run --network=internal  # No internet access

# Kubernetes network policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-metadata
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
```

**Ref**: CWE-918

---

## Review Process

1. **Docker**: Find docker-compose.yml, Dockerfiles → check privileged, USER, caps
2. **Ports**: List exposed ports → verify only necessary, localhost binding
3. **Firewall**: Check UFW/iptables → verify restrictive rules
4. **Secrets**: Search PASSWORD, SECRET → verify not hardcoded, .env gitignored
5. **Images**: Check FROM statements → verify pinned versions, not :latest
6. **Permissions**: Run `find / -perm 777` → check for overly permissive files
7. **Services**: Run `systemctl list-units` → check for unnecessary services
8. **Resources**: Check compose files → verify mem/cpu limits set
9. **Root filesystem**: Check read_only flags → verify enabled where possible
10. **Metadata**: Test metadata endpoint access from containers

## Report Format

```
**[CRITICAL] Privileged Docker Container**
- **Category**: Infrastructure & Configuration
- **Location**: `docker-compose.yml:12`
- **Issue**: Container running with privileged: true
- **Risk**: Container can escape to host, full system compromise
- **Evidence**: `privileged: true` under frigate service
- **Fix**: Remove privileged flag, add only needed capabilities with cap_add
- **Reference**: CWE-250
```

## Common References

CWE-214 (info exposure through process env) | CWE-250 (execution with unnecessary privileges) | CWE-269 (improper privilege management) | CWE-312 (cleartext storage) | CWE-400 (resource exhaustion) | CWE-522 (insufficiently protected credentials) | CWE-668 (exposure of resource) | CWE-732 (incorrect permissions) | CWE-770 (allocation without limits) | CWE-798 (hardcoded credentials) | CWE-918 (SSRF) | CWE-1104 (untrusted component) | CWE-1188 (insecure default)
