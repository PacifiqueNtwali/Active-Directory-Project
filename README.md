# Active Directory Attack Detection Lab (Splunk + Sysmon)

## Overview
I designed and deployed a small on-prem style Active Directory environment to simulate common attack scenarios and validate security telemetry using Splunk.

This project demonstrates my ability to:
- Build and configure a Windows domain
- Generate realistic attack activity
- Ingest and analyze endpoint and directory telemetry
- Troubleshoot real infrastructure and logging failures

> Inspired by MyDFIR’s AD detection series. I extended the lab through independent troubleshooting, attack tool pivoting, and additional detection validation beyond the guide.

---

## Architecture

![Logical Diagram](https://i.imgur.com/5klzKDq.png)


**Environment Flow**
- Windows Server 2022 → Domain Controller (AD DS, DNS)
- Windows 10 → Domain-joined endpoint
- Ubuntu Server → Splunk Enterprise (SIEM)
- Kali Linux → Attacker
- Logs shipped via Splunk Universal Forwarder + Sysmon

---

## Environment Details

### Systems
- Windows Server 2022 (Domain Controller)
- Windows 10 (Domain endpoint)
- Ubuntu Server 22.04 (Splunk)
- Kali Linux (Attacker)

  ![Systems](https://imgur.com/y3yMz15.png)

### Tools
- Splunk Enterprise
- Splunk Universal Forwarder
- Sysmon
- Atomic Red Team
- Hydra

---

## Attack Scenarios Simulated

### 1️⃣ RDP Brute Force Attack
- Multiple failed logons against a domain user
- Generated clustered authentication failures

 ![Attack Simulated](https://imgur.com/a/yxzXGfl.png)

## Shows Kali ip

[![kali ip](https://imgur.com/a/thom19t.png)
](https://i.imgur.com/CYJxu6R.png)

<img width="537" height="281" alt="image" src="https://github.com/user-attachments/assets/3ceb2dbe-5125-4c82-9b59-0959ae64c561" />

---

### 2️⃣ Successful Authentication After Brute Force
- Confirmed successful login after failures
- Verified attacker source IP (Kali Linux)

![Successful Authentication After Brute Force](https://i.imgur.com/X7pL9Qe.png)



![kali ip](https://imgur.com/a/thom19t.png)

---

### 3️⃣ Active Directory User Attribute Changes
- Modified password policies on a domain user
- Observed inconsistent but valid Event ID 4738 telemetry

📍 Screenshot:  
`screenshots/02-detections/4738-user-changed.png`

---

### 4️⃣ Atomic Red Team Simulation
- Executed Atomic test for local account creation
- Confirmed creation and deletion events in Splunk

📍 Screenshot:  
`screenshots/02-detections/art-newlocaluser.png`

---

## Telemetry Validation

| Event ID | Meaning |
|--------|--------|
| 4625 | Failed logon (brute force indicator) |
| 4624 | Successful logon |
| 4738 | AD user account modified |
| 4720 / 4726 | Account created / deleted |

Splunk queries used for detection are stored in `/queries`.

---

## Troubleshooting & Fixes

### ❌ Splunk Web Not Accessible (Port 8000)
**Problem**
- Windows endpoint could not access Splunk UI

**Fix**
- Opened TCP 8000 inbound/outbound in Windows Defender Firewall
- Allowed port 8000 via UFW on Ubuntu

📍 Screenshot:  
`screenshots/03-troubleshooting/splunk-8000-fixed.png`

---

### ❌ Crowbar Attack Tool Failed
**Problem**
- Crowbar did not function reliably in my environment

**Fix**
- Pivoted to Hydra and successfully generated authentication telemetry

📍 Screenshot:  
`screenshots/03-troubleshooting/hydra-success.png`

---

## Key Skills Demonstrated
- Active Directory administration
- Windows authentication internals
- SIEM ingestion and querying
- Security event analysis
- Network and firewall troubleshooting
- Attack simulation & detection mapping

---

## Reproduction (High-Level)
1. Build 4 VMs on the same NAT network
2. Assign static IPs and verify routing
3. Install Splunk Enterprise and enable receiving on port 9997
4. Install Sysmon + Universal Forwarder on Windows hosts
5. Promote Windows Server to Domain Controller
6. Join Windows 10 to the domain
7. Execute attacks and validate telemetry in Splunk

---

## Screenshots Folder Structure
All screenshots are intentionally organized:

