---
title: "W08 — OSPF Cloud (Cisco + MikroTik)"
course: CST8371
term: 25F
week: 08
activity: lab
points: 10
due: {YYYY-MM-DD HH:MM-0400}
submit_to: TFTP
filename: w08-ospf-cloud-{username}.txt
co: [CO1, CO2, CO3, CO4]
aliases: ["W08 OSPF Cloud"]
tags:
  - course/cst8371
  - term/25F
  - week/w08
  - activity/lab
  - topic/ospf
  - topic/multivendor
  - device/cisco-ios
  - device/mikrotik-routeros
  - skill/verify
---
# W06 — Hybrid OSPF - Cisco + MikroTik (Cloud)

> **Builds on W04/W05.** Keep the **EDGE / CORE / DIST** triangle and upstream REMOTE. This week, you add **two MikroTik routers (T1, T2)** and form **multi‑vendor adjacencies** on a shared Ethernet segment.

---
## Topology changes from W04/W05
- **Add a broadcast segment** (VMware bridged network) connecting **EDGE g0/0/3**, **T1 ether1**, **T2 ether1**. Use **10.U.10.0/24**.
- **Keep triangle links** (EDGE–CORE 10.U.12.0/29, EDGE–DIST 10.U.13.0/29, CORE–DIST 10.U.23.0/29) and **CORE Vlan20** 10.U.20.0/24 **unchanged**.
- **Upstream** stays **EDGE g0/0/0 → 203.0.113.U/24 → REMOTE** (default originates here).
- **Optional internal T1↔T2 link** (host‑only): **10.U.22.0/24** between **T1 ether2** and **T2 ether2** to demonstrate an alternate path.
- **Loopbacks for IDs:**
  - Cisco: Lo100 already **10.U.100.1/.2/.3** on EDGE/CORE/DIST.
  - MikroTik: create **Loop22** (dummy/lo) on T1/T2 as **10.U.100.11/32** and **10.U.100.12/32** and use them as Router‑IDs.

**Updated diagram placeholder**  
![Topology (Cisco + MikroTik)](sandbox:/mnt/data/w04-topology.png)

> **[IMAGE]:** Please replace with a diagram showing the new shared LAN **10.U.10.0/24** between **EDGE g0/0/3**, **T1 ether1**, **T2 ether1**, plus the optional **10.U.22.0/24** link between **T1 ether2** and **T2 ether2**.

---
## Start Here — Week 06
**Window (ET):** {Start Day, Mon DD @ hh:mm} → **{Due Day, Mon DD @ hh:mm}** — {late policy}

- **What to submit (10 pts):** `w08-ospf-cloud-{username}.txt`
- **Where:** *TFTP* (ops‑only proof lines)
- **Rules:** exact filename; **no** running configs; include device **prompt + command** lines; Cisco IOS + RouterOS evidence.
- **Estimated time:** **2h30m**.

### Mini Quick‑Ref (Cisco + MikroTik)
- **Cisco enable:** `int g0/x; ip ospf U area 0` • passive pattern `router ospf U` → `passive-interface default` → `no passive-interface g0/x`
- **Cisco verify:** `show ip ospf neighbor` • `show ip ospf interface g0/x` • `show ip route | i ^O|^O*E[12]`
- **MikroTik enable:** `/routing ospf instance set [find] router-id=10.U.100.11` • `/routing ospf area add name=backbone area-id=0.0.0.0` • `/routing ospf interface-template add interfaces=ether1 area=backbone`
- **MikroTik verify:** `/routing ospf neighbor print` • `/routing ospf database print` • `/ip route print where dst-address=0.0.0.0/0`
- **Debug (short):** Cisco `debug ip ospf adj` • MikroTik `/log print where message~"OSPF"`

---
## Objectives (CO map)
- **CO1 — Multi‑vendor adjacencies & DR/BDR:** Form OSPF neighbours on the shared LAN and identify **DR/BDR** roles across Cisco & MikroTik.
- **CO2 — Default route across vendors:** Originate default at EDGE and prove T1/T2 learn **O*E2 0.0.0.0/0** (or RouterOS equivalent).
- **CO3 — Exchange & timers:** Capture brief evidence of **DBD/LSR/LSU** bring‑up and confirm **Hello 10/Dead 40** on the shared LAN.
- **CO4 — Reconvergence & uniqueness:** Fail one link and show path change end‑to‑end; include **`show tcp brief` from CORE** or **RouterOS `/ip ssh active print`** for uniqueness.

> **CO map:** CO1 → Task 1 • CO2 → Task 2 • CO3 → Task 3 • CO4 → Task 4

---

## Network Topology

![Lab Topology](img/w06-topology.png)


---
## Addressing additions (new segment)

|          Node | Interface    | Address/Mask  | Notes                         |
| ------------: | :----------- | :------------ | :---------------------------- |
|          EDGE | g0/0/3       | 10.U.10.1/24  | Shared LAN with T1/T2, Area 0 |
| T1 (RouterOS) | ether1       | 10.U.10.11/24 | Shared LAN, Area 0            |
| T2 (RouterOS) | ether1       | 10.U.10.12/24 | Shared LAN, Area 0            |
|            T1 | ether2 (opt) | 10.U.22.1/24  | Host‑only link to T2          |
|            T2 | ether2 (opt) | 10.U.22.2/24  | Host‑only link to T1          |

Loopbacks/IDs (RouterOS):  
`/interface bridge add name=lo` • `/ip address add address=10.U.100.11/32 interface=lo` (T1) • `...100.12/32` (T2) • set instance RID to the lo address.

---
## Tasks

### Task 0 — Baseline checks (from W04/W05)
- [ ] Cisco neighbours Full on transits (p2p); CORE Vlan20 **Passive**; default `O*E2 0.0.0.0/0` on CORE/DIST.

---
### Task 1 — Form multi‑vendor adjacencies & DR/BDR (CO1)
**Goal:** OSPF neighbours/adjacencies on the shared **10.U.10.0/24** segment; identify **DR/BDR**.

**Cisco (EDGE)**
```bash
interface g0/0/3
 ip address 10.U.10.1 255.255.255.0
 ip ospf U area 0
 ! do NOT set p2p here; keep default broadcast for DR election demo
```
**MikroTik (T1 & T2)**
```bash
# set Router-ID to loopback
/routing ospf instance set [find] router-id=10.U.100.11   # T1 (use ...12 on T2)
# create backbone area (if not present)
/routing ospf area add name=backbone area-id=0.0.0.0
# enable OSPF on the shared LAN interface
/routing ospf interface-template add interfaces=ether1 area=backbone
```
**Verify**
```bash
# Cisco
show ip ospf neighbor
show ip ospf interface g0/0/3 | include Network Type|State|Priority
# MikroTik
/routing ospf neighbor print detail
```
*Expect:* one device elected **DR**, another **BDR**; others **DROTHER** with **2‑Way** to each other and **Full** to DR/BDR.

#### Cert Card — Task 1
- **Commands:** Cisco `show ip ospf neighbor` • `show ip ospf interface g0/0/3` • RouterOS `/routing ospf neighbor print detail`
- **Pass if:** Both platforms show the same DR/BDR roles on 10.U.10.0/24; neighbours **Full** with DR/BDR.
- **Pitfall:** RID/priority changes are **non‑preemptive**—flap interface if needed.

#### CO1 — Verification & Collection of Information
```diff
=== CO1 – Multi‑vendor adjacencies & DR/BDR ===
```
```bash
show ip ospf neighbor
show ip ospf interface g0/0/3 | include Network Type|State|Priority
/routing ospf neighbor print detail
```
`!-- Proves cross‑vendor adjacencies and DR/BDR election on the shared LAN.`

---
### Task 2 — Default route propagation to MikroTik (CO2)
**Goal:** T1/T2 learn the default originated by EDGE.

**On EDGE (Cisco)** — already configured in W04, confirm:
```bash
show ip route 0.0.0.0
show run | section router ospf
```
If missing, add:
```bash
ip route 0.0.0.0 0.0.0.0 g0/0/0 203.0.113.254
router ospf U
 default-information originate
```
**Verify downstream**
```bash
# MikroTik
/ip route print where dst-address=0.0.0.0/0
# Cisco CORE/DIST (control)
show ip route | include ^O\*E2 0.0.0.0/0
```
*Expect:* RouterOS shows an **OSPF external** 0/0 (E2 by default).

#### Cert Card — Task 2
- **Commands:** EDGE `show ip route 0.0.0.0` • RouterOS `/ip route print where dst-address=0.0.0.0/0` • CORE/DIST `show ip route | i ^O*E2 0.0.0.0/0`
- **Pass if:** Default exists on EDGE and appears on both T1 and T2 as OSPF external.
- **Pitfall:** Static not fully specified on EDGE → not originated.

#### CO2 — Verification & Collection of Information
```diff
=== CO2 – Default route across vendors ===
```
```bash
show ip route 0.0.0.0              # EDGE
/ip route print where dst-address=0.0.0.0/0    # T1 or T2
```
`!-- Proves default exists at EDGE and is learned by MikroTik via OSPF.`

---
### Task 3 — Exchange & timers across vendors (CO3)
**Goal:** Observe bring‑up on both platforms and confirm timers on the shared LAN.

**Short debug burst**
```bash
# Cisco
debug ip ospf adj
# MikroTik (watch log)
/log print where message~"OSPF"
```
Bounce **EDGE g0/0/3** or **T1 ether1** once, then stop debug (`undebug all`).

**Verify timers & state**
```bash
# Cisco
show ip ospf interface g0/0/3 | include Hello|Dead|State
# MikroTik
/routing ospf interface print detail where interfaces=ether1
```
*Expect:* Hello **10** / Dead **40** on both sides; adjacency returns to **Full** with DR/BDR.

#### Cert Card — Task 3
- **Commands:** Cisco `debug ip ospf adj` → `undebug all` • `show ip ospf interface g0/0/3 | i Hello|Dead|State` • RouterOS `/routing ospf interface print detail`
- **Pass if:** Logs show state changes and timers match the defaults.
- **Pitfall:** Leaving debug running; always stop it.

#### CO3 — Verification & Collection of Information
```diff
=== CO3 – Exchange & timers ===
```
```bash
show ip ospf interface g0/0/3 | include Hello|Dead|State
/routing ospf interface print detail where interfaces=ether1
```
`!-- Confirms bring‑up states and timer parity on both platforms.`

---
### Task 4 — Reconvergence path change (CO4)
**Goal:** Show path change from the **PC (CORE LAN)** to **REMOTE** or to a MikroTik loopback when a link is failed.

**Induce failure**
- Shut **EDGE g0/0/1** (to CORE) **or** disable **T1 ether1** briefly.

**Verify paths**
```bash
# From PC (Linux or Windows)
traceroute -n 203.0.113.U   # or: tracert -d 203.0.113.U

# Optional: to MikroTik loopback
traceroute -n 10.U.100.11
```
**Control‑plane proof**
```bash
# Cisco CORE
show ip route 0.0.0.0
show ip ospf neighbor
show tcp brief
# MikroTik
/routing ospf neighbor print
```
*Expect:* Path reroutes via the remaining devices; default remains installed; CORE includes a unique **TCB** line.

#### Cert Card — Task 4
- **Commands:** `traceroute`/`tracert` from PC • CORE `show ip route 0.0.0.0` • CORE `show tcp brief` • RouterOS `/routing ospf neighbor print`
- **Pass if:** PC path changes during failure; default persists; unique TCB recorded.

#### CO4 — Verification & Collection of Information
```diff
=== CO4 – Reconvergence & uniqueness ===
```
```bash
traceroute -n 203.0.113.U
show ip route 0.0.0.0
show tcp brief
/routing ospf neighbor print
```
`!-- Shows path change, default presence, and includes a unique control‑plane marker from CORE.`

---
## Submission checklist
- [ ] **Filename:** `w08-ospf-cloud-{username}.txt` (UTF‑8)
- [ ] CO sections present and in order (CO1–CO4)
- [ ] Cisco + MikroTik evidence included
- [ ] Cert Cards satisfied for each task

## Grading rubric (10 pts)
| Criteria (CO)                             | Description                                      | Pts |
|------------------------------------------|--------------------------------------------------:|----:|
| **CO1 — Multi‑vendor adjacencies**       | DR/BDR roles consistent across platforms         |  3 |
| **CO2 — Default route across vendors**    | MikroTik learns 0/0 via OSPF external           |  3 |
| **CO3 — Exchange & timers**              | Bring‑up/timers evidenced on both platforms      |  2 |
| **CO4 — Reconvergence & TCB**            | Path change shown; CORE `show tcp brief` present|  1 |
| Professionalism                          | Filename, ordering, clarity                      |  1 |
| **Total**                                |                                                  | **10** |

---
## Cleanup (if needed)
```bash
erase startup-config
reload
```

*RouterOS:* `/system reset-configuration no-defaults=yes` (confirm before running).

