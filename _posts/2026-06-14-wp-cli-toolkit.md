---
title: "Project: WP-CLI Toolkit — Managing WordPress from the Command Line"
date: 2026-06-14 07:00:00 +0100
categories: [Projects, WordPress]
tags: [wordpress, wpcli, bash, linux, rhel, automation, sysadmin, plugins, database]
---

Every WordPress administrator faces this scenario: a plugin update breaks the site, the dashboard returns a white screen, and the client is panicking. No browser access. No admin panel. The only tool available is the terminal. This is exactly where WP-CLI becomes essential. This project builds an interactive Bash menu that covers the most critical WordPress management operations entirely from the command line.

**Repo:** [biroue10/wp-cli-toolkit](https://github.com/biroue10/wp-cli-toolkit)

---

## What is WP-CLI?

WP-CLI (WordPress Command Line Interface) is the official command-line tool for managing WordPress installations. It allows you to perform any WordPress operation without a browser — install plugins, create users, update core, export the database, and much more.

**Why it matters in production:**

| Situation | Without WP-CLI | With WP-CLI |
|-----------|----------------|-------------|
| Plugin breaks admin dashboard | Manually edit the database to deactivate the plugin | `wp plugin deactivate broken-plugin` |
| Client forgot admin password | Access phpMyAdmin, find the user, hash a new password manually | `wp user update admin --user_pass=newpass` |
| Site migration to new domain | Export DB, open in text editor, find-replace thousands of URLs, re-import | `wp search-replace old-domain.com new-domain.com` |
| WordPress core update | Click through the admin UI | `wp core update` |
| Database backup before maintenance | Log into phpMyAdmin, export manually | `wp db export backup.sql` |

WP-CLI turns multi-step manual processes into single commands.

---

## Installing WP-CLI on RHEL

```bash
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
```

**What each step does:**

- `curl -O` — download the WP-CLI phar file (a PHP archive — a single self-contained executable)
- `chmod +x` — make it executable
- `sudo mv /usr/local/bin/wp` — move it to a system-wide location so any user can call `wp`

**Verify installation:**
```bash
wp --info
```

```
OS:           Linux 6.12.0-211.20.1.el10_2.x86_64
PHP binary:   /usr/bin/php
PHP version:  8.3.31
MySQL binary: /usr/bin/mariadb
WP-CLI version: 2.12.0
```

**Important:** When running WP-CLI as root (via sudo), always add `--allow-root`. WP-CLI refuses to run as root by default as a safety measure — `--allow-root` explicitly overrides this.

---

## The Toolkit — Architecture

```bash
WP_PATH="/var/www/monsite"           # WordPress installation directory
WP_CLI="/usr/local/bin/wp"           # Full path to WP-CLI binary
DATE=$(date '+%Y-%m-%d_%H-%M-%S')   # Timestamp for backup file names
BACKUP_DIR="/var/backups/wp-toolkit" # Where database backups are stored
```

**Why store `WP_CLI` as a variable?**

`sudo` uses a restricted PATH — it doesn't include `/usr/local/bin/` by default. Calling `wp` directly in a sudo context fails with "command not found". Using the full path `/usr/local/bin/wp` ensures the binary is always found regardless of which user runs the script.

---

## Requirement Checks

Before doing anything, the script verifies the environment is correct:

```bash
check_requirements() {
    if [ ! -f "$WP_CLI" ]; then
        log "ERROR: WP-CLI not found at $WP_CLI"
        exit 1
    fi

    if [ ! -d "$WP_PATH" ]; then
        log "ERROR: WordPress not found at $WP_PATH"
        exit 1
    fi

    log "Requirements OK."
}
```

| Check | Condition | Meaning |
|-------|-----------|---------|
| `[ ! -f "$WP_CLI" ]` | WP-CLI binary not found | Cannot manage WordPress without WP-CLI |
| `[ ! -d "$WP_PATH" ]` | WordPress directory missing | Wrong path configured |

`exit 1` signals failure to any calling process (cron, another script). `exit 0` signals success. This matters when scripts call each other — the caller reads the exit code to know if the operation succeeded.

---

## The Interactive Menu

```bash
show_menu() {
    echo "============================="
    echo "   WP Toolkit — $WP_PATH"
    echo "============================="
    echo "  1. Plugin management"
    echo "  2. Theme management"
    echo "  3. User management"
    echo "  4. Update WordPress core"
    echo "  5. Database backup"
    echo "  6. Search and replace in DB"
    echo "  0. Exit"
    echo "============================="
    echo -n "Choose an option: "
    read CHOICE
}
```

`read CHOICE` — pauses execution and waits for user input. Whatever the user types is stored in the variable `CHOICE`. The main loop then uses a `case` statement to call the right function.

**The main loop:**

```bash
while true; do
    show_menu
    case $CHOICE in
        1) manage_plugins ;;
        2) sudo $WP_CLI theme list --path="$WP_PATH" --allow-root ;;
        3) manage_users ;;
        4) sudo $WP_CLI core update --path="$WP_PATH" --allow-root ;;
        5) backup_database ;;
        6) search_replace ;;
        0) log "Goodbye." && exit 0 ;;
        *) log "Invalid option." ;;
    esac
done
```

`while true` keeps the menu running after each action. Without it, the script would exit after one operation. The only exit point is option `0` which calls `exit 0`.

`case` is cleaner than `if/elif/elif` when handling multiple fixed values. Each case ends with `;;` (equivalent to `break` in other languages). `*` is the default case — catches anything that doesn't match.

---

## Plugin Management

```bash
manage_plugins() {
    echo "  1. List all plugins"
    echo "  2. Install a plugin"
    echo "  3. Activate a plugin"
    echo "  4. Deactivate a plugin"
    echo "  5. Update all plugins"
    echo "  6. Delete a plugin"
    read ACTION

    case $ACTION in
        1) sudo $WP_CLI plugin list --path="$WP_PATH" --allow-root ;;
        2) echo -n "Plugin slug: " && read PLUGIN
           sudo $WP_CLI plugin install "$PLUGIN" --activate --path="$WP_PATH" --allow-root ;;
        3) echo -n "Plugin slug: " && read PLUGIN
           sudo $WP_CLI plugin activate "$PLUGIN" --path="$WP_PATH" --allow-root ;;
        4) echo -n "Plugin slug: " && read PLUGIN
           sudo $WP_CLI plugin deactivate "$PLUGIN" --path="$WP_PATH" --allow-root ;;
        5) sudo $WP_CLI plugin update --all --path="$WP_PATH" --allow-root ;;
        6) echo -n "Plugin slug: " && read PLUGIN
           sudo $WP_CLI plugin delete "$PLUGIN" --path="$WP_PATH" --allow-root ;;
    esac
}
```

**Key WP-CLI plugin commands explained:**

**`wp plugin list`** — shows all installed plugins with status and version:
```
+-----------+----------+--------+---------+
| name      | status   | update | version |
+-----------+----------+--------+---------+
| akismet   | inactive | none   | 5.7     |
| hello     | inactive | none   | 1.7.2   |
| wordfence | active   | none   | 8.2.2   |
+-----------+----------+--------+---------+
```

**`wp plugin install wordfence --activate`** — downloads from wordpress.org, installs, and activates in one command:
```
Installing Wordfence Security (8.2.2)
Downloading installation package...
Installing the plugin...
Plugin installed successfully.
Activating 'wordfence'...
Plugin 'wordfence' activated.
Success: Installed 1 of 1 plugins.
```

**`wp plugin deactivate broken-plugin`** — the most critical command for incident response. When a plugin causes a white screen or 500 error, this deactivates it without touching the database manually.

**`wp plugin update --all`** — updates every plugin in one command. Essential for security maintenance — outdated plugins are the #1 attack vector for WordPress sites.

**What is a plugin slug?** The slug is the unique identifier of a plugin on wordpress.org. For "Wordfence Security", the slug is `wordfence`. For "WooCommerce", it's `woocommerce`. It's the last part of the plugin URL: `wordpress.org/plugins/wordfence`.

---

## User Management

```bash
manage_users() {
    case $ACTION in
        1) sudo $WP_CLI user list --path="$WP_PATH" --allow-root ;;
        2) sudo $WP_CLI user create "$USERNAME" "$EMAIL" \
               --user_pass="$PASSWORD" --role="$ROLE" \
               --path="$WP_PATH" --allow-root ;;
        3) sudo $WP_CLI user update "$USERNAME" \
               --user_pass="$PASSWORD" \
               --path="$WP_PATH" --allow-root ;;
        4) sudo $WP_CLI user delete "$USERNAME" \
               --reassign=1 \
               --path="$WP_PATH" --allow-root ;;
    esac
}
```

**WordPress user roles:**

| Role | Capabilities |
|------|-------------|
| `administrator` | Full access — manage everything |
| `editor` | Manage and publish all posts and pages |
| `author` | Write and publish their own posts only |
| `contributor` | Write posts but cannot publish |
| `subscriber` | Read only — no content creation |

**`wp user list`** output:
```
+----+------------+---------------+---------------------+
| ID | user_login | display_name  | user_registered     |
+----+------------+---------------+---------------------+
| 1  | admin      | admin         | 2026-06-14 04:59:57 |
+----+------------+---------------+---------------------+
```

**`wp user create`** — creates a user with role:
```bash
wp user create john john@example.com --user_pass=SecurePass! --role=editor
```

**`wp user update admin --user_pass=NewPassword`** — resets a password instantly. Used when a client is locked out and the "forgot password" email is not working (common issue with SMTP misconfiguration).

**`wp user delete john --reassign=1`** — the `--reassign=1` flag is critical. Every post in WordPress belongs to a user. Deleting a user without reassigning their content permanently deletes all their posts. `--reassign=1` transfers all content to user ID 1 (the main admin) before deletion. Content is preserved.

---

## Database Backup

```bash
backup_database() {
    sudo mkdir -p "$BACKUP_DIR"
    BACKUP_FILE="${BACKUP_DIR}/wp-db-${DATE}.sql"
    sudo $WP_CLI db export "$BACKUP_FILE" --path="$WP_PATH" --allow-root
    sudo gzip "$BACKUP_FILE"
    log "Backup saved: ${BACKUP_FILE}.gz"
}
```

**`wp db export`** — dumps the entire WordPress database to a `.sql` file. This includes:
- All posts, pages, and custom post types
- All comments
- All users and their metadata
- All plugin settings
- All WordPress options (site URL, theme, widget config)
- All WooCommerce orders, products, and customer data (if WooCommerce is installed)

`gzip` compresses the file immediately after export. A typical WordPress database dumps to 5-50MB uncompressed — compressed it's 80-90% smaller.

**Backup file naming:**
```
wp-db-2026-06-14_06-19-00.sql.gz
```

Timestamp in the filename means backups never overwrite each other. You can keep 30 daily backups and know exactly when each one was taken.

**Always backup before:**
- WordPress core updates
- Plugin updates
- Theme changes
- Database search-replace operations
- Any custom code deployment

---

## Search and Replace

```bash
search_replace() {
    echo -n "Search for: " && read SEARCH
    echo -n "Replace with: " && read REPLACE
    sudo $WP_CLI search-replace "$SEARCH" "$REPLACE" --path="$WP_PATH" --allow-root
}
```

**`wp search-replace`** — scans every table and every column in the WordPress database and replaces matching strings. It handles serialized PHP data correctly — something a simple SQL `REPLACE()` cannot do.

**Why serialized data matters:**

WordPress stores some data in serialized PHP format:
```
a:2:{s:3:"url";s:22:"http://old-domain.com/";s:4:"name";s:4:"Site";}
```

If you do a plain SQL find-replace on this, you change the string length but not the length indicator in the serialized format — this corrupts the data silently. WP-CLI handles this automatically.

**Most common use case — site migration:**
```
Search:  http://old-domain.com
Replace: https://new-domain.com
```

This updates every URL in the database:
- Post content links
- Image URLs in the media library
- Plugin settings storing the domain
- Widget configurations
- Option values like `siteurl` and `home`

**Output:**
```
Success: Made 47 replacements.
```

---

## Bash Concepts Used

| Concept | Where | Purpose |
|---------|-------|---------|
| Variables | `WP_PATH`, `WP_CLI`, `DATE` | Configuration in one place |
| Functions | `check_requirements`, `manage_plugins`, `manage_users` | Organized, reusable code blocks |
| `case` statement | Menu routing, plugin/user sub-menus | Clean multi-branch logic |
| `while true` | Main loop | Keep menu running after each action |
| `read` | `read CHOICE`, `read PLUGIN` | Capture user input |
| `exit 0` / `exit 1` | Requirements check, quit option | Signal success or failure to calling processes |
| `[ ! -f ]` / `[ ! -d ]` | Requirement checks | Test if file/directory exists |
| `&&` | `log "Goodbye." && exit 0` | Run second command only if first succeeds |

---

## Incident Response Scenarios

**Scenario 1 — Plugin white screen**
```
Symptom: Site returns blank page after plugin update
Action:  Option 1 → Deactivate plugin → wp plugin deactivate plugin-name
Result:  Site restored in seconds
```

**Scenario 2 — Admin locked out**
```
Symptom: Client forgot password, reset email not working
Action:  Option 3 → Change user password → enter new password
Result:  Client regains access immediately
```

**Scenario 3 — Site migration**
```
Symptom: Site moved to new domain, all internal links broken
Action:  Option 6 → Search: old-domain.com → Replace: new-domain.com
Result:  All 47 URL occurrences updated in one operation
```

**Scenario 4 — Pre-maintenance backup**
```
Symptom: About to update WordPress core
Action:  Option 5 → Database backup → compressed .sql.gz saved
Result:  Rollback point created before any changes
```

---

## What's Next

- WP-03: WordPress Multisite — host multiple sites on one WordPress installation
- WP-04: WordPress Hardening — secure a WordPress site against attacks
- WP-05: WordPress Backup — automated full backup (files + database) with retention

**Repo:** [github.com/biroue10/wp-cli-toolkit](https://github.com/biroue10/wp-cli-toolkit)
