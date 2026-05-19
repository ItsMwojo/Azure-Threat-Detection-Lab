# Azure-Threat-Detection-Lab

## Overview
A hands-on home lab built in Microsoft Azure to simulate real attacks, 
ingest logs into Microsoft Sentinel, and build custom detection rules 
that raise incidents in Microsoft Defender XDR.


---

## Architecture

| Resource | Name | Region |
|---|---|---|
| Subscription | ThreatLabSubscription | — |
| Virtual Machine | VulnVM (Windows Server 2025) | West US 2 |
| Log Analytics Workspace | ThreatLabWorkspace2 | West US 2 |
| Microsoft Sentinel | On ThreatLabWorkspace2 | West US 2 |
| Data Collection Rule | LabDCR3 | West US 2 |
| Azure Monitor Agent | Running on VulnVM | — |

---

## Lab Phases

### Phase 1 — Foundation
- Created dedicated Azure subscription
- Deployed Log Analytics Workspace
- Enabled Microsoft Sentinel

### Phase 2 — Vulnerable VM
- Deployed Windows Server 2025 VM
- Disabled Windows Defender and Firewall
- Enabled WinRM
- Created weak local admin account
- Opened RDP (3389) and SMB (445) in NSG

### Phase 3 — Log Pipeline
- Installed Azure Monitor Agent on VulnVM
- Created Data Collection Rule via Windows Security Events via AMA connector
- Confirmed SecurityEvent table populating in Sentinel

### Phase 4 — Brute Force Detection
- Simulated repeated failed logon attempts
- Built KQL detection rule triggering on 3+ failures in 5 minutes
- Confirmed incident raised in Microsoft Defender

### Phase 5 — Backdoor Admin Account Detection
- Simulated attacker creating a hidden admin account
- Built KQL detection rule watching for Event ID 4720
- Confirmed High severity incident raised in Microsoft Defender

### Phase 6 — Workbook Dashboard
- Built Sentinel Workbook with failed logon chart, top targeted accounts table, and new admin accounts panel

### Phase 7 — Suspicious Recon Commands
- Simulated attacker running common reconnaissance commands
- Built KQL detection rule watching for known recon process names via Event ID 4688

---

## Contents
- [Detection Rules](DETECTION-RULES.md)
- [Problems and Fixes](PROBLEMS.md)
- [Screenshots](SCREENSHOTS.md)

---

## Key Lessons Learned

1. Always verify agent status inside the VM itself. Portal status cannot be trusted, spent a lot of time very confused there.
2. Every resource must be in the same Azure region or the pipeline silently breaks, leftover resources from a old lab caused me pain with that.
3. Microsoft-Event and Microsoft-SecurityEvent are different streams that write to different tables, I would get them mixed up.
4. Set analytics rule lookback to at least 30 minutes to account for ingestion delay, not giving it enough time made me edit it thinking its not working.
5. The SecurityEvent table does not exist until the first event arrives. This is normal.

---

## Tools Used
- Microsoft Azure
- Microsoft Sentinel
- Microsoft Defender XDR
- Azure Monitor Agent
- KQL (Kusto Query Language)
- PowerShell
