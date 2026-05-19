# Problems and Fixes

A log of every major problem hit during this lab and how it was resolved.

---

## Problem 1 — Azure Portal Said Agent Was Installed But It Wasn't

**What happened:**
The Azure portal showed "Provisioning succeeded" for the Azure Monitor Agent.
Inside the VM the RuntimeSettings folder was completely empty.
No config had ever been pushed to the machine so the agent had nothing to act on.

**How I fixed it:**
Deleted and reinstalled the agent using the Windows Security Events via AMA 
connector inside Sentinel instead of installing it manually.
The RuntimeSettings folder populated correctly and the agent started running.

---

## Problem 2 — Logs Landing in the Wrong Table

**What happened:**
Data was flowing but into a table called Event instead of SecurityEvent.
Most Sentinel detection rules are written against SecurityEvent so none of them worked.

**How I fixed it:**
The Data Collection Rule was using the wrong stream type (Microsoft-Event).
Deleted the old DCR and rebuilt it using the Windows Security Events via AMA 
connector which automatically configures the correct stream (Microsoft-SecurityEvent).

---

## Problem 3 — Region Mismatch Silently Breaking the Pipeline

**What happened:**
The original Log Analytics Workspace was created in East US.
The VM ended up in West US 2 due to VM size availability.
Data Collection Rules cannot send data across regions so nothing flowed through.
No error message was shown anywhere.

**How I fixed it:**
Created a new Log Analytics Workspace in West US 2 to match the VM region.
Enabled Sentinel on the new workspace and rebuilt the DCR pointing to it.

---

## Problem 4 — Analytics Rule Never Firing

**What happened:**
The detection rule was configured with a 5 minute lookback window.
Events take 5-10 minutes to ingest into Sentinel.
By the time data arrived it was already outside the query window so the rule never triggered.

**How I fixed it:**
Changed the lookback period from 5 minutes to 30 minutes.
This gives enough buffer for ingestion delay while still catching recent activity.

---

## Problem 5 — Defender Portal Showing Wrong Workspace

**What happened:**
The original ThreatLabWorkspace (East US) was connected to Defender as the primary workspace.
All logs were going to ThreatLabWorkspace2 so Defender was looking at an empty workspace.

**How I fixed it:**
Disconnected the old workspace in Defender portal settings.
Connected ThreatLabWorkspace2 as the primary workspace.
