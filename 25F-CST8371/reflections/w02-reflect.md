# Weekly Reflect — W02 IPv6 Routing & Link-Local Next-Hops

**Week:** W02  
**Date:** {date}  
**Student:** {your_name}

> **Ops-only evidence policy:** Use operational commands (e.g., `show ipv6 route`, `show ipv6 neighbors <if>`, `ping ipv6 …`, `traceroute …`). Avoid `show running-config`.

---

## Always True Rules

### Always Rule 1
**Rule (one line):**  
If the next hop is a **link-local (fe80::/10)** address, then the static route **must** include the **exit interface + that LLA** (LLA alone is ambiguous).

**In my own words (1–2 sentences):**  
Link-locals are only unique on a link. IOS needs the interface to know *which* neighbour you mean; otherwise, the route won’t install/forward correctly.

**Proof lines (pick two, 1–2 lines each):**
- `R1# show ipv6 route static | include 2001:db8:192` → `… via FE80::12, GigabitEthernet0/0/0`
- `R1# show ipv6 neighbors g0/0/0` → `FE80::12 … GigabitEthernet0/0/0`
- `R1# ping ipv6 fe80::12 source g0/0/0` → `Reply from FE80::12`

**If this breaks next week, first move:**  
Run `show ipv6 neighbors g0/0/0` on **R1** to confirm the next-hop LLA is on the same link you referenced in the route.

---

### Always Rule 2
**Rule (one line):**  
If `traceroute` reaches a router but traffic doesn’t pass through it—and links/neighbours look good—**verify IPv6 forwarding** is enabled (`ipv6 unicast-routing`).

**In my own words (1–2 sentences):**  
Interfaces can be up and pingable, but without the global forwarding knob, the router won’t route transit traffic. It’s the first sanity check when the path dies at a router.

**Proof lines (pick two, 1–2 lines each):**
- `R2# show ipv6 protocols` → `IPv6 routing enabled` *(or “not enabled” if broken)*
- `PC> tracert 2001:db8:192::69` → shows the hop where forwarding stops
- `R2# show ipv6 route` → confirms routes exist but forwarding was off

**If this breaks next week, first move:**  
Run `show ipv6 protocols` on the suspected router; if disabled, enable forwarding and retest the path.

> **Note:** Although we’re studying IPv6 here, this rule applies to **IPv4** as well—especially on **Layer-3 switches**. If `traceroute` dies at a switch, confirm **`ip routing`** is enabled and routes/VLAN SVIs are present (`show ip route`, `show ip interface brief`).

---

## CER Micro-Cards (filled examples for W02)

**What is a CER micro-card?**  
A tiny 3-liner to lock in a concept: **Claim → Evidence → Reasoning**.

### CER #1 — LLA next-hop + exit IF
```perl
Claim: A static route with a link-local next hop must include the matching exit interface, or traffic will black-hole.  
Evidence: R1# show ipv6 route static | inc 2001:db8:192 -> ... via FE80::12, Gi0/0/0  
R1# show ipv6 neighbors Gi0/0/0 -> FE80::12 ... on Gi0/0/0  
Reasoning: The next-hop LLA lives on Gi0/0/0; tying the route to the same link makes the next hop unambiguous and forwardable.
```

### CER #2 — IPv6 forwarding knob
```perl
Claim: When traceroute stops at a router, the first check is whether IPv6 forwarding is enabled.  
Evidence: R2# show ipv6 protocols -> IPv6 Routing Protocol is not enabled  
PC> tracert 2001:db8:192::69 -> stops at R2  
Reasoning: Interfaces and neighbours can be healthy, but without global forwarding, the device will not route transit traffic; enabling it restores the path.
```


---

## Ops Lexicon — New This Week

**knob** *(ops slang)*  
A configuration **toggle/flag** you turn on/off to change device behaviour.  
- Examples:  
  - `ipv6 unicast-routing` — the **IPv6 forwarding knob** (enables routing of IPv6 through the box)  
  - `ip routing` — the **IPv4 forwarding knob** (common on L3 switches)  
- Why it matters: Without the knob, interfaces can be **up/up** and pingable, but the device behaves like a **host**, not a **router**.

**IP forwarding** *(a.k.a. Layer-3 forwarding / routing)*  
The process of receiving a packet on one interface and **forwarding** it out another based on the routing table.  
- IPv6 enable/verify: `ipv6 unicast-routing`; quick proof: `show ipv6 protocols` (should say “IPv6 routing enabled”) or a `traceroute` that goes **through** the device.  
- IPv4 analogue: `ip routing` on L3 switches; quick proof: `traceroute` passes through and `show ip route` contains non-connected routes.


---

## Reflection Questions (answer briefly)

1. **Key insight:** What idea from this week will most change how you troubleshoot IPv6 next time?  
   {2–3 sentences}

2. **LLA ambiguity (deep dive):** Why is a **link-local address (fe80::/10)** considered *ambiguous* when used as a next hop?  
   - Explain the **scope/uniqueness boundary** and why the router needs the **exit interface**.  
   - Name a concrete scenario where LLA-only breaks (e.g., multi-access link or multiple interfaces with neighbours using fe80::1).  
   - Include one **ops proof line** you’d use to tie an LLA to the correct link.

3. **Tripwire & prevention:** What mistake or failure pattern will you watch for, and what’s your first prevention step?  
   {1–2 sentences}

4. **Proof discipline:** Which two commands will you run *first* to prove link health and next-hop correctness? Why those two?  
   - {cmd_1} — {why}  
   - {cmd_2} — {why}

5. **Transfer:** Where else (another lab or real gear) would these Always Rules apply without changes (include IPv4/L3 switch context)?  
   {1–2 sentences}

---

*File naming suggestion:* `reflections/w02-reflect-{username}.md`



