# W05 Challenge — OSPF Tuning (Troubleshoot Tickets)

**Week:** W05  
**Student:** {username}  
**Topology U:** 250  (e.g., 10.250.0.0/16 addressing as per lab diagram)  
**Devices:** EDGE (ID 1), CORE (ID 2), DIST (ID 3), REMOTE (203.0.113.0/24)  
**Ops-only policy:** Submit *operational commands + outputs* only. **Do not** use `show running-config`.  
**Reminder:** Use **interface-mode OSPF** (`ip ospf 1 area 0` under transit interfaces). No `network` statements.

---

## Target Baseline (for both tickets)
1) Single-area OSPF (**area 0**) with adjacencies on all campus links.  
2) Transit subnets:  
   - EDGE–CORE: `10.250.12.0/29`  
   - CORE–DIST: `10.250.23.0/29`  
   - EDGE–DIST: `10.250.13.0/29`  
3) Loopbacks:  
   - EDGE `Lo100 10.250.100.1/32`, CORE `Lo100 10.250.100.2/32`, DIST `Lo100 10.250.100.3/32`.  
4) Default path from campus toward REMOTE via **EDGE → 203.0.113.254**.

---

## Ticket A — “Same Prefix, Different Math” (Route to 10.250.20.0/24)

**Ticket:**  
NOC logged that the OSPF route to **10.250.20.0/24** does not agree between routers.
- On **EDGE**, `show ip route 10.250.20.0` shows learned via **OSPF** with an eleven metric value.
- On **DIST**, the **same prefix** is also learned via **OSPF** but the **metric is different**

>**NOC** - Network Operations Center

Interfaces on both paths are meant to be the **same speed**, but the **costs** don’t line up. Your task is to find out why the cost math differs and fix it minimally.

**Your job:**  
- Diagnose why OSPF **costs** don’t line up across routers that have the **same physical link speeds**.  
- Make a **minimal** change that restores **consistent costing** campus-wide.  
- Prove the change by showing a clear **after** cost summary and one **route metric** delta.

**Constraints:**  
- Don’t change interface speeds/encapsulation.  
- Keep changes scoped to **OSPF behaviour**, not IP addressing.

**Suggested ops proof lines (from EDGE/DIST):**
```bash
# You NEED to submit the outputs of these commands
show ip route | include 10.250.20.0
```

**Success (must pass all):**
- A short **before/after** text note explaining what changed in the **metric** for **10.250.20.0/24** on at least one router (based on the `show ip route | include 10.250.20.0` line).
- The route to **10.250.20.0/24** shows a **metric change** on at least one router after your fix.
- Costs are now **consistent** across EDGE, CORE, DIST for identical links (explain briefly using your route metric comparison).

---

## Ticket B — “User Can’t Reach TFTP”

**Symptom report:**  
*A user on PC `10.250.20.20/24` says they cannot reach the **TFTP server 192.0.2.69**.*

**Your job:**  
- Trace and fix the **control-plane** cause preventing the campus from using a valid **default path** toward REMOTE.  
- Apply the **fewest** changes in the **right place**.  
- Prove both control-plane (route present) and data-plane (path works).

**Ops proof lines:**
```bash
# You NEED to submit the output of these commands
EDGE # show ip route | include Gateway
CORE # show ip route 0.0.0.0

# Connectivity Test
CORE # tracert 192.0.2.69
```

**Success (must pass all):**
- A one-line **diagnosis** explaining why the campus did **not** have a usable default via EDGE.  
- Exactly **one** minimal fix path chosen and justified (stick to OSPF-correct behaviour).  
- `O*E2 0.0.0.0/0` visible downstream **and** first hop toward REMOTE = **EDGE**; PC can ping `192.0.2.69`.

---

## Submission Format (plain text)

**Filename:** `w05-challenge-{username}.txt`

Use these exact keys (one line each). Keep IOS commands minimal and focused.

```plaintext
A.finding=<why EDGE and DIST show different OSPF metrics for 10.250.20.0/24>
A.fix=<the exact commands you applied>

A.verify1=EDGE# show ip route | include 10.250.20.0  -> should show metric
A.verify2=DIST# show ip route | include 10.250.20.0  -> should show metric

B.finding=<why campus wasn’t reaching REMOTE (default path issue)>
B.fix=<the exact commands you applied>

B.verify1=EDGE# show ip route | include Gateway  -> prove default in RIB
B.verify1=CORE# show ip route 0.0.0.0.           -> prove learned default

ConnectivityTest=PC> tracert 192.0.2.69  ; via CORE - EDGE - reach OK
```
---

**Marking (5 pts total)**
- **Ticket A (2 pts)**
    - Clear finding + minimal correct fix — 1
    - EDGE and CORE `show ip route | include 10.250.20.0` show **metric change** - 1
- **Ticket B (2 pts)**
    - Clear finding + minimal correct fix — 1
    - Control-plane proofs: **EDGE** default in RIB **and** **CORE** `O*E2 0.0.0.0/0` present — 1
- **Connectivity Test (1 pt)**
    - `PC> tracert 192.0.2.69`  via CORE -> EDGE (reachability to server) — 1

---

**Reminder:** Keep evidence lean (only the lines that prove your claim). No screenshots, no `show run`.

