title W05 — Reflect: OSPF Foundations, Identity, Neighbours and Evidence
activity reflect
deliverable reflections/r05-{username}.txt
points 0

# Weekly Reflect — W05: OSPF Foundations, Identity, Neighbours and Evidence

Week: W05  
Date: {date}  
Student: {your_name}  

> Ops-only evidence policy: Use operational commands: `show ip ospf`, `show ip ospf interface brief`, `show ip ospf interface <interface>`, `show ip ospf neighbor`, `show ip route <destination> <mask>`, `ping`, `tracert -d`. Do not use `show running-config` as evidence.

---

## Submission Contract

| Item | Requirement |
|---|---|
| Action | Answer each prompt using evidence from your Lab 05 checkpoint files (C01–C05). |
| Evidence | Include at least one exact command/output proof line for each reflection question. |
| Success Indicator | Your answer names what the evidence showed (identity, neighbour state, route, or reachability) and what it did not show. |
| Failure Signal | Generic answers, missing command output, or claims that treat a neighbour, a route and a working ping as the same proof. |

---

## Always True Rules

### Always Rule 1 — The Router ID identifies the router; the process ID is only local

Rule (one line):  
If two routers use different process IDs, they can still become neighbours; the Router ID must be unique in the domain, and the area must match on the shared link.

In my own words (1–2 sentences):  
{write what each of process ID, Router ID and area ID identifies, and which of the three has to agree between neighbours}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show ip ospf | include Routing Process|Router ID → {process and Router ID line}`
- `{DEVICE}# show ip ospf interface loopback100 | include Loopback100|Area → {Area line}`

If this breaks next week, first move:  
{read the active Router ID from `show ip ospf` before assuming which identity a neighbour table is showing}

---

### Always Rule 2 — Up/up interfaces do not guarantee a neighbour

Rule (one line):  
If Hello and Dead intervals, area, subnet or network type disagree on the shared link, the interface stays up/up and the adjacency still fails.

In my own words (1–2 sentences):  
{write why a surviving alternate path can hide a failed adjacency, and what evidence you needed beyond "the link is up"}

Proof lines (pick two, 1–2 lines each):
- `{EDGE}# show ip ospf interface g0/0/2 | include Timer intervals → {timer line}`
- `{EDGE}# show logging | include ADJCHG|Mismatched|Dead R → {log line}`

If this breaks next week, first move:  
{compare the interface details on both ends of the link before changing anything}

---

### Always Rule 3 — Passive advertises the prefix; not-in-OSPF does neither

Rule (one line):  
A passive interface that is in OSPF advertises its prefix but sends no Hellos and forms no neighbour; an interface excluded from OSPF does not advertise its prefix and forms no neighbour.

In my own words (1–2 sentences):  
{write why a missing neighbour on an interface does not by itself tell you whether it is passive or excluded}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ip ospf interface vlan20 | include Process ID|Area|Passive → {line}`
- `{EDGE}# show ip ospf interface g0/0/0 → {line}`

If this breaks next week, first move:  
{run `show ip ospf interface <interface>` and read the participation and passive lines, not the neighbour table}

---

### Always Rule 4 — A neighbour, a route and a reachability test are three different proofs

Rule (one line):  
A FULL neighbour does not prove a route is installed, and an installed route identifies the selected forwarding path but does not prove the return path or end-to-end delivery.

In my own words (1–2 sentences):  
{write what each of the three kinds of evidence answers, and which one you would still need after the other two}

Proof lines (pick two, 1–2 lines each):
- `{EDGE}# show ip ospf neighbor → {neighbour line}`
- `{CORE}# show ip route ospf → {route line}`
- `{PC}> tracert -d <destination> → {first hop and result}`

If this breaks next week, first move:  
{check each layer in order: neighbour state, then route, then a test from the user's host}

---

## CER Micro-Cards

> What is a CER micro-card? A tiny 3-line check: Claim → Evidence → Reasoning.

### CER 1 — Timer mismatch

Claim:  
{the adjacency failed because of the timer disagreement, while the interface stayed up/up}

Evidence:  
`{DEVICE}# {command} → {paste one exact line from your FAULT state}`

Reasoning:  
{explain why this line points to a Hello/Dead disagreement rather than a cabling or addressing fault}

---

### CER 2 — Passive versus excluded

Claim:  
{VLAN20 was in OSPF and passive, while EDGE G0/0/0 was not in OSPF at all}

Evidence:  
`{DEVICE}# {command} → {paste one exact line for each interface}`

Reasoning:  
{explain how each line separates "advertised, no Hellos" from "not participating"}

---

### CER 3 — Route is not reachability

Claim:  
{the route learned through OSPF identified the forwarding path, and the host test proved delivery}

Evidence:  
`{DEVICE}# show ip route <destination> → {route line}`  
`{PC}> ping or tracert -d <destination> → {result line}`

Reasoning:  
{explain what the route line showed and what only the host test could show}

---

## Ops Lexicon — New This Week

Router ID  
The dotted-decimal identifier of a router in the OSPF domain: the configured `router-id`, otherwise the highest eligible loopback IPv4 address, otherwise the highest eligible active interface IPv4 address.

Process ID  
The number that identifies one OSPF process on one router. It is local and need not match a neighbour's.

Area ID  
The OSPF area an interface belongs to. It must agree on the shared link; this lab uses Area 0 only.

Hello / Dead interval  
The timers for sending Hellos and for declaring a neighbour down. They must match on the shared link.

2-WAY / FULL  
2-WAY means each router sees itself in the other's Hello. FULL means that adjacency has completed database synchronization.

DR / BDR / DROTHER  
On a broadcast segment, the elected Designated and Backup Designated Routers form FULL adjacencies with the others, which are DROTHER. Two DROTHER routers normally stay 2-WAY with each other. Point-to-point links show `FULL/-`.

Passive interface  
An OSPF-enabled interface that advertises its prefix but sends no Hellos and forms no neighbour.

Default-information originate  
The command that advertises an existing default route from EDGE into OSPF.

---

## Reflection Questions

1. IDENTITY: Which command and output showed your active Router ID and process number on one device, and which of the process ID, Router ID and area ID must agree with a neighbour?  
{2–3 sentences + one proof line}

2. NEIGHBOUR STATE: In your neighbour output, what did `FULL/-` and `FULL/DR` or `FULL/BDR` tell you about the type of each link? What would `2-WAY/DROTHER` between two routers on a larger broadcast segment mean?  
{2–3 sentences + one proof line}

3. TIMERS: What evidence showed the link was up while the adjacency was not, and what evidence identified the cause instead of assuming it from a missing neighbour entry?  
{2–3 sentences + two proof lines}

4. PASSIVE VERSUS EXCLUDED: Which line proved VLAN20 was passive, and which proved G0/0/0 was not in OSPF? Why did the same prefix appear on EDGE for one and not the other?  
{2–3 sentences + two proof lines}

5. Reusable knowledge: A neighbour is FULL. What else must you inspect before claiming a user can reach a server? What is the first thing you would check next time a router has no OSPF-learned routes?  
{2–3 sentences}

---

File naming suggestion: `reflections/r05-{username}.txt`
