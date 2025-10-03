# W06 Challenge — OSPF Show-Only Forensics 

**Week:** W06  
**Student:** {username}  
**Devices:** EDGE (ID 1), CORE (ID 2), DIST (ID 3)

> **Ops-only rule:** Diagnose with **only** these commands:  
> `show ip protocols` • `show ip ospf neighbor` • `show ip ospf interface brief`  
> Describe fixes in words; do **not** use `show running-config`.

---

## How to answer
For each ticket: (1) **Finding** — paste the *exact line(s)* from the Evidence Pack that prove the issue; (2) **Fix** — describe the *minimal* change(s) you would make; (3) **Verify** — predict what the *allowed* show commands will look like after the fix (one line each).

Each ticket is independent of the others.

---

## Ticket A — “The Long Way Around”

**Trouble Ticket A:** Traffic to **DIST’s loopback (10.U.100.3)** from **EDGE** prefers to go **via CORE**, even though there's a direct link.

### Evidence Pack
```
EDGE# show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/0/1      U     0               10.U.12.1/29       1     DROTH 1/1
Gi0/0/2      U     0               10.U.13.1/29       10    DR    1/1

EDGE# show ip route ospf | include 10.U.100.3
O    10.U.100.3 [110/11] via 10.U.12.2, 00:00:14, GigabitEthernet0/0/1

CORE# show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/0/1      U     0               10.U.12.2/29       10    DR    1/1
Gi0/0/2      U     0               10.U.23.2/29       10    DROTH 1/1

```

> **Finding focus:** Check interface costs and routing decisions. Even though **EDGE–DIST** is directly connected via `Gi0/0/2`, the route to **DIST's loopback** is being learned **via CORE**. This usually happens when **interface cost** is higher on the direct path or **OSPF reference bandwidth** is mismatched.

---

## Ticket B — “It Exists on CORE, But DIST Never Learns It”

**Trouble Ticket B:** Hosts in **VLAN200 (10.250.200.0/24)** can ping **CORE**, but routers beyond CORE (such as **DIST**) have no route to that subnet. As a result, pings from **DIST** to any address in 10.250.200.0/24 fail, and `show ip route ospf` on **DIST** shows no entry for the network.

### Evidence Pack
```
CORE# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan200                10.250.200.2    YES manual up                    up
Gi0/0/1                10.250.12.2     YES manual up                    up
Gi0/0/0                10.250.23.2     YES manual up                    up

CORE# show ip ospf interface brief
Interface    PID   Area   IP Address/Mask   Cost  State Nbrs F/C
Gi0/0/1      1     0      10.250.12.2/29   10    DR    1/1
Gi0/0/0      1     0      10.250.23.2/29   10    DROTH 1/1

DIST# show ip route ospf | include 10.250.200.0  
<no output>
```

> **Finding focus:** Compare CORE’s **Layer-3 interfaces** (`show ip interface brief`) with CORE’s **OSPF interface participation**. Then check **DIST’s OSPF routes** for the corresponding subnet to confirm whether the information propagated downstream.

---

## Ticket C — “Weird Neighbour Behaviour”

**Symptom:** The OSPF neighborship between **CORE** and **DIST** never reaches **FULL** state. Instead, both routers show each other stuck in **EXSTART** or **EXCHANGE** with frequent flapping. No routes are exchanged between them, and topology updates appear inconsistent.

### Evidence Pack
```
CORE# show ip protocols | include ID
  Router ID 10.250.100.2

DIST# show ip protocols | include ID
  Router ID 10.250.100.2

CORE# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.250.100.1      1   FULL/DR         00:00:36    10.250.12.1     Gi0/0/1
10.250.100.2      1   EXSTART/BDR     00:00:39    10.250.23.3     Gi0/0/0

DIST# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.250.100.1      1   FULL/DR         00:00:35    10.250.13.1     Gi0/0/0
10.250.100.2      1   EXSTART/DR      00:00:38    10.250.23.2     Gi0/0/2
```

>**Finding focus**: Read each router’s OSPF Router ID from the protocols summary and compare it to the neighbour state seen on the shared link in the `show ip ospf neighbour` command. Use the combination of IDs and state progression (**EXSTART/EXCHANGE/FULL**) to infer whether the two devices **agree** at the control plane.
>**Be aware**: Some OSPF changes require the `clear ip ospf` command to take effect.

---

## Submission Format (plain text)
**Filename:** `w06-challenge-{username}.txt`

```plaintext
A.finding=<explain why CORE was silent>
A.fix=<minimal config you would apply>
A.verify=<CORE#show ip protocols | include Passive --> Expected: >

B.finding=<explain why DIST never learns the route>
B.fix=<minimal config you would apply>
B.verify=<DIST# show ip route ospf | include 10.250.200.0 --> Expected:>

C.finding=<explain why neigh state CORE and DIST>
C.fix=<minimal config you would apply>
C.verify=<DIST# show ip protocols | include ID --> Expected: >
C.verify=<DIST# show ip ospf neighbor | include 10.250.100.2 --> Expected: >
```

---
### Marking (6 pts total)
- **Ticket A (2 pts)** — precise show-only finding (1), minimal fix + show verification (1)  
- **Ticket B (2 pts)** — precise show-only finding (1), minimal fix + show verification (1)  
- **Ticket C (2 pts)** — precise show-only finding (1), minimal fix + show verification (1)


