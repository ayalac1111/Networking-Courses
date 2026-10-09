title W06 — Reflect: OSPF Route Learning, Cost and Path Evidence
activity reflect
deliverable reflections/r06-{username}.txt
points 0

# Weekly Reflect — W06: OSPF Route Learning, Cost and Path Evidence

Week: W06  
Date: {date}  
Student: {your_name}  

> Ops-only evidence policy: Use operational commands: `show ip ospf interface <interface>`, `show ip ospf neighbor`, `show ip route <destination> <mask>`, `show ip cef <destination>`, `show interfaces <interface> | include BW`, `ping`. Use your own Lab 06 capture files; do not use `show running-config` as evidence.

---

## Submission Contract

| Item | Requirement |
|---|---|
| Action | Answer each prompt using evidence from your Lab 06 checkpoint files (C01–C04). |
| Evidence | Include at least one exact command/output proof line for each reflection question. |
| Success Indicator | Your answer names what the evidence showed (role, neighbour state, cost, route or path) and what it did not show. |
| Failure Signal | Generic answers, missing command output, or claims that treat a neighbour, a role or a lower priority number as the same proof as a route or a path. |

---

## Always True Rules

### Always Rule 1 — Priority fills an empty role; it does not remove a healthy holder

Rule (one line):  
If a DR already exists on a broadcast segment, a router that arrives later with a higher priority or Router ID does not replace it; priority and Router ID rank the candidates only when a role has to be filled.

In my own words (1–2 sentences):  
{write what decided who held DR and BDR in your captures, and why the order of events mattered as much as the numbers}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show ip ospf interface <interface> | include State|Priority|Designated → {state, priority or DR/BDR line}`
- `{DEVICE}# show ip ospf neighbor → {a FULL/DR, FULL/BDR or 2-WAY/DROTHER line}`

If this breaks next week, first move:  
{read State, Priority and the DR and BDR lines on the segment before predicting who should hold a role}

---

### Always Rule 2 — A FULL neighbour is not a route

Rule (one line):  
If the two ends of a link describe it differently, such as a different network type, the neighbour can reach FULL and the routes still do not cross, with no log message to warn you.

In my own words (1–2 sentences):  
{write which evidence stopped being enough once the neighbour was FULL, and where you looked next}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show ip ospf interface <interface> | include Network Type → {network type line}`
- `{DEVICE}# show ip route <destination> <mask> → {route line, or the absence of one}`

If this breaks next week, first move:  
{compare the network type and interface details on both ends of the link, then the route, before changing anything}

---

### Always Rule 3 — Cost is the evidence for a path; the lowest total wins

Rule (one line):  
If you can read each outgoing interface cost along a path and add the cost advertised for the destination, you can explain the metric on the installed route instead of accepting it.

In my own words (1–2 sentences):  
{write how reference bandwidth and interface bandwidth produced each cost, and how the costs became the metric you saw on the route}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show interfaces <interface> | include BW → {bandwidth line}`
- `{DEVICE}# show ip route <destination> <mask> → {metric line}`

If this breaks next week, first move:  
{read the cost on each interface of the competing paths, add them, and compare the total to the route's metric}

---

### Always Rule 4 — A cost you set on one router is a cost every router adds up

Rule (one line):  
If a router advertises a link at a changed cost, every router that learns a path through it includes that cost in its own total, even though nothing on those routers changed.

In my own words (1–2 sentences):  
{write what the installed next hops, the forwarding entry and a traffic test each showed, and which of them you needed after the other two}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show ip route <destination> <mask> → {next hop line or lines}`
- `{DEVICE}# show ip cef <destination> → {forwarding next hop line}`
- `{DEVICE}# ping <destination> source <interface> → {result}`

If this breaks next week, first move:  
{check which router changed a cost, then re-read the route on the router that picked the unexpected path}

---

## CER Micro-Cards

> What is a CER micro-card? A tiny 3-line check: Claim → Evidence → Reasoning.

### CER 1 — Role held on the segment

Claim:  
{name the router that held DR in one of your captured states and the state you are describing}

Evidence:  
`{DEVICE}# {command} → {paste one exact line from that state's capture}`

Reasoning:  
{explain why the numbers and the order of events in that state produced this holder and not another}

---

### CER 2 — FULL with no route

Claim:  
{state what the neighbour table showed and what the routing table did not show}

Evidence:  
`{DEVICE}# {command} → {paste one exact line for each}`

Reasoning:  
{explain what the line you pasted could not tell you, and which other line located the fault}

---

### CER 3 — Why this path was chosen

Claim:  
{name the next hop the router chose to reach the destination, in the state you are describing}

Evidence:  
`{DEVICE}# show ip route <destination> <mask> → {route line}`  
`{DEVICE}# show interfaces <interface> | include BW → {bandwidth line}`

Reasoning:  
{explain how the costs on the competing paths add up to this choice, using your own numbers}

---

## Ops Lexicon — New This Week

Incumbent  
The router that already holds a role on a broadcast segment. A later arrival does not replace it by ranking higher.

Priority  
The interface value that ranks candidates when a DR or BDR must be filled. Priority 0 makes the router ineligible for either role. It is not a route cost.

Network type  
How OSPF treats a link: broadcast links elect a DR and BDR; point-to-point links have exactly two routers and do not. Both ends must describe the link the same way.

Cost  
The value OSPF assigns to an outgoing interface: reference bandwidth divided by interface bandwidth, with a minimum of 1, unless an explicit cost overrides it.

Reference bandwidth  
The scale OSPF divides by when it calculates automatic cost. It must be the same on every router.

Path cost / metric  
The sum of the outgoing interface costs along a path, plus the cost advertised for the destination. The lowest total is installed.

ECMP  
Equal-cost multipath: two or more next hops with the same total cost, installed together for one destination.

CEF  
The forwarding table the router uses to send packets. `show ip cef` shows the next hops that forwarding will use.

---

## Reflection Questions

1. ELECTION: Which capture showed who held DR or BDR in one state, and which values and which event in the sequence explain that result? What did the result show about what priority does and does not do?  
{2–3 sentences + one proof line}

2. FULL, NO ROUTE: In your mismatch capture, what did the neighbour output show, and what did it leave out? Which evidence located the cause instead of assuming it?  
{2–3 sentences + two proof lines}

3. COST: Using your own numbers, how did the interface costs along the path become the metric on the route, and what changed in that explanation after the reference-bandwidth change?  
{2–3 sentences + two proof lines}

4. PATH CHOICE: After the link failure and the recovery, which evidence told you which next hops were installed and which path was in use? What did the cost you set on one router change on a router you did not configure?  
{2–3 sentences + two proof lines}

5. Reusable knowledge: In Lab 06, where did your prediction differ from what you captured, and what did you do next? Next time OSPF picks a path you did not expect, what is the first thing you would check, and why before changing anything?  
{2–3 sentences}

---

File naming suggestion: `reflections/r06-{username}.txt`
