# W12 — Reflect: Extended ACLs & Subnet Policy

**Week:** W12\
**Date:** {date}\
**Student:** {your\_name}\
**Ops-only policy:** Use operational commands (device prompt + command). No `show running-config`. Packet Tracer friendly outputs.

---

## Always True Rules (write in your own words)

### Always Rule 1 — Extended ACLs filter source, destination, and services

**Rule (one line):** Extended ACLs match on **source IP**, **destination IP**, and optionally **protocols/ports**.

**In my words (1–2 sentences):**\
Unlike standard ACLs, extended ACLs let you block specific services between subnets — like HTTP to a web server or DNS from one host. This gives precise control of who can access what.

**Proof lines (pick two):**

- `# show access-lists ACL-VM`
- `# show ip interface Gi0/0/x | include access list`

**If this breaks next week, first move:**\
Make sure you’re using an extended ACL and that source/destination/IP/protocol all match correctly.

---

### Always Rule 2 — ACLs are most effective close to the source

**Rule (one line):** Placing ACLs **inbound on source-facing interfaces** saves router effort and bandwidth.

**Proof lines (pick two):**

- `# show access-lists ACL-VM`
- `# ping 192.0.2.80`

**If this breaks next week, first move:**\
Double-check which interface receives the traffic first and move the ACL there.

---

### Always Rule 3 — Implicit deny is always active

**Rule (one line):** ACLs end with an invisible `deny ip any any` unless overridden.

**Proof lines (pick two):**

- `# show access-lists ACL-VM` → denied packets with no explicit rule
- `# nslookup ns.cnap.cst` → no DNS response from blocked host

**If this breaks next week, first move:**\
Add a final `permit ip any any` to prevent accidental full blocking.

---

## Create Micro-Cards (CER)

> **Claim → Evidence → Reasoning**

**CER 1 — ICMP block from VM**

- **Claim:** VM cannot ping into the PC subnet.
- **Evidence:** `# show access-lists ACL-VM` → ICMP echo denied line hit
- **Reasoning:** ACL correctly stops VM subnet from reaching PC devices via ping.

**CER 2 — Service-based denial**

- **Claim:** VM was denied HTTP to the web server at `192.0.2.80`.
- **Evidence:** `# show access-lists ACL-VM` → `deny tcp ... eq 80` matched
- **Reasoning:** Extended ACLs filtered by protocol and port to control access.

**CER 3 — Specific host blocked in PC subnet**

- **Claim:** Host `198.18.U.129` was denied DNS service.
- **Evidence:** `# show access-lists ACL-PC` → `deny udp host ... eq 53`
- **Reasoning:** ACL enforced a policy targeting one device while allowing the subnet.

---

## Ops Lexicon — New This Week

- **Extended ACL:** ACL that filters traffic by source, destination, protocol, and port.
- **Named ACL:** An ACL with a custom name, supporting easier edits and review.
- **Interface inbound/outbound:** Direction ACLs inspect packets: entering (in) or exiting (out) an interface.
- **Service Filtering:** Blocking or permitting specific services like HTTP, DNS, SSH using port numbers.
- **Permit IP Any Any:** A catch-all ACE allowing all remaining traffic not previously denied.
- **Hit counter:** Shows how many times each ACL rule has matched traffic.

---

## Reflect Questions

1. Why does placing ACLs on inbound interfaces help filter more efficiently?
2. What was your process to test whether HTTP and DNS traffic were working or denied?
3. Provide output showing a denied packet from ACL-VM and a permitted one from ACL-PC.


4. If you were a network admin, how would you use extended ACLs to protect internal services?

---

## Time & Confidence

- **Time spent (hh**\*\*:mm\*\*\*\*):\*\* \_\_\_\_\_\_\_\_
- **Confidence (0–5):** \_\_\_\_\_\_\_\_ (0 = lost, 5 = I could teach it)

---

## Appendix — Your best one-liner

Paste your best ops verification line proving an extended ACL rule worked

