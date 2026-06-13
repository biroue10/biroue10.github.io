---
title: "My Roadmap: From Service Desk to Senior Systems Engineer"
date: 2026-06-13 00:00:00 +0100
categories: [Meta, Roadmap]
tags: [roadmap, linux, sysadmin, career, junior, senior, docker, kubernetes, ansible, aws, rhcsa]
pin: false
---

This is the complete roadmap I'm following — from Service Desk Analyst to Junior Systems Administrator, then all the way to Senior Systems Engineer. Every module has a side project. No theory without practice.

**Full roadmap on GitHub:** [biroue10/syseng-journey](https://github.com/biroue10/syseng-journey/blob/main/FULL_ROADMAP.md)

---

## Where I Am Now

```
Service Desk  ──►  [Junior SysAdmin]  ──►  SysAdmin  ──►  Senior SysEng
     └── Linux ── Web stack ── Containers ── Cloud ── SRE ──┘
```

Currently building toward **Junior Systems Administrator**.

---

## Level 1 — Junior Systems Administrator

### Linux Core

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| Linux Fundamentals | users, permissions, systemd, logs | User provisioning script | ✅ |
| Linux Internals | LVM, memory, processes, performance | Performance dashboard | ✅ |
| Bash Scripting | functions, parsing, error handling, cron | Server audit tool | ⬜ |

### Networking

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| Networking Fundamentals | IP, TCP/UDP, routing, subnets | Network mapper | ⬜ |
| DNS & DHCP | BIND, Unbound, records, leases | Home lab DNS server | ⬜ |
| Network Security | firewalld, tcpdump, nmap, VPN | IDS monitor | ⬜ |

### Web & Application Stack

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| Nginx | vhosts, reverse proxy, SSL | Multi-tenant Nginx platform | ✅ |
| MariaDB | SQL, backup, replication | DB backup & monitoring | ✅ |
| PHP-FPM / LEMP | PHP-FPM, OpCache, WordPress | WordPress production deploy | ⬜ |

### Monitoring

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| Prometheus + Grafana | metrics, dashboards, alerting | Full monitoring stack | ✅ |
| Log Management | Loki, rsyslog, logrotate | Centralized log platform | ⬜ |

### Security

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| System Hardening | SSH, SELinux, Fail2ban, auditd | CIS hardening playbook | 🔄 |
| PKI & Certificates | OpenSSL, Let's Encrypt, internal CA | Internal PKI | ⬜ |
| Vulnerability Management | OpenSCAP, Lynis, patch management | Patch manager | ⬜ |

### Automation

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| Ansible | roles, templates, vault | LEMP stack automation | ✅ |
| Git & CI/CD | branches, GitHub Actions, pipelines | Automated deploy pipeline | ⬜ |

### Containers

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| Docker | images, Dockerfile, Compose | Containerized LEMP stack | 🔄 |
| Container Security | Trivy, distroless, secrets | Secure container pipeline | ⬜ |

---

## Level 2 — Systems Administrator

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| Kubernetes Fundamentals | pods, deployments, services, RBAC | LEMP on K8s | ⬜ |
| Kubernetes Operations | upgrades, limits, probes, PDB | Production k3s cluster | ⬜ |
| Helm | charts, values, publish | Helm chart library | ⬜ |
| K8s Monitoring | Prometheus Operator, network policies | K8s observability | ⬜ |
| High Availability | HAProxy, Galera, Keepalived | Full HA lab | 🔄 |
| Backup & DR | rsync, xtrabackup, RTO/RPO | Enterprise backup system | ⬜ |
| Chaos Engineering | Chaos Toolkit, game days | Chaos testing framework | ⬜ |
| AWS Core | EC2, S3, VPC, IAM, RDS | AWS infrastructure (Terraform) | ⬜ |
| AWS Operations | CloudWatch, Systems Manager, costs | AWS governance tool | ⬜ |
| GitOps | ArgoCD, Flux, drift detection | GitOps-driven K8s | ⬜ |
| HashiCorp Vault | dynamic secrets, PKI, K8s integration | Secrets management platform | ⬜ |
| Terraform | IaC, modules, remote state | Complete IaC lab | ⬜ |

---

## Level 3 — Senior Systems Engineer

| Module | Key Skills | Side Project | Status |
|--------|-----------|-------------|--------|
| SRE Fundamentals | SLO, error budget, toil, post-mortems | SRE dashboard | ⬜ |
| Advanced Observability | Jaeger, Tempo, OpenTelemetry, profiling | Full observability platform | ⬜ |
| Performance Engineering | profiling, k6, flamegraphs, eBPF | Performance testing framework | ⬜ |
| Zero Trust | mTLS, Keycloak SSO, OPA, Vault | Zero trust lab | ⬜ |
| DevSecOps | SAST, DAST, container scanning, InSpec | DevSecOps pipeline | ⬜ |
| System Design | Redis, Kafka, microservices, CAP | Scalable architecture | ⬜ |
| Platform Engineering | Backstage, golden paths, IDP | Internal developer platform | ⬜ |

---

## Certifications

```
RHCSA → RHCE → CKA → AWS SAA → CKS
```

| Cert | Level | Validates |
|------|-------|-----------|
| RHCSA (EX200) | Junior | Linux administration |
| RHCE (EX294) | Mid | Ansible automation |
| CKA | Mid-Senior | Kubernetes administration |
| AWS SAA | Mid-Senior | Cloud architecture |
| CKS | Senior | Kubernetes security |

---

## The Rule

Every module ends with a working side project committed to GitHub.
No certification without the project behind it.
No theory without a server that proves it works.

**Progress is tracked live:** [github.com/biroue10/syseng-journey](https://github.com/biroue10/syseng-journey)
