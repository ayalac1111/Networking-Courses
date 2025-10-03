# Weekly Reflect — W06: OSPF Hybrid, Multi-Vendor, and Control Plane Ops

**Week:** W06  
**Date:** {date}  
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip ospf neighbor`, `show ip route`, `show ip protocols`, `ip address print`, Winbox summaries). Avoid `show running-config` or static screenshots.

---

## Always True Rules

### Always Rule 1 — Router-ID must be **unique** across the OSPF domain
**Rule (one line):**
If two routers have the **same Router-ID**, OSPF adjacency **will not form correctly** and may stay stuck in **EXSTART/EXCHANGE**.

**In my own words (1–2 sentences):**
The Router-ID acts like a unique identifier for LSAs and neighbors. Duplicate IDs cause confusion in SPF and neighbor state flapping.

**Proof lines (pick two):**
- `DIST# show ip protocols | include ID`
- `CORE# show ip ospf neighbor | include EXSTART`

**If this breaks next week, first move:**
Confirm all RIDs are unique and stable (loopback preferred); if needed, clear OSPF process to re-elect.

---

### Always Rule 2 — Passive interfaces **advertise**, but never neighbor
**Rule (one line):**
Passive OSPF interfaces **still advertise** connected networks but **do not form** OSPF neighbor relationships.

**In my own words (1–2 sentences):**
Use passive on LAN-facing interfaces to avoid unnecessary neighbor formation. Use non-passive on router-router links.

**Proof lines (pick two):**
- `R? # show ip protocols | include Passive`
- `R? # show ip ospf interface | include Passive`

**If this breaks next week, first move:**
Remove passive from transit interfaces (e.g., EDGE↔DIST), or confirm it's only applied to user LAN SVIs.

---

### Always Rule 3 — OSPF prefers **lowest cost** path
**Rule (one line):**
OSPF selects routes with the **lowest cumulative cost**, and cost is determined by interface bandwidth (or manual override).

**In my own words (1–2 sentences):**
If two paths exist, OSPF picks the one with the lower cost. If costs are misaligned (e.g., 10 vs 1), traffic takes the longer route.

**Proof lines (pick two):**
- `EDGE# show ip route | include 10.U.100.3`
- `EDGE# show ip ospf interface brief`

**If this breaks next week, first move:**
Check cost on each interface and verify if the shortest physical path has the lowest metric.

---

## CER Micro-Cards (Claim → Evidence → Reasoning)

**CER 1 — Router-ID collision**  
- **Claim:** OSPF neighborship between CORE and DIST was stuck in EXSTART.  
- **Evidence:** Both `show ip protocols` showed RID 10.250.100.2.  
- **Reasoning:** OSPF cannot form full adjacency between routers with duplicate IDs.

**CER 2 — Passive interface blocked neighbor**  
- **Claim:** DIST did not form OSPF neighbor on Gi0/0/2.  
- **Evidence:** `show ip ospf interface` shows passive enabled on Gi0/0/2.  
- **Reasoning:** Passive suppresses hellos, so no neighbor is formed.

**CER 3 — Wrong cost caused long path**  
- **Claim:** EDGE took the long path to reach DIST’s loopback via CORE.  
- **Evidence:** OSPF route to 10.U.100.3 pointed to CORE, not DIST.  
- **Reasoning:** Cost on the direct EDGE↔DIST link was higher, so OSPF chose the alternate route.

---

## Ops Lexicon — New This Week

- **Router-ID (RID):** OSPF unique identifier; must be unique in domain.
- **EXSTART/EXCHANGE:** Adjacency states that fail if RIDs are duplicated.
- **Cost:** OSPF path metric; lower is better. Manual or calculated from bandwidth.
- **Passive Interface:** Suppresses hellos; no neighbors but still advertises prefix.
- **Winbox IP OSPF Tab:** MikroTik GUI to verify neighbors and interfaces.

---

## Reflect Questions

1. What made this week more challenging when working across Cisco and MikroTik? Mention one CLI-vs-GUI learning point.
2. How did you detect a Router-ID conflict, and how would you resolve it safely in a real environment?
3. Show an example where cost manipulation fixed a routing path. Include the before/after proof.
4. What would happen if you forgot to passive a user VLAN SVI?
5. Which command (Cisco or MikroTik) became your go-to for checking OSPF state?

---

## Time & Confidence
- **Time spent (hh:mm):** ________  
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Best one-liner this week
Paste your favourite verification command from this week (prompt + command + result), e.g.:
```
DIST# show ip route | include 10.250.200.0  -> (missing before fix, appears after)
```

