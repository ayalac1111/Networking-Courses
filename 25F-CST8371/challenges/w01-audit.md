# W01 Audit — Running Config Analysis

## What is an **Audit**?  
An **Audit** is a *read/diagnose/plan* exercise: you analyze a **static running-config** provided to you, identify what’s wrong, prescribe the **minimal fixes**, and list how you would **verify** those fixes. It does **not** require live devices. This measures config-reading speed, fault recognition, and the ability to select the right verifications. 

---
## Week 1 Baseline

For this exercise, use the following **target state**:

1. **Remote access:** VTY must allow **SSH only (SSH v2)** using the local user `admin`.  
2. **Logging:** **Syslog** must point to **192.0.2.250**.  
3. **Time:** The **Audit router** acts as the **NTP master (stratum 5)** for all internal routers.

> **Goal:** Identify **three** violations of this baseline, write the **minimal IOS fix** for each, and include **one verify command** per issue.

---
## What to Do
- Read the **embedded** running-config in the section below.
- Write exactly **three** `findings`, **three** `fix` lines (minimal!), and **three** `verify` lines.
- Keep each value **one line**; be concise and professional.

---
## Deliverable
- **Filename:** `w01-audit-username.txt` (all lowercase; e.g., `w01-audit-ayalac.txt`)
- **Where:** Brightspace → Assignment folder **01-audit**
- **Points:** **3 points total** — **1 point per issue** (*finding + minimal fix + a reasonable verify line*).
- **Weighting:** The **Challenges/Audits** category is **7%** of the final course grade.

---
### Audit Submission Template

```plaintext
findings1=<one clear issue>
findings2=<one clear issue>
findings3=<one clear issue>
fix1=<exact IOS line>
fix2=<exact IOS line>
fix3=<exact IOS line>
verify1=<command -> expected “good”>
verify2=<command -> expected “good”>
verify3=<command -> expected “good”>
```

**What each field means:**
- `findings#` — WHAT is wrong (one-liners). Name the risk/symptom, not the whole config.
- `fix#` — the **minimal** IOS command(s) to repair those issues (each on its own line).
- `verify#` — the command you’d run **and** the single “good” you expect (proof line).

**Tips**
- **Findings**: be specific (e.g., “Telnet permitted on VTY”, “Syslog host wrong”, “CDP disabled on Gi0/0”).
- **Fixes**: choose the **fewest lines** that achieve the baseline; avoid cosmetic changes.
- **Verify**: name the command and the one-line success (e.g., `show ip ssh -> Version 2`).

---

### Week 1 - `RTR-AUDIT-1` Router Configuration

```plaintext

! running-config  — Week 1 Audit
! Purpose: identify misconfigurations blocking the Week-1 baseline and prescribe minimal fixes.

version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname RTR-AUDIT-1
ip domain-name cst8371.net
!
username admin privilege 15 secret 5 $1$abc$abcdefghijklmnopqrstu
!
interface GigabitEthernet0/0
 description Uplink to Remote
 ip address 203.0.113.250 255.255.255.0
!
interface GigabitEthernet0/1
 description LAN
 ip address 10.20.20.1 255.255.255.0
!
ip ssh version 2
!
logging trap informational
logging host 192.0.2.50
!

!
line vty 0 4
 transport input ssh
 exec-timeout 10 0
!
end
```

---
## Sample Audit Exercise

You are given the following **running-config**. Identify what violates the **target state**, then propose **minimal** fixes.

**Target state**
- VTY allows **SSH only (SSH v2)** using local user `admin`
- **Syslog** points to **198.51.50.50**
- **Neighbour discovery (CDP)** is visible on the **uplink (Gi0/0)**

```plaintext
! Sample running-config
! Your task: find baseline violations and prescribe minimal fixes.
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname RTR-AUDIT-SAMPLE
ip domain-name example.local
!
username admin privilege 15 secret 5 $1$abc$abcdefghijklmnopqrstu
!
interface GigabitEthernet0/0
 description Uplink to Remote
 ip address 203.0.113.100 255.255.255.0
 no cdp enable                    ! ISSUE: CDP disabled on the Remote uplink
!
interface GigabitEthernet0/1
 description LAN
 ip address 10.10.20.1 255.255.255.0
!
ip ssh version 2                  ! OK: SSHv2 already enabled
!
logging trap informational
logging host 198.51.100.50        ! ISSUE: wrong syslog host (should be 198.51.50.50)
!
line vty 0 4
 login local
 transport input telnet           ! ISSUE: Telnet allowed; must be SSH-only
 exec-timeout 10 0
!
end
```
Sample audit submission (one line per key)
```plaintext
findings1=VTY allows Telnet; access must be SSH-only.
findings2=Syslog host set to 198.51.100.50 (incorrect).
findings3=CDP disabled on uplink Gi0/0.
fix1=transport input ssh
fix2=logging host 198.51.50.50
fix3=cdp enable
verify1=show running-config | include transport input -> transport input ssh
verify2=show logging | include 198.51.50.50 -> logging to 198.51.50.50
verify3=show cdp neighbors -> Remote appears on Gi0/0
```

### Why this is “minimal”
- Only the lines required to satisfy the baseline are changed (no cosmetic edits).
- SSHv2 is already set globally, so we don’t add extra SSH commands.
- Each verify is a single, checkable line tied to the fix.
---
## How to work (suggested)
1. Skim in the order of the target state: **VTY/SSH → Logging → CDP (uplinks) → IP/routes**.  
2. Write **three** precise findings.  
3. Add the **fewest** IOS commands that fix them.  
4. For each issue, write **one** verify command and the “good” you expect.  
5. Save and submit your text file in Brightspace.

---
### Why this matters
Being able to troubleshoot by **reading running-configs** is a core networking skill: it builds speed in spotting policy gaps, choosing **minimal** fixes, and proving the result with **focused verification**.

