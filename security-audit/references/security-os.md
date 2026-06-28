# OS & Server Security

## Scope

SSH hardening | User accounts | Sudo configuration | System patches | Service hardening | File permissions | System logging | Firewall | Kernel security | Account policies | Boot security | Network configuration

## Vulnerabilities

### 1. Weak SSH Configuration

**Search**: `sshd_config|PermitRootLogin|PasswordAuthentication|Port.*22|PubkeyAuthentication|AllowUsers`

**Issue**: Password authentication enabled | Root login allowed | Default SSH port | No key-only auth | Weak ciphers | No fail2ban

**Severity**: Critical (unauthorized access)

**Fix**: Key-only auth | Disable root login | Non-default port | Strong ciphers | fail2ban

**Evidence patterns**:
```
# Vulnerable /etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
Port 22
PermitEmptyPasswords yes
ChallengeResponseAuthentication yes
```

**Secure patterns**:
```
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Port 45  # Non-default
PermitEmptyPasswords no
MaxAuthTries 3
LoginGraceTime 30
AllowUsers youruser  # Whitelist users
Protocol 2
X11Forwarding no
PermitUserEnvironment no

# Strong ciphers only
Ciphers aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512,hmac-sha2-256
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# Restart SSH
systemctl restart sshd

# fail2ban for SSH
apt install fail2ban
# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = 45
maxretry = 3
bantime = 3600
```

**Ref**: CWE-287, CWE-306, A07:2021

---

### 2. Excessive Sudo Privileges

**Search**: `sudo|sudoers|NOPASSWD|ALL.*ALL|visudo|wheel|admin`

**Issue**: User has NOPASSWD for all commands | ALL=(ALL) ALL | Service accounts with sudo | No command whitelisting

**Severity**: High (privilege escalation)

**Fix**: Whitelist specific commands | Require password | Principle of least privilege | No NOPASSWD for sensitive commands

**Evidence patterns**:
```
# Vulnerable /etc/sudoers
user ALL=(ALL) NOPASSWD: ALL  # Full sudo without password!
%wheel ALL=(ALL) ALL  # All wheel users can sudo everything
service_account ALL=(ALL) NOPASSWD: /bin/bash  # Service can get shell!
```

**Secure patterns**:
```
# /etc/sudoers - use visudo to edit
user ALL=(ALL) /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx
# Specific commands only

# Require password
user ALL=(ALL) /usr/sbin/reboot
# Password required

# No NOPASSWD for shells
# Never allow: /bin/bash, /bin/sh, /usr/bin/su

# Service accounts - no sudo
# Service accounts shouldn't have sudo at all

# Audit sudo usage
Defaults logfile="/var/log/sudo.log"
Defaults log_input, log_output
```

**Ref**: CWE-250, CWE-269, A01:2021

---

### 3. Outdated System Packages

**Search**: `apt.*update|yum.*update|dnf.*update|pacman.*-Syu|last.*update|kernel.*version`

**Issue**: Old kernel | Unpatched packages | Known CVEs | No automatic updates | Last update > 30 days

**Severity**: High (vulnerable to known exploits)

**Fix**: Regular updates | Unattended-upgrades | Security-only updates | Monitor CVEs

**Evidence patterns**:
```
# Check last update
stat /var/log/apt/history.log
ls -lt /var/log/apt/ | head

# Old kernel
uname -r  # Compare with latest
apt list --upgradable | grep linux-image

# Vulnerable packages
apt list --upgradable
yum check-update
```

**Secure patterns**:
```
# Update regularly
apt update && apt upgrade -y
yum update -y
dnf upgrade -y

# Automatic security updates (Ubuntu/Debian)
apt install unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades

# /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";

# RHEL/CentOS - yum-cron
yum install yum-cron
systemctl enable yum-cron

# Check for vulnerabilities
apt install debsecan
debsecan

# Or use Ubuntu's built-in
ubuntu-security-status
```

**Ref**: CWE-1035, CWE-1104, A06:2021

---

### 4. Weak User Account Security

**Search**: `passwd|useradd|adduser|/etc/shadow|/etc/passwd|password.*policy|pam\.d`

**Issue**: Weak passwords | No password expiry | Shared accounts | Default accounts not disabled | No account lockout

**Severity**: High (unauthorized access)

**Fix**: Strong password policy | Password expiry | Disable default accounts | Account lockout | Individual accounts

**Evidence patterns**:
```
# Check for weak passwords
john --wordlist=/usr/share/wordlists/rockyou.txt /etc/shadow

# No password expiry
chage -l username
# Maximum: 99999 days (no expiry!)

# Default accounts enabled
grep -v nologin /etc/passwd | grep -v false
# Look for guest, admin, test accounts

# No lockout policy
grep pam_tally2 /etc/pam.d/common-auth
# Not configured
```

**Secure patterns**:
```
# Password policy (PAM)
# /etc/pam.d/common-password (Debian/Ubuntu)
password requisite pam_pwquality.so retry=3 minlen=12 dcredit=-1 ucredit=-1 ocredit=-1 lcredit=-1

# Password expiry
chage -M 90 username  # Max 90 days
chage -m 7 username   # Min 7 days between changes
chage -W 14 username  # Warn 14 days before expiry

# Account lockout
# /etc/pam.d/common-auth
auth required pam_tally2.so onerr=fail audit silent deny=5 unlock_time=900

# Disable default/unused accounts
usermod -L guest
usermod -s /sbin/nologin ftp

# Strong passwords
apt install libpam-pwquality
# /etc/security/pwquality.conf
minlen = 12
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
```

**Ref**: CWE-521, CWE-307, A07:2021

---

### 5. Unnecessary Services Running

**Search**: `systemctl.*list|service.*status|netstat.*-tulpn|ss.*-tulpn|ports.*listening`

**Issue**: Telnet, FTP, rsh enabled | Debug services in production | Unused daemons | Unnecessary network services

**Severity**: Medium (increased attack surface)

**Fix**: Disable unused services | Minimal installation | Remove unnecessary packages

**Evidence patterns**:
```
# List running services
systemctl list-units --type=service --state=running

# Listening ports
netstat -tulpn
ss -tulpn

# Risky services
systemctl status telnet
systemctl status vsftpd
systemctl status rsh
```

**Secure patterns**:
```
# Disable unnecessary services
systemctl disable telnet
systemctl stop telnet
systemctl disable vsftpd
systemctl disable rsh
systemctl disable cups  # If no printing needed

# Remove packages
apt remove --purge telnetd rsh-server vsftpd
yum remove telnet-server rsh-server vsftpd

# List installed packages
apt list --installed | grep -E 'telnet|ftp|rsh'

# Minimal installation
# Use minimal/server variants
# Don't install desktop environments on servers

# Only essential services
systemctl list-unit-files --state=enabled
```

**Ref**: CWE-1188, A05:2021

---

### 6. Insecure File Permissions

**Search**: `find.*-perm.*777|chmod.*777|world.writable|/etc/shadow|/etc/passwd|/root|\.ssh/`

**Issue**: World-writable files | Readable /etc/shadow | Insecure SSH keys | Writable system directories

**Severity**: High (privilege escalation, data access)

**Fix**: Proper permissions | Restrict sensitive files | No 777 | Principle of least privilege

**Evidence patterns**:
```
# Find world-writable files
find / -type f -perm 0777 2>/dev/null
find / -type d -perm 0777 2>/dev/null

# Check critical files
ls -l /etc/shadow  # Should be 640 or 600
ls -l /etc/passwd  # Should be 644
ls -l ~/.ssh/      # Should be 700
ls -l ~/.ssh/id_rsa  # Should be 600
```

**Secure patterns**:
```
# Critical file permissions
chmod 600 /etc/shadow
chmod 644 /etc/passwd
chmod 700 /root
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/authorized_keys

# Fix world-writable files
find / -type f -perm 0777 -exec chmod 755 {} \; 2>/dev/null
find / -type d -perm 0777 -exec chmod 755 {} \; 2>/dev/null

# Secure /tmp (noexec, nosuid)
mount -o remount,noexec,nosuid,nodev /tmp

# /etc/fstab
tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev 0 0

# Audit file permissions
apt install aide
aide --init
aide --check
```

**Ref**: CWE-732, A05:2021

---

### 7. Missing System Logging

**Search**: `rsyslog|syslog|journalctl|/var/log|auditd|logrotate`

**Issue**: Logging disabled | Logs not rotated | No audit logs | Logs world-readable | No centralized logging

**Severity**: Medium (no forensic trail)

**Fix**: Enable logging | Log rotation | Auditd for security events | Secure log permissions | Central log server

**Evidence patterns**:
```
# Check logging service
systemctl status rsyslog
systemctl status systemd-journald

# Check audit daemon
systemctl status auditd

# Log rotation
ls -l /etc/logrotate.d/
cat /var/log/syslog | wc -l  # Very large = no rotation?

# World-readable logs
find /var/log -type f -perm /o=r
```

**Secure patterns**:
```
# Enable rsyslog
systemctl enable rsyslog
systemctl start rsyslog

# Enable auditd
apt install auditd
systemctl enable auditd
systemctl start auditd

# Audit rules
# /etc/audit/rules.d/audit.rules
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
-w /etc/sudoers -p wa -k sudoers_changes
-w /var/log/auth.log -p wa -k auth_log_changes
-w /bin/su -p x -k su_execution
-w /usr/bin/sudo -p x -k sudo_execution

# Log rotation
# /etc/logrotate.d/rsyslog
/var/log/syslog {
    rotate 7
    daily
    compress
    delaycompress
    notifempty
}

# Secure log permissions
chmod 640 /var/log/auth.log
chmod 640 /var/log/syslog

# Centralized logging
# Configure rsyslog to send to log server
*.* @@logserver.example.com:514
```

**Ref**: CWE-778, A09:2021

---

### 8. Weak Firewall Configuration

**Search**: `ufw|iptables|firewalld|nftables|allow.*any|ACCEPT.*0\.0\.0\.0`

**Issue**: Firewall disabled | Default allow | No rules | Overly permissive | Services exposed to internet

**Severity**: High (unauthorized access)

**Fix**: Default deny incoming | Whitelist specific ports | Source IP restrictions | Enable firewall

**Evidence patterns**:
```
# UFW status
ufw status
# Status: inactive

# iptables default policy
iptables -L
# Chain INPUT (policy ACCEPT)  # Should be DROP!

# firewalld
firewall-cmd --state
systemctl status firewalld
```

**Secure patterns**:
```
# UFW (Ubuntu/Debian)
ufw default deny incoming
ufw default allow outgoing
ufw allow from 192.168.1.0/24 to any port 22  # SSH from LAN only
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable

# iptables
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables-save > /etc/iptables/rules.v4

# firewalld (RHEL/CentOS)
firewall-cmd --set-default-zone=drop
firewall-cmd --zone=public --add-service=ssh --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

**Ref**: CWE-668, A05:2021

---

### 9. Kernel Security Features Disabled

**Search**: `sysctl|/etc/sysctl\.conf|kernel\.|net\.ipv4|aslr|exec-shield`

**Issue**: ASLR disabled | IP forwarding enabled when not needed | Source routing enabled | ICMP redirects accepted

**Severity**: Medium (makes exploitation easier)

**Fix**: Enable ASLR | Disable IP forwarding (unless router) | Harden network stack

**Evidence patterns**:
```
# Check ASLR
cat /proc/sys/kernel/randomize_va_space
# 0 = disabled (bad!)
# 2 = fully enabled (good)

# IP forwarding
cat /proc/sys/net/ipv4/ip_forward
# 1 = enabled (bad unless router)

# ICMP redirects
cat /proc/sys/net/ipv4/conf/all/accept_redirects
# 1 = enabled (bad)
```

**Secure patterns**:
```
# /etc/sysctl.conf or /etc/sysctl.d/99-security.conf

# Enable ASLR
kernel.randomize_va_space = 2

# Disable IP forwarding (unless router)
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Ignore source routed packets
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Enable TCP SYN cookies
net.ipv4.tcp_syncookies = 1

# Disable ping (optional)
net.ipv4.icmp_echo_ignore_all = 1

# Apply settings
sysctl -p
```

**Ref**: CWE-693, A05:2021

---

### 10. No Intrusion Detection

**Search**: `aide|tripwire|ossec|fail2ban|rkhunter|chkrootkit`

**Issue**: No file integrity monitoring | No IDS/IPS | No rootkit detection | No intrusion detection

**Severity**: Low (detection, not prevention)

**Fix**: Install AIDE/Tripwire | fail2ban | rkhunter | Log monitoring

**Evidence patterns**:
```
# Check for IDS tools
dpkg -l | grep -E 'aide|tripwire|ossec|fail2ban'
systemctl status fail2ban
systemctl status ossec
```

**Secure patterns**:
```
# File integrity monitoring - AIDE
apt install aide
aideinit
cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
aide --check

# Cron job for daily checks
0 5 * * * /usr/bin/aide --check | mail -s "AIDE Report" admin@example.com

# fail2ban (brute force protection)
apt install fail2ban
systemctl enable fail2ban

# Rootkit detection
apt install rkhunter chkrootkit
rkhunter --check
chkrootkit

# Automated scans
0 3 * * * /usr/bin/rkhunter --check --skip-keypress --report-warnings-only

# OSSEC (comprehensive IDS)
# See https://www.ossec.net/
```

**Ref**: CWE-778, A09:2021

---

