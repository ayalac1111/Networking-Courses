# Weekly Reflect — W11: NAT Foundations — Translating Enterprise Traffic

**Week:** W11
**Date:** {date}
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ip nat translations`, `show ip nat statistics`, `show ip nat interfaces`). Avoid `show running-config` or static screenshots.

---

## Always True Rules

### Always Rule 1 — NAT always translates at the boundary
**Rule (one line):**
NAT performs address translation **only on packets crossing between inside and outside** interfaces.

**In my own words (1–2 sentences):**
Packets moving within the same side of the network are never translated; only traffic that crosses the NAT boundary is eligible. This explains why `show ip nat translations` only lists flows that traverse inside ↔ outside.

**Proof lines (pick two):**
- `R1# show ip nat statistics`
- `R1# show ip nat translations`

**If this breaks next week, first move:**
Verify the interface roles with `show ip nat interfaces`.

---

### Always Rule 2 — Inside local and inside global are two views of the same address
**Rule (one line):**
Each translation entry shows an **inside-local → inside-global** mapping representing the private-to-public conversion of the same device.

**In my own words (1–2 sentences):**
Inside local and inside global are not two different devices — they're the same device's address, viewed before and after translation. Confusing the two is the most common NAT mistake.

**Proof lines (pick two):**
- `R1# show ip nat translations | include Inside`
- `PC1> ping 8.8.8.8` followed by `R1# show ip nat translations`

**If this breaks next week, first move:**
Check the ACL and NAT pool used for translation; confirm packets match the permit statement.

---

### Always Rule 3 — NAT order matters (ACL → pool → overload)
**Rule (one line):**
NAT processes packets in the order: **ACL match → pool selection → translation/overload.**

**In my own words (1–2 sentences):**
Before any address gets translated, the ACL has to select the traffic; only then does the router pick an address from the pool (or overload one) to do the translation. If any step in that order fails, no translation appears.

**Proof lines (pick two):**
- `R1# show ip nat statistics` (displaying pool usage and hits)
- `R1# show ip nat translations verbose`

**If this breaks next week, first move:**
Check for ACL mismatch or pool exhaustion — most common cause of missing translations.

---

## Common NAT Misunderstanding
> 🚫 **Don't configure both inside and outside on the same interface.**
> NAT requires one boundary direction per interface — traffic must cross from *inside* to *outside* to be translated.

---

## Create Micro-Cards (CER)
> **Claim → Evidence → Reasoning**

**CER 1 — Inside traffic successfully translated**
- **Claim:** NAT successfully translated inside traffic to a public address.
- **Evidence:** `R1# show ip nat translations` → `10.10.U.10 → 198.51.100.10`
- **Reasoning:** Private inside addresses were mapped to public globals for Internet reachability.

**CER 2 — Dynamic pool exhausted**
- **Claim:** The dynamic pool exhausted after reaching maximum sessions.
- **Evidence:** `R1# show ip nat statistics` shows pool "exhausted."
- **Reasoning:** Translation failed because all available public IPs in the pool were already in use.

**CER 3 — PAT enables port-based sharing**
- **Claim:** PAT (Overload) enabled multiple inside hosts to share one public address.
- **Evidence:** `R1# show ip nat translations` → multiple inside locals sharing one global address, distinguished by port.
- **Reasoning:** PAT adds port numbers to distinguish unique flows sharing the same inside-global address.

---

## Ops Lexicon — New This Week

| Term | Definition |
|:-----|:------------|
| **Inside local** | Private address seen inside the network (before translation). |
| **Inside global** | Public address used to represent an inside device to the outside. |
| **Outside local** | External host address as seen from inside (after translation). |
| **PAT (Port Address Translation)** | Also called NAT overload — many inside hosts share one public IP by using unique port numbers. |
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

---

## Time & Confidence
- **Time spent (hh:mm):** ________
- **Confidence (0–5):** ________ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner
Paste your best ops verification line proving NAT works:
```
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 198.51.100.10:1045 10.10.U.10:1045   172.16.0.10:80    172.16.0.10:80
```
