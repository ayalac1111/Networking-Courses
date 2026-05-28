title W04 — Reflect: IPv6 Route Selection and Backup Path Evidence
activity reflect
deliverable reflections/w04-reflect-{username}.md
points 0

# Weekly Reflect — W04: IPv6 Route Selection and Backup Path Evidence

Week: W04  
Date: {date}  
Student: {your_name}  

> Ops-only evidence policy: Use operational commands first: `show ipv6 protocols`, `show ipv6 route`, `show ipv6 route <destination>`, `tracert`, `traceroute6`, `tracepath6`, and host route displays. Use `show running-config | include ipv6 route` only when you need to prove that an inactive floating route is configured.

---

## Submission Contract

| Item | Requirement |
|---|---|
| Action | Answer each prompt using evidence from your W04 lab output. |
| Evidence | Include at least one exact command/output proof line for each reflection question. |
| Success Indicator | Your answer identifies the selected route, next hop, route purpose, and what the evidence proves. |
| Failure Signal | Generic answers, missing command output, or claims that do not match route-table evidence. |

---

## Always True Rules

### Always Rule 1 — A route lookup proves the selected route

Rule (one line):  
If `show ipv6 route <destination>` returns a route, the device has selected a forwarding decision for that destination.

In my own words (1–2 sentences):  
{write how a route lookup identifies the prefix match, next hop, and selected path}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ipv6 route 2001:db8:192::69 → ...`
- `{RU}# show ipv6 route 2010:acad:U:a::10 → ...`
- `{RU}# show ipv6 route 2010:acad:U:b::20 → ...`

If this breaks next week, first move:  
{run `show ipv6 route <destination>` on the device making the forwarding decision before changing configuration}

---

### Always Rule 2 — Forward path and return path are different proofs

Rule (one line):  
If PC-A or Alpine reaches the TFTP server, the forward path must leave through CORE/RU and the return path must come back through RU toward the student `/48`.

In my own words (1–2 sentences):  
{write why CORE default route evidence proves the forward path, while RU summary-route evidence proves the return path}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ipv6 route 2001:db8:192::69 → selected default route toward RU`
- `{RU}# show ipv6 route 2010:acad:U:a::10 → summary route back to LAN-A`
- `{RU}# show ipv6 route 2010:acad:U:b::20 → summary route back to SITE-B`
- `{PC-A}> tracert 2001:db8:192::69 → path reaches TFTP`
- `{Alpine}$ traceroute6 2001:db8:192::69 → path reaches TFTP`

If this breaks next week, first move:  
{check the route lookup in both directions before changing addresses or routes}

---

### Always Rule 3 — Floating routes are backup routes because of administrative distance

Rule (one line):  
If two static routes match the same destination, the lower administrative distance route is selected and the higher administrative distance route waits as the floating backup.

In my own words (1–2 sentences):  
{write how the PRIMARY route wins before failure and the SECONDARY route appears after PRIMARY is removed}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ipv6 route 2001:db8:192::69 → PRIMARY selected before failure`
- `{CORE}# show ipv6 route 2001:db8:192::69 → SECONDARY selected after PRIMARY shutdown`
- `{RU}# show ipv6 route 2010:acad:U:a::10 → SECONDARY return path after failure`
- `{RU}# show ipv6 route 2010:acad:U:b::20 → SECONDARY return path after failure`

If this breaks next week, first move:  
{confirm whether the preferred route is still present; then check the floating route administrative distance}

---

### Always Rule 4 — Link-local next hops require the outgoing interface

Rule (one line):  
If an IPv6 static route uses a link-local next hop, the route must include the outgoing interface because the same link-local address can exist on more than one link.

In my own words (1–2 sentences):  
{write how you identified the neighbour link-local address on the SECONDARY link and why the interface was required}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ipv6 route 2001:db8:192::69 → SECONDARY route shows link-local next hop plus interface`
- `{RU}# show ipv6 route 2010:acad:U:a::10 → SECONDARY route shows link-local next hop plus interface`
- `{RU}# show ipv6 route 2010:acad:U:b::20 → SECONDARY route shows link-local next hop plus interface`
- `{DEVICE}# show ipv6 neighbors → neighbour LLA appears on the SECONDARY interface`

If this breaks next week, first move:  
{confirm the next hop is on the correct local link and that the static route includes the outgoing interface}

---

## CER Micro-Cards

> What is a CER micro-card? A tiny 3-line check: Claim → Evidence → Reasoning.

### CER 1 — Selected route

Claim:  
{the device selected this route for the destination}

Evidence:  
`{DEVICE}# show ipv6 route <destination> → {paste one exact line}`

Reasoning:  
{explain why the prefix match and next hop prove the forwarding decision}

---

### CER 2 — Return path

Claim:  
{RU has a valid return path to the student network}

Evidence:  
`RU# show ipv6 route 2010:acad:U:a::10 → {paste one exact line}`  
or  
`RU# show ipv6 route 2010:acad:U:b::20 → {paste one exact line}`

Reasoning:  
{explain why the `/48` summary route covers the LAN-A and SITE-B prefixes}

---

### CER 3 — Floating route activation

Claim:  
{the SECONDARY path became active after the PRIMARY path was removed}

Evidence:  
`CORE# show ipv6 route 2001:db8:192::69 → {paste one exact line}`

Reasoning:  
{explain how the selected next hop/interface changed after PRIMARY shutdown}

---

## Ops Lexicon — New This Week

Default route  
Route to `::/0`. Used when no more specific route matches the destination.

Summary route  
A route that represents multiple more-specific networks using one larger prefix.

Floating static route  
A backup static route with a higher administrative distance than the preferred route.

Administrative distance  
A preference value used to choose between routes to the same destination. Lower is preferred.

Recursive static route  
A static route that names only the next-hop address. The device must resolve that next hop through another connected route.

Fully specified static route  
A static route that includes both the outgoing interface and the next-hop address.

Forward path  
The path traffic takes from the source toward the destination.

Return path  
The path reply traffic takes back to the original source.

Route lookup  
A command/output check that shows which route the device selected for a destination.

---

## Reflection Questions

1. Route lookup: Which command best proved the selected route from CORE to the TFTP server? Paste one proof line and identify the selected next hop.  
{2–3 sentences + one proof line}

2. Forward path vs return path: Which command proved the forward path, and which command proved the return path for either PC-A or Alpine?  
{2–3 sentences + two proof lines}

3. Floating route: What changed in the route table after the PRIMARY path was shut down? Identify the command, selected route, and backup path evidence.  
{2–3 sentences + one proof line}

4. Link-local next hop: How did you prove that the floating route used a link-local next hop and the correct outgoing interface?  
{2–3 sentences + one proof line}

5. Evidence boundary: What did your traceroute prove, and what did it not prove?  
{1–2 sentences + one proof line}

---

File naming suggestion: `reflections/w04-reflect-{username}.md`

