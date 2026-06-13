---
title: "Project: Net-Mapper — Detect Every Device on Your Network"
date: 2026-06-13 08:00:00 +0100
categories: [Projects, Bash]
tags: [bash, linux, networking, nmap, security, scripting, rhel, sysadmin]
---

A systems administrator must know what is connected to their network at all times. An unknown device on your network is a security risk. This Bash script scans the local network, lists all connected devices, and detects changes between runs — new devices appearing or disappearing.

**Repo:** [biroue10/net-mapper](https://github.com/biroue10/net-mapper)

---

## The Problem

When a new device connects to your network, you want to know about it. This could be:
- A legitimate new device (new employee laptop, new server)
- An unauthorized device (someone connected to your WiFi without permission)
- A compromised device appearing after being infected

Checking manually is impossible. A script that runs on schedule and reports changes is the right approach.

---

## The Tool — nmap

`nmap` (Network Mapper) is the standard tool for network scanning. Used by sysadmins and security engineers worldwide.

```bash
nmap -sn 192.168.11.0/24
```

- `-sn` — ping scan only, no port scanning — just detect who is alive
- `192.168.11.0/24` — scan all 254 addresses in the subnet

**Sample raw output:**
```
Starting Nmap 7.92 at 2026-06-13 05:05 +01
Nmap scan report for 192.168.11.1
Host is up (0.0075s latency).
Nmap scan report for 192.168.11.101
Host is up (0.011s latency).
Nmap scan report for 192.168.11.103
Host is up (0.00014s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 3.27 seconds
```

---

## How the Script Works

### The Logic

```
First run:
  → Scan network → save to REPORT_FIRST
  → No previous scan to compare → nothing to diff

Second run:
  → Copy REPORT_FIRST (previous scan) to REPORT_SECOND
  → Scan network again → save to REPORT_FIRST
  → diff REPORT_SECOND vs REPORT_FIRST → show changes
```

Each run keeps the previous scan as a reference. Changes between runs are immediately visible.

---

### Variables

```bash
PLAGE="192.168.11.0/24"
REPORT_FIRST="/tmp/scan_actuel.txt"
REPORT_SECOND="/tmp/nouveau_scan.txt"
```

The network range and file paths are defined once as variables. Changing the network range only requires editing one line.

---

### Filtering with grep

The raw nmap output includes latency times which change between every scan — comparing them directly would always show differences even when no device changed.

Solution: keep only the lines containing IP addresses:

```bash
nmap -sn $PLAGE | grep "Nmap scan report" | tee $REPORT_FIRST
```

**Before filtering:**
```
Nmap scan report for 192.168.11.1
Host is up (0.0075s latency).       ← changes every run
Nmap scan report for 192.168.11.101
Host is up (0.011s latency).        ← changes every run
```

**After filtering:**
```
Nmap scan report for 192.168.11.1
Nmap scan report for 192.168.11.101
```

Now comparisons are meaningful — only actual device changes appear.

---

### The compare_scan function

```bash
compare_scan(){
    print_section "COMPARAISON"
    if [ -f $REPORT_FIRST ]; then
        cp $REPORT_FIRST $REPORT_SECOND
        scan_network
    fi
    diff $REPORT_SECOND $REPORT_FIRST
}
```

- `if [ -f $REPORT_FIRST ]` — checks if a previous scan exists
- `cp $REPORT_FIRST $REPORT_SECOND` — saves the old scan before overwriting it
- `scan_network` — runs a fresh scan into `REPORT_FIRST`
- `diff $REPORT_SECOND $REPORT_FIRST` — compares old vs new

---

## Real Test — Device Detection

**Scenario:** A phone connects to the network between two runs.

```
=============================
  COMPARAISON
=============================

=============================
  scanning en cours
=============================
Nmap scan report for 192.168.11.1
Nmap scan report for 192.168.11.101
Nmap scan report for 192.168.11.103
1a2
> Nmap scan report for 192.168.11.101
```

`>` means the device appeared in the new scan — it was not there before. The script detected the phone connecting to the network.

**Reading diff output:**

| Symbol | Meaning |
|--------|---------|
| `>` | New device — appeared in the latest scan |
| `<` | Device gone — was in the previous scan, now missing |
| Nothing | No change — network is identical |

---

## Full Script

```bash
#!/bin/bash

PLAGE="192.168.11.0/24"
REPORT_FIRST="/tmp/scan_actuel.txt"
REPORT_SECOND="/tmp/nouveau_scan.txt"

print_section() {
    echo ""
    echo "============================="
    echo "  $1"
    echo "============================="
}

scan_network() {
    print_section "scanning en cours"
    nmap -sn $PLAGE | grep "Nmap scan report" | tee $REPORT_FIRST
}

compare_scan() {
    print_section "COMPARAISON"
    if [ -f $REPORT_FIRST ]; then
        cp $REPORT_FIRST $REPORT_SECOND
        scan_network
    fi
    diff $REPORT_SECOND $REPORT_FIRST
}

main() {
    compare_scan
}

main
```

---

## Bash Concepts Used

| Concept | Where |
|---------|-------|
| Variables | `PLAGE`, `REPORT_FIRST`, `REPORT_SECOND` |
| Functions | `print_section`, `scan_network`, `compare_scan`, `main` |
| Pipes `\|` | Chain nmap → grep → tee |
| `tee` | Display output AND save to file simultaneously |
| `if [ -f ]` | Check if file exists before comparing |
| `cp` | Copy previous scan before overwriting |
| `diff` | Compare two files and show differences |

---

## What's Next

- Add hostname resolution — show device name next to IP
- Alert via email when a new device is detected
- Schedule with cron — run every hour automatically
- Log all detected changes with timestamps

**Repo:** [github.com/biroue10/net-mapper](https://github.com/biroue10/net-mapper)
