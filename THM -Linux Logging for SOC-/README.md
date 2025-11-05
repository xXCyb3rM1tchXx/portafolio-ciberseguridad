# Portfolio of Evidence — Linux Logging for SOC
**File:** `README.md`  
**Author:** Mitch  
**Date:** 2025-11-03  
**Description:** Linux Logging for SOC — TryHackMe (Room)  
**Task:** 4  
**Tools:** TryHackMe

---

## Table of Contents
- [General Instructions](#general-instructions)  
- [Learning Objectives](#learning-objectives)  
- [Prerequisites](#prerequisites)  
- [Task 2 — Log Format](#task-2---log-format)  
- [Task 3 — Authentication Logs](#task-3---authentication-logs)  
- [Task 4 — Generic System Logs](#task-4---generic-system-logs)  
- [Task 6 — Audit Daemon](#task-6---audit-daemon)  
- [Conclusions](#conclusions)  
- [Credits](#credits)  
- [Annexes (suggested)](#annexes-suggested)

---

## General Instructions
Organized documentation for the 'Linux Logging for SOC (THM)' exercises.
This document describes the steps performed in the TryHackMe *Linux Logging for SOC* room.  
Each exercise includes: **Objective**, **Useful Commands**, **Procedure**, and **Result**.

---

## Learning Objectives
- Explore Linux authentication, runtime, and system logs.  
- Learn commands and pitfalls when working with logs.  
- Discover how tools like `auditd` monitor and report events.  
- Practice each log source on the attached VM.

---

## Prerequisites
- Access to the TryHackMe VM (or any Linux host with logs).  
- `sudo` privileges to read `/var/log/*` and run `ausearch`.  
- Basic knowledge of bash shell.

---

## Task 2 — Log Format

**Objective:** Identify the time sync domain and Yama kernel message.

**Useful Commands:**
```bash
pwd
ls -la
grep -i "timesync" /var/log/syslog
grep -i "yama" /var/log/syslog
```

**Result / Flag:**
```
TIME_SERVER_DOMAIN: CTF1_timesync
YAMA_KERNEL_MESSAGE: CTF2_yama
```

---

## Task 3 — Authentication Logs

**Objective:** Analyze authentication logs to detect failed SSH attempts and new sudo users.

**Useful Commands:**
```bash
sudo grep "ssh" /var/log/auth.log | grep -E 'Failed'
sudo grep -i "useradd" /var/log/auth.log
```

**Result / Flag:**
```
SSH_FAILED_IP: CTF3_Failed Password
NEW_SUDO_USER: CTF4_useradd
```

---

## Task 4 — Generic System Logs

**Objective:** Determine installed `unzip` version and find a flag in `.bash_history`.

**Useful Commands:**
```bash
sudo grep "unzip" /var/log/dpkg.log
sudo find / -name ".bash_history" 2>/dev/null
sudo cat /root/.bash_history
```

**Result / Flag:**
```
UNZIP_VERSION: CTF5_unzip
BASH_HISTORY_FLAG: CTF5_unzip
```

---

## Task 6 — Audit Daemon

**Objective:** Analyze `auditd` logs for file access, downloads, and scans.

**Useful Commands:**
```bash
sudo ausearch -i -k file_thmsecret
sudo ausearch -i -k proc_wget
sudo ausearch -i | grep -i "naabu"
```

**Result / Flag:**
```
SECRET_OPENED_AT: CTF7_file-thmsecret
WGET_ORIGINAL_FILENAME: CTF8_proc_wge_GitHub
NAABU_SCANNED_RANGE: CTF9_network range
```

---

## Conclusions
- Investigated Linux log sources: `/var/log/syslog`, `/var/log/auth.log`, `/var/log/dpkg.log`, and `auditd`.  
- Tools used: `grep`, `find`, `ausearch`, and manual `.bash_history` inspection.  
- evidences: store screenshots in  `evidences/`

---

## Credits
- **Room:** TryHackMe — *Linux Logging for SOC*  
- **Author:** Mitch  
- **Date:** 2025-11-03

---

## Annexes
- `evidences/` — Screenshots and `.txt` outputs.  