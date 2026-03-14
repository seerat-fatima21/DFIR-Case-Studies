# DFIR Case Study: Web Application Compromise via Unrestricted File Upload

## Overview

This case study documents a forensic investigation of a simulated web application attack using a provided PCAP file. The objective was to reconstruct the full attack chain, from initial reconnaissance through to data exfiltration, by analyzing network traffic and applying core DFIR methodology.

The investigation was conducted using **Wireshark** as the primary analysis tool, supported by **NetworkMiner** for artifact extraction, and **WHOIS/GeoIP** tools for attacker attribution.

---

## Attack Summary

By analyzing the packet capture, I was able to identify a targeted attack originating from IP `117.11.88.124`, geolocated to **China**. The attacker exploited an **unrestricted file upload vulnerability** in the web application, successfully deploying a PHP webshell (`image.php`) to the `/reviews/uploads/` directory. This gave them remote command execution on the server.

The attack was conducted entirely over plain HTTP with no obfuscation or encoding — meaning the full payload and commands were visible in the packet capture, which made attribution and reconstruction straightforward.

---

## Investigation Walkthrough

### Phase 1: Reconnaissance
The attacker began with standard HTTP GET requests to browse the application and identify the attack surface. Nothing appeared immediately alarming, but the volume and pattern of requests indicated automated or scripted activity.

### Phase 2: Exploitation — File Upload
The critical finding was a **POST request** attempting to upload a PHP file to `/reviews/upload.php`. The first attempt failed, but the second attempt succeeded. The application had no server-side validation of file type or content, allowing the attacker to upload a malicious PHP webshell disguised as an image.

- **Webshell name:** `image.php`
- **Upload path:** `/reviews/uploads/.`
- **Vulnerability:** No file type validation, no extension allowlist

### Phase 3: Remote Code Execution
Once the webshell was in place, the attacker used HTTP GET requests to pass commands through it. This gave them the ability to execute arbitrary commands on the server under the web server's user context.

### Phase 4: Data Exfiltration
The attacker targeted `/etc/passwd` for exfiltration, a standard post-exploitation step to enumerate system users and potentially crack offline passwords. The file contents were returned in the HTTP response, confirming successful exfiltration.

### Phase 5: C2 Communication
Outbound connections were observed on **port 8080** and **port 443**, indicating the attacker established a command and control channel to maintain persistent access after the initial compromise.

---

## Key Findings

- **~47% of total network traffic** during the incident was malicious
- The attacker's User-Agent string (`Mozilla/5.0 (X11; Linux x86_64; rv:109.0)`) suggested a Linux-based machine running Firefox, consistent with a Kali Linux attacker environment
- No encryption or obfuscation was used, suggesting the attacker was either inexperienced or confident the traffic wouldn't be monitored
- The entire attack chain from the first malicious packet to exfiltration was reconstructable from the PCAP alone

---

## MITRE ATT&CK Mapping

| Technique | Tactic | ID |
|-----------|--------|----|
| Exploit Public-Facing Application | Initial Access | T1190 |
| Web Shell | Persistence | T1505.003 |
| Data from Local System | Collection | T1005 |
| Exfiltration Over C2 Channel | Exfiltration | T1041 |

---

## Recommendations

The root cause of this compromise was a missing server-side file validation check — a simple fix that would have prevented the entire attack chain.

**To prevent similar incidents:**
- Enforce server-side file type validation using allowlists (not just client-side checks)
- Store uploaded files outside the web root or in a non-executable directory
- Implement a WAF rule to detect and block webshell upload patterns
- Deploy IDS/IPS monitoring for unusual outbound traffic on ports 8080 and 443
- Conduct regular web application vulnerability assessments

---

## Tools Used
- **Wireshark** — Packet capture analysis and stream reconstruction
- **NetworkMiner** — File and credential extraction from PCAP
- **WHOIS / GeoIP** — Attacker IP attribution
- **MITRE ATT&CK Navigator** — Technique mapping

---

> ⚠️ **Disclaimer:** This analysis was performed on a provided lab PCAP file in a controlled educational environment. All findings are simulated for learning purposes only.
