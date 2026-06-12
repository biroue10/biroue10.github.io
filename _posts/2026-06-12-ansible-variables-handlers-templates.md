---
title: "Ansible — Variables, Handlers and Jinja2 Templates"
date: 2026-06-12 00:00:00 +0100
categories: [Automation, Projects]
tags: [ansible, jinja2, templates, handlers, variables, nginx, rhel, iac, roles, idempotence, devops, linux]
---

Module 08 continues. The skeleton from last session (roles for Nginx, MariaDB, PHP-FPM)
worked — but every value was hardcoded. Today: variables to centralise config,
handlers to restart services only when needed, and Jinja2 templates to generate
config files dynamically.

**Repo:** [biroue10/ansible-lemp-stack](https://github.com/biroue10/ansible-lemp-stack)

---

## The problem with hardcoded values

```yaml
- name: Install Nginx
  dnf:
    name: nginx        # hardcoded
    state: present
```

If you manage 50 servers with different configs, hardcoded tasks become unmaintainable.
One value changes → you edit it in 50 places → you introduce errors.

---

## Variables — defaults/main.yml

Each role has a `defaults/` directory for default variable values:

```yaml
# roles/nginx/defaults/main.yml
nginx_package: nginx
nginx_service: nginx
nginx_worker_processes: auto
```

Reference them in tasks with Jinja2 syntax. When a value **starts** with `{{`, wrap it in quotes — otherwise YAML misparses it as a dictionary:

```yaml
- name: Install Nginx
  dnf:
    name: "{{ nginx_package }}"
    state: present

- name: Start nginx
  service:
    name: "{{ nginx_service }}"
    state: started
    enabled: true
```

**Rule:** Never use hyphens in Ansible variable names — use underscores.

```yaml
php_fpm_service: php-fpm   # ✅ variable name uses underscore, value can have hyphen
php-fpm_service: php-fpm   # ❌ causes a parsing error
```

---

## Handlers — restart only when config changes

Restarting Nginx on every playbook run drops active connections unnecessarily.
A handler only runs when explicitly notified — and is only notified when the
triggering task reports `changed`.

```yaml
# roles/nginx/handlers/main.yml
- name: restart nginx
  service:
    name: "{{ nginx_service }}"
    state: restarted
```

```yaml
# roles/nginx/tasks/main.yml
- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx    # must match handler name exactly
```

The flow:

```
Config unchanged → copy task: ok      → handler not triggered → no restart
Config changed   → copy task: changed → handler triggered     → nginx restarts
```

---

## Jinja2 Templates — dynamic config files

The `copy` module transfers a static file — identical on every server.
The `template` module processes Jinja2 variables first, then copies the result.

One template → infinite configurations.

### Create the template

```bash
cp /etc/nginx/nginx.conf roles/nginx/templates/nginx.conf.j2
```

Replace hardcoded values with variables:

```nginx
worker_processes {{ nginx_worker_processes }};
```

### Use the template module

```yaml
- name: Deploy nginx config
  template:
    src: nginx.conf.j2          # Ansible looks in roles/nginx/templates/
    dest: /etc/nginx/nginx.conf
  notify: restart nginx
```

### Proof it works

Change `nginx_worker_processes: auto` to `nginx_worker_processes: 2` in `defaults/main.yml` and run:

```
TASK [nginx : Deploy nginx config] ──── changed
RUNNING HANDLER [nginx : restart nginx] changed
```

```bash
grep worker_processes /etc/nginx/nginx.conf
# worker_processes 2;
```

Change it back to `auto` → `changed=0`, handler not triggered.

---

## Final role structure

```
roles/nginx/
├── defaults/
│   └── main.yml          # variables: package, service, worker_processes
├── files/
│   └── nginx.conf        # static reference file
├── handlers/
│   └── main.yml          # restart nginx
├── tasks/
│   └── main.yml          # install → start → deploy config
└── templates/
    └── nginx.conf.j2     # Jinja2 template
```

---

## Key takeaways

- `defaults/main.yml` is the single place to change a value — it propagates everywhere
- Handlers enforce idempotence for service restarts — they never run unnecessarily
- The **repo is the source of truth** — never edit `/etc/nginx/nginx.conf` directly on the server
- `template` over `copy` as soon as any value may differ between servers or environments

---

**Next:** Ansible Vault — encrypting secrets so passwords are never stored in plain text.
