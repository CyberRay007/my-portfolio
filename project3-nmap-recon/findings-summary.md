# Findings Summary — Project 3: Network Reconnaissance

**Target:** 192.168.56.101 (csec VM — Ubuntu Linux)  
**Scanner:** Kali Linux 2026.1 (192.168.56.102)  
**Scan Date:** April 28, 2026  
**Analyst:** Favour (cyberray007)

---

## Executive Summary

A full network reconnaissance engagement was conducted against the target host `192.168.56.101` within an isolated VirtualBox lab environment. The target was found to be running **three exposed TCP services**, with one representing a **critical severity vulnerability** (ProFTPD 1.3.3c backdoor). Two additional services are running outdated versions with documented CVEs. Immediate remediation is recommended.

---

## Risk Summary

| Risk Level | Count |
|---|---|
| 🔴 Critical | 1 |
| 🟠 High | 1 |
| 🟡 Medium | 2 |
| 🟢 Low | 1 |

---

## Open Ports

| Port | Protocol | Service | Version | Risk |
|---|---|---|---|---|
| 21 | TCP | FTP | ProFTPD 1.3.3c | 🔴 Critical |
| 22 | TCP | SSH | OpenSSH 7.6p1 | 🟡 Medium |
| 80 | TCP | HTTP | Apache httpd 2.4.29 | 🟠 High |
| 5353 | UDP | mDNS | Zeroconf | 🟢 Low |

---

## Critical Finding — ProFTPD 1.3.3c Backdoor

**CVE:** CVE-2010-4221  
**CVSS Score:** 10.0 (Critical)  
**Description:** ProFTPD version 1.3.3c was distributed with a malicious backdoor injected into the source code via a compromised distribution server in November 2010. The backdoor listens on port 6200 and provides unauthenticated root shell access.

**Exploitation potential:** HIGH — Metasploit module available (`exploit/unix/ftp/proftpd_133c_backdoor`)

**Recommendation:** Replace ProFTPD immediately. If FTP is required, upgrade to latest stable release and configure with TLS (FTPS). Preferably, migrate to SFTP over OpenSSH.

---

## Scan Commands Reference

```bash
# Phase 1 - Connectivity
ping -c 5 192.168.56.101

# Phase 2 - Host Discovery
nmap -sn 192.168.56.0/24

# Phase 3 - TCP SYN Scan
sudo nmap -sS 192.168.56.101

# Phase 4 - Service/Version Scan
sudo nmap -sV -sC 192.168.56.101

# Phase 5 - UDP Scan
sudo nmap -sU 192.168.56.101

# Phase 6 - XML Export
sudo nmap -sS -sV -oX scan.xml 192.168.56.101
xsltproc scan.xml -o scan.html
```
