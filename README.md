#Active-Directory-Project — Partial Lab Documentation

Status: Work in progress — documenting completed work only (I'll finish the rest later).
Guided run: Apr 14, 2025 → Apr 21, 2025
Credit: Adapted from MyDfir (YouTube)

Summary (what I completed)

I built the initial sections of an on-prem AD lab and documented the steps I performed through network/design, VM deployment, and initial telemetry collection setup. This README covers only the parts I completed:

Part 1 — Logical diagram

Part 2 — VM installation & base OS setup (VirtualBox)

Part 3 — Network config, Splunk install on Ubuntu, Splunk UF and Sysmon prep on Windows hosts

I will document the remaining steps (AD promotion, domain join, attack telemetry generation, detections) later.

Objective (short)

Create a small on-premises lab that demonstrates AD + endpoint telemetry ingestion into Splunk so I can later exercise detection scenarios.

What I learned / demonstrated

Creating a clear logical diagram for the lab

Deploying and configuring VMs in VirtualBox (Windows Server 2022, Windows 10, Ubuntu Server 22.04, Kali)

Creating a NAT network and assigning static IPs

Installing Splunk Enterprise on Ubuntu and enabling it at boot

Installing Splunk Universal Forwarder (UF) on Windows and preparing inputs.conf

Installing Sysmon (with community config) on Windows hosts

Troubleshooting connectivity issues (Windows Defender firewall, ufw on Ubuntu)

Using Hydra as an alternative to Crowbar for demonstrations

Lab components (as deployed)

VirtualBox NAT network: 192.168.10.0/24

Splunk (Ubuntu Server 22.04): 192.168.10.10 — Splunk Web :8000, receive UF :9997

Windows Server 2022 (AD placeholder VM)

Windows 10 (target endpoint)

Kali Linux (attacker)

Sysmon + Splunk UF installed on Windows Server and Windows 10

Files / assets included

draw.io logical diagram (referenced in project notes)

screenshots taken during installation and troubleshooting

local inputs.conf snippets used on Windows UF (referenced below)

Quick steps I executed
Part 1 — Logical diagram

Created a draw.io diagram showing Splunk, AD/DC, Windows 10 target, Kali attacker, switch, router, and cloud.

Marked attacker host red; used dotted green arrows to indicate logs/forwarding to Splunk.

Annotated icons with roles and IP addresses to reference during setup.

Part 2 — VM setup (VirtualBox)

Deployed VMs:

Windows Server 2022 (intended AD/DC)

Windows 10 (target)

Ubuntu Server 22.04 (Splunk indexer)

Kali Linux (attacker)

Allocated extra CPU/RAM/disk for Splunk VM because it will index/search.

Took snapshots after base OS installs.

Part 3 — Network & Splunk + UF + Sysmon setup

Created VirtualBox NAT network 192.168.10.0/24 and attached all VMs to it.

Set Splunk VM static IP on Ubuntu using netplan. Example used:

network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.10.10/24]
      gateway4: 192.168.10.1
      nameservers:
        addresses: [8.8.8.8]


Then sudo netplan apply and verified with ip a.

Installed Splunk Enterprise (.deb) on Ubuntu:

sudo dpkg -i splunk-<version>.deb

cd /opt/splunk/bin && sudo ./splunk start

Created admin user when prompted

Enabled boot-start: sudo ./splunk enable boot-start -user splunk

Prepared Splunk Universal Forwarder (UF) on Windows:

Downloaded UF installer and installed on the Windows hosts.

Created a local inputs.conf (placed in C:\Program Files\SplunkUniversalForwarder\etc\system\local\) to collect Application, System, Security and Sysmon logs and set index = endpoint.

Ensured the Splunk UF service runs under Local System (if permission issues occurred).

Installed Sysmon and applied a vetted configuration (e.g., Olaf/other) on both Windows hosts to capture high-fidelity telemetry.

In Splunk Web (post-install):

Created index endpoint.

Enabled receiving on port 9997.

Validated: searched index=endpoint in Splunk Search & Reporting and confirmed events and host names appeared.

Troubleshooting & notes

Splunk web initially unreachable from the Windows host (attempted http://192.168.10.10:8000).

Root cause: firewall rules. Resolved by adding inbound/outbound rules in Windows Defender for port 8000 and updating ufw on Ubuntu to allow traffic from the Windows VM.

Splunk UF service sometimes lacked privileges to read Windows event logs. Fixed by switching the service account to Local System.

Crowbar in the guided walkthrough did not function in my environment; used Hydra for a demo instead.

After configuring inputs.conf, the Splunk UF service must be restarted for changes to take effect.

Example inputs.conf (local)

Place this under:
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf

[WinEventLog://Application]
disabled = 0
index = endpoint

[WinEventLog://System]
disabled = 0
index = endpoint

[WinEventLog://Security]
disabled = 0
index = endpoint

[monitor://C:\Windows\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx]
disabled = 0
index = endpoint
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

Next (to be documented later)

Promote Windows Server to Domain Controller, create domain users, and join Windows 10 to the domain

Run Atomic Red Team / PowerShell / Kali techniques to generate telemetry and validate detections

Create Splunk searches/dashboards for the telemetry
