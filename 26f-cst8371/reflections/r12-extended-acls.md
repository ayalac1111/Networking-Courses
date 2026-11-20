# Weekly Reflect — W12: Extended ACLs — Precise Traffic Control

**Week:** W12
**Date:** {date}
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip access-lists`, `show ip interface <if> | include access list`, `ping`, `curl`, `tftp`, `ssh`). Avoid `show running-config` or static screenshots.

---

## Always True Rule

### Placement follows what the policy needs to see, not just "closer is better"

**Rule (one line):**
An extended ACL goes as close as possible to whichever side — source or destination — actually lets it enforce the intended policy without side effects; "closer to the source" is the common case, not a fixed rule.

**In my own words (1–2 sentences):**
Because extended ACLs match on source, destination, protocol, and port together, placing them near the source usually stops unwanted traffic earliest and cheapest — that's what Lab 12's `POLICY-VM` and `POLICY-PC` do. But when the ACL's job is to protect the device itself from the outside world, like `POLICY-EXTERNAL` on EDGE, "the source" is the entire Internet — so the only placement that makes sense is inbound on the interface facing that source, right at the destination being protected.

**Proof lines (pick two, from Lab 12):**
- `show ip access-lists POLICY-VM` (or `POLICY-PC`, `POLICY-EXTERNAL`) — non-zero hits on the intended deny lines
- `show ip interface <interface> | include access list` — confirms which interface and direction the policy is actually bound to

**If this breaks next week, first move:**
Before assuming the ACL logic is wrong, check `show ip interface <if> | include access list` first — a policy with perfectly correct match statements does nothing if it's bound to the wrong interface or the wrong direction.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**

**CER 1 — Source-side placement stops traffic before it costs anything**
- **Claim:** `POLICY-VM` and `POLICY-PC` are placed inbound on CORE's subnet-facing interfaces, not on EDGE near the Internet.
- **Evidence:** `show ip access-lists POLICY-VM` shows deny hits on CORE's `Gi0/0/2`; the denied traffic never appears in EDGE's NAT translation table at all.
- **Reasoning:** Denying traffic at the interface closest to where it originates means it never crosses the transit link, never reaches NAT, and never wastes bandwidth or translation-table space on a flow that was always going to be blocked.

**CER 2 — Destination-side placement, when the ACL protects the destination itself**
- **Claim:** `POLICY-EXTERNAL` is placed inbound on EDGE's public interface, not "near the source" (which would mean everywhere on the Internet).
- **Evidence:** `show ip access-lists POLICY-EXTERNAL` shows deny hits for inbound SSH and inbound ICMP echo *requests*, while outbound-initiated traffic (including echo *replies*) still succeeds.
- **Reasoning:** When there is no single, closer source interface to filter at — because the source is the entire outside world — the only placement that protects the device is inbound on the interface facing that world, right at the destination being defended.

---

## Reflect Questions

1. Lab 12 places `POLICY-VM` and `POLICY-PC` inbound on CORE, close to the source, but `POLICY-EXTERNAL` inbound on EDGE, close to the destination. Using the "what does the policy need to see" rule, explain why the placement changes for `POLICY-EXTERNAL`.
2. Give one piece of evidence that proves `POLICY-EXTERNAL` blocks inbound ping *requests* without blocking the replies your own outbound pings depend on. Which ICMP keyword makes that distinction possible?
3. For one of your three named policies, give the hit-counter evidence that proves the permit-all catch-all line is actually being reached — not just that the earlier deny lines exist.
4. Standard ACLs are placed close to the destination to avoid collateral blocking (the standard ACL placement rule). Explain, in one or two sentences, why that same reasoning does not apply to `POLICY-VM`/`POLICY-PC` in this lab.

---

## Time & Confidence
- **Time spent (hh:mm):** ________
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner
Paste the pair of proof lines (ACL hit-counter evidence + the interface/direction it's bound to) that best demonstrates the Always True Rule above:
```
CORE# show ip access-lists POLICY-VM
Extended IP access list POLICY-VM
    10 deny tcp 192.168.U.64 0.0.0.31 host 192.0.2.80 eq 80 (n matches)
    ...
    50 deny icmp 192.168.U.64 0.0.0.31 192.168.U.0 0.0.0.63 echo log (n matches)
    60 permit ip any any (n matches)

CORE# show ip interface GigabitEthernet0/0/2 | include access list
  Inbound  access list is POLICY-VM
```
