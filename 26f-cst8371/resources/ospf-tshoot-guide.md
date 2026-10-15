# OSPF Troubleshooting Guide

Work through each section **in order** and stop at the first thing that is wrong. Fix one thing, then re-check.

Contents: **1. Neighbours** · **2. Routes** · **3. Parameters (quick reference)**

---

## 1. Neighbours: no adjacency

Start with `show ip ospf neighbor`. Healthy is `FULL` (`FULL/DR`, `FULL/BDR`, or `FULL/-` on point-to-point).

| State | Meaning | Go to |
|---|---|---|
| neighbour missing | hellos not exchanged or rejected | Steps 1 to 4 |
| `INIT` | I hear them, they do not hear me | Step 2 on the **other** router |
| `2WAY` | normal between two non-DR routers on broadcast; otherwise a priority or network-type problem | `show ip ospf interface` |
| stuck `EXSTART` / `EXCHANGE` | MTU mismatch | compare MTU on both ends |

### Step 1: Is OSPF running on the interface?

Check this first. If the interface is down, or OSPF is not enabled on it, no hello is ever sent and nothing below can work.

| Check | Command | Healthy |
|---|---|---|
| Interface up and addressed | `show ip interface brief \| exclude unassigned` | `up` / `up` and the right address |
| Link works | `ping <peer address>` | 100 percent (this does **not** prove OSPF will form) |
| OSPF enabled, in which area | `show ip ospf interface brief` | interface is listed, correct Area |

Not listed: enable it with `ip ospf 1 area 0` under the interface (or add a `network` statement under `router ospf 1`).

### Step 2: What must match (the requirements)

Two routers become neighbours only if **all** of these agree on the shared link:

| Requirement | Must be | Seen in |
|---|---|---|
| Subnet and mask | same subnet, same mask (/28 vs /29 fails) | `show ip ospf interface <int>` → `Internet Address` |
| Area | same area | `show ip ospf interface brief` → `Area` |
| Hello / Dead timers | identical | `show ip ospf interface <int>` → `Timer intervals` |
| Network type | compatible (broadcast with broadcast, point-to-point with point-to-point) | `show ip ospf interface <int>` → `Network Type` |
| Router ID | **different** on each router | `show ip ospf` |
| Authentication | same type and key (if used) | `show running-config interface <int>` |
| MTU | same | `show interface <int>` |

**Watch the mismatch live with debug:**

```text
R1# debug ip ospf hello
```

Wait about 10 seconds, then `undebug all`. Read the lines like this:

```text
OSPF-1 HELLO Et0/1: Send hello to 224.0.0.5 area 0 from 10.0.12.1
OSPF-1 HELLO Et0/1: Rcv hello from 10.0.100.2 area 0 10.0.12.2
OSPF-1 HELLO Et0/1: Mismatched hello parameters from 10.0.12.2
OSPF-1 HELLO Et0/1: Dead R 40 C 40, Hello R 10 C 10, Mask R 255.255.255.240 C 255.255.255.248
```

| Part | What it tells you |
|---|---|
| `Send hello ... from 10.0.12.1` | this interface is enabled and **not passive** |
| `Rcv hello from 10.0.100.2 ... 10.0.12.2` | the **neighbour router ID** (`10.0.100.2`) and the neighbour's interface address (`10.0.12.2`) |
| `Mismatched hello parameters` | the neighbour's hello was **rejected**; the next line says why |
| `R` / `C` | **R** = value **R**eceived from the neighbour, **C** = value **C**onfigured on this router. Any pair that differs is the problem |
| `Mask R ... C ...` | subnet mask mismatch |
| no `Send` lines at all | interface not enabled or passive: back to Step 1 and Step 4 |
| `Send` lines but no `Rcv` | the neighbour is not sending: debug on the other router |

If the lines do not show the cause, use `debug ip ospf adj` as well; area, authentication and network-type problems are reported there.

**Fix what differs.** Change the value on the side that does not match the design (compare both ends):

| Problem | Fix (interface unless noted) |
|---|---|
| Wrong mask or address | `ip address <address> <mask>` |
| Wrong area | `ip ospf 1 area <n>` |
| Hello / Dead differ | `ip ospf hello-interval <s>` and `ip ospf dead-interval <s>`, or `no` the override |
| Network type differs | `ip ospf network {broadcast \| point-to-point}` |
| Authentication differs | match the `ip ospf authentication` and key, or `no` it |
| MTU differs | `ip mtu <bytes>` to match |

### Step 3: Duplicate router IDs

If Steps 1 and 2 show nothing wrong, check the router IDs. Two routers with the same ID will not form a full adjacency, and the console may print:

```text
%OSPF-4-DUP_RTRID_NBR: OSPF detected duplicate router-id 10.0.100.1 from 10.0.12.2 on interface Ethernet0/1
```

Compare the ID on both routers (`show ip ospf` → `Routing Process "ospf 1" with ID ...`; the ID in `show ip ospf neighbor` and in the debug `Rcv hello from` line is the neighbour's). To change it:

```text
R1(config)# router ospf 1
R1(config-router)# router-id 10.0.100.1
R1# clear ip ospf process        ! the new ID only applies after this
```

### Step 4: Is the interface passive?

A passive interface is advertised but sends no hellos, so no neighbour forms on it.

| Check | Command | Look for |
|---|---|---|
| Passive list | `show ip protocols` | `Passive Interface(s):` |
| One interface | `show ip ospf interface <int>` | `No Hellos (Passive interface)` |

Fix: `router ospf 1` → `no passive-interface <int>`. Passive is **correct** on a user LAN that has no router (the design may require it); it is only wrong on a link to another router.

---

## 2. Routes: missing or wrong route

Start with `show ip route <destination>` on the router that should have the route. Neighbours must be `FULL` first (Section 1).

### How to read the route

```text
O    10.0.30.1/32 [110/11] via 10.0.12.2, 00:05:12, Ethernet0/1
                           via 10.0.21.2, 00:05:12, Ethernet0/2
```

| Part | Meaning |
|---|---|
| `O`, `O IA`, `O E2`, `O*E2` | OSPF intra-area, inter-area, external; `*` = candidate default |
| `[110/11]` | **administrative distance** / **metric (cost)**. 110 is OSPF; another distance or a code such as `S` means a different source is winning |
| `via` lines | next hops. Equal-cost paths show several; fewer than the design means one link or adjacency is down |
| `% Network not in table` | no route of any kind: continue below |

### Step 1: What must be true for the route to exist

| Requirement | Check | Command |
|---|---|---|
| Neighbours are `FULL` | state and count | `show ip ospf neighbor` |
| The **origin** router has OSPF on the interface that holds the network | interface listed with its Area | `show ip ospf interface brief` (on the origin) |
| The origin advertises it (the network is not a passive-excluded or missing interface) | the prefix is in the database | `show ip ospf database` |
| No filtering or wrong area on the way | same area on both ends | `show ip ospf interface brief` |
| For a default or external route: the origin injects it | `default-information originate` under OSPF; the static default exists | `show running-config \| section ospf`, `show ip route 0.0.0.0` |
| No better route from another source | AD lower than 110 | `show ip route <destination>` |

### Step 2: Read the OSPF database

`show ip ospf database` shows what is advertised, even when no route was installed.

| Section | Link ID | Adv Router | Use it to |
|---|---|---|---|
| Router Link States | the router's own router ID | the same ID | confirm the router is advertising at all |
| Net Link States | the DR's interface address | the DR's router ID | confirm a broadcast segment has a DR |
| Type-5 AS External | the external prefix (for example `0.0.0.0`) | the router ID that injected it | confirm the default route is originated, and by whom |

| Finding | Meaning |
|---|---|
| Prefix **not** in the database | the origin is not advertising it: fix the origin (interface in OSPF, `default-information originate`) |
| In the database, **not** in the routing table | a better route from another source, a filter, or the next hop is not reachable |
| In the database and the table, **wrong** next hops or metric | check cost and `bandwidth` on the links, and whether one adjacency is down |

### Step 3: Test the path

| Test | Command | Reading |
|---|---|---|
| Forward and return path | `ping <destination> source <interface>` | the source must be a network the far side can route back to |
| Hop by hop | `traceroute <destination>` | where it stops is where the route is missing |
| Host check | on the host: `ping <destination>` and confirm its gateway | a host also needs a default gateway |

If the ping goes out but never returns, check the **return** route (the far router, or the static route on the ISP).

### Fixes

| Problem | Fix |
|---|---|
| Network not advertised | `ip ospf 1 area <n>` on the interface holding it (or a `network` statement) |
| No default route in OSPF | `router ospf 1` → `default-information originate` on the router with the static default |
| Wrong cost / path | `ip ospf cost <n>`, `bandwidth`, or `auto-cost reference-bandwidth` (same on all routers) |
| Static route overriding OSPF | remove it with `no ip route ...` if it is not in the design |
| Change did not take effect | `clear ip ospf process` |

---

## 3. Parameters: quick reference

| Question | Command | Look for |
|---|---|---|
| Router ID, reference bandwidth, areas | `show ip ospf` | `ID`, `Reference bandwidth unit`, area list |
| Which interfaces run OSPF | `show ip ospf interface brief` | Area, Cost, State, `Nbrs F/C` |
| One interface in detail | `show ip ospf interface <int>` | Network Type, Cost, Priority, Hello/Dead, DR/BDR, passive |
| Passive interfaces, networks, distance | `show ip protocols` | `Passive Interface(s)`, routing for networks |
| Neighbours and roles | `show ip ospf neighbor` | neighbour ID, state, role, interface |
| What OSPF is configured | `show running-config \| section ospf` | compare line by line with the design |
| What is advertised | `show ip ospf database` | Router, Net and External LSAs |

## Habits

- Compare **both ends** of every link side by side.
- Make the smallest change, then re-check neighbours and routes before the next one.
- After a repair, confirm nothing else broke: neighbours still `FULL`, routes still show the expected next hops.
- Remove a wrong setting with `no` plus the same command. Do not rebuild the whole OSPF configuration.
- Always `undebug all` when you finish a debug.
