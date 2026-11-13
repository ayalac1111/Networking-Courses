# Weekly Reflect — W11: NAT Operation and Troubleshooting — Proving Translation

**Week:** W11
**Date:** {date}
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip nat translations`, `show ip nat statistics`, `ping`, `traceroute`). Avoid `show running-config` or static screenshots.

---

## Always True Rule

### NAT success and end-to-end success are different questions
**Rule (one line):**
Proving that NAT translated a packet does **not** prove the connection worked — and proving a connection failed does not by itself mean NAT is the cause.

**In my own words (1–2 sentences):**
A translation entry in `show ip nat translations` tells you the packet crossed the NAT boundary and got a new address — nothing more. The rest of the path (routing, ACLs beyond the NAT rule, the destination itself) still has to hold for the connection to actually succeed.

**Proof lines (pick two, from your Challenge 03 trouble tickets):**
- `show ip nat translations` (entry present)
- `ping` or `traceroute` result to the same destination, run immediately after

**If this breaks next week, first move:**
Re-run the 5-step checklist from the lecture (traffic generated → selected → resource available → translating → return traffic allowed) — find the first step that fails, not the first thing that looks unusual.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**

**CER 1 — Translation proven, connection still fails**
- **Claim:** NAT translated the traffic correctly, but the connection did not complete.
- **Evidence:** `show ip nat translations` shows a matching entry for the attempt; the follow-up `ping`/connection test still times out.
- **Reasoning:** A present translation entry only proves the packet was selected and translated — it says nothing about routing beyond the NAT device, ACLs elsewhere in the path, or whether the destination itself is reachable. The fault lies past NAT, not inside it.

**CER 2 — NAT is the actual fault**
- **Claim:** The connection failure is caused by NAT itself, not something downstream.
- **Evidence:** `show ip nat translations` shows no entry for the attempt; `show access-list <n>` shows 0 hits on the ACL that should have selected the traffic.
- **Reasoning:** Zero ACL hits means the traffic was never even offered to the translation engine — the fault is upstream of translation (selection), so nothing downstream (routing, destination) can be ruled in or out yet.

---

## Reflect Questions

1. From one of your Challenge 03 trouble tickets, give one command output that proved translation *did* happen, and one separate piece of evidence that proved the connection still failed. Explain why the first does not contradict the second.
2. For a ticket where the ACL hit count stayed at 0, which of the 5 troubleshooting-checklist steps failed first, and how does the hit count prove that specifically (rather than a later step)?
3. Give the smallest evidence set (name the exact commands) that would let you rule NAT itself *out* as the cause of a failed connection, without checking every rule from scratch.
4. In C03, the pool existed and was correctly referenced, yet the connection failed. Which checklist step catches this, and why doesn't checking "does the pool exist" catch it?

---

## Time & Confidence
- **Time spent (hh:mm):** ________
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner
Paste the pair of proof lines (translation evidence + connection test) that best demonstrates the Always True Rule above:
```
{username}-EDGE# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
tcp 203.0.113.17:1045  10.10.U.12:1045    192.0.2.69:22      192.0.2.69:22

{username}-CORE# ping 192.0.2.69 source Loopback{U}
Sending 5, 100-byte ICMP Echos to 192.0.2.69, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```
