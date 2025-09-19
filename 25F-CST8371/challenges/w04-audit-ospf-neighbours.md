# W04 Audit — We Are Not Neighbours

## What is an **Audit**?
An **Audit** is a *read/diagnose/plan* exercise: you analyze a **static running-config** provided to you, identify what’s wrong, prescribe the **minimal fixes**, and list how you would **verify** those fixes. No live devices are required. This measures config-reading speed, fault recognition, and choosing the right verifications.

---

## Week 4 Baseline (Neighbouring)
For this exercise, use the following **target state**:

1. **Neighbouring:** OSPF adjacencies must form on all three **point-to-point** links (R1–R2, R2–R3, R3–R1) in **area 0**.  
2. **Interface mode only:** Enable OSPF **under each transit interface**; **do not** use `network` statements.  
3. **Timers & masks:** **Hello/Dead** timers **match** across each link; **IP/masks** match on each point-to-point pair.

> **Goal:** Identify **three** violations of this baseline (mask mismatch, area mismatch, hello/dead mismatch), write the **minimal IOS fix** for each, and include **one verify command** per issue.

---

## What to Do
- Read the **embedded** running-configs below for **R1, R2, R3**.  
- Write exactly **three** `findings`, **three** `fix` lines (minimal!), and **three** `verify` lines.  
- Keep each value **one line**; be concise and professional.

---

## Deliverable
- **Filename:** `w04-audit-{username}.txt` (all lowercase; e.g., `w04-audit-ayalac.txt`)  
- **Where:** Brightspace → Assignment folder **W04-audit**  
- **Points:** **3 points total** — **1 point per issue** (*finding + minimal fix + a reasonable verify line*).  
- **Weighting:** Counts under **Challenges/Audits** in the course grade.

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
- `findings#` — WHAT is wrong (one-liners).  
- `fix#` — the **fewest** IOS command(s) that repair the issue.  
- `verify#` — the command you’d run **and** the single “good” you expect (proof line).

---

## Week 4 — Router Configurations (R1, R2, R3)

### Topology addressing (for reference)
- **Link-A (R1 Gi0/0 ↔ R2 Gi0/0):** 10.0.12.0/30 (R1 .1 /30, R2 .2 **should** be /30)  
- **Link-B (R2 Gi0/1 ↔ R3 Gi0/1):** 10.0.23.0/30 (.2 and .3, both /30)  
- **Link-C (R3 Gi0/2 ↔ R1 Gi0/2):** 10.0.31.0/30 (.3 and .1, both /30)

> **Expected good:** all three links are **area 0**, timers **match**, neighbours reach **FULL**.

---

### R1 — `RTR-NEIGH-1`

```plaintext
version 15.4
hostname RTR-NEIGH-1
!
interface GigabitEthernet0/0
 description Link-A to R2
 ip address 10.0.12.1 255.255.255.252     
 ip ospf 1 area 0
!
interface GigabitEthernet0/2
 description Link-C to R3
 ip address 10.0.31.1 255.255.255.252     
 ip ospf 1 area 0                          
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
router ospf 10
 router-id 1.1.1.1
end
```

### R2 — `RTR-NEIGH-2`

```plaintext
version 15.4
hostname RTR-NEIGH-2
!
interface GigabitEthernet0/0
 description Link-A to R1
 ip address 10.0.12.2 255.255.255.0       
 ip ospf 1 area 0
!
interface GigabitEthernet0/1
 description Link-B to R3
 ip address 10.0.23.2 255.255.255.252     
 ip ospf 1 area 1                         
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
router ospf 10
 router-id 2.2.2.2
end
```

### R3 — `RTR-NEIGH-3`

```plaintext
version 15.4
hostname RTR-NEIGH-3
!
interface GigabitEthernet0/1
 description Link-B to R2
 ip address 10.0.23.3 255.255.255.252     
 ip ospf 1 area 0                          
!
interface GigabitEthernet0/2
 description Link-C to R1
 ip address 10.0.31.3 255.255.255.252     
 ip ospf 1 area 0
 ip ospf hello-interval 5                 
 ip ospf dead-interval 20
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
router ospf 100
 router-id 3.3.3.3
end
```

---

## Your Task (exactly three issues)
Identify the three issues, then provide the **minimal** fix and a **single-line verify** for each. Keep the style identical to the template above.

---

## How to work (suggested)
1. Skim in baseline order: **Link-A → Link-B → Link-C**.  
2. Write **three** precise findings.  
3. Add the **fewest** IOS commands to fix them.  
4. For each issue, write **one** verify command and the “good” you expect.  
5. Save and submit your text file in Brightspace.

---

### Why this matters
Reading configs to troubleshoot OSPF neighbour formation builds speed in spotting policy gaps, choosing **minimal** fixes, and proving the result with focused verification.

