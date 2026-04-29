# Project 3 — Network Reconnaissance & Enumeration with Nmap

> **Lab Environment:** VirtualBox Home Lab | Kali Linux 2026.1 (Attacker) → Ubuntu Linux (Target: csec VM)  
> **Date:** April 28, 2026  
> **Author:** Favour (cyberray007)  
> **Difficulty:** Beginner → Intermediate  
> **Category:** Reconnaissance | Network Enumeration | Port Scanning

---

## 🎯 Objective

Perform structured network reconnaissance against a target VM (`192.168.56.101`) using **Nmap**, progressing through host discovery, TCP/UDP port scanning, service fingerprinting, OS detection, and professional report generation — simulating the **reconnaissance phase** of a real-world penetration test or red team engagement.

---

## 🗺️ Lab Environment

```
┌─────────────────────────────────────────────────────┐
│              VirtualBox Host-Only Network            │
│                  192.168.56.0/24                     │
│                                                      │
│   ┌─────────────────┐      ┌──────────────────────┐ │
│   │   Kali Linux    │      │    csec (Ubuntu)     │ │
│   │  192.168.56.102 │◄────►│   192.168.56.101     │ │
│   │   (Attacker)    │      │     (Target)         │ │
│   └─────────────────┘      └──────────────────────┘ │
│                                                      │
│   Also discovered:                                   │
│   192.168.56.1   (VirtualBox Gateway)                │
│   192.168.56.100 (DHCP Server)                       │
└─────────────────────────────────────────────────────┘
```

| Machine | IP Address | Role | OS |
|---|---|---|---|
| Kali Linux 2026.1 | 192.168.56.102 | Attacker / Scanner | Kali Linux |
| csec VM | 192.168.56.101 | Target | Ubuntu Linux |
| VirtualBox Gateway | 192.168.56.1 | Gateway | — |
| DHCP Server | 192.168.56.100 | DHCP | — |

---

## 🔬 Scanning Methodology

This project follows a **layered enumeration approach**, progressing from broad discovery to deep service analysis:

```
Phase 1: Verify Connectivity      →  ping / ip a
Phase 2: Host Discovery           →  nmap -sn (ping sweep)
Phase 3: TCP Port Scan            →  nmap -sS (SYN scan)
Phase 4: Service & Version Scan   →  nmap -sV -sC
Phase 5: UDP Port Scan            →  nmap -sU
Phase 6: Output & Reporting       →  nmap -oX (XML export)
```

---

## 📋 Phase-by-Phase Findings

### Phase 1 — Connectivity Verification

**Command used:**
```bash
ip a          # Check attacker interfaces
ping -c 5 192.168.56.102   # Verify target is reachable
```

**Result:** ✅ Ping successful — 5/5 packets received, 0% packet loss  
Average RTT: 8.582ms — target is live and reachable on the host-only network.

---

### Phase 2 — Host Discovery (Ping Sweep)

**Command used:**
```bash
nmap -sn 192.168.56.0/24
```

**Result:** 4 live hosts discovered across the /24 subnet.

| Host | Status | Notes |
|---|---|---|
| 192.168.56.1 | Up | VirtualBox gateway |
| 192.168.56.100 | Up (0.0031s) | MAC: Oracle VirtualBox NIC |
| 192.168.56.101 | Up (0.0093s) | MAC: Oracle VirtualBox NIC — **Primary Target** |
| 192.168.56.102 | Up | Kali attacker machine |

> 💡 **Analyst Note:** The `-sn` flag performs host discovery only (no port scan). All 4 hosts responded — the MAC addresses confirm this is a VirtualBox environment.

---

### Phase 3 — TCP SYN Scan (Stealth Scan)

**Command used:**
```bash
sudo nmap -sS 192.168.56.101
```

**Result:** 3 open TCP ports discovered, 997 closed (reset).

| Port | Protocol | State | Service |
|---|---|---|---|
| 21 | tcp | **open** | ftp |
| 22 | tcp | **open** | ssh |
| 80 | tcp | **open** | http |

> 💡 **Analyst Note:** SYN scan (`-sS`) is the default stealth scan — it sends SYN packets and analyses responses without completing the TCP handshake, making it less likely to be logged by applications (though still visible at network level).

---

### Phase 4 — Service Version & Script Scan

**Command used:**
```bash
sudo nmap -sV -sC 192.168.56.101
```

**Detailed Findings:**

| Port | Service | Product | Version | Extra Info |
|---|---|---|---|---|
| 21/tcp | FTP | **ProFTPD** | **1.3.3c** | — |
| 22/tcp | SSH | **OpenSSH** | **7.6p1** | Ubuntu 4ubuntu0.7; protocol 2.0 |
| 80/tcp | HTTP | **Apache httpd** | **2.4.29** | Ubuntu |

**SSH Host Keys discovered:**
```
2048 d6:01:90:39:2d:8f:46:fb:03:86:73:b3:3c:54:7e:54 (RSA)
 256 f1:f3:c0:dd:ba:a4:85:f7:13:9a:da:3a:bb:4d:93:04 (ECDSA)
 256 12:e2:98:2d:a3:e7:36:4f:be:6b:ce:36:6b:7e:0d:9e (ED25519)
```

**HTTP Server Header:** `Apache/2.4.29 (Ubuntu)`  
**HTTP Title:** Site does not have a title (text/html)

**OS Detection:** Unix, Linux — CPE: `cpe:/o:linux:linux_kernel`

> 🚨 **Security Finding:** **ProFTPD 1.3.3c** is a notably vulnerable version — it contains the infamous **backdoor vulnerability (CVE-2010-4221)** introduced via a compromised source tarball. This version should be flagged as a **critical risk** in a real engagement.

> 🚨 **Security Finding:** **Apache 2.4.29** is outdated and has multiple known CVEs. The current stable release is significantly newer.

---

### Phase 5 — UDP Scan

**Command used:**
```bash
sudo nmap -sU 192.168.56.101
```

> ⚠️ UDP scanning is slow by nature — this scan took **1773.12 seconds** (29+ minutes). This is normal. In real engagements, you would limit UDP scans to top ports: `--top-ports 100`.

**Significant UDP ports (open|filtered):**

| Port | State | Service | Note |
|---|---|---|---|
| 68/udp | open\|filtered | dhcp | DHCP client |
| 5353/udp | **open** | zeroconf | mDNS — confirmed open |
| 6000/udp | open\|filtered | X11 | Potential GUI exposure |
| 2002/udp | open\|filtered | globe | — |

> 💡 **Analyst Note:** `open|filtered` means Nmap cannot determine if the port is open or filtered because UDP does not send a response for open ports — only closed ports send ICMP "port unreachable." The confirmed open port is **5353/udp (mDNS/Zeroconf)**.

---

### Phase 6 — XML Output & Report Generation

**Command used:**
```bash
sudo nmap -sS -sV -oX scan.xml 192.168.56.101
xsltproc scan.xml -o scan.html     # Convert XML to HTML report
```

**Output files generated:**
- `scan.xml` — machine-readable Nmap output (importable to Metasploit, Nessus, etc.)
- `scan.html` — human-readable HTML report (rendered in browser)

---

## 🔴 Vulnerabilities Identified

| # | Service | Version | Vulnerability | Severity | CVE Reference |
|---|---|---|---|---|---|
| 1 | ProFTPD | 1.3.3c | Backdoor command execution | **CRITICAL** | CVE-2010-4221 |
| 2 | Apache httpd | 2.4.29 | Multiple CVEs (EOL version) | **HIGH** | Various |
| 3 | OpenSSH | 7.6p1 | Username enumeration | **MEDIUM** | CVE-2018-15919 |
| 4 | FTP Port 21 | ProFTPD 1.3.3c | Anonymous login check needed | **MEDIUM** | — |
| 5 | mDNS 5353/udp | Zeroconf | Service/host information disclosure | **LOW** | — |

---

## 🛡️ Recommendations

| Finding | Recommendation |
|---|---|
| ProFTPD 1.3.3c | **Immediately patch or replace** — this version has a known backdoor. Upgrade to latest stable or switch to vsftpd/sftp. |
| Apache 2.4.29 | Upgrade to current stable Apache (2.4.62+). Apply security patches. |
| OpenSSH 7.6p1 | Upgrade to 9.x series. Disable password auth, enforce key-based login only. |
| FTP (Port 21) | Disable FTP entirely if not needed. Use SFTP (SSH) instead. Never allow anonymous FTP. |
| UDP Exposure | Firewall unused UDP ports. Restrict mDNS to internal segments only. |
| HTTP (Port 80) | Force HTTPS. Deploy TLS certificate. Set security headers (HSTS, CSP, X-Frame-Options). |

---

## 📁 Repository Structure

```
project3-nmap-recon/
├── README.md                   ← This file (full writeup)
├── scans/
│   ├── scan.xml                ← Nmap XML output
│   └── scan.html               ← HTML report
├── screenshots/
│   ├── 01-network-connectivity.png
│   ├── 02-host-discovery.png
│   ├── 03-tcp-syn-scan.png
│   ├── 04-service-version-scan.png
│   ├── 05-udp-scan.png
│   ├── 06-xml-export.png
│   └── 07-html-report.png
└── reports/
    └── findings-summary.md     ← Executive summary
```

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Nmap 7.98 | Network scanner — host discovery, port scanning, service detection |
| xsltproc | XML-to-HTML transformation for Nmap report |
| VirtualBox | Hypervisor for lab environment |
| Kali Linux 2026.1 | Attacker OS |

---

## 📚 Key Nmap Flags Reference

| Flag | Description |
|---|---|
| `-sn` | Ping sweep (host discovery, no port scan) |
| `-sS` | TCP SYN (stealth) scan |
| `-sV` | Service version detection |
| `-sC` | Default NSE script scan |
| `-sU` | UDP scan |
| `-oX` | Output to XML format |
| `-A` | Aggressive scan (OS detect + version + scripts + traceroute) |
| `--top-ports N` | Scan top N most common ports |

---

## 🔗 Related Projects

- [Project 1 — SIEM Home Lab with Splunk](../project1-siem-splunk/)
- [Project 2 — Threat Hunting with MITRE ATT&CK](../project2-threat-hunting/)
- [IR-001 — Phishing Email Incident Response](../IR-001-phishing/)

---

## ⚠️ Disclaimer

> This project was conducted in a **private, isolated VirtualBox lab environment** for educational purposes only. All scanning and enumeration was performed against virtual machines I own and control. Performing unauthorized network scanning against systems you do not own is **illegal** and unethical.

---

*Part of the [30-Day Cybersecurity Lab Project Series](../README.md) by cyberray007*
