---
title: "Project: WordPress Hardening — 8 Security Protections in One Script"
date: 2026-06-15 00:01:00 +0000
categories: [Projects, WordPress]
tags: [wordpress, security, nginx, rhel, fail2ban, bash, hardening, php]
---

A default WordPress installation is functional but not secure. File permissions are too permissive, the admin exposes the WordPress version, XML-RPC is open to brute force attacks, and there is nothing stopping someone from uploading a PHP webshell. This project applies 8 security protections to a WordPress site running on RHEL with Nginx, and packages everything into a single script: `harden-wordpress.sh`.

**Repo:** [biroue10/wordpress-hardening](https://github.com/biroue10/wordpress-hardening)

---

## Why WordPress Hardening Matters

WordPress powers 43% of the web — which makes it the most targeted CMS by attackers. Most compromised WordPress sites are not hacked through zero-days. They are compromised through:

- Weak file permissions exposing credentials
- Outdated plugins with known vulnerabilities
- Brute force attacks on wp-login.php
- Webshells uploaded via the media uploader
- Admin accounts with file editing capabilities

Every protection in this project addresses one of these attack vectors directly.

---

## Protection 1 — wp-config.php Permissions

`wp-config.php` contains the database name, username, and password for MariaDB. On a default WordPress installation, this file is readable by every process on the server.

**Check current permissions:**
```bash
ls -l /var/www/monsite/wp-config.php
-rwxr-xr-x. 1 apache apache 3662 wp-config.php
```

`r-xr-x` means group and others can read and execute this file. Any process running on the server can read the database credentials.

**Fix:**
```bash
chmod 640 /var/www/monsite/wp-config.php
```

`640` means:
- Owner (apache): read + write
- Group (apache): read only
- Others: nothing

`wp-config.php` is a PHP file — it never needs to be executable.

---

## Protection 2 — Block PHP Execution in Uploads

The `wp-content/uploads/` directory is writable by WordPress to store images and media. If an attacker gains access to upload a file, they can upload a PHP webshell — a script that gives them remote control of the server via the browser.

**The attack:**
```bash
# Attacker uploads shell.php disguised as an image
curl http://site.com/wp-content/uploads/shell.php
# → Full server access
```

**Fix — Nginx location block:**
```nginx
location ~* /wp-content/uploads/.*\.php$ {
    deny all;
}
```

**Critical: order matters in Nginx.** This block must appear **before** the `location ~ \.php$` block. Nginx evaluates regex location blocks in order — the first match wins. If the PHP execution block comes first, it matches before the deny rule and the webshell executes.

**Verify:**
```bash
echo "<?php echo 'hacked'; ?>" | sudo tee /var/www/monsite/wp-content/uploads/test.php
curl http://192.168.11.103/wp-content/uploads/test.php
# → 403 Forbidden
```

---

## Protection 3 — Disable File Editor in Admin

WordPress includes a built-in file editor in the admin dashboard (Appearance → Theme Editor). An attacker who compromises an admin account can inject PHP code directly into theme files — without any server access.

**Fix — add to wp-config.php:**
```php
define('DISALLOW_FILE_EDIT', true);
```

After this, the Theme Editor and Plugin Editor menus disappear from the WordPress admin entirely.

---

## Protection 4 — Hide WordPress Version

By default, WordPress announces its version in the HTML source of every page:

```html
<meta name="generator" content="WordPress 7.0" />
```

This tells attackers exactly which version-specific vulnerabilities to target.

**Fix — Must-Use Plugin:**

A Must-Use plugin (mu-plugin) is a PHP file placed in `wp-content/mu-plugins/`. It loads automatically on every request, cannot be deactivated from the admin, and runs before regular plugins.

```bash
sudo mkdir -p /var/www/monsite/wp-content/mu-plugins
```

```php
<?php
remove_action('wp_head', 'wp_generator');
```

The `remove_action` call unhooks the function that outputs the generator meta tag.

**Verify:**
```bash
curl -s http://192.168.11.103/ | grep "generator"
# → no output
```

---

## Protection 5 — Block XML-RPC

XML-RPC is a legacy WordPress protocol that allowed external applications to control WordPress remotely. It has been replaced by the REST API but remains active by default.

**Why it is dangerous:**

XML-RPC supports a `system.multicall` method that allows testing hundreds of username/password combinations in a single HTTP request. This bypasses rate limiting and makes brute force attacks extremely fast.

**Fix — Nginx:**
```nginx
location = /xmlrpc.php {
    deny all;
}
```

`location =` means exact match — faster than regex, applies only to that specific URL.

**Verify:**
```bash
curl -s -o /dev/null -w "%{http_code}" http://192.168.11.103/xmlrpc.php
# → 403
```

---

## Protection 6 — Security Headers

HTTP response headers instruct the browser on how to handle the page. Without security headers, browsers apply permissive defaults that enable several attack vectors.

**Before hardening:**
```
Server: nginx/1.26.3
X-Powered-By: PHP/8.3.31
```

**After hardening:**
```
Server: nginx
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: no-referrer-when-downgrade
```

**Added to Nginx config:**
```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
add_header Referrer-Policy "no-referrer-when-downgrade";
server_tokens off;
```

| Header | Protection |
|--------|-----------|
| `X-Frame-Options: SAMEORIGIN` | Prevents clickjacking — blocks your site from being embedded in iframes on other domains |
| `X-Content-Type-Options: nosniff` | Prevents MIME-type sniffing — browser must use the declared content type |
| `X-XSS-Protection: 1; mode=block` | Activates XSS filter in older browsers |
| `Referrer-Policy` | Controls what URL information is sent when users click links |
| `server_tokens off` | Removes Nginx version from Server header |

---

## Protection 7 — Hide PHP Version

`X-Powered-By: PHP/8.3.31` reveals the exact PHP version. This is set by PHP itself, not Nginx.

**Fix — php.ini:**
```ini
expose_php = Off
```

```bash
sudo systemctl restart php-fpm
```

**Verify:**
```bash
curl -I http://192.168.11.103/
# X-Powered-By header is gone
```

---

## Protection 8 — Fail2ban on wp-login.php

`wp-login.php` is the WordPress login page. It is publicly accessible and has no built-in rate limiting — an attacker can attempt thousands of passwords automatically.

**Fail2ban filter** (`/etc/fail2ban/filter.d/wordpress.conf`):
```ini
[Definition]
failregex = ^<HOST> .* "POST /wp-login.php
ignoreregex =
```

This regex matches any POST request to `wp-login.php` in the Nginx access log. Every login attempt is a POST request.

**Fail2ban jail** (`/etc/fail2ban/jail.local`):
```ini
[wordpress]
enabled  = true
filter   = wordpress
logpath  = /var/log/nginx/access.log
maxretry = 5
findtime = 300
bantime  = 3600
```

| Parameter | Value | Meaning |
|-----------|-------|---------|
| `maxretry` | 5 | 5 failed attempts triggers a ban |
| `findtime` | 300 | Within a 5-minute window |
| `bantime` | 3600 | IP banned for 1 hour |

**Check active jail:**
```bash
sudo fail2ban-client status wordpress
```

---

## The Script

All 8 protections are packaged in `harden-wordpress.sh`. Each function checks if the protection is already in place before applying it — safe to run multiple times.

```bash
sudo ./harden-wordpress.sh
```

Output:
```
[2026-06-15] Starting WordPress hardening...
================================
[2026-06-15] Fixing wp-config.php permissions...
[2026-06-15] Done: wp-config.php set to 640
[2026-06-15] Disabling file editor...
[2026-06-15] Done: DISALLOW_FILE_EDIT added
[2026-06-15] Hiding WordPress version...
[2026-06-15] Done: WordPress version hidden via mu-plugin
...
================================
[2026-06-15] Hardening complete.

Protections applied:
  [OK] wp-config.php permissions: 640
  [OK] File editor disabled
  [OK] WordPress version hidden
  [OK] PHP blocked in uploads
  [OK] XML-RPC blocked
  [OK] Security headers added
  [OK] PHP version hidden
  [OK] Fail2ban WordPress jail active
```

---

## What's Next

- WP-05: WordPress Backup — automated full backup (files + database) with retention and restore script
- WP-06: WordPress Performance — OpCache, FastCGI cache, compression, benchmarks

**Repo:** [github.com/biroue10/wordpress-hardening](https://github.com/biroue10/wordpress-hardening)
