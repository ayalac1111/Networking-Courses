title W07 — Reflect: OSPF Diagnosis from Evidence
activity reflect
deliverable reflections/r07-{username}.txt
points 0

# Weekly Reflect — W07: OSPF Diagnosis from Evidence

Week: W07  
Date: {date}  
Student: {your_name}  

> Ops-only evidence policy: Use operational commands: `show ip protocols`, `show ip ospf interface <interface>`, `show ip ospf neighbor`, `show ip route <destination> <mask>`. Use your own capture files from Lab 05 and Lab 06. If you worked through the Challenge 02 CML example, you may use those captures too. Do not use `show running-config` as evidence.

---

## Submission Contract

| Item | Requirement |
|---|---|
| Action | Answer each prompt using evidence from your own Lab 05 and Lab 06 captures (and the Challenge 02 CML example, if you used it). |
| Evidence | Include at least one exact command/output proof line for each reflection question. |
| Success Indicator | Your answer names what the evidence showed (identity, interface behaviour, neighbour state or parameter) and what it did not show. |
| Failure Signal | Generic answers, missing command output, or claims that treat an empty neighbour table as the same proof as the cause of the problem. |

---

## Always True Rules

### Always Rule 1 — A Router ID names the router, and it must be unique

Rule (one line):  
If two routers in the same OSPF domain use the same Router ID, their adjacency cannot be relied on; each router must be identified by a value no other router uses.

In my own words (1–2 sentences):  
{write how one of your routers got its Router ID, and why a value that no other router uses matters}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show ip protocols | include Router ID → {Router ID line}`
- `{DEVICE}# show ip ospf neighbor → {neighbour ID column showing the other router's ID}`

If this breaks next week, first move:  
{read the Router ID on each router and compare them before changing any interface setting}

---

### Always Rule 2 — A passive interface still advertises its network but forms no neighbour

Rule (one line):  
If an interface is passive, OSPF still advertises the network on it and sends no Hellos, so no neighbour can form there; use it on an interface that faces only end hosts, never on a link to another router.

In my own words (1–2 sentences):  
{write which of your interfaces faces only hosts, what you expect OSPF to do on it, and what it must not do on a router-to-router link}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show ip protocols | include Passive → {passive interface line}`
- `{DEVICE}# show ip ospf interface <interface> | include Hello|Passive → {Hello or passive line}`
- `{DEVICE}# show ip route <destination> <mask> → {route to the network behind the passive interface, as seen from another router}`

If this breaks next week, first move:  
{check whether the interface faces a router or only hosts before deciding whether it should be passive}

---

### Always Rule 3 — Two routers become neighbours only when the values that must agree do agree

Rule (one line):  
If the two ends of a link disagree on a value OSPF checks before forming a neighbour, such as the area, the subnet or the Hello and Dead timers, no neighbour forms, and the neighbour table is empty whichever value is wrong.

In my own words (1–2 sentences):  
{write which values you compared on the two ends of a link, and which output showed you each end}

Proof lines (pick two, 1–2 lines each):
- `{DEVICE}# show ip ospf interface <interface> | include Area|Hello → {area and timer lines}`
- `{DEVICE}# show ip ospf neighbor → {a neighbour line, or the absence of one}`

If this breaks next week, first move:  
{compare the interface details on both ends of the link, one value at a time, before changing anything}

---

## CER Micro-Cards

> What is a CER micro-card? A tiny 3-line check: Claim → Evidence → Reasoning.

### CER 1 — How a router was identified

Claim:  
{name one router and the Router ID it used in your captures}

Evidence:  
`{DEVICE}# {command} → {paste one exact line}`

Reasoning:  
{explain where that Router ID came from and what would go wrong if another router used the same one}

---

### CER 2 — An interface that should or should not form a neighbour

Claim:  
{name an interface and state whether a neighbour should form on it, in the state you are describing}

Evidence:  
`{DEVICE}# {command} → {paste one exact line for the interface setting}`  
`{DEVICE}# {command} → {paste one exact line for the neighbour table}`

Reasoning:  
{explain what the interface faces and why that makes the evidence expected or a problem}

---

### CER 3 — Why a neighbour did or did not form

Claim:  
{state what the neighbour table showed on a link in the state you are describing}

Evidence:  
`{DEVICE}# {command} → {paste one exact line from each end of the link}`

Reasoning:  
{explain which values you compared, and what the empty or full neighbour table could not tell you by itself}

---

## Ops Lexicon — New This Week

Router ID  
The 32-bit value that identifies a router in OSPF. It is set manually or chosen by the router from its addresses, and it must be unique in the domain.

Passive interface  
An interface that OSPF advertises but does not use to send Hellos, so no neighbour forms on it. It is meant for interfaces that face only end hosts.

Hello and Dead interval  
How often a router sends Hellos on an interface, and how long it waits without hearing one before it declares the neighbour down. Both ends of a link must use the same values.

Adjacency  
The neighbour relationship between two routers. The neighbour table lists it with a state; `FULL` means the routers have exchanged their databases.

Area ID  
The OSPF area an interface belongs to. Both ends of a link must be in the same area.

---

## Reflection Questions

1. ROUTER ID: Which Router ID did each router in your capture use, and how did it get it? What would you check first if two routers showed the same one?  
{2–3 sentences + one proof line}

2. PASSIVE: Which interface faces only hosts in your topology? What did the passive setting stop, and what did it leave working? Which line showed each?  
{2–3 sentences + two proof lines}

3. NEIGHBOUR: On one link, which values did you compare on the two ends? What did the neighbour table show, and what did it leave out?  
{2–3 sentences + two proof lines}

4. EVIDENCE: Pick one problem you met in Lab 05 or Lab 06 (or in the Challenge 02 example). Which evidence located the cause instead of assuming it?  
{2–3 sentences + one proof line}

5. Reusable knowledge: Next time two OSPF routers do not become neighbours, what is the first thing you would check, and why before changing anything?  
{2–3 sentences}

---

File naming suggestion: `reflections/r07-{username}.txt`
