---
title: "Project: WordPress Deployer — Automated WordPress Installation on RHEL"
date: 2026-06-14 05:00:00 +0100
categories: [Projects, WordPress]
tags: [wordpress, bash, nginx, php, mariadb, rhel, linux, automation, sysadmin, wpcli]
---

Installing WordPress manually is a sequence of repetitive steps — create the database, download the files, edit the config, set permissions, configure Nginx. Do it once and you understand it. Do it ten times and you automate it. This script deploys a complete WordPress site on RHEL in one command.

**Repo:** [biroue10/wordpress-deployer](https://github.com/biroue10/wordpress-deployer)

---

## How WordPress Works — The Full Stack

Before automating anything, you need to understand what you're automating.

```
Browser sends request
       ↓
     Nginx
       ↓
   Is it PHP?
   ├── No  → Nginx serves the file directly (HTML, CSS, images)
   └── Yes → Nginx passes it to PHP-FPM
                    ↓
                PHP-FPM executes the WordPress code
                    ↓
              WordPress queries MariaDB
                    ↓
              MariaDB returns data
                    ↓
              PHP-FPM returns HTML to Nginx
                    ↓
              Nginx sends HTML to browser
```

Four components, one request. Each has a specific role — none of them can be removed.

---

## PHP CLI vs PHP-FPM

`php --version` shows PHP installed — but that's PHP CLI (Command Line Interface). You run it manually in the terminal for scripts and tasks.

**PHP-FPM** is different. It's a permanent service that stays running in the background, waiting for Nginx to send it PHP files to execute:

```
php-fpm: master process
├── php-fpm: pool www    ← worker, waiting
├── php-fpm: pool www    ← worker, waiting
├── php-fpm: pool www    ← worker, waiting
├── php-fpm: pool www    ← worker, waiting
└── php-fpm: pool www    ← worker, waiting
```

5 workers ready to handle 5 simultaneous requests. Without PHP-FPM running as a service, Nginx cannot execute PHP.

---

## PHP Extensions WordPress Requires

WordPress doesn't work with a bare PHP installation. It needs specific extensions:

| Extension | Purpose |
|-----------|---------|
| `php-mysqlnd` | PHP ↔ MariaDB communication — without this, WordPress cannot connect to its database |
| `php-json` | JSON data handling — used by the REST API and many plugins |
| `php-xml` | XML processing — required for feeds, sitemaps, some plugins |
| `php-mbstring` | Multi-byte character support — without this, accented characters and non-Latin scripts break |
| `php-gd` | Image library — WordPress uses it to resize uploaded images and generate thumbnails |

```bash
sudo dnf install -y php-mysqlnd php-json php-xml php-mbstring php-gd
sudo systemctl restart php-fpm
```

---

## The Script — Step by Step

### Variables

```bash
SITE_NAME="monsite"
SITE_DOMAIN="monsite.local"
WEB_ROOT="/var/www/${SITE_NAME}"
DB_NAME="${SITE_NAME}_db"
DB_USER="${SITE_NAME}_user"
DB_PASS="WordPress2025!"
DB_ROOT_PASS="root_password"
WP_URL="https://wordpress.org/latest.tar.gz"
```

All configuration lives at the top. `${SITE_NAME}` is reused in `WEB_ROOT`, `DB_NAME`, and `DB_USER` — changing one variable updates everything automatically.

---

### Web Root

```bash
create_web_root() {
    sudo mkdir -p "${WEB_ROOT}"
    sudo chown -R apache:apache "${WEB_ROOT}"
    sudo chmod -R 755 "${WEB_ROOT}"
}
```

| Command | Purpose |
|---------|---------|
| `mkdir -p` | Create directory and all parent directories |
| `chown -R apache:apache` | Give ownership to the Apache/PHP user — it needs to write files |
| `chmod -R 755` | Owner: full access · Group/Others: read + execute only |

755 means the web server can read and navigate the directory, but only the owner can write. This prevents other users or processes from modifying site files.

---

### Database

```bash
create_database() {
    sudo mysql -u root -p"${DB_ROOT_PASS}" <<EOF
CREATE DATABASE IF NOT EXISTS ${DB_NAME} CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASS}';
GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';
FLUSH PRIVILEGES;
EOF
}
```

`<<EOF` sends multiple lines directly to mysql — same as opening the MariaDB prompt and typing each command, but automated.

`utf8mb4` is important: it's the character set that supports emojis and all Unicode characters. The older `utf8` in MySQL/MariaDB only supports 3-byte characters — emojis require 4 bytes and would fail silently.

---

### WordPress Download

```bash
install_wordpress() {
    cd /tmp
    curl -O "${WP_URL}"
    tar -xzf latest.tar.gz
    sudo cp -R wordpress/. "${WEB_ROOT}/"
    sudo chown -R apache:apache "${WEB_ROOT}"
    rm -rf /tmp/wordpress /tmp/latest.tar.gz
}
```

Download to `/tmp` — a temporary directory — not directly to the web root. Extract there, copy the files, then clean up. `/tmp` is automatically cleaned on reboot. No leftover archives on the server.

`tar -xzf`:
- `-x` — extract
- `-z` — decompress gzip
- `-f` — file to process

---

### wp-config.php

```bash
configure_wordpress() {
    sudo cp "${WEB_ROOT}/wp-config-sample.php" "${WEB_ROOT}/wp-config.php"
    sudo sed -i "s/database_name_here/${DB_NAME}/" "${WEB_ROOT}/wp-config.php"
    sudo sed -i "s/username_here/${DB_USER}/" "${WEB_ROOT}/wp-config.php"
    sudo sed -i "s/password_here/${DB_PASS}/" "${WEB_ROOT}/wp-config.php"
}
```

WordPress ships with `wp-config-sample.php` — a template. Copy it first, then replace the placeholder values with `sed`. The original sample stays intact as a reference.

`sed -i "s/old/new/"` — in-place substitution. Replaces text directly in the file without opening an editor.

---

### Nginx Virtual Host

```nginx
server {
    listen 80;
    server_name monsite.local;
    root /var/www/monsite;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Two key blocks:

**`location /`** — `try_files` tells Nginx: look for the file, then the directory, then fall back to `index.php`. This is what makes WordPress permalinks work — `/about/` doesn't map to a real directory, Nginx passes it to WordPress via `index.php`.

**`location ~ \.php$`** — any URL ending in `.php` goes to PHP-FPM via a Unix socket. A socket is faster than a TCP port — it's a direct file-based connection between two processes on the same machine.

---

## Heredoc and Variable Escaping

Writing Nginx configs inside bash scripts has a pitfall. Nginx uses `$uri`, `$args`, `$document_root` — all starting with `$`. Bash also uses `$` for variables. Inside a heredoc, bash tries to expand `$uri` as a bash variable — it finds nothing and substitutes an empty string.

**The fix:** single-quoted heredoc `<< 'EOF'` tells bash to treat everything literally — no variable expansion.

```bash
cat > /tmp/nginx-wp.conf << 'NGINXEOF'
    try_files $uri $uri/ /index.php?$args;   ← $uri stays as $uri
NGINXEOF
sed -i "s|SITE_DOMAIN_PLACEHOLDER|${SITE_DOMAIN}|g" /tmp/nginx-wp.conf
```

Use placeholders for bash variables, replace them after with `sed`. Nginx variables stay untouched.

---

## WP-CLI Installation

```bash
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
```

WP-CLI is the command-line interface for WordPress. It replaces the browser-based installer and allows full WordPress management from the terminal:

```bash
sudo /usr/local/bin/wp core install \
  --path=/var/www/monsite \
  --url=http://monsite.local \
  --title="Biroue Lab" \
  --admin_user=admin \
  --admin_password=Admin2025! \
  --admin_email=your@email.com \
  --allow-root
```

```
Success: WordPress installed successfully.
```

---

## Verification

```bash
curl -I http://monsite.local
```

```
HTTP/1.1 200 OK
Server: nginx/1.26.3
X-Powered-By: PHP/8.3.31
Link: <http://monsite.local/wp-json/>; rel="https://api.w.org/"
```

`200 OK` — WordPress is running. The `Link` header confirms the WordPress REST API is active.

---

## Bash Concepts Used

| Concept | Where |
|---------|-------|
| Variables | All configuration at the top |
| Functions | `create_web_root`, `create_database`, `install_wordpress`, `configure_wordpress`, `configure_nginx` |
| Heredoc `<<EOF` | Send multiple SQL commands to mysql |
| Single-quoted heredoc `<<'EOF'` | Prevent bash from expanding Nginx variables |
| `sed -i` | In-place file substitution for wp-config.php |
| Pipes `\|` | curl → tar, sed replacements |
| `curl -O` | Download files from URL |

---

## What's Next

- WP-CLI toolkit — manage WordPress entirely from the command line
- WordPress hardening — security best practices for production
- HTTPS with Let's Encrypt — automatic certificate for the WordPress site
- WooCommerce setup — deploy a full e-commerce store

**Repo:** [github.com/biroue10/wordpress-deployer](https://github.com/biroue10/wordpress-deployer)
