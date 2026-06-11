---
title: "Start Here — Welcome to My Syseng Journey"
date: 2026-06-01 00:00:00 +0100
categories: [Linux]
tags: [rhel, linux, sysadmin, devops, ansible, roadmap, syseng-journey]
pin: true
---

Welcome. This blog documents my journey from **Service Desk Analyst** to **Senior Systems Engineer** — hands-on, on a real RHEL 10 server, one module at a time.

Every post here comes from something I actually built, broke, and fixed. No tutorials copied without practice. No theory without verification.

---

## Who I am

**Biroue Isaac** — Service Desk Analyst L2 at Insight, based in Casablanca.

4+ years in IT support, now systematically building the infrastructure engineering skills required to move into a Senior Systems Engineer role.

Full profile → [About](/about/)

---

## The roadmap

I follow a structured 38-module curriculum built from real Senior Systems Engineer job requirements:

```
Foundation → Services → High Availability → Automation → Observability → Cloud
```

| Track | Modules | Status |
|-------|---------|--------|
| **Linux & System Administration** | 01, 06 | ✅ Done |
| **Web Stack** | 02, 15 | ✅ Done |
| **Databases** | 03, 07, 25 | ✅ / 🔄 |
| **Monitoring & Observability** | 04, 14 | ✅ / ⬜ |
| **High Availability** | 07, 21 | 🔄 In progress |
| **Automation** | 08, 13, 28 | 🔄 In progress |
| **Security** | 09, 32 | ⬜ Upcoming |
| **Containers & K8s** | 11, 12 | ⬜ Upcoming |
| **Cloud** | 30 | ⬜ Upcoming |

Full roadmap → [syseng-journey/ROADMAP.md](https://github.com/biroue10/syseng-journey/blob/main/ROADMAP.md)

---

## Where to start reading

| If you're interested in… | Start with… |
|--------------------------|-------------|
| Nginx load balancing | [High Availability](/categories/high-availability/) |
| MariaDB replication | [Databases](/categories/databases/) |
| Ansible automation | [Automation](/categories/automation/) |
| Prometheus & Grafana | [Monitoring](/categories/monitoring/) |
| SELinux troubleshooting | [Security](/categories/security/) |
| All projects with code | [Projects](/categories/projects/) |

---

## Lab environment

```yaml
OS:      Red Hat Enterprise Linux 10.2
Kernel:  6.12.0
Access:  SSH + Tailscale VPN
Stack:   Nginx · MariaDB · Ansible · Prometheus · Grafana
```

---

## Follow the journey

- GitHub: [github.com/biroue10](https://github.com/biroue10)
- All notes and configs: [syseng-journey](https://github.com/biroue10/syseng-journey)
- Ansible project: [ansible-lemp-stack](https://github.com/biroue10/ansible-lemp-stack)

If something here helped you or you have a question, leave a comment below.
