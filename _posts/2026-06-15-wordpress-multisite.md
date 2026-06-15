---
title: "Project: WordPress Multisite — One Installation, Multiple Sites"
date: 2026-06-15 00:00:00 +0000
categories: [Projects, WordPress]
tags: [wordpress, multisite, nginx, rhel, linux, php-fpm, sysadmin, network]
---

WordPress powers 43% of the web. But most people don't realize that a significant portion of those sites don't each have their own WordPress installation — they share one. That's exactly what WordPress Multisite is: a single WordPress installation that hosts a network of independent sites. This is the foundation of WordPress.com. This project activates and configures WordPress Multisite on a RHEL server with Nginx, building a network of 4 sites from a single codebase.

**Repo:** [biroue10/wordpress-multisite](https://github.com/biroue10/wordpress-multisite)

---

## What is WordPress Multisite?

WordPress Multisite is a built-in WordPress feature that transforms a single installation into a network of sites. Each site in the network has:

- Its own posts, pages, and media
- Its own theme (chosen from themes available network-wide)
- Its own activated plugins (from plugins installed network-wide)
- Its own admin users
- Its own URL

But they all share:

- The same WordPress core files (`/var/www/monsite/`)
- The same database server (separate tables, same MariaDB instance)
- The same PHP-FPM and Nginx

**Why this matters at scale:**

WordPress.com hosts over 100 million sites. Running a separate WordPress installation for each would be impossible to maintain. Multisite is what makes it feasible — one codebase to update, one security patch to apply, thousands of sites updated simultaneously.

---

## Subdirectory vs Subdomain

WordPress Multisite supports two URL structures:

| Mode | URL | Requires |
|------|-----|---------|
| Subdirectory | `site.com/blog/` | Nothing extra |
| Subdomain | `blog.site.com` | Wildcard DNS record `*.site.com` |

This project uses **subdirectory** mode — simpler for a lab environment without a real domain. The final network:

```
http://192.168.11.103/          → Main site: Biroue Lab
http://192.168.11.103/myblog/   → Biroue Blog
http://192.168.11.103/shop/     → Biroue Shop
http://192.168.11.103/dev/      → Biroue Dev
```

---

## Step 1 — Allow Multisite in wp-config.php

The first step is telling WordPress to enable the Multisite feature. Add one line to `wp-config.php` above the `/* That's all, stop editing! */` marker:

```php
define('WP_ALLOW_MULTISITE', true);
```

This does not activate Multisite yet — it only unlocks the network installer in the admin panel.

---

## Step 2 — Deactivate All Plugins

WordPress requires all plugins to be deactivated before activating the network. Active plugins can interfere with the network setup process.

```bash
sudo /usr/local/bin/wp plugin deactivate --all --path=/var/www/monsite --allow-root
```

Output:
```
Plugin 'wordfence' deactivated.
Success: Deactivated 1 of 3 plugins.
```

---

## Step 3 — Run the Network Installer

Navigate to `http://<server-ip>/wp-admin/network.php`. WordPress presents the network creation form:

- **Installation Type:** Sub-directories
- **Network Title:** Biroue Lab Network
- **Admin Email:** biroueisaac@gmail.com

After clicking **Install**, WordPress generates the configuration to add to `wp-config.php`:

```php
define( 'MULTISITE', true );
define( 'SUBDOMAIN_INSTALL', false );
define( 'DOMAIN_CURRENT_SITE', '192.168.11.103' );
define( 'PATH_CURRENT_SITE', '/' );
define( 'SITE_ID_CURRENT_SITE', 1 );
define( 'BLOG_ID_CURRENT_SITE', 1 );
```

Each constant explained:

| Constant | Value | Purpose |
|----------|-------|---------|
| `MULTISITE` | true | Activates the Multisite network |
| `SUBDOMAIN_INSTALL` | false | Uses subdirectories, not subdomains |
| `DOMAIN_CURRENT_SITE` | server IP | The main domain of the network |
| `PATH_CURRENT_SITE` | `/` | Root path of the network |
| `SITE_ID_CURRENT_SITE` | 1 | ID of the main site in the network |
| `BLOG_ID_CURRENT_SITE` | 1 | Blog ID of the main site |

---

## Step 4 — Configure Nginx for Multisite

This is the critical step that most tutorials gloss over. The default Nginx configuration handles a single WordPress site. With Multisite, Nginx receives requests for `/myblog/`, `/shop/`, `/dev/` — paths that don't physically exist on disk.

**The problem:** Nginx looks for `/var/www/monsite/myblog/index.php`. That directory doesn't exist. Without specific rewrite rules, Nginx returns 404.

**The solution** — add rewrite rules to the Nginx config:

```nginx
location / {
    try_files $uri $uri/ /index.php?$args;
}

if (!-e $request_filename) {
    rewrite /wp-admin$ $scheme://$host$uri/ permanent;
    rewrite ^(/[^/]+)?(/wp-.*) $2 last;
    rewrite ^(/[^/]+)?(/.*\.php) $2 last;
}
```

**What each rewrite does:**

**`if (!-e $request_filename)`** — only applies if the requested file does not exist on disk. This avoids interfering with real files (images, CSS, JS).

**`rewrite /wp-admin$ ... permanent`** — when accessing `/myblog/wp-admin` (no trailing slash), redirect to `/myblog/wp-admin/`. The WordPress admin requires the trailing slash.

**`rewrite ^(/[^/]+)?(/wp-.*) $2 last`** — when a sub-site requests `/myblog/wp-content/themes/...`, rewrite to `/wp-content/themes/...`. All sub-sites share the same `wp-content` directory. The files live in one place.

**`rewrite ^(/[^/]+)?(/.*\.php) $2 last`** — when a sub-site requests `/myblog/index.php`, rewrite to `/index.php`. There is one PHP entry point for the entire network. WordPress internally figures out which site to serve based on the URL.

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

## Step 5 — Create Sub-Sites

In Network Admin (`/wp-admin/network/`), go to **Sites → Add New**:

| Site Address | Title | Result |
|-------------|-------|--------|
| myblog | Biroue Blog | `192.168.11.103/myblog/` |
| shop | Biroue Shop | `192.168.11.103/shop/` |
| dev | Biroue Dev | `192.168.11.103/dev/` |

**Note:** WordPress reserves certain words as site addresses: `blog`, `page`, `feed`, `wp-admin`, `wp-content`, `wp-includes`. Using `myblog` instead of `blog` avoids this restriction.

---

## Step 6 — Reactivate Plugins

```bash
sudo /usr/local/bin/wp plugin activate --all --path=/var/www/monsite --allow-root
```

Output:
```
Plugin 'akismet' activated.
Plugin 'hello' activated.
Plugin 'wordfence' network activated.
Success: Activated 3 of 3 plugins.
```

Note `wordfence` shows **network activated** — it applies automatically to all sites in the network. Individual site admins cannot deactivate it.

---

## Verification

```bash
curl -s -o /dev/null -w "myblog: %{http_code}\n" http://192.168.11.103/myblog/
curl -s -o /dev/null -w "shop:   %{http_code}\n" http://192.168.11.103/shop/
curl -s -o /dev/null -w "dev:    %{http_code}\n" http://192.168.11.103/dev/
```

```
myblog: 200
shop:   200
dev:    200
```

All three sub-sites respond with HTTP 200.

---

## Network Admin vs Site Admin

One of the most important concepts in WordPress Multisite is the two-level administration hierarchy:

| Level | Who | Access | URL |
|-------|-----|--------|-----|
| **Super Admin** | Network administrator | Full control — install plugins/themes, create/delete sites, manage all users | `/wp-admin/network/` |
| **Site Admin** | Individual site owner | Manage their site only — cannot install plugins, only activate from network-available plugins | `/wp-admin/` |

This separation is what makes Multisite viable for hosting platforms. The hosting provider controls which plugins and themes are available. Site owners customize within those boundaries.

**In practice at WordPress.com:** Automattic is the Super Admin. Every WordPress.com site owner is a Site Admin — they can choose their theme, install approved plugins, manage their content. They cannot install arbitrary PHP code or modify the server.

---

## Bash Concepts Reinforced

| Command | What it does |
|---------|-------------|
| `wp plugin deactivate --all` | Deactivate every plugin in one command |
| `wp plugin activate --all` | Reactivate every plugin in one command |
| `curl -w "%{http_code}"` | Check HTTP response code without downloading the page |
| `nginx -t` | Test Nginx configuration before reloading |

---

## What's Next

- WP-04: WordPress Hardening — secure a WordPress site against attacks (file permissions, wp-login.php protection, security headers, Fail2ban)
- WP-05: WordPress Backup — automated full backup with retention and restore script

**Repo:** [github.com/biroue10/wordpress-multisite](https://github.com/biroue10/wordpress-multisite)
