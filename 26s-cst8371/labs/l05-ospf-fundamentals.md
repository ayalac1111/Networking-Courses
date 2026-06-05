# Lab 05 — OSPF Fundamentals

---

## Section A — Start Here

### A1 - Overview

This lab introduces single-area OSPFv2 on a three-router topology. You will form OSPF neighbours using interface-based activation, set stable Router IDs with Loopback100, advertise CORE VLAN20 as a passive interface, and verify learned routes plus default route propagation from EDGE.

The purpose of this lab is not only to make OSPF work. The purpose is to prove, from command output, that the control plane and data plane behave as expected.

- Control-plane proof: OSPF Router IDs, OSPF-enabled interfaces, neighbour state, and route installation.
- Data-plane proof: PC traceroute before and after a transit-link failure.
- Evidence rule: verification commands may be full output while you are checking; collection commands may be filtered so the submitted file stays readable.

### A1.1 — Mini Quick-Ref

| Task | Command / Indicator |
|---|---|
| Enable OSPF on an interface | `interface <int>` → `ip ospf U area 0` |
| Set stable Router ID | `router ospf U` → `router-id 10.U.100.x` |
| Prove identity | `show ip ospf` |
| Prove OSPF interface participation | `show ip ospf interface brief` |
| Prove neighbours | `show ip ospf neighbor` → `FULL` |
| Advertise LAN but block neighbours | `ip ospf U area 0` on VLAN20 + `passive-interface vlan 20` |
| Inject default route | Static default on EDGE + `default-information originate` |
| Prove default learned | CORE/DIST show `O*E2 0.0.0.0/0` |
| Prove reconvergence | PC `tracert -d 203.0.113.254` changes path after CORE-EDGE link failure |

### A2 - Why This Lab Is Important

- OSPF neighbour formation is the foundation for dynamic route learning.
- Router ID, interface participation, and passive interfaces determine what OSPF advertises.
- Evidence must prove both protocol state and user-facing reachability.
- This lab creates the baseline for later OSPF tuning, DR/BDR, cost, timers, and convergence labs.

### A3 - Objectives / Evidence Map

| Objective | Evidence checkpoint |
|---|---|
| Build the addressed baseline with PC on VLAN20 | C00 |
| Configure OSPF Router IDs and advertise Loopback100 | C01 |
| Form FULL OSPF neighbours on transit links | C02 |
| Observe neighbour formation using current state and logs | C03 |
| Advertise CORE VLAN20 as passive | C04 |
| Propagate a default route from EDGE | C05 |
| Prove path change after a transit-link failure | C06 |

---

## Section B — Topology and Addressing

### B1 — Topology

![Lab 05 Topology](../images/l05-topology.png)
>Alpine not used in this lab
### B2 — Addressing Plan

| Device               | Interface         | Network         | IP/Mask                  | Notes                                   |
| -------------------- | ----------------- | --------------- | ------------------------ | --------------------------------------- |
| EDGE                 | Loopback100       | Loopback        | `10.U.100.1/32`          | Router ID and OSPF advertisement        |
| CORE                 | Loopback100       | Loopback        | `10.U.100.2/32`          | Router ID and OSPF advertisement        |
| DIST                 | Loopback100       | Loopback        | `10.U.100.3/32`          | Router ID and OSPF advertisement        |
| EDGE                 | G0/0/0 to REMOTE  | REMOTE-UPLINK   | `203.0.113.U/24`         | Upstream exit; not in OSPF              |
| REMOTE               | Interface to EDGE | REMOTE-UPLINK   | `203.0.113.254/24`       | Upstream next hop                       |
| EDGE                 | G0/0/1 to CORE    | EDGE–CORE       | `10.U.12.1/29`           | Transit, Area 0                         |
| CORE                 | G0/0/1 to EDGE    | EDGE–CORE       | `10.U.12.2/29`           | Transit, Area 0                         |
| EDGE                 | G0/0/2 to DIST    | EDGE–DIST       | `10.U.13.1/29`           | Transit, Area 0                         |
| DIST                 | G0/0/1 to EDGE    | EDGE–DIST       | `10.U.13.3/29`           | Transit, Area 0                         |
| CORE                 | G0/0/2 to DIST    | CORE–DIST       | `10.U.23.2/29`           | Transit, Area 0                         |
| DIST                 | G0/0/2 to CORE    | CORE–DIST       | `10.U.23.3/29`           | Transit, Area 0                         |
| CORE                 | VLAN20            | CORE VLAN20     | `10.U.20.2/24`           | Default gateway; passive OSPF interface |
| PC                   | NIC               | CORE VLAN20     | `10.U.20.20/24`          | Evidence host and user endpoint; gateway `10.U.20.2` |
| TFTP/HTTP/DNS server | Server LAN        | Remote services | `192.0.2.69 / .80 / .53` | Remote services                         |

---

## Section C — Lab Tasks and Evidence

### C0 — Build and Prove the Baseline

#### Action

1. Create `l05-username.txt` on the PC desktop.
2. Apply the addressing table to EDGE, CORE, DIST, and PC.
3. Configure Loopback100 on EDGE, CORE, and DIST.
4. Configure CORE VLAN20 and the PC access port.
5. Shut unused CORE access ports and move them to VLAN666.
6. Configure SSH on EDGE, CORE, and DIST.
7. SSH from PC to CORE on the `10.U.20.2` address.

Baseline service requirements:

```text
NTP:       Synchronize lab devices to EDGE.  EDGE should be set to master 3.
Syslog:    Send logging messages to PC `10.U.20.20`.
PC tool:   Run Tftpd64 Syslog service during the lab.
```

SSH requirement for routers and switches:

```
Username: admin
Password: cisco
Privilege level: 15
Domain name: cst8371.net
SSH version: 2
```

CORE VLAN requirements:

```text
VLAN 20 name:  <username>-VLAN20
VLAN 666 name: <username>-VLAN666
No active access port remains in VLAN 1.
PC-facing port is an access port in VLAN20.
```

#### Verification

```plaintext
From all routers/swtich:
show ip interface brief | exclude unassigned|down
show ip ssh

CORE:
show vlan brief
show tcp brief

PC:
ping 10.U.20.2
ssh -l admin 10.U.20.2
```

#### Success Indicator / Failure Signal

| Evidence                     | Success Indicator                                        | Failure Signal                                              |
| ---------------------------- | -------------------------------------------------------- | ----------------------------------------------------------- |
| `show ip interface brief`    | Required physical interfaces and Loopback100 are `up/up` | Interface administratively down, protocol down, or wrong IP |
| `show vlan brief`            | VLAN20 and VLAN666 exist; PC port is in VLAN20           | Host port in VLAN1 or required VLAN is missing              |
| `show ip ssh`                | SSH version 2 enabled                                    | SSH disabled or version 1                                   |
| `PC> ping 10.U.20.2`         | PC reaches CORE VLAN20 gateway                           | Timeout or unreachable                                      |
| `PC> ssh -l admin 10.U.20.2` | PC opens SSH access to CORE                              | Connection refused or timeout                               |

#### C00 — Collection of Information

In `l05-<username>.txt`, create:

```text
=== C00 – Baseline interface, VLAN, host, and SSH state ===
```

Include the command lines and outputs.

Use restricted commands in the submitted evidence file so the file stays readable:

```plaintext
From all routers/switches:
show ip interface brief | exclude unassigned|down
show ip ssh

CORE:
show vlan brief
show tcp brief

PC:
ping 10.U.20.2
```

Add this comment line:

```text
!-- Proves base addressing, VLAN20, PC access, and SSH are stable before OSPF is enabled.
```

---

### C1 — Establish OSPF Identity

#### Goal

Give each router a stable OSPF identity before any neighbour relationships are formed.

#### Why this matters

OSPF uses the Router ID to identify routers inside the link-state database. If the Router ID is unstable or wrong, later neighbour and route evidence becomes harder to interpret. Loopback100 provides a predictable /32 identity for EDGE, CORE, and DIST.

#### Expected behaviour

After this task, each router runs OSPF process `U` and advertises only Loopback100 into Area 0. No transit neighbours are expected yet.

#### Action

Stable OSPF identity comes before neighbour formation.

Configure EDGE using the command sequence below.

EDGE:

```plaintext
!- Enable OSPF and set the router's identity
router ospf U
router-id 10.U.100.1

!- Enable the loopback100 in OSPF
interface loopback100
ip ospf U area 0
```

Configure CORE and DIST with the same required design values from Section B2:

| Device | OSPF process | Router ID | Loopback100 OSPF area |
|---|---:|---|---:|
| CORE | `U` | `10.U.100.2` | `0` |
| DIST | `U` | `10.U.100.3` | `0` |

#### Verification

From all devices:

```plaintext
show ip ospf
show ip ospf interface loopback100
```

#### Success Indicator / Failure Signal

| Evidence | Success Indicator | Failure Signal |
|---|---|---|
| `show ip ospf` | Process `U`; Router ID equals `10.U.100.1`, `.2`, `.3` | Wrong process ID, wrong Router ID, or missing OSPF process |
| `show ip ospf interface loopback100` | `Process ID U`, `Area 0` | Loopback not listed, wrong area, or wrong process |

#### Checkpoint

| Do | Verify | Expect |
|---|---|---|
| Set OSPF process `U`, Router ID, and Loopback100 Area 0 | `show ip ospf`; `show ip ospf interface loopback100` | Correct Router ID on each router; Loopback100 in Area 0; no transit neighbours required yet |

#### C01 — Collection of Information

In the evidence file, include the command lines and the outputs.

```text
=== C01 – Router IDs and loopback Area 0 ===
```

Use restricted commands in the submitted evidence file so the file stays readable:

```plaintext
From all devices:

show ip ospf | include Routing Process|Router ID
show ip ospf interface loopback100 | include Process ID|Area
```

Add this comment line:

```text
!-- Proves each router has stable OSPF identity and Loopback100 participates in single-area OSPFv2 Area 0.
```

---

### C2 — Form Transit Neighbours

#### Goal

Enable OSPF only on router-to-router transit links so EDGE, CORE, and DIST form adjacencies.

#### Why this matters

Transit links are where routers exchange OSPF hello packets, form neighbours, and share link-state information. EDGE G0/0/0 faces the upstream REMOTE network and must not participate in the student OSPF domain.

#### Expected behaviour

Each router should see two OSPF neighbours in `FULL` state. EDGE G0/0/0 should not appear in the OSPF interface list.

#### Action

Enable OSPF Area 0 only on inter-router transit links.

EDGE:

```plaintext
interface g0/0/1
ip ospf U area 0
ip ospf network point-to-point

interface g0/0/2
ip ospf U area 0
ip ospf network point-to-point
```

Enable g0/0/1 and g0/0/2 on CORE and DIST as well.

Do not enable OSPF on EDGE G0/0/0.

#### Verification

From all devices:

```plaintext
show ip ospf neighbor
show ip ospf interface brief
```

#### Success Indicator / Failure Signal

| Evidence | Success Indicator | Failure Signal |
|---|---|---|
| `show ip ospf neighbor` | Each router shows two neighbours in `FULL` state | `INIT`, `2WAY`, `EXSTART`, no neighbour, or wrong neighbour ID |
| `show ip ospf interface brief` | Transit interfaces listed in Area 0 | Transit interface absent, wrong area, or wrong process |
| EDGE interface list | EDGE G0/0/0 absent from OSPF interface list | EDGE G0/0/0 appears in OSPF |

#### Checkpoint

| Do | Verify | Expect |
|---|---|---|
| Enable OSPF on transit links only | `show ip ospf neighbor`; `show ip ospf interface brief` | Two `FULL` neighbours per router; EDGE G0/0/0 absent from OSPF |

#### C02 — Collection of Information

```text
=== C02 – Transit neighbour formation ===
```

Include the command lines and outputs.

Use these commands in the evidence file:

```plaintext
From all devices:

show ip ospf neighbor
show ip ospf interface brief

From EDGE:
show ip ospf interface gi0/0/0
```

Add this comment line:

```text
!-- Proves OSPF neighbours form only on intended transit links and EDGE G0/0/0 is excluded from OSPF.
```

---

### C3 — Observe Neighbour Formation

#### Goal

Record both the current OSPF neighbour state and any available log entries that show the neighbour formation event.

#### Why this matters

The neighbour table proves the current state. The log supports the timeline of how the adjacency formed. Logs are supporting evidence only; they do not replace current neighbour proof.

#### Expected behaviour

Neighbour tables show `FULL`. Log output may show OSPF adjacency-change events. If the log buffer does not contain OSPF entries, recreate the event by shutting and re-enabling the relevant transit interface, then collect all neighbours again.

#### Action

Capture protocol-state evidence after neighbours have formed.

> **Operational note**
> If OSPF log entries are not visible, shut the relevant transit interface and bring it back up again. Then collect neighbour evidence from EDGE, CORE, and DIST so all neighbours are represented.

#### Verification


```plaintext
From all devices
show ip ospf neighbor
show logging | include OSPF|ADJCHG
```

#### Success Indicator / Failure Signal

| Evidence | Success Indicator | Failure Signal |
|---|---|---|
| `show ip ospf neighbor` | Neighbours are visible in `FULL` state | Missing neighbour or adjacency not `FULL` |
| `show logging \| include OSPF\|ADJCHG` | Log buffer shows OSPF neighbour events, if still present | No log entries because buffer rotated or logging disabled |
| Evidence boundary | Neighbour table proves current state; logs support that formation occurred | Student relies only on logs without current neighbour proof |

#### Checkpoint

| Do | Verify | Expect |
|---|---|---|
| Capture neighbour state and log support | `show ip ospf neighbor`; `show logging \| include OSPF\|ADJCHG` | Current neighbours are `FULL`; logs show adjacency events if still present |

#### C03 — Collection of Information

```text
=== C03 – OSPF neighbour observation evidence ===
```

Include the command lines and outputs.

Use these commands in the evidence file:

```plaintext
From all devices:
show logging | include OSPF|ADJCHG
```

Add this comment line:

```text
!-- Proves current OSPF neighbour state and records available log evidence of adjacency formation.
```

---

### C4 — Advertise the User LAN Without Neighbours

#### Goal

Advertise the PC LAN `10.U.20.0/24` into OSPF while preventing OSPF neighbour formation on the user-facing VLAN.

#### Why this matters

End-user networks should be reachable through OSPF, but end hosts do not run OSPF. The passive-interface pattern advertises the network while suppressing OSPF hello packets on VLAN20.

#### Expected behaviour

CORE VLAN20 appears in OSPF Area 0 and is passive. EDGE and DIST learn `10.U.20.0/24` through OSPF.

#### Action

Advertise the CORE user LAN into OSPF Area 0 without allowing neighbour formation on VLAN20.

CORE:

```plaintext
!-- Enable the interface in OSPF
interface vlan 20
ip ospf U area 0

!-- Set the interface as passive
router ospf U
passive-interface vlan 20
```

End hosts use the network. End hosts do not participate in OSPF.

#### Verification



```plaintext
From CORE:
show ip ospf interface vlan 20
show ip ospf interface brief

From EDGE and DIST:
show ip route ospf | begin Gateway
```

#### Success Indicator / Failure Signal

| Evidence | Success Indicator | Failure Signal |
|---|---|---|
| `show ip ospf interface vlan 20` | `Process ID U`, `Area 0`, and `Passive` | VLAN20 not OSPF-enabled or not passive |
| `show ip ospf interface brief` | VLAN20 listed on CORE; no neighbour forms on VLAN20 | VLAN20 missing or neighbour-capable |
| EDGE/DIST route evidence | Route to `10.U.20.0/24` learned through OSPF | User LAN missing from OSPF route table |

#### Checkpoint

| Do | Verify | Expect |
|---|---|---|
| Add CORE VLAN20 to OSPF and make it passive | `show ip ospf interface vlan 20`; `show ip route ospf` | VLAN20 is passive; EDGE/DIST learn `10.U.20.0/24` |

#### C04 — Collection of Information

```text
=== C04 – Passive user LAN advertisement ===
```

Include the command lines and outputs.

```plaintext
From CORE:
show ip ospf interface vlan 20
show ip ospf interface brief

From EDGE and DIST:
show ip route ospf | begin Gateway
```

Add this comment line:

```text
!-- Proves CORE VLAN20 is advertised into Area 0 while remaining passive and non-neighbour-forming.
```

---

### C5 — Preview Default Route Propagation

#### Goal

Make EDGE the exit point for unknown destinations and advertise that default path to CORE and DIST.

#### Why this matters

OSPF does not automatically advertise a default route. EDGE must first have a valid static default route, then explicitly originate it into OSPF.

#### Expected behaviour

EDGE shows a static default route pointing to `203.0.113.254` through G0/0/0. CORE and DIST learn `O*E2 0.0.0.0/0`. PC traceroute exits the VLAN20 LAN toward CORE and then the OSPF domain.

#### Action

On EDGE, configure a fully specified static default route using the upstream interface and next hop. Then originate that default into OSPF.

EDGE:

```plaintext
!-- set a default route
ip route 0.0.0.0 0.0.0.0 g0/0/0 203.0.113.254

!-- Distribute the route in OSPF
router ospf U
default-information originate
```

Use PC for the user path proof.

#### Verification


```plaintext
From all devices:
show ip route 0.0.0.0

PC> tracert -d 203.0.113.254
```

#### Success Indicator / Failure Signal

| Evidence | Success Indicator | Failure Signal |
|---|---|---|
| EDGE `show ip route 0.0.0.0` | Static default points to `203.0.113.254` and `G0/0/0` | Default missing, next hop missing, or exit interface missing |
| CORE/DIST route output | `O*E2 0.0.0.0/0` appears | No default route or default not learned through OSPF |
| PC traceroute | Path exits PC LAN through CORE toward EDGE/REMOTE | Timeout before CORE or path does not leave CORE LAN |

#### Checkpoint

| Do | Verify | Expect |
|---|---|---|
| Configure EDGE static default and originate it into OSPF | `show ip route 0.0.0.0`; PC `tracert` | EDGE has static default; CORE/DIST learn `O*E2`; PC path exits toward REMOTE |

#### C05 — Collection of Information

```text
=== C05 – Preview default route propagation ===
```

Include the command lines and outputs.

Use restricted commands in the submitted evidence file so the file stays readable:

```plaintext
From EDGE:
show ip route 0.0.0.0

From CORE / DIST:
show ip route | include O\*E2|0.0.0.0

PC> tracert -d 203.0.113.254
```

Add this comment line:

```text
!-- Proves EDGE has a static default and CORE/DIST learn the default route from OSPF.
```

---

### C6 — Preview Path Change After Failure

#### Goal

Prove that OSPF maintains forwarding after a transit-link failure by moving traffic from the direct CORE→EDGE path to the alternate CORE→DIST→EDGE path.

#### Why this matters

A routing protocol is useful only if the data path adapts when the topology changes. This task connects OSPF neighbour state, the routing table, and PC traceroute output.

#### Expected behaviour

Before the failure, PC traffic uses CORE→EDGE. During the failure, CORE loses the direct EDGE neighbour but keeps the default route through the remaining OSPF path. After restoration, all expected neighbours return to `FULL`.

#### Action

Capture the baseline path, shut the EDGE-to-CORE transit interface, observe the alternate path, then restore the link.

1. From PC, capture the baseline path:

```plaintext
PC> tracert -d 203.0.113.254
```

2. On EDGE, shut the interface g0/0/1 to CORE:

3. Wait until CORE no longer lists EDGE as a neighbour on the CORE-to-EDGE link.

4. Capture failure-state evidence:

```plaintext
CORE:
show ip ospf neighbor
show ip route 0.0.0.0

PC> tracert -d 203.0.113.254
```

5. Restore the link before leaving the lab:

6. Confirm restoration:

```plaintext
From all devices:
show ip ospf neighbor
```

#### Verification

You need to submit the test, 

```plaintext
PC> tracert -d 203.0.113.254

!-- EDGE - Shutdown g0/0/1

CORE# show ip ospf neighbor
CORE# show ip route 0.0.0.0
PC> tracert -d 203.0.113.254

!-- EDGE - No shutdown g0/0/1

From all devices:
show ip ospf neighbor
```

#### Success Indicator / Failure Signal

| Evidence | Success Indicator | Failure Signal |
|---|---|---|
| Baseline traceroute | Path uses PC → CORE → EDGE/REMOTE | No path or incorrect first hop |
| CORE neighbour output during failure | CORE-to-EDGE adjacency absent; CORE-to-DIST remains `FULL` | All adjacencies lost or wrong link failed |
| CORE default route during failure | Default route remains present through OSPF | Default route removed with no alternate path |
| Post-failure traceroute | Path uses PC → CORE → DIST → EDGE/REMOTE | Traffic stops at CORE or no alternate path |
| Restoration neighbour output | EDGE, CORE, and DIST return to expected `FULL` neighbour state | Link left down or adjacency does not reform |

#### Checkpoint

| Do | Verify | Expect |
|---|---|---|
| Shut EDGE G0/0/1, test path, restore link | CORE neighbour table, CORE default route, PC traceroute | CORE loses direct EDGE adjacency only; default remains; path moves through DIST; neighbours return to `FULL` after restore |

#### C06 — Collection of Information

```text
=== C06 – Preview path change and restoration ===
```

Include the command lines and outputs.

Use these commands in the evidence file:

```plaintext
PC> tracert -d 203.0.113.254

!-- EDGE - Shutdown g0/0/1

PC> tracert -d 203.0.113.254

!-- EDGE - No shutdown g0/0/1
```

Add this comment line:

```text
!-- Proves OSPF can preserve forwarding through the alternate CORE-DIST-EDGE path after the CORE-EDGE link fails, then return to the restored topology.
```


## Section D — Submission and Cleanup

### D1 — Submission File

Submit one evidence file to the course TFTP server.

| File | Required Content | Source |
|---|---|---|
| `l05-<username>.txt` | C00–C06 evidence collected during the lab | Student-created evidence file on PC |

Your evidence file must contain:

```text
=== C00 – Baseline interface, VLAN, host, and SSH state ===
=== C01 – Router IDs and loopback Area 0 ===
=== C02 – Transit neighbour formation ===
=== C03 – OSPF neighbour observation evidence ===
=== C04 – Passive user LAN advertisement ===
=== C05 – Preview default route propagation ===
=== C06 – Preview path change and restoration ===
```

Each checkpoint must include:

```text
Device prompt + command
Raw command output
One comment line beginning with !--
```

---

### D2 — Submit from PC

Submit the file by TFTP from PC:

```plaintext
PC> tftp 192.0.2.69 PUT l05-<username>.txt
```

Your submission is your responsibility.

Before leaving the lab, prove:

```text
1. TFTP transfer completed.
2. File name is l05-<username>.txt.
3. File contains C00–C06 section labels.
4. File has non-zero size.
```

---

### D3 — Cleanup

After submission is confirmed, clean up routers using the provided TCL script.

On all devices:

```
tclsh clean.tcl
```

Now you can turn off your router.

---

## End of Lab 05 — OSPF Fundamentals