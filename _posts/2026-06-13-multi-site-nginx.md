---
title: "Project: Multi-Site Nginx — 3 Websites on One Server"
date: 2026-06-13 10:00:00 +0100
categories: [Projects, Nginx]
tags: [nginx, linux, rhel, webserver, virtualhost, sysadmin, configuration]
---

One of the most common tasks for a systems administrator is hosting multiple websites on a single server. This project configures 3 independent virtual hosts on one RHEL server — each with its own domain, files, and log files.

**Repo:** [biroue10/multi-site-nginx](https://github.com/biroue10/multi-site-nginx)

---

## How Nginx Routes Traffic to the Right Site

When a browser sends a request, it includes the domain name in the HTTP header:

```
GET / HTTP/1.1
Host: site1.local
```

Nginx reads this `Host` header and matches it against all `server_name` directives across all config files. The matching block handles the request. This is called a **Virtual Host**.

```
Request: Host: site1.local  →  matches server_name site1.local  →  serves /var/www/site1
Request: Host: site2.local  →  matches server_name site2.local  →  serves /var/www/site2
Request: Host: site3.local  →  matches server_name site3.local  →  serves /var/www/site3
```

3 sites, 1 IP address, 1 Nginx process.

---

## Configuration

Each site gets its own file in `/etc/nginx/conf.d/`. This keeps configurations isolated — disabling a site means removing one file, not editing a monolithic config.

**site1.conf:**
```nginx
server {
    listen 80;
    server_name site1.local;
    root /var/www/site1;
    index index.html;
    access_log /var/log/nginx/site1-access.log;
    error_log  /var/log/nginx/site1-error.log;
}
```

| Directive | Purpose |
|-----------|---------|
| `listen 80` | Accept HTTP traffic on port 80 |
| `server_name` | Domain that triggers this block |
| `root` | Directory where site files live |
| `index` | Default file to serve |
| `access_log` | Requests log — this site only |
| `error_log` | Errors log — this site only |

---

## Local DNS with /etc/hosts

There is no real DNS server in this lab. `/etc/hosts` maps domain names to IPs locally — the system checks it before querying DNS.

```bash
sudo bash -c 'echo "127.0.0.1 site1.local site2.local site3.local" >> /etc/hosts'
```

Now `site1.local` resolves to `127.0.0.1` without any DNS server.

---

## Testing

```bash
sudo nginx -t && sudo systemctl reload nginx

curl http://site1.local
curl http://site2.local
curl http://site3.local
```

**Output:**
```html
<html><body><h1>Site 1</h1></body></html>
<html><body><h1>Site 2</h1></body></html>
<html><body><h1>Site 3</h1></body></html>
```

Each site responds correctly from its own directory.

---

## Separate Logs

Each site writes to its own log:

```
/var/log/nginx/site1-access.log
/var/log/nginx/site2-access.log
/var/log/nginx/site3-access.log
```

**Why it matters:** When multiple sites share a single log file, troubleshooting requires filtering out all other sites' traffic. Separate logs mean you go directly to the right file. In production with high traffic, this saves significant time during incidents.

**Log entry after curl:**
```
127.0.0.1 - - [13/Jun/2026:06:52:59 +0100] "GET / HTTP/1.1" 200 48 "-" "curl/8.12.1"
```

| Field | Value | Meaning |
|-------|-------|---------|
| `127.0.0.1` | Client IP | Request came from localhost |
| `GET / HTTP/1.1` | Request | Method, path, protocol |
| `200` | Status | Success |
| `48` | Bytes | Response size |
| `curl/8.12.1` | User-Agent | Tool that made the request |

---

## Key Concepts

- **Virtual Host** — multiple sites on one IP, separated by domain name
- **`server_name`** — the directive Nginx uses to match incoming requests
- **`conf.d/`** — each file is one site — clean, isolated, easy to manage
- **Separate logs** — essential for troubleshooting in multi-site environments
- **`/etc/hosts`** — local DNS override, no external server needed

---

## What's Next

- HTTPS with Let's Encrypt — automate certificate issuance for each domain
- Custom error pages — branded 404/500 pages per site
- Rate limiting — protect each site independently

**Repo:** [github.com/biroue10/multi-site-nginx](https://github.com/biroue10/multi-site-nginx)
