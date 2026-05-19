# Detection Rules

All custom KQL analytics rules built during this lab.
Each rule is deployed in Microsoft Sentinel and raises incidents in Microsoft Defender XDR.

---

## Rule 1 — Brute Force Login Attempts

**What it detects:**
Repeated failed logon attempts against the machine within a short window.
Indicative of a brute force or password spraying attack.

**Event ID:** 4625 (Failed Logon)
**Severity:** Medium

**Attack simulated:**
```powershell
1..10 | ForEach-Object {
    net use \\localhost\IPC$ /user:fakeuser$_ wrongpassword123 2>$null
    net use \\localhost\IPC$ /delete 2>$null
}
```

**KQL:**
```kql
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by Computer, bin(TimeGenerated, 5m)
| where FailedAttempts >= 3
```

**Rule settings:**
- Run frequency: every 5 minutes
- Lookback: 30 minutes
- Alert threshold: greater than 0 results

---

## Rule 2 — Backdoor Admin Account Created

**What it detects:**
A new local user account being created on the machine.
Indicative of an attacker creating a backdoor account for persistent access.

**Event ID:** 4720 (User Account Created)
**Severity:** High

**Attack simulated:**
```powershell
New-LocalUser -Name "backdoor" -Password (ConvertTo-SecureString "Password123!" -AsPlainText -Force)
Add-LocalGroupMember -Group "Administrators" -Member "backdoor"
```

**KQL:**
```kql
SecurityEvent
| where EventID == 4720
| where TimeGenerated > ago(30m)
```

**Rule settings:**
- Run frequency: every 5 minutes
- Lookback: 30 minutes
- Alert threshold: greater than 0 results

---

## Rule 3 — Suspicious Recon Commands Detected

**What it detects:**
Known attacker reconnaissance commands being run on the machine.
Indicative of an attacker enumerating users, privileges, and stored credentials.

**Event ID:** 4688 (Process Creation)
**Severity:** Medium

**Attack simulated:**
```powershell
whoami /priv
net localgroup administrators
query user
cmdkey /list
```

**KQL:**
```kql
SecurityEvent
| where EventID == 4688
| where CommandLine has_any ("whoami", "cmdkey", "query user", "net localgroup")
| where TimeGenerated > ago(30m)
```

**Rule settings:**
- Run frequency: every 5 minutes
- Lookback: 30 minutes
- Alert threshold: greater than 0 results
