---
title: "Project: DB-Guardian — Automated MariaDB Backup with Retention and Logging"
date: 2026-06-14 02:00:00 +0100
categories: [Projects, Bash]
tags: [bash, linux, rhel, mariadb, mysql, backup, cron, automation, sysadmin, scripting]
---

Every production database needs automated backups. Relying on manual backups is a mistake — when a failure happens, it always happens at the worst time. This project automates MariaDB backups nightly: dumps all databases, compresses them, names files with timestamps, deletes backups older than 7 days, and logs every action.

**Repo:** [biroue10/db-guardian](https://github.com/biroue10/db-guardian)

---

## Why Automated Backups Matter

A database failure without a recent backup means data loss. Scenarios that make backups critical:

- Accidental `DROP TABLE` or `DROP DATABASE`
- Failed migration that corrupts data
- Ransomware encrypting your database files
- Hardware failure

Manual backups fail because humans forget. A script running on cron never forgets.

---

## What the Script Does

```
1. Connect to MariaDB
2. List all user databases (exclude system databases)
3. For each database:
   → mysqldump → compress with gzip → save to /var/backups/db-guardian/
4. Delete backups older than 7 days
5. Log every step with timestamp
```

Each backup file is named with the database name and exact timestamp:

```
monsite_db_2026-06-14_02-21-56.sql.gz
mysql_2026-06-14_02-21-56.sql.gz
sys_2026-06-14_02-21-56.sql.gz
```

No file is ever overwritten. Each run produces new files.

---

## Key Variables

```bash
BACKUP_DIR="/var/backups/db-guardian"
DB_USER="root"
DB_PASS="your_password"
RETENTION_DAYS=7
DATE=$(date '+%Y-%m-%d_%H-%M-%S')
LOG_FILE="/var/log/db-guardian.log"
```

All configuration is at the top. Changing the retention period or backup directory requires editing one line.

---

## The log() Function

```bash
log() {
    echo "[$DATE] $1" | tee -a "$LOG_FILE"
}
```

Every message gets a timestamp and goes to two places simultaneously:
- The terminal (visible when run manually)
- `/var/log/db-guardian.log` (persistent history)

`tee -a` — the `-a` flag appends to the log file instead of overwriting it. Without `-a`, each backup run would erase all previous log entries.

---

## Listing Databases

```bash
DATABASES=$(mysql -u "$DB_USER" -p"$DB_PASS" -e "SHOW DATABASES;" 2>/dev/null \
    | grep -Ev "Database|information_schema|performance_schema")
```

`SHOW DATABASES` returns all databases including system ones. The `grep -Ev` filters out:

| Excluded | Reason |
|----------|--------|
| `Database` | Header line from mysql output |
| `information_schema` | System database — recreated automatically by MariaDB |
| `performance_schema` | System database — recreated automatically by MariaDB |

Only user databases get backed up.

---

## The Backup Function

```bash
backup_databases() {
    log "Starting database backup..."

    DATABASES=$(mysql -u "$DB_USER" -p"$DB_PASS" -e "SHOW DATABASES;" 2>/dev/null \
        | grep -Ev "Database|information_schema|performance_schema")

    for DB in $DATABASES; do
        FILENAME="${BACKUP_DIR}/${DB}_${DATE}.sql.gz"
        mysqldump -u "$DB_USER" -p"$DB_PASS" "$DB" 2>/dev/null | gzip > "$FILENAME"
        log "Backed up: $DB → $FILENAME"
    done
}
```

`mysqldump` exports the full database as SQL. The output is piped directly into `gzip` — no uncompressed file is ever written to disk. This saves space and is faster than compressing after the fact.

---

## Automatic Cleanup

```bash
cleanup_old_backups() {
    log "Removing backups older than $RETENTION_DAYS days..."
    find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
    log "Cleanup done."
}
```

`find` with `-mtime +7` matches files last modified more than 7 days ago. `-delete` removes them. Without this, backups would accumulate indefinitely and fill the disk.

---

## Sample Log Output

```
[2026-06-14_02-21-56] ==============================
[2026-06-14_02-21-56] DB-Guardian backup started
[2026-06-14_02-21-56] ==============================
[2026-06-14_02-21-56] Starting database backup...
[2026-06-14_02-21-56] Backed up: monsite_db → /var/backups/db-guardian/monsite_db_2026-06-14_02-21-56.sql.gz
[2026-06-14_02-21-56] Backed up: mysql → /var/backups/db-guardian/mysql_2026-06-14_02-21-56.sql.gz
[2026-06-14_02-21-56] Backed up: sys → /var/backups/db-guardian/sys_2026-06-14_02-21-56.sql.gz
[2026-06-14_02-21-56] Backed up: test_replication → /var/backups/db-guardian/test_replication_2026-06-14_02-21-56.sql.gz
[2026-06-14_02-21-56] Removing backups older than 7 days...
[2026-06-14_02-21-56] Cleanup done.
[2026-06-14_02-21-56] Backup completed successfully
[2026-06-14_02-21-56] ==============================
```

---

## Cron Scheduling

```bash
sudo crontab -e
```

```
0 2 * * * /home/biroue/db-guardian/db-guardian.sh
```

| Field | Value | Meaning |
|-------|-------|---------|
| `0` | minute | At minute 0 |
| `2` | hour | At 2:00 AM |
| `*` | day of month | Every day |
| `*` | month | Every month |
| `*` | day of week | Every day of the week |

The backup runs every night at 02:00 without any manual action.

---

## Bash Concepts Used

| Concept | Where |
|---------|-------|
| Variables | `BACKUP_DIR`, `RETENTION_DAYS`, `DATE` |
| Functions | `log`, `backup_databases`, `cleanup_old_backups`, `main` |
| Command substitution `$()` | Capture database list from mysql |
| `for` loop | Iterate over each database |
| Pipe `\|` | Chain mysqldump → gzip |
| `tee -a` | Write to screen and append to log file |
| `find -mtime -delete` | Locate and remove old files |
| `2>/dev/null` | Suppress password warnings from mysql |

---

## What's Next

- Email alert on backup failure — know immediately if something breaks
- Rsync to a remote server — offsite backup for disaster recovery
- Restore script — restore a specific database from a `.sql.gz` file

**Repo:** [github.com/biroue10/db-guardian](https://github.com/biroue10/db-guardian)
