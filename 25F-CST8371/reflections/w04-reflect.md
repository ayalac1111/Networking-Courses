# Weekly Reflect — W04: Fundamentals of OSPF (Neighbouring, Passive, Default)

**Week:** W04  
**Date:** {date}  
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip ospf neighbor`, `show ip ospf interface g0/x`, `show ip protocols | include Passive|Routing for`, `show ip route 0.0.0.0`). Avoid `show running-config`.

---

## Always True Rules

### Always Rule 1 — OSPF neighbours form only when link parameters match
**Rule (one line):**  
**If** two routers share the **same IP subnet**, **same Area ID**, and **matching Hello/Dead (and auth/MTU if configured)** on a link, **then** OSPF forms an adjacency that reaches **FULL**.

**In my own words (1–2 sentences):**  
{explain why masks define L3 adjacency, Area binds LSDB scope, and Hello/Dead drive liveness}

**Proof lines (pick two, 1–2 lines each):**
- `R? # show ip ospf interface g0/x | include Internet Address|Area|Hello|Dead`
- `R? # show ip ospf neighbor | include FULL|EXSTART|Gi0/x`
- `R? # show ip interface g0/x | include Internet address`

**If this breaks next week, first move:**  
{compare Hello/Dead and Area on both ends; confirm masks are /30↔/30 on p2p links and MTU/auth match if present}

---

### Always Rule 2 — Passive interfaces advertise, but do not neighbour
**Rule (one line):**  
**If** an interface is **passive** under OSPF, **then** the network is advertised but **no hellos** are sent/received and **no neighbour** can form on that interface.

**In my own words (1–2 sentences):**  
{clarify policy: user-edge LANs passive; router-to-router transit not passive}

**Proof lines (pick two, 1–2 lines each):**
- `R? # show ip protocols | include Passive`
- `R? # show ip ospf interface vlan{LAN} | include Passive|Process ID|Area`
- `R? # show ip ospf neighbor | include vlan{LAN}` *(should show none)*

**If this breaks next week, first move:**  
{audit passive list: make transits non-passive; keep user SVIs passive}

---

### Always Rule 3 — Default-information originates only with a real default (unless *always*)
**Rule (one line):**  
**If** the edge router has a default in its **RIB**, `default-information originate` will inject `0.0.0.0/0` to neighbours; **without** a RIB default you must use `always`.

**In my own words (1–2 sentences):**  
{explain OSPF depends on RIB presence; downstream sees `O*E2 0.0.0.0/0` (or `O IA` in stub)}

**Proof lines (pick two, 1–2 lines each):**
- `EDGE# show ip route | include ^S\*|^Gateway` *(prove a real default exists)*
- `EDGE# show ip ospf database external | include 0.0.0.0` *(normal area)*
- `DOWN# show ip route 0.0.0.0 | include O\*E2|O IA`
- `DOWN# traceroute 8.8.8.8 | include 1  ` *(first hop = edge)*

**If this breaks next week, first move:**  
{add static/default on EDGE or enable `default-information originate always`; then verify downstream route and path}

---

## CER Micro-Cards (create from the audit activity)

> **What is a CER micro-card?** A tiny 3‑liner to lock in a concept: **Claim → Evidence → Reasoning**.  

---

## Ops Lexicon — New This Week

**Adjacency States:** `Down → Init → 2-Way → ExStart → Exchange → Loading → FULL` (p2p should end at **FULL**).

**Router ID (RID):** 32‑bit identifier for OSPF; duplicates cause issues on multi-access.

**Area 0 (Backbone):** Mandatory transit area; all other areas must connect logically to Area 0.

**Interface‑mode OSPF:** `interface g0/x` then `ip ospf 1 area 0` (we do **not** use `network` statements).

**Hello/Dead:** Default **10/40** on many IOS p2p links; must match across the link.

**Passive Interface:** Advertises connected network into OSPF but suppresses hellos; no neighbours form.

**Route Types:** `O` (intra-area), `O IA` (inter-area), `O E1/E2` (external). Default often appears as **`O*E2 0.0.0.0/0`**.

**Network Types:** `point-to-point`, `broadcast`, `non-broadcast`, etc. DR/BDR election applies on broadcast/NBMA; not on p2p.

---

## Reflection Questions (answer briefly)

1. **Neighbour recipe:** Which exact parameter was wrong **on each broken link** in the audit (mask, area, hello/dead)? Give **one** proof line for each.  
   {3–6 sentences total}

2. **Passive policy:** Which interface(s) in your pod should be **passive** and why? Add one line that proves it is passive.  
   {2–3 sentences + one proof line}

3. **Default path:** How did you prove the default was originated and selected downstream? Provide one control‑plane line and one data‑plane line.  
   {2–3 sentences}

4. **RID & stability (optional):** Did you confirm unique RIDs? What command would reveal a duplicate and what symptom would you expect?  
   {1–2 sentences}

---

## Quick Checklist — What you *actually* built
- [ ] R1–R2 (p2p) neighbour **FULL**, Area 0, masks match  
- [ ] R2–R3 (p2p) neighbour **FULL**, Area 0  
- [ ] R3–R1 (p2p) neighbour **FULL**, timers match  
- [ ] User/edge interfaces **passive**  
- [ ] Downstream routers learn `O*E2 0.0.0.0/0` from EDGE

---

## Time & Confidence
- **Time spent (hh:mm):** ________  
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best *one‑liner* this week
Paste your single favourite verification line (prompt + command + proof), e.g.:

```bash
R2# show ip ospf neighbor | include Gi0/1 -> FULL
```


---

*File naming suggestion:* `reflections/w03-reflect-{username}.md`
