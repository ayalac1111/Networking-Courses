---
title: W05 — OSPF Tuning & Troubleshooting (Interface‑Based)
course: CST8371
term: 25F
week: 5
activity: lab
points: 10
due: 2025-10-03T15:30:00
submit_to: TFTP
filename: w05-ospf-{username}.txt
co:
  - CO1
  - CO2
  - CO3
  - CO4
aliases:
  - W05 OSPF Tuning
tags:
  - course/cst8371
  - term/25F
  - week/w05
  - activity/lab
  - topic/ospf
  - topic/single-area
  - device/cisco-ios
  - skill/verify
---
# W05 — OSPF Tuning & Troubleshooting (builds on W04)

> Continue with the exact **W04 topology**.

---
## Start Here — Week 05
**Window (ET):** Friday, Sep 26th @ 4:30 → **Friday, Oct 3rd @ 3:30 PM** — No late submissions.

- **What to submit (10 pts):** `w05-ospf-{username}.txt`
- **Where:** *TFTP* (ops‑only proof lines)
- **Rules:** exact filename; **no** `show run`; include device **prompt + command** lines only.
- **Estimated time:** **2h30m** (builds on W04).

### Mini Quick‑Ref (W05)
- **Network type:** `show ip ospf interface g0/0/x` • set: `ip ospf network point-to-point` *(or remove to use broadcast)*
- **Cost & ref‑bw:** `ip ospf cost <n>` • `auto-cost reference-bandwidth <Mbps>` • `show ip ospf interface | i Cost`
- **DR/BDR (broadcast only):** `ip ospf priority <0–255>` • `show ip ospf neighbor`
- **Debug (short bursts):** `debug ip ospf adj` • `debug ip ospf hello` • stop: `undebug all`
- **Route view:** `show ip route | i E2` • `show ip cef 0.0.0.0/0`

---
## Objectives (CO map)
- **CO1 — Network types & DR control:** Compare **point‑to‑point** vs **broadcast** on one inter‑router link; perform and verify a **DR/BDR** election.
- **CO2 — Path selection tuning:** Use **reference‑bandwidth** and **per‑interface cost** to demonstrate **ECMP** and preferred path selection.
- **CO3 — Troubleshooting signals:** Use **Hello/Dead** awareness and a **short debug burst** to observe **adjacency bring‑up** states.
- **CO4 — Evidence & uniqueness:** Provide a clean ops‑only bundle and include **`show tcp brief` from CORE** to mark your submission.

> **CO map:** CO1 → Task 1 • CO2 → Task 2 • CO3 → Task 3 • CO4 → Task 4

---

## Network Topology

![Lab Topology](img/w04-topology.png)

---

## Tasks

### Task 0 — Baseline from W04
- [ ] Create **`w05-ospf-{username}.txt`**. You will submit this file to the TFTP server in the lab.
- [ ] Confirm neighbours Full on **EDGE–CORE** and **EDGE–DIST**.
- [ ] Confirm **CORE Vlan20** is **Passive** and in **Area 0**.
- [ ] Confirm default `O*E2 0.0.0.0/0` exists on **CORE** and **DIST**.
- [ ] Keep your console on **EDGE** and ssh into **CORE** and **DIST** using their loopback address.
- [ ] Do **not** `clear ip ospf process` except when explicitly asked; do **not** reset OSPF settings until after CO4 collection.

---
### Task 1 — Network Type & DR/BDR (CO1)

**Goal:** See the operational difference between **p2p** and **broadcast** on Ethernet, and control **DR/BDR** when in broadcast.

**Why:** On Ethernet, **broadcast** networks elect **DR/BDR** and use **Hello 10 / Dead 40** by default. On **point-to-point**, there’s no DR/BDR, and adjacencies often reconverge faster (fewer roles, simpler LSAs).

**Steps**
- [ ] On **EDGE–CORE** link, set **broadcast** (default) by **removing** p2p if present.  
- [ ] Influence election using **priority U** on **CORE-Gi0/0/1** (0 = never DR/BDR)
- [ ] Verify neighbour states.   *Expect:* on this segment CORE,  should become the **DR**.  Ensure to clear the OSPF process.
- [ ] On **EDGE–DIST** link, set **broadcast** (default) by **removing** p2p if present.  
- [ ] **EDGE-Gi0/0/2**  should not participate in the election (0 = never DR/BDR). 
- [ ] Verify neighbour states.   *Expect:* on this segment,  **DIST** should become the **DR**; **EDGE** should become a **DROTHER**

<details> 

<summary>Task 1 — Commands (EDGE - CORE)</summary>

```bash
# Remove p2p (broadcast is default) on BOTH ends
EDGE(config)# interface gi0/0/1
EDGE(config-if)# no ip ospf network point-to-point

CORE(config)# interface gi0/0/1
CORE(config-if)# no ip ospf network point-to-point

# Influence DR election on CORE
CORE(config)# interface gi0/0/1
CORE(config-if)# ip ospf priority {U}     # e.g., 250

# Non-preemptive: force a new election (ONE side only)
CORE# clear ip ospf process

```
</details>


#### CO1 — Verification & Collection of Information
```diff
=== CO1 – Network type & DR/BDR control ===
```
```bash
# From EDGE/CORE/DIST
show ip ospf neighbor
```
`!-- Proves broadcast vs p2p behaviour and DR/BDR control;`

---
### Task 2 — Cost, Reference‑Bandwidth & ECMP (CO2)

**Goal:** Show how **cost** and **reference-bandwidth** affect path selection and then make **CORE** use **both paths** (ECMP) to **EDGE Lo1**.

**Steps**
- [ ]  Create **EDGE Lo1**: `10.{U}.1.1/32` and enable OSPF (interface-mode).
- [ ]  From **CORE**, **check the route metric** to `10.{U}.1.1/32` (record the metric).
- [ ]  Set a **domain-wide reference-bandwidth = 10000** on **EDGE, CORE, DIST**.
- [ ]  Run `show ip ospf` on **all three routers** to check the **reference-bandwidth** value.
- [ ]  From **CORE**, **check the route metric again** to `10.{U}.1.1/32` (note the change).
- [ ]  Observe the **other path’s cost** from **DIST** (not installed on CORE yet).
- [ ]  Make **CORE** use **both paths** to `10.{U}.1.1/32` by **changing one interface cost** (tie the totals).
- [ ]  From **CORE**, **check the route again** to confirm **ECMP** (two next-hops).

<details> <summary>Task 2 — Commands (partial)</summary>

Set domain-wide reference-bandwidth (all three routers)

```
R(config)# router ospf U 
R(config-router)# auto-cost reference-bandwidth 10000 
```

See the “other path’s” cost (not installed on CORE) from DIST

```
DIST# show ip route 10.{U}.1.1 
```

Force ECMP on CORE (tie the two totals)

```
CORE(config)# interface g0/0/1         
! CORE→EDGE (adjust if your port differs) 
CORE(config-if)# ip ospf cost N             ! example: makes CORE totals to Lo1 equal  
! choose a N value that equals: 
!  (CORE→EDGE cost) + 1  ==  (CORE→DIST + DIST→EDGE) + 1
```

</details>

#### CO2 — Verification & Collection of Information
```diff
=== CO2 – Cost & ECMP ===
```
```bash
# Reference-bandwidth evidence (all routers)
show ip protocol | inc Reference


# Route to EDGE Lo1 after Reference (CORE + DIST)
show ip route 10.{U}.1.1


# ECMP proof on CORE (two next-hops after you tie totals)
show ip cef 10.{U}.1.1
show ip route 10.{U}.1.1

```
`!-- Shows ECMP (two adjacencies) when costs equal`


---
### Task 3 — Observe Adjacency Bring‑up (CO3)

**Goal:** Trigger a brief adjacency issue by **changing CORE’s OSPF timers** on one link, observe **ExStart/Exchange/Loading**, and then **align timers** to restore **FULL**.

**Steps**
- [ ] On **CORE→EDGE** link, **change Hello/Dead** on CORE only to create a **timer mismatch** (adjacency should drop).
- [ ] Start a very short **debug** to capture bring-up lines (then turn it off).
- [ ] **Verify** neighbour state and interface **Hello/Dead** values on both sides.
- [ ] **Align timers** (set EDGE to match CORE), then confirm the adjacency returns to **FULL**.

<details>

<summary>Task 3 — Commands (timer mismatch → observe → fix)</summary>

Create the mismatch on **CORE** (Ethernet default is Hello 10 / Dead 40)

```bash
CORE(config)# interface g0/0/1           ! CORE ↔ EDGE transit 
CORE(config-if)# ip ospf hello-interval 1 
CORE(config-if)# ip ospf dead-interval 4
```

Brief debug to watch bring-up (keep it short)

```bash
CORE# debug ip ospf adj 
CORE# debug ip ospf hello  
```
# Nudge the link to re-negotiate on CORE only 
```bash
CORE(config)# interface g0/0/1 
CORE(config-if)# shutdown 
CORE(config-if)# no shutdown  
```
# Stop all debug ASAP -- Check you are getting your hello with incorrect timers
```bash
CORE# undebug all
```

Verify the impact (mismatch should prevent/tear down FULL) - in EDGE/CORE

```bash
# show ip ospf neighbor 
# show ip ospf interface g0/0/1 | include Hello|Dead|State  
```

Verify:_ 
CORE shows **Hello 1 / Dead 4**; 
EDGE remains **Hello 10 / Dead 40**; 
adjacency **not FULL** due to **timer mismatch**.

</details>

#### CO3 — Verification & Collection of Information
```diff
=== CO3 – Adjacency bring-up (debug evidence) ===
```
```bash
show ip ospf interface g0/0/1 | include Hello|Dead|State
! paste the last ~5 lines from your debug window showing the mismatch
```
`!-- Proves timer values and observed adjacency states during bring-up.`

---
### Task 4 — Evidence bundle & uniqueness (CO4)

**Goal:** Submit a clean, minimal set of proofs, including a **TCP control block** from CORE.


**Collect (single bundle, per device where relevant)**
```bash
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
show ip protocols
show tcp brief    ! CORE only — include a visible SSH TCB line
```

#### CO4 — Verification & Collection of Information
```diff
=== CO4 – Final evidence & unique TCB ===
```
```bash
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
show ip protocols
show tcp brief    ! CORE only — include a visible SSH TCB line
```
`!-- Minimal bundle showing neighbour health, interface participation, default reachability, and a unique CORE TCB.`

---
## Submission checklist
- [ ] **Filename:** `w05-ospf-{username}.txt` (UTF‑8)
- [ ] CO sections present and in order (CO1–CO4)
- [ ] Submitted to *TFTP* within the window

## Verification 
```bash
ssh -l cisco 192.0.2.69 #-- password cisco
ls -l /var/tftp/*{username}*
```


## Grading rubric (10 pts)
| Criteria (CO)                    |                                              Description |    Pts |
| -------------------------------- | -------------------------------------------------------: | -----: |
| **CO1 — Net types & DR control** | Broadcast vs p2p proven; DR/BDR influenced; p2p restored |      3 |
| **CO2 — Cost & ECMP**            |              Ref‑bw set; ECMP demonstrated and explained |      3 |
| **CO3 — Debug & timers**         |                Bring‑up states observed; timers verified |      2 |
| **CO4 — Evidence & TCB**         |            Clean bundle incl. `show tcp brief` from CORE |      2 |
| **Total**                        |                                                          | **10** |

---
## Cleanup
```bash
erase startup-config
delete vlan.dat
reload
```

