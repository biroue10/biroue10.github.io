---
title: "High Availability — Zero Downtime in Production"
date: 2026-06-11 04:00:00 +0100
categories: [High Availability]
tags: [nginx, load-balancing, mariadb, replication, keepalived, haproxy, spof, rto, rpm, vip, rhel]
pin: false
---

A production system that goes fully offline for a single server failure is not production-ready.
This category documents how to design and operate systems that survive hardware failures,
maintenance windows and traffic spikes.

---

## What this category covers

| Topic | Status |
|-------|--------|
| SPOF, RTO, RPO — concepts | ✅ |
| Nginx load balancer — upstream, algorithms | ✅ |
| Passive health checks | ✅ |
| Sticky sessions vs centralized sessions | ✅ |
| MariaDB master/slave replication | ✅ |
| Keepalived — VIP failover | ⏸ |
| HAProxy — frontend/backend, ACL | ⬜ |
| Galera Cluster — multi-master | ⬜ |

Posts in this category cover **Module 07 — High Availability** and **Module 21 — HAProxy** of the [roadmap](https://github.com/biroue10/syseng-journey/blob/main/ROADMAP.md).
