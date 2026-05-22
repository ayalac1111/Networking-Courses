---
title: "W03 — Reflect: IPv6 Neighbour Discovery"
activity: reflect
deliverable: reflections/w03-reflect-{username}.md
points: 0
---

# Weekly Reflect — W03: Neighbour Discovery Protocol (NDP)

**Week:** W03  
**Date:** {date}  
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ipv6 interface`, `show ipv6 neighbors <if>`, `show logging | include ND`, `ip -6 addr`, `ip -6 route`, `ip -6 neigh`, `ping -6 …`). Avoid `show running-config`.

---

## Always True Rules

### Always Rule 1 — RAs drive host routes
**Rule (one line):**  
 **If** a router advertises prefix X:/64 on a link, **then** hosts form GUAs and a **default via the router’s LLA**.

**In my own words (1–2 sentences):**  
{write how RA → SLAAC + default via LLA; mention A/M/O flags and router lifetime}

**Proof lines (pick two, 1–2 lines each):**
- `{HOST}$ ip -6 route | grep default  → default via fe80::… dev …`
- `{EDGE}# show ipv6 interface g0/0/1 | include advertise|prefix|Joined`
- `{EDGE}# show logging | include (Received RS|Sending RA)` *(if available on your platform)*

**If this breaks next week, first move:**  
{run `{HOST}$ ip -6 route` and `{EDGE}# show ipv6 interface g0/0/1 | include advertise|lifetime` to confirm RA is present; check that Router Lifetime > 0}

---

### Always Rule 2 — NS/NA repopulate the neighbour cache (NUD)
**Rule (one line):**  
If I clear neighbours and send traffic to a new IPv6 target on‑link, **NS/NA** resolve L3→L2 and the entry moves **INCOMPLETE → REACHABLE**, later ageing to **STALE**.

**In my own words (1–2 sentences):**  
{explain the first‑ping effect and how NUD keeps mappings fresh}

**Proof lines (pick two, 1–2 lines each):**
- `{EDGE}# show ipv6 neighbors | include Gi0/0/1|INCOMPLETE|REACH|STALE`
- `{HOST}$ ip -6 neigh | head -n 3`
- `{EDGE}# show logging | include Neighbor (Solicitation|Advertisement)` *(if available)*

**If this breaks next week, first move:**  
{clear the table and generate known-good traffic; verify L2/port/VLAN and check solicited-node group membership}

---

### Always Rule 3 — DAD blocks on‑link duplicates
**Rule (one line):**  
If two nodes configure the **same IPv6 address** on the **same link**, **DAD** rejects the second instance and the address remains **tentative/duplicate**.

**In my own words (1–2 sentences):**  
{explain DAD probe (NS from :: to solicited-node multicast) and why the duplicate is refused}

**Proof lines (pick two, 1–2 lines each):**
- `{EDGE}# show ipv6 interface g0/0/1 | include Duplicate|tentative` *(router-side)*
- `{VM}$ ip -6 addr show dev eth0 | grep -E 'tentative|dadfailed'` *(host-side if the host loses)*

**If this breaks next week, first move:**  
{confirm both devices truly share the same L2 and /64; try the duplicate on the other node to see which side loses}

---

## CER Micro-Cards (copy the ones from the lab)

> **What is a CER micro-card?** A tiny 3‑liner to lock in a concept: **Claim → Evidence → Reasoning**.  


---

## Ops Lexicon — New This Week

**Router Advertisement (RA)**  
IPv6 control message sent by routers advertising on‑link prefixes, lifetimes, flags (`A/M/O`), MTU, hop-limit, and **router preference**. Hosts use it to form addresses and pick a **default via LLA**.

**Router Solicitation (RS)**  
Host’s request to prompt an immediate RA (instead of waiting the periodic interval).

**Neighbour Solicitation (NS)** / **Neighbour Advertisement (NA)**  
Pair used for address resolution and reachability. NS asks "who has this IPv6?", NA replies with the L2 address; also used by **NUD** to probe stale entries.

**NUD — Neighbour Unreachability Detection**  
Mechanism that maintains neighbour cache correctness: states typically **INCOMPLETE → REACHABLE → STALE → (DELAY/PROBE)**.

**DAD — Duplicate Address Detection**  
Special NS sent from `::` to the **solicited‑node multicast** of the tentative address; a detected conflict marks the address **tentative/duplicate (dadfailed)**.

**Solicited‑node multicast**  
Group `ff02::1:ffXX:XXXX` derived from the last 24 bits of an IPv6 address; NS targets this group.

**IID — Interface Identifier (EUI‑64)**  
Lower 64 bits of an address; EUI‑64 forms IID by inserting `ff:fe` into the MAC and flipping the U/L bit. Some hosts use stable or temporary IIDs instead of EUI‑64.

---

## Reflection Questions (answer briefly)

1. **RA to routes:** What did the **A/M/O** flags and **Router Lifetime** tell your host to do this week? Give one proof line that convinced you.  
   {2–3 sentences}

2. **NUD in practice:** Which **NUD** states did you actually observe, and what action moved them (e.g., first ping, idle timer)?  
   {2–3 sentences}

3. **DAD tripwire:** Describe one realistic way a duplicate address might appear in a lab or workplace. What is your **first check** and why?  
   {1–2 sentences}

4. **IID source:** Was your host’s IID **EUI‑64, stable, or temporary**? How did you tell?  
   {1–2 sentences + one proof line}

---

*File naming suggestion:* `reflections/w03-reflect-{username}.md`

