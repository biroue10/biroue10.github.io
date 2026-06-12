---
title: "Ansible Vault — Never Store a Password in Plain Text"
date: 2026-06-12 01:00:00 +0100
categories: [Automation, Projects]
tags: [ansible, vault, security, mariadb, secrets, encryption, iac, rhel, devops, linux]
---

The last piece of Module 08. Variables and templates handle configuration — Vault
handles credentials. A password committed in plain text to GitHub is a password
that has leaked.

**Repo:** [biroue10/ansible-lemp-stack](https://github.com/biroue10/ansible-lemp-stack)

---

## The problem

```yaml
# roles/mariadb/defaults/main.yml
mariadb_root_password: SecurePass123!   # visible to anyone with repo access
```

`defaults/` is version-controlled and readable. Secrets go in a separate encrypted
file — `vars/main.yml` — that Ansible decrypts at runtime using a vault password
you provide on the command line.

---

## Create the encrypted file

```bash
mkdir -p roles/mariadb/vars
ansible-vault create roles/mariadb/vars/main.yml
```

Ansible prompts for a vault password, then opens an editor. Write the secret inside:

```yaml
---
mariadb_root_password: SecurePass123!
```

Save and quit. The file on disk looks like this:

```
$ANSIBLE_VAULT;1.1;AES256
63663737626334383438656433316236366630336438313438623835666163613032333230393639
...
```

This is what gets committed to Git. Without the vault password, it is unreadable.

---

## Use vault variables in tasks

Vault variables work exactly like regular variables — Ansible decrypts transparently at runtime:

```yaml
- name: Set MariaDB root password
  mysql_user:
    name: root
    password: "{{ mariadb_root_password }}"
    login_unix_socket: /var/lib/mysql/mysql.sock
    login_user: root
    login_password: "{{ mariadb_root_password }}"
```

---

## Run the playbook

```bash
ansible-playbook -i inventory/hosts playbook.yml --ask-become-pass --ask-vault-pass
```

Two password prompts:
1. **BECOME password** — sudo on the server
2. **Vault password** — to decrypt `vars/main.yml`

---

## Useful vault commands

```bash
ansible-vault view roles/mariadb/vars/main.yml    # read without decrypting to disk
ansible-vault edit roles/mariadb/vars/main.yml    # edit in place
ansible-vault rekey roles/mariadb/vars/main.yml   # change the vault password
```

---

## Issues hit along the way

### `mariadb_root_password` is undefined
The `vars/` directory didn't exist before `ansible-vault create`.
Fix: `mkdir -p roles/mariadb/vars` first.

### Access denied for user 'root' (using password: NO)
The first run had already set the root password. Subsequent runs need to
authenticate before they can change it. Fix: add `login_user` and `login_password`
to the `mysql_user` task.

### `mariadb.service` refused to restart
`mariadb-check-socket` reads all config files in `/etc/my.cnf.d/` alphabetically
and takes the last value for each option. The slave config from Module 07
(`mariadb-slave.cnf`) defined `port=3307` and a different socket path, which
overrode the master's values because `slave` comes after `server` alphabetically.

Fix:
- Renamed `mariadb-slave.cnf` → `00-mariadb-slave.cnf` so it is read first
- Added `port = 3306` explicitly to `mariadb-server.cnf` so the master always wins

---

## Module 08 complete

| Concept | Status |
|---------|--------|
| Roles | ✅ |
| Variables (defaults) | ✅ |
| Handlers | ✅ |
| Jinja2 Templates | ✅ |
| Vault | ✅ |

Next: **Module 09 — System Security** — hardening, Fail2ban, Let's Encrypt, SELinux advanced.
