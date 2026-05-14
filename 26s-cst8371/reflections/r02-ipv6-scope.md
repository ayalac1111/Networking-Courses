# W02 Reflect — IPv6 Address Scope Evidence

**Week:** W02  
**Student:** `{your_name}`  
**Lab evidence file:** `l02-<username>.txt`

> **Ops-only evidence policy:** Use evidence from your Lab 02 outputs. Use command output such as `show ipv6 interface brief`, `show ipv6 route static`, `show ipv6 route <prefix>`, `show ip ssh`, `show tcp brief`, and `ping`. Do not paste full running configurations.

---

## 1. IPv6 Scope Observation Table

Use addresses from your own Lab 02 evidence. Copy one observed example of each address type.

| Address observed | Device/interface | Type: LLA / ULA / GUA | Boundary type | What the evidence proves |
|---|---|---|---|---|
|  |  | LLA | Protocol scope boundary |  |
|  |  | ULA | Routing-policy / administrative boundary |  |
|  |  | GUA | Globally routable address scope |  |

Required boundary model:

```text
LLA boundary = protocol scope boundary.
ULA boundary = routing-policy / administrative boundary.
GUA boundary = globally routable address scope.
```

---

## 2. Always Rules

### Always Rule 1 — Classify the address before troubleshooting reachability

**Rule:**  
Before troubleshooting IPv6 reachability, identify whether the address is LLA, ULA, or GUA.

**In my own words:**  
`{1–2 sentences}`

**Proof line from my lab:**  
Paste one line from `show ipv6 interface brief` that shows an LLA, ULA, or GUA address.

```text
{paste proof line}
```

**First move next time:**  
When an IPv6 ping fails, I will first run:

```text
{command}
```

because:

```text
{reason}
```

---

### Always Rule 2 — ULA reachability depends on routing policy

**Rule:**  
ULA addresses can be routed internally when routes and policy allow it, but ULA prefixes are not intended to be carried across the public Internet.

**In my own words:**  
`{1–2 sentences}`

**Proof line from my lab:**  
Paste one line that proves ULA reachability was allowed or blocked in the lab.

```text
{paste proof line}
```

Acceptable evidence examples:
- Site A PC ping to Site B PC succeeds.
- Site A PC ping to External Server fails.
- `R2# show ipv6 route fd00:acad:U:a::/64` shows no route.
- Site C PC ping to Site D fails because of the boundary policy.

**First move next time:**  
To check whether a ULA destination should be reachable, I will run:

```text
{command}
```

because:

```text
{reason}
```

---

## 3. CER Micro-Cards

A **CER Micro-Card** is a short technical reasoning note.  
  
CER means:  

| Part | Meaning | What to write |  
|---|---|---|  
| **Claim** | What you believe is true | One clear technical statement |  
| **Evidence** | What proves it | One command output line, ping result, or route-table result |  
| **Reasoning** | Why the evidence supports the claim | One sentence connecting the evidence to the network behaviour |  
  
A CER Micro-Card is not a paragraph reflection. It should be short enough to reuse later when troubleshooting.  
  
Use this format:

A CER micro-card uses three lines:

```text
Claim:
Evidence:
Reasoning:
```

Example:

```plaintext
Claim: Site A and Site B can communicate using ULA because internal routing allows it.
Evidence: Site A PC> ping fd00:acad:100:b::10 -> replies received.
Reasoning: The successful ping proves the ULA destination is reachable inside the internal routing domain.
```

Use evidence from your own Lab 02 outputs.

### CER #1 — ULA internal communication

```text
Claim: Site A and Site B can communicate using ULA because routing/policy allows it inside the internal domain.

Evidence:
{paste one ping result or route evidence}

Reasoning:
{one sentence explaining why this proves internal ULA reachability, not Internet reachability}
```

---

### CER #2 — ULA boundary

```text
Claim: ULA addresses are not automatically blocked by IPv6 forwarding; their boundary is administrative/routing policy.

Evidence:
{paste one failed ping or route lookup showing the boundary}

Reasoning:
{one sentence explaining how routing policy or boundary policy controls the result}
```

---

### CER #3 — GUA external reachability

```text
Claim: Site D can reach the External Server because both use GUA addressing and the required GUA route exists.

Evidence:
{paste Site D ping to External Server or show ipv6 route static evidence}

Reasoning:
{one sentence explaining why this path is allowed}
```

---

## 4. Evidence Limits Statement

Complete the statements using your Lab 02 evidence.

```text
This lab proves:
1.
2.
3.

This lab does not prove:
1.
2.
```

Minimum expected content:

```text
This lab proves IPv6 interface identity and the operational scope of LLA, ULA, and GUA addresses.

This lab does not prove that every IPv6 address has Internet reachability. It also does not prove that all ULA networks can reach each other automatically.
```

---

## 5. Reflection Questions

Answer briefly. Use one command or test result in each answer.

### Q1 — Interface identity

Which command best proves IPv6 interface identity in this lab?

```text
Command:
What it proves:
What it does not prove:
```

---

### Q2 — ULA reachability

Why can Site A reach Site B, but Site A cannot reach the External Server?

```text
Answer:
Evidence:
```

---

### Q3 — Site C boundary

Why does Site C fail to reach Site D or the External Server?

```text
Answer:
Evidence:
```

---

### Q4 — GUA reachability

Why can Site D reach the External Server?

```text
Answer:
Evidence:
```

---

### Q5 — First troubleshooting command

Next time an IPv6 ping fails, what command will you run first?

```text
Command:
Why this command first:
```

---

## 6. Ops Lexicon — New This Week

Complete each term in your own words.

| Term | My operational definition | Lab evidence that showed it |
|---|---|---|
| LLA |  |  |
| ULA |  |  |
| GUA |  |  |
| IPv6 scope |  |  |
| Routing policy |  |  |
| Evidence limit |  |  |
