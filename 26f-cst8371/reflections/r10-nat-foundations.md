# Weekly Reflect — W10: NAT Foundations — Translating Enterprise Traffic

**Week:** W10
**Date:** {date}
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip nat translations`, `show ip nat statistics`). Avoid `show running-config` or static screenshots.

---

## Always True Rules

### Always Rule 1 — NAT always translates at the boundary
**Rule (one line):**
NAT performs address translation **only on packets crossing between inside and outside** interfaces.

**In my own words (1–2 sentences):**
Packets moving within the same side of the network are never translated; only traffic that crosses the NAT boundary is eligible. This explains why `show ip nat translations` only lists traffic that crosses inside ↔ outside (plus any static mappings you configured).

**Proof lines (pick two):**
- `RA# show ip nat statistics`
- `RA# show ip nat translations`

**If this breaks next week, first move:**
Verify the interface roles: the Inside and Outside interface lists in `show ip nat statistics`.

---

### Always Rule 2 — Inside local and inside global are two views of the same address
**Rule (one line):**
Each translation entry shows an **inside-local → inside-global** mapping representing the private-to-public conversion of the same device.

**In my own words (1–2 sentences):**
Inside local and inside global are not two different devices — they're the same device's address, viewed before and after translation. Confusing the two is the most common NAT mistake.

**Proof lines (pick two):**
- `RA# show ip nat translations | include <inside local address>`
- `VM:~$ ping -c 4 192.0.2.69` followed by `RA# show ip nat translations`

**If this breaks next week, first move:**
Re-read the Inside local and Inside global columns in `show ip nat translations` before deciding which address belongs to which side.

---

### Always Rule 3 — NAT order matters (ACL → pool → overload)
**Rule (one line):**
NAT processes packets in the order: **ACL match → pool selection → translation/overload.**

**In my own words (1–2 sentences):**
Before any address gets translated, the ACL has to select the traffic; only then does the router pick an address from the pool (or overload one) to do the translation. If any step in that order fails, no translation appears.

**Proof lines (pick two):**
- `RA# show ip nat statistics` (displaying pool usage and hits)
- `RA# show ip nat translations verbose`

**If this breaks next week, first move:**
Check that the ACL matches the inside source addresses, then that the pool or interface named in the rule is the one you intended (`show ip nat statistics`).

### Always Rule 4 — NAT is stateful: the table is the evidence
**Rule (one line):**
NAT creates a translation entry when the first matching packet crosses the boundary and removes it on timeout or `clear`; the translation table, not the configuration, shows what NAT is doing now.

**In my own words (1–2 sentences):**
The configuration only says what *can* be translated. A dynamic or PAT entry exists only after traffic has crossed, and it disappears when it ages out or is cleared. A static mapping, such as a port-forward, is in the table from the start. So you generate traffic first, then read the table.

**Proof lines (pick two):**
- `RA# clear ip nat translation *` followed by `RA# show ip nat translations` (only static entries remain)
- `RA# show ip nat statistics` (total translations, hits and misses change after new traffic)
- `RA# show ip nat translations` right after generating traffic from the inside host

**If this breaks next week, first move:**
Generate fresh traffic from the inside host, then read `show ip nat translations` immediately. An empty table after a `clear` is expected until traffic crosses again.

---

## Common NAT Misunderstanding
> **Don't configure both inside and outside on the same interface.**
> NAT requires one boundary direction per interface — traffic must cross from *inside* to *outside* to be translated.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**

**CER 1 — Inside traffic successfully translated**
- **Claim:** NAT successfully translated inside traffic to a public address.
- **Evidence:** `RA# show ip nat translations` → `172.16.9.46 → 203.0.113.U`
- **Reasoning:** Private inside addresses were mapped to public globals for Internet reachability.

**CER 2 — Dynamic pool exhausted**
- **Claim:** The dynamic pool ran out of free addresses.
- **Evidence:** `RA# show ip nat statistics` shows the pool's allocated count at its total and the misses counter increasing.
- **Reasoning:** Translation failed because all available public IPs in the pool were already in use.

**CER 3 — PAT enables port-based sharing**
- **Claim:** PAT (Overload) let multiple flows share one public address.
- **Evidence:** `RA# show ip nat translations` → multiple entries sharing one inside global address, distinguished by port.
- **Reasoning:** PAT adds port numbers to distinguish unique flows sharing the same inside-global address.

**CER 4 — Static port-forward gives the outside a way in**
- **Claim:** An outside host can reach an inside server through a static port-forward.
- **Evidence:** `RA# show ip nat translations` → the static entry, `tcp {public address:port} → {inside server address:port}`, present before any traffic.
- **Reasoning:** The mapping was configured explicitly, so it exists without inside traffic and lets the outside initiate the connection.

---

## Ops Lexicon — New This Week

| Term | Definition |
|:-----|:------------|
| **Inside local** | Private address seen inside the network (before translation). |
| **Inside global** | Public address used to represent an inside device to the outside. |
| **Outside local** | Address of an outside host as seen from the inside network. In this lab it matches the outside global, because NAT here translates only the inside side. |
| **Outside global** | Address of an outside host as seen from the outside network. |
| **PAT (Port Address Translation)** | Also called NAT overload — many inside hosts share one public IP by using unique port numbers. |
| **Static NAT / port-forward** | A fixed mapping from a public address and port to an inside address and port. It is in the table before any traffic and stays until removed. |
| **NAT pool** | Configured range of public addresses used for dynamic translation. |
| **ACL for NAT** | Defines which traffic is eligible for translation based on source criteria. |
| **show ip nat translations/statistics** | Verification commands to confirm translation entries and pool usage. |

---

## Reflect Questions

1. How does NAT help conserve IPv4 address space while still enabling global access?
2. What is the difference between **inside local** and **inside global** addresses?
3. Show a proof line where PAT created multiple translations for one public IP.
4. Explain how pool exhaustion affects new sessions and what `show ip nat statistics` reveals.
5. List two symptoms that indicate a missing NAT ACL or mis-assigned interface roles.
6. Why does a static port-forward entry appear in `show ip nat translations` before any traffic, while a PAT entry does not?

---

## Time & Confidence
- **Time spent (hh:mm):** ________
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner
Paste your best ops verification line proving NAT works:
```
RA# show ip nat translations
Pro  Inside global           Inside local            Outside local           Outside global
{pro} {inside global:port}   {inside local:port}     {outside local:port}    {outside global:port}
```
