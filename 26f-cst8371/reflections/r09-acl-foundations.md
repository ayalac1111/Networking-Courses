# Weekly Reflect — W09: ACL Foundations — Standard ACLs & Wildcard Masks

**Week:** W09
**Date:** {date}
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip access-lists`, `show ip interface`, `show users`). Avoid `show running-config` or static screenshots.

---

## Always True Rules

### Always Rule 1 — Standard ACLs check source only
**Rule (one line):**
A **standard ACL** matches packets **only on source address**, never on destination.

**In my own words (1–2 sentences):**
Even if traffic is destined for a protected network, ACL logic only looks at the source IP to decide permit or deny. This is why a standard ACL can stop traffic from one source, but cannot be told to stop it only when it is headed to one destination.

**Proof lines (pick two):**
- `R1# show ip access-lists BLOCK-PC1`
- `R2# show ip interface <interface> | include access list`

**If this breaks next week, first move:**
Check whether your ACL logic is matching source (not destination); verify the ACL type (standard vs extended).

---

### Always Rule 2 — ACLs process top-down, first-match wins
**Rule (one line):**
ACLs read from the top and stop at the first match — later lines are ignored once a decision is made.

**Proof lines (pick two):**
- `R1# show ip access-lists BLOCK-PC1`
- `PC1:~$ ping -c 4 198.18.U.78`

**If this breaks next week, first move:**
Reorder or insert ACL entries logically — more specific conditions before general permits.

---

### Always Rule 3 — Implicit deny is always last
**Rule (one line):**
Every ACL ends with an **invisible deny all**, even if you don't type it.

**Proof lines (pick two):**
- `R2# show ip access-lists MGMT-R2` → hit counter on the last line after a denied attempt
- `PC2:~$ ssh admin@198.18.U.2` (expect no login prompt)

**If this breaks next week, first move:**
Explicitly add `permit any` at the end if needed for testing or gradual policy rollout.

---

### Always Rule 4 — Standard ACLs are placed close to the destination
**Rule (one line):**
Because a standard ACL cannot see the destination, place it as close as possible to the network it protects — placing it near the source risks blocking traffic meant for other destinations.

**In my own words (1–2 sentences):**
Placing a standard ACL far from its intended target widens its "blast radius" — it can end up filtering traffic that was never headed for the protected subnet in the first place.

**Proof lines (pick two):**
- `R2# show ip interface <interface> | include access list`
- `R1# show ip access-lists BLOCK-PC1` (permit/deny hit counts after testing traffic to different destinations)

**If this breaks next week, first move:**
Re-trace the traffic path from source to destination and re-apply the ACL on the interface/direction closest to the protected resource.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**

**CER 1 — Blocked source denied**
- **Claim:** Packets from the Accounting network toward SRV-Payroll were denied.
- **Evidence:** `R2# show ip access-lists BLOCK-ACCOUNTING` → `deny 198.18.U.64, wildcard bits 0.0.0.15 ({n} matches)`
- **Reasoning:** The counter moved, so the packets reached the deny line, and the ACL matched them by their source address.

**CER 2 — Legit traffic allowed**
- **Claim:** Packets from PC1 toward SRV-Payroll were permitted.
- **Evidence:** `R2# show ip access-lists BLOCK-ACCOUNTING` → `permit any ({n} matches)`
- **Reasoning:** The ACL denied only the Accounting source and let every other source through.

**CER 3 — Placement confirmed by hit counts**
- **Claim:** BLOCK-PC1 stopped PC1 toward Accounting while PC1 still reached SRV-Payroll.
- **Evidence:** `R1# show ip access-lists BLOCK-PC1` → `deny host 198.18.U.46 ({n} matches)`, and `PC1:~$ ping -c 4 198.18.U.110` → `0% packet loss`
- **Reasoning:** The interface and direction you choose decide which traffic the ACL sees; traffic to other destinations was not blocked, which supports the "close to destination" placement rule for standard ACLs.

---

## Ops Lexicon — New This Week

- **Wildcard mask:** A binary pattern used to define which bits in an IP address are checked (0 = must match, 1 = ignore).
- **Standard ACL:** Access control list that filters traffic by **source IP only**.
- **Extended ACL:** ACL that filters by source, destination, protocol, and port.
- **Implicit deny:** The unseen final rule that drops all packets not previously permitted.
- **Stateless filtering:** An ACL tests every packet on its own, with no memory of earlier packets, so a reply is filtered like any other packet.
- **Access-class:** Command used to bind a standard ACL to VTY lines for management-plane hardening.

---

## Reflect Questions

1. Why do BLOCK-ACCOUNTING and BLOCK-PC1 inspect **source** instead of **destination** addresses?
2. What would happen if BLOCK-ACCOUNTING were placed near its **source** (on R1) instead of near its **destination**? Name one flow that would be blocked and should not be.
3. Show two `show ip access-lists` lines proving both a permit and a deny match occurred.
4. What part of ACL behaviour confirms the "first-match wins" rule?
5. Describe one placement decision you had to reason through this week (BLOCK-ACCOUNTING or BLOCK-PC1) and why you chose that interface and direction.

---

## Time & Confidence
- **Time spent (hh:mm):** ________
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner
Paste your best ops verification line proving ACL logic worked.
