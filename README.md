# AgentTesla SOC Incident Investigation
### SOC Tier 2, Full Incident Response Report | Globex Manufacturing Ltd

---

## 📋 Project Overview

This repository documents a complete **SOC Tier 2 incident investigation** conducted as part of a hands-on cybersecurity project at **AltSchool Africa**. The investigation covers a real-world phishing and malware exfiltration scenario at a simulated multinational organisation, **Globex Manufacturing Ltd**.

On **December 4, 2024**, a staff member reported opening a suspicious email attachment. The investigation confirmed an active **AgentTesla infostealer** infection that resulted in the exfiltration of browser credentials, session cookies, and keylogger output to an attacker-controlled FTP server.

The investigation was conducted across **two phases**:

| Phase | Investigation Type | Key Tool |
|---|---|---|
| Phase 1 | Phishing Email Analysis | Email header analysis, VirusTotal, emldump |
| Phase 2 | Network Traffic Analysis | Wireshark PCAP investigation |

---

## 👩‍💻 Analyst

| Field | Detail |
|---|---|
| **Name** | Iniobong Johnson |
| **Role** | SOC Analyst Tier 2 (Project) |
| **Institution** | AltSchool Africa |
| **Incident Date** | 04 December 2024 |
| **Report Date** | 31 May 2026 |
| **Classification** | CONFIDENTIAL — INTERNAL USE ONLY |

---

## 🎯 Incident Summary

### What Happened
A Globex Manufacturing staff member received a **phishing email** disguised as a purchase quotation from a Turkish supplier (`acronas[.]com[.]tr`). The email contained a malicious attachment named `TECHNICAL SPECIFICATIONS.TAR`, which was actually a **Win32 RAR Archive** containing an **AgentTesla Trojan Dropper**.

Once executed, the malware:
1. Performed external IP reconnaissance via `api[.]ipify[.]org`
2. Resolved the attacker's FTP domain `ftp[.]ercolina-usa[.]com`
3. Established a **cleartext FTP connection** on Port 21
4. Exfiltrated **browser credentials, session cookies, and keylogger output**
5. Entered a **20-minute dormant collection phase** before re-establishing contact

---

## 📧 Phase 1 — Phishing Email Analysis

### Key Findings

| Field | Finding |
|---|---|
| Sender | `sertan@acronas[.]com[.]tr` |
| Originating IP | `94[.]141[.]120[.]32` (DGTL TECH UK LLP) |
| SPF Result | Softfail — IP not authorised |
| DKIM Result | None |
| DMARC Result | None |
| Spoofing Detected | Yes — Highly probable |
| Social Engineering | Pretexting — Purchase Quotation lure |
| Attachment Name | `TECHNICAL SPECIFICATIONS.TAR` |
| True File Type | Win32 RAR Archive (v4) — disguised as .TAR |
| Malware Family | AgentTesla Trojan Dropper |
| VirusTotal Detections | 49 / 72 vendors flagged as malicious |

### Cryptographic Hashes

| Hash Type | Value |
|---|---|
| MD5 | `b7635c9cc63619099419c68a2bf0d390` |
| SHA1 | `98683c9ee69dd591473629d231aacb1020db91b4` |
| SHA256 | `5c98308c69c84a57214442e2cadc9f8f0fcdbab8e6050f9915ac336b6f1d59f0` |

### Risk Rating
> ⚠️ **HIGH** — Execution of this payload would result in silent credential theft, browser session hijacking, and lateral network penetration.

📄 [View Full Phishing Analysis Report](./01_Phishing_Analysis/Globex_Phishing_Email_Analysis.pdf)

---

## 🌐 Phase 2 — Network Traffic Analysis

### Key Findings

| Field | Finding |
|---|---|
| Victim IP | `10[.]12[.]4[.]101` |
| Victim Host | `DESKTOP-VJCRXEB` |
| Victim User | `gary.strickman` |
| FTP Server IP | `192[.]254[.]225[.]136` |
| FTP Domain | `ftp[.]ercolina-usa[.]com` |
| FTP Username | `ben@ercolina-usa[.]com` |
| FTP Password | `nXe 0M~WkW&n` (captured in cleartext) |
| Total Packets | 157 (87 directed to attacker IP) |
| Exfiltration Port | Port 21 — Unencrypted FTP |

### Files Exfiltrated via FTP STOR Commands

| File Prefix | Content | Timestamp |
|---|---|---|
| `PW_` | Harvested browser credentials | 21:21:03 |
| `CO_` | Google Chrome session cookies | 21:21:03 |
| `CO_` | Microsoft Edge session cookies | 21:21:04 |
| `KL_` | Keylogger output — recorded keystrokes | 21:41:08 |

### Attack Timeline

| # | Timestamp | Event |
|---|---|---|
| 1 | — | Email received by victim (not in PCAP) |
| 2 | — | Attachment opened (not in PCAP) |
| 3 | 21:20:56 | Malware execution — DNS query to `api[.]ipify[.]org` |
| 4 | 21:21:02 | DNS resolution — `ftp[.]ercolina-usa[.]com` → `192[.]254[.]225[.]136` |
| 5 | 21:21:02 | FTP login — cleartext credentials transmitted |
| 6 | 21:21:03 | Data exfiltration Phase 1 — PW_ and CO_ files uploaded |
| 7 | 21:21:04 | Data exfiltration Phase 2 — Edge cookies uploaded |
| 8 | 21:21:05 – 21:41:03 | Idle / collection — keylogger active for 20 minutes |
| 9 | 21:41:07 | Secondary DNS lookup — re-establishing FTP connection |
| 10 | 21:41:08 | KL_ keylogger file uploaded — secondary exfiltration complete |
| 11 | Post 21:41:08 | No further traffic observed in PCAP window |

📄 [View Full Network Analysis Report](./02_Network_Analysis/Globex_Network_Traffic_Analysis.pdf)

---

## 🚨 Master IOC Table

### Email-Based IOCs

| IOC Type | Indicator Value |
|---|---|
| Sender Email | `sertan@acronas[.]com[.]tr` |
| Originating IP | `94[.]141[.]120[.]32` |
| Serving IP | `94[.]73[.]145[.]234` |
| Sender Domain | `acronas[.]com[.]tr` |
| URL | `hxxp[://]acronas[.]com[.]tr` |
| File Name | `TECHNICAL SPECIFICATIONS.TAR` |
| MD5 Hash | `b7635c9cc63619099419c68a2bf0d390` |
| SHA1 Hash | `98683c9ee69dd591473629d231aacb1020db91b4` |
| SHA256 Hash | `5c98308c69c84a57214442e2cadc9f8f0fcdbab8e6050f9915ac336b6f1d59f0` |

### Network-Based IOCs

| IOC Type | Indicator Value |
|---|---|
| Victim IP | `10[.]12[.]4[.]101` |
| FTP Server IP | `192[.]254[.]225[.]136` |
| FTP Domain | `ftp[.]ercolina-usa[.]com` |
| Reconnaissance Domain | `api[.]ipify[.]org` |
| FTP Username | `ben@ercolina-usa[.]com` |
| FTP Password | `nXe 0M~WkW&n` |
| Network Port | `21` (Outbound FTP) |

---

## 🛡️ Defense Recommendations

### Immediate Actions
- Isolate host `10[.]12[.]4[.]101` from the network
- Block all outbound traffic to `192[.]254[.]225[.]136`
- Block outbound FTP Port 21 at the perimeter firewall
- Reset all credentials for user `gary.strickman`
- Invalidate active Chrome and Edge browser sessions
- Preserve the full PCAP file as forensic evidence
- Quarantine the malicious attachment hash across all endpoints

### Long-Term Controls
- Block all outbound FTP traffic permanently at the firewall
- Deploy an Endpoint Detection and Response (EDR) solution
- Implement DNS filtering to block newly observed domains
- Enable MFA on all corporate accounts
- Deploy email attachment sandboxing at the mail gateway
- Enforce DMARC, DKIM, and SPF policies
- Conduct regular phishing awareness training for all staff

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Wireshark | PCAP network traffic analysis |
| emldump.py | Email parsing and IOC extraction |
| eioc.py | Automated IOC extraction from email file |
| VirusTotal | Malware hash analysis |
| DomainTools | Reverse IP lookup |
| sha256sum / md5sum | Cryptographic hash extraction |
| Sublime Text | Raw email header inspection |

---

## 📁 Deliverables

| File | Description |
|---|---|
| [Phishing Analysis Report](./01_Phishing_Analysis/Globex_Phishing_Email_Analysis.pdf) | Full Phase 1 investigation report |
| [Network Analysis Report](./02_Network_Analysis/Globex_Network_Traffic_Analysis.pdf) | Full Phase 2 investigation report |
| [Executive Summary Deck](./03_Executive_Summary/Globex_SOC_Executive_Summary.pptx) | 8-slide presentation for stakeholders |
| [IOC Summary Table](./04_IOC_Summary/IOC_Summary_Table.xlsx) | Consolidated IOC reference document |

---

> *This project was completed as part of the AltSchool Africa cybersecurity curriculum.*
>

Submitted by: Iniobong Johnson · Inib12
