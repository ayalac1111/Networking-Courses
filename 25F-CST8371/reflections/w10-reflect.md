# W10 — Reflect: Standard ACLs & Wildcard Masks

**Week:** W10  
**Date:** {date}  
**Student:** {your_name}  
**Ops-only policy:** Use operational commands (device prompt + command). No `show running-config`. Packet Tracer friendly outputs.

---

## Always True Rules (write in your own words)

### Always Rule 1 — Standard ACLs check source only
**Rule (one line):** A **standard ACL** matches packets **only on source addresses**, never on destinations.

**In my words (1–2 sentences):**  
Even if traffic is destined for a protected network, ACL logic only looks at the **source IP** to decide permit or deny. This is why spoof prevention relies on blocking fake source addresses, not destination ones.

**Proof lines (pick two):**
- `CORE# show access-lists PROTECT-PC`
- `CORE# show ip interface g0/0/0 | include access list`

**If this breaks next week, first move:**  
Check whether your ACL logic is placed close to the source; verify the ACL type (standard vs extended).

---

### Always Rule 2 — ACLs process top-down, first-match wins
**Rule (one line):** ACLs read from the top and stop at the first match — later lines are ignored once a decision is made.

**Proof lines (pick two):**
- `CORE# show access-lists PROTECT-PC`
- `EDGE# ping 198.18.U.130 source 203.0.113.U`

**If this breaks next week, first move:**  
Reorder or insert ACL entries logically — deny conditions before general permits.

---

### Always Rule 3 — Implicit deny is always last
**Rule (one line):** Every ACL ends with an **invisible deny all**, even if you don’t type it.

**Proof lines (pick two):**
- `CORE# show access-lists PROTECT-PC` → hit counters for deny and permit lines  
- `EDGE# ping 198.18.U.130 source 198.18.U.129` (expect drop)

**If this breaks next week, first move:**  
Explicitly add `permit any` at the end if needed for testing or gradual policy rollout.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**

**CER 1 — Spoofed source blocked**  
- **Claim:** Packets spoofing a PC address were denied at CORE.  
- **Evidence:** `CORE# show access-lists PROTECT-PC` → `deny 198.18.U.128 0.0.0.63 (3 matches)`  
- **Reasoning:** ACL successfully stopped packets pretending to originate from the protected PC subnet.

**CER 2 — Legit traffic allowed**  
- **Claim:** External or VM-sourced packets were permitted to the PC subnet.  
- **Evidence:** `CORE# show access-lists PROTECT-PC` → `permit any (5 matches)`  
- **Reasoning:** The ACL logic allowed valid packets while maintaining anti-spoofing protection.

**CER 3 — ACL order confirmed**  
- **Claim:** Moving a `deny` line above a `permit` changed outcomes as expected.  
- **Evidence:** Before/after `show access-lists` hit counters differ by sequence.  
- **Reasoning:** Confirms the “first-match wins” processing order of ACLs.

---

## Ops Lexicon — New This Week
- **Wildcard mask:** A binary pattern used to define which bits in an IP address are checked (0 = must match, 1 = ignore).  
- **Standard ACL:** Access control list that filters traffic by **source IP only**.  
- **Extended ACL:** ACL that filters by **source + destination + protocol + port**.  
- **Implicit deny:** The unseen final rule that drops all packets not previously permitted.  
- **Anti-spoofing:** Security technique using ACLs to block packets claiming to originate from internal subnets when they arrive from external links.  
- **Access-group:** Command used to bind an ACL to an interface in a specific direction (`in` or `out`).

---

## Reflect Questions

1) Why does the “Protect PC Network” ACL inspect **source** instead of **destination** addresses? 
2) What would happen if the ACL were applied **outbound** on EDGE instead of **inbound** on CORE?  
3) Show two `show access-lists` lines proving both a permit and a deny match occurred.  
4) What part of ACL behaviour confirms the “first-match wins” rule?  
5) Describe one improvement you made to your ACL order or mask logic during testing.

---

## Time & Confidence
- **Time spent (hh:mm):** ________  
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner
Paste your best ops verification line proving ACL logic worked
