---
title: "Security — Defense in Depth on RHEL"
date: 2026-06-11 07:00:00 +0100
categories: [Security]
tags: [selinux, fail2ban, hardening, ssh, firewall, openssl, certbot, auditd, cis, rhel]
pin: false
---

Security is not a feature you add at the end — it is a layer you build into every decision.
This category documents system hardening, SELinux, certificate management and compliance
on RHEL 10.

---

## What this category covers

| Topic | Status |
|-------|--------|
| SELinux — enforcing mode, contexts, booleans | ✅ |
| SELinux — semanage, audit2allow, ausearch | ✅ |
| Firewall — firewall-cmd, zones, rich rules | ✅ |
| HTTPS — self-signed certificates, openssl | ✅ |
| SSH hardening — keys, port, AllowUsers | ⬜ |
| Fail2ban — SSH and Nginx brute force | ⬜ |
| Let's Encrypt with Certbot | ⬜ |
| CIS Benchmark — RHEL hardening | ⬜ |
| OpenSCAP — compliance scanning | ⬜ |
| auditd — system activity logging | ⬜ |

Posts in this category cover **Module 09 — Security** and **Module 32 — Compliance** of the [roadmap](https://github.com/biroue10/syseng-journey/blob/main/ROADMAP.md).
