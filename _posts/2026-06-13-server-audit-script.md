---
title: "Project: Server Audit Script — Know What's Running on Your Server"
date: 2026-06-13 06:00:00 +0100
categories: [Projects, Bash]
tags: [bash, linux, scripting, security, audit, rhel, sysadmin, monitoring]
---

A systems administrator needs to know the state of their server at any moment. This project is a Bash script that audits a RHEL server and generates a structured report covering disk space, service health, failed SSH logins, sudo access, and open ports.

**Repo:** [biroue10/server-audit](https://github.com/biroue10/server-audit)

---

## Why This Project

Every time I SSH into a server, I want to answer five questions immediately:

1. Is disk space critical?
2. Are all critical services running?
3. Is anyone trying to brute-force SSH?
4. Who has sudo access?
5. Which ports are exposed?

Checking these manually every time is slow and error-prone. A script runs in seconds and gives a consistent, structured report every time.

---

## What the Script Does

```
sudo ./audit.sh
```

The script runs five checks and writes everything to `/tmp/audit-report.txt`:

---

### 1. Disk Space

```bash
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$DISK_USAGE" -gt 80 ]; then
    echo "WARNING: Disk usage is at ${DISK_USAGE}% - Action required!"
else
    echo "OK: Disk usage is at ${DISK_USAGE}%"
fi
```

Checks the root partition usage. If it exceeds 80%, the script flags it as a warning. A full disk crashes services silently — MariaDB stops writing, Nginx can't log, applications fail in unexpected ways.

**Sample output:**
```
=============================
  DISK SPACE
=============================
Filesystem              Size  Used  Avail  Use%  Mounted on
/dev/mapper/rhel-root    70G  6.0G   64G    9%   /
/dev/sda2               960M  355M  606M   37%   /boot
/dev/mapper/rhel-home   398G  9.9G  389G    3%   /home

OK: Disk usage is at 9%
```

---

### 2. Services Status

```bash
SERVICES=("nginx" "mariadb" "sshd" "firewalld")

for SERVICE in "${SERVICES[@]}"; do
    if systemctl is-active --quiet "$SERVICE"; then
        echo "OK:      $SERVICE is running"
    else
        echo "WARNING: $SERVICE is NOT running"
    fi
done
```

Loops through all critical services and checks each one with `systemctl is-active`. The `--quiet` flag suppresses output — we only care about the exit code (0 = running, non-zero = stopped).

**Sample output:**
```
=============================
  SERVICES STATUS
=============================
OK:      nginx is running
OK:      mariadb is running
OK:      sshd is running
OK:      firewalld is running
```

---

### 3. Failed SSH Logins

```bash
FAILED=$(journalctl -u sshd --since "24 hours ago" | grep "Failed password" | wc -l)
echo "Failed login attempts: $FAILED"
journalctl -u sshd --since "24 hours ago" | grep "Failed password" | tail -5
```

Reads the SSH service logs from the last 24 hours, counts failed password attempts, and shows the 5 most recent. On a public-facing server, hundreds of attempts per day is normal — thousands is a signal to review Fail2ban configuration.

**Sample output:**
```
=============================
  FAILED SSH LOGINS (last 24h)
=============================
Failed login attempts: 0
```

---

### 4. Sudo Access

```bash
getent group wheel | cut -d: -f4
grep -v "^#" /etc/sudoers | grep -v "^$"
```

Lists who is in the `wheel` group (sudo access on RHEL) and shows active sudoers entries. In a production environment, this list should be minimal and reviewed regularly. Every sudo user is a potential privilege escalation vector.

**Sample output:**
```
=============================
  USERS WITH SUDO ACCESS
=============================
Members of wheel group (sudo):
biroue
```

---

### 5. Open Ports

```bash
ss -tlnp | grep LISTEN
```

Lists all TCP ports in LISTEN state with the process name. Every open port is an attack surface. This check revealed that MariaDB was listening on `0.0.0.0:3306` — accessible from any interface — instead of `127.0.0.1` only.

**Sample output:**
```
=============================
  OPEN PORTS
=============================
LISTEN  0.0.0.0:22    sshd
LISTEN  0.0.0.0:3306  mariadbd   ← exposed on all interfaces
LISTEN  [::]:80       nginx
LISTEN  *:9090        prometheus
LISTEN  *:3000        grafana
```

---

## Key Bash Concepts Used

| Concept | Example in script |
|---------|-----------------|
| Variables | `DATE=$(date '+%Y-%m-%d %H:%M:%S')` |
| Command substitution `$(...)` | Capture command output into a variable |
| Functions | `check_disk()`, `check_services()` |
| Arrays | `SERVICES=("nginx" "mariadb" "sshd")` |
| For loop | Iterate over all services |
| If/else condition | Alert when disk > 80% |
| Redirection `>` and `>>` | Write report to file |
| Pipes `\|` | Chain commands together |
| `awk`, `cut`, `grep`, `wc` | Parse and extract data |
| `2>&1` | Redirect errors into the report |

---

## Real Finding

Running the script revealed that MariaDB was listening on all interfaces (`0.0.0.0:3306`). A database should never be publicly accessible. This is the value of a regular audit — it catches misconfigurations before an attacker does.

---

## What's Next

The next iteration will:
- Add CPU and RAM usage checks
- Send an email alert when a WARNING is detected
- Schedule automatically via cron every morning at 6:00

**Repo:** [github.com/biroue10/server-audit](https://github.com/biroue10/server-audit)
