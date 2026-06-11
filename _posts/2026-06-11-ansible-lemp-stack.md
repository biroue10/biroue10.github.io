---
title: "Project: Ansible LEMP Stack Deployment"
date: 2026-06-11 00:00:00 +0100
categories: [Projects, Ansible]
tags: [ansible, nginx, mariadb, php, rhel, automation, iac]
---

Automated deployment of a full LEMP stack on RHEL 10 using Ansible roles.
This is the first project in my infrastructure-as-code series — the goal is to
go from a bare RHEL server to a fully running Nginx + MariaDB + PHP-FPM stack
with a single command.

**GitHub:** [biroue10/ansible-lemp-stack](https://github.com/biroue10/ansible-lemp-stack)

---

## Stack

| Component  | Package              | Service    |
|------------|----------------------|------------|
| Web server | `nginx`              | `nginx`    |
| Database   | `mariadb-server`     | `mariadb`  |
| PHP runtime| `php` + `php-fpm`    | `php-fpm`  |

---

## Project structure

```
ansible-lemp-stack/
├── inventory/
│   └── hosts
├── roles/
│   ├── nginx/
│   │   └── tasks/main.yml
│   ├── mariadb/
│   │   └── tasks/main.yml
│   └── php/
│       └── tasks/main.yml
└── playbook.yml
```

Ansible **roles** keep each component isolated — nginx, mariadb and php each
live in their own directory with their own tasks. The main playbook just
assembles them:

```yaml
- name: Deploy LEMP Stack
  hosts: lemp
  become: true
  roles:
    - nginx
    - mariadb
    - php
```

---

## Key concept: idempotence

Running the playbook twice produces the same result as running it once.
On the first run, Ansible installs and starts the services (`changed=3`).
On every subsequent run, it checks the desired state against the current state
and does nothing if they already match (`changed=0`).

```
# First run
PLAY RECAP
localhost : ok=7  changed=3  unreachable=0  failed=0

# Second run — nothing changed
PLAY RECAP
localhost : ok=7  changed=0  unreachable=0  failed=0
```

This is the foundation of infrastructure-as-code: the playbook describes *what
the system should look like*, not *what steps to execute*.

---

## Deploy in one command

```bash
git clone https://github.com/biroue10/ansible-lemp-stack.git
cd ansible-lemp-stack
ansible-playbook -i inventory/hosts playbook.yml --ask-become-pass
```

---

## What's next

This is the v1 skeleton. Upcoming iterations will add:

- **Variables** — configurable ports, package versions, database names
- **Handlers** — restart nginx only when the config actually changes
- **Templates (Jinja2)** — generate `nginx.conf` from variables
- **Vault** — encrypt database passwords at rest
