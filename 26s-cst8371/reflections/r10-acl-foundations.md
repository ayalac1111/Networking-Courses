# Weekly Reflect — W10: ACL Foundations — Standard ACLs & Wildcard Masks

**Week:** W10
**Date:** {date}
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip access-lists`, `show ip interface`, `show logging`). Avoid `show running-config` or static screenshots.

---

## Always True Rules

### Always Rule 1 — Standard ACLs check source only
**Rule (one line):**
A **standard ACL** matches packets **only on source address**, never on destination.

**In my own words (1–2 sentences):**
Even if traffic is destined for a protected network, ACL logic only looks at the source IP to decide permit or deny. This is why spoof prevention relies on blocking fake source addresses, not destination ones.

**Proof lines (pick two):**
- `CORE# show access-lists PROTECT-PC`
- `EDGE# show ip interface GigabitEthernet0/0/2 | include access list`

**If this breaks next week, first move:**
Check whether your ACL logic is matching source (not destination); verify the ACL type (standard vs extended).

---

### Always Rule 2 — ACLs process top-down, first-match wins
**Rule (one line):**
ACLs read from the top and stop at the first match — later lines are ignored once a decision is made.

**Proof lines (pick two):**
- `EDGE# show access-lists PROTECT-VM`
- `EDGE# ping 198.18.U.130 source 203.0.113.U`

**If this breaks next week, first move:**
Reorder or insert ACL entries logically — more specific conditions before general permits.

---

### Always Rule 3 — Implicit deny is always last
**Rule (one line):**
Every ACL ends with an **invisible deny all**, even if you don't type it.

**Proof lines (pick two):**
- `EDGE# show access-lists PROTECT-VM` → hit counters for deny and permit lines
- `EDGE# ping 198.18.U.130 source 198.18.U.129` (expect drop)

**If this breaks next week, first move:**
Explicitly add `permit any` at the end if needed for testing or gradual policy rollout.

---

### Always Rule 4 — Standard ACLs are placed close to the destination
**Rule (one line):**
Because a standard ACL cannot see the destination, place it as close as possible to the network it protects — placing it near the source risks blocking traffic meant for other destinations.

**In my own words (1–2 sentences):**
Placing a standard ACL far from its intended target widens its "blast radius" — it can end up filtering traffic that was never headed for the protected subnet in the first place.

**Proof lines (pick two):**
- `EDGE# show ip interface GigabitEthernet0/0/2 | include PROTECT-VM`
- `EDGE# show access-lists PROTECT-VM` (permit/deny hit counts at the VM-facing interface)

**If this breaks next week, first move:**
Re-trace the traffic path from source to destination and re-apply the ACL on the interface/direction closest to the protected resource.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**

**CER 1 — Spoofed source blocked**
- **Claim:** Packets spoofing a PC address were denied at CORE.
- **Evidence:** `EDGE# show access-lists PROTECT-PC` → `deny 198.18.U.128 0.0.0.63 (3 matches)`
- **Reasoning:** ACL successfully stopped packets pretending to originate from the protected PC subnet.

**CER 2 — Legit traffic allowed**
- **Claim:** External or VM-sourced packets were permitted to the PC subnet.
- **Evidence:** `EDGE# show access-lists PROTECT-PC` → `permit any (5 matches)`
- **Reasoning:** The ACL logic allowed valid packets while maintaining anti-spoofing protection.

**CER 3 — Placement confirmed by hit counts**
- **Claim:** PROTECT-VM correctly filters only PC-sourced traffic headed to the VM segment.
- **Evidence:** `EDGE# show access-lists PROTECT-VM` → permit/deny hit counts at Gi0/0/2, no unrelated traffic blocked.
- **Reasoning:** Placing the ACL at the interface closest to the VM subnet confirms the "close to destination" placement rule for standard ACLs.

---

## Ops Lexicon — New This Week

- **Wildcard mask:** A binary pattern used to define which bits in an IP address are checked (0 = must match, 1 = ignore).
- **Standard ACL:** Access control list that filters traffic by **source IP only**.
- **Extended ACL:** ACL that filters by source, destination, protocol, and port.
- **Implicit deny:** The unseen final rule that drops all packets not previously permitted.
- **Anti-spoofing:** Security technique using ACLs to block packets claiming to originate from internal subnets when they actually arrive from external links.
- **Access-class:** Command used to bind a standard ACL to VTY lines for management-plane hardening.

---

## Reflect Questions

1. Why does the PROTECT-PC ACL inspect **source** instead of **destination** addresses?
2. What would happen if PROTECT-VM were applied **inbound** on EDGE's transit interface instead of **outbound** on the VM-facing interface?
3. Show two `show access-lists` lines proving both a permit and a deny match occurred.
4. What part of ACL behaviour confirms the "first-match wins" rule?
5. Describe one placement decision you had to reason through this week (PROTECT-PC or PROTECT-ALS) and why you chose that interface and direction.

---

## Time & Confidence
- **Time spent (hh:mm):** ________
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner
Paste your best ops verification line proving ACL logic worked.
