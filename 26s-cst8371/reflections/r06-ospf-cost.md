# W05 — Reflect: OSPF Tuning (Costs, DR/BDR, Default Path)

**Week:** W05  
**Date:** {date}  
**Student:** {your_name}  
**Ops-only policy:** Use operational commands (device prompt + command). No `show running-config`. Packet Tracer friendly outputs.

---

## Always True Rules (write in your own words)

### Always Rule 1 — Cost math must be consistent
**Rule (one line):** If routers have links of the same speed, their **OSPF interface costs** must align; otherwise routes will show **different metrics** for the same prefix.

**In my words (1–2 sentences):**  
{explain why inconsistent reference-bandwidth or manual cost makes metrics disagree}

**Proof lines (pick two):**
- `EDGE# show ip route | include 10.250.20.0  -> [110/metric]`  
- `CORE# show ip route | include 10.250.20.0  -> [110/metric]`

**If this breaks next week, first move:**  
Compare route metrics on two routers for the same prefix; normalise the underlying cost knob.

---

### Always Rule 2 — Cost knobs: ref‑bw vs manual cost (keep it consistent)
**Rule (one line):** The **manual interface cost** (`ip ospf cost <n>`) overrides bandwidth/reference‑bandwidth math; pick **one strategy** and apply it consistently across the domain.

**Proof lines (pick two):**
- `EDGE# show ip route | include 10.250.20.0  -> [110/metric]`  
- `CORE# show ip route | include 10.250.20.0  -> [110/metric]`

**If this breaks next week, first move:**  
Remove ad‑hoc manual costs or align `auto-cost reference-bandwidth` everywhere; then re‑check the route metric lines.

---

### Always Rule 3 — DR is non‑preemptive
**Rule (one line):** If a **DR** is elected on a broadcast/NBMA segment, it **keeps** the role until it **fails** or the **OSPF process resets**—raising another router’s priority **does not** replace the current DR by itself.

**Proof lines (pick two):**
- `R? # show ip ospf neighbor | include FULL|DR|BDR`  
- `R? # show ip ospf interface g0/x | include Priority|State`

**If this breaks next week, first move:**  
To change DR on purpose, shut/no shut one side or `clear ip ospf process` on the segment; then verify DR/BDR roles.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**. Keep each to 3 lines.

**CER 1 — Same prefix, different metric**  
- **Claim:** EDGE and CORE showed different OSPF metrics for `10.250.20.0/24`.
- **Evidence:** {paste two lines: `EDGE# show ip route | include 10.250.20.0` and `CORE# …` before fix}
- **Reasoning:** Cost was computed with differing assumptions; standardising the cost knob aligned metrics.

**CER 2 — Default path wasn’t exported**  
- **Claim:** Downstream lacked `O*E2 0.0.0.0/0`.
- **Evidence:** {`EDGE# show ip route | include ^S*|^Gateway` shows no default; `CORE# show ip route 0.0.0.0` empty}
- **Reasoning:** Origination depends on an EDGE RIB default (or `always`). After fix, the LSA appeared and traffic followed EDGE.

**CER 3 — DR role persists**  
- **Claim:** Changing a router’s priority did **not** replace the current DR.  
- **Evidence:** {`R? # show ip ospf neighbor` before/after; DR stayed the same until a reset/failure}  
- **Reasoning:** OSPF DR election is **non‑preemptive**.

---

## Ops Lexicon — New This Week
- **Reference-bandwidth:** Global OSPF setting used to derive interface **cost** from bandwidth; must match across the domain for consistent metrics.
- **Interface cost (manual):** Per-interface override (`ip ospf cost <n>`) that directly sets cost regardless of bandwidth.
- **Metric:** The **second number** in the OSPF bracketed pair in `show ip route` `[110/metric]`; lower is preferred.
- **Administrative distance (AD):** The **first number** in `[110/metric]`; OSPF AD is typically **110**.
- **Default-information originate:** Command to advertise `0.0.0.0/0` into OSPF; requires an EDGE default in the RIB unless `always` is specified.
- **DR/BDR (Broadcast/NBMA):** DR is elected and remains until failure or process reset; **non‑preemptive** behaviour.

---

## Reflect Questions

1) Name **two** knobs that directly change OSPF **interface cost**, and state which one you used in the challenge.  
2) Packet Tracer doesn’t show reference-bandwidth with `show ip ospf`. How did you still **justify** that costing was aligned?  
3) Give one example of something that **doesn’t** alter OSPF cost but could still affect convergence or adjacency (and why).
4) In one sentence, explain why AD stayed **110** but the **metric** changed in your outputs.  
5) Show one **before/after** route line for `10.250.20.0/24` and explain the improvement in plain language.  
6) (DR/BDR) How would you safely replace a DR on a broadcast segment? What proof line shows the new DR?

---

## Time & Confidence
- **Time spent (hh:mm):** ________  
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one‑liner
Paste your favourite verification line from this week (prompt + command + proof), e.g.:
```
CORE# show ip route | include 10.250.20.0  -> [110/2] (was [110/20])
```

