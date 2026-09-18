# W03 Reflect — SLAAC and NDP Evidence

**Week:** W03  
**Student:** `{your_name}`  
**Lab evidence file:** `l03-c0N-{username}.txt`

> **Ops-only evidence policy:** Use evidence from your Lab 03 outputs. Use command output such as `show ipv6 interface brief`, `show logging | include ...`, `show ipv6 neighbors`, `ip -6 addr show`, `ip -6 route`, and `ping -6`. Do not paste full running configurations.

---

## 1. NDP Evidence Observation Table

Use evidence from your own Lab 03 checkpoints. Copy one observed example of each process.

| Process observed | Checkpoint | Where it showed up | What the evidence proves |
|---|---|---|---|
|  | C02 | EDGE's log (RS/RA) |  |
|  | C03 | EDGE's log + `show ipv6 neighbors` (NS/NA, NUD state) |  |
|  | C04 | EDGE's log + `ip -6 addr show` on VM (DAD) |  |

Required boundary model:

```text
RA proves EDGE offered a prefix and itself as default router — not that VM will use it.
SLAAC proves VM built its own GUA from that prefix — not that the address is reachable elsewhere.
NS/NA proves L3-to-L2 resolution occurred; the NUD state (INCOMPLETE/REACHABLE/STALE) shows how
current that mapping is, not just that it once happened.
DAD proves the network refused a duplicate — it does not decide which device "owns" the address in
the abstract, only which one loses the collision.
```

---

## 2. Always Rules

### Always Rule 1 — An RA is what makes SLAAC possible, not the other way around

**Rule:**  
If EDGE's `Gi0/0/1` sends no Router Advertisement, VM never proceeds to SLAAC — no RA, no address, no default route, no matter what VM itself is configured to do.

**In my own words:**  
`{1–2 sentences}`

**Proof line from my lab:**  
Paste one line from your C02 evidence showing EDGE's RA (the `Sending solicited RA` or `prefix` line).

```text
{paste proof line}
```

**First move next time:**  
If a host comes up with no IPv6 address at all, I will first run:

```text
{command}
```

on `{EDGE or VM}` because:

```text
{reason}
```

---

### Always Rule 2 — NS/NA resolves an address; the NUD state says how much to trust it

**Rule:**  
A neighbour entry existing in `show ipv6 neighbors` proves NS/NA happened at some point. Only a `REACH` state proves the mapping is current — `STALE` means it hasn't been re-confirmed recently.

**In my own words:**  
`{1–2 sentences}`

**Proof line from my lab:**  
Paste one line from your C03 evidence showing a NUD state transition or a `REACH`/`STALE` entry.

```text
{paste proof line}
```

**First move next time:**  
If a neighbour entry looks wrong, I will first run:

```text
{command}
```

because:

```text
{reason}
```

---

### Always Rule 3 — DAD blocks the duplicate, not both addresses

**Rule:**  
When VM deliberately claimed EDGE's own address in C04, DAD didn't fail both copies — it failed the one added after the address was already in use.

**In my own words:**  
`{1–2 sentences}`

**Proof line from my lab:**  
Paste your C04 evidence line showing which side kept the address and which was marked duplicate/`dadfailed tentative`.

```text
{paste proof line}
```

**First move next time:**  
If I suspect a duplicate address, I will first run:

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

| Part | Meaning | What to write |
|---|---|---|
| **Claim** | What you believe is true | One clear technical statement |
| **Evidence** | What proves it | One command output line, ping result, or log line |
| **Reasoning** | Why the evidence supports the claim | One sentence connecting the evidence to the network behaviour |

A CER Micro-Card is not a paragraph reflection. It should be short enough to reuse later when troubleshooting.

Use evidence from your own Lab 03 outputs.

### CER #1 — RA drives SLAAC

```text
Claim: VM formed its GUA because EDGE's Gi0/0/1 sent a Router Advertisement carrying the LAN prefix.

Evidence:
{paste your C02 proof line}

Reasoning:
{one sentence explaining why the RA, not VM's own configuration, is what produced the address}
```

---

### CER #2 — NUD state, not just cache presence

```text
Claim: VM's neighbour entry reaching REACH proves EDGE re-confirmed it recently — not just that NS/NA
happened once, at some point in the past.

Evidence:
{paste your C03 proof line showing a NUD state}

Reasoning:
{one sentence explaining what would be different about a STALE entry instead}
```

---

### CER #3 — DAD picks a loser, not two failures

```text
Claim: DAD rejected VM's duplicate of EDGE's address because EDGE was already using it — not because
IPv6 forbids the address itself.

Evidence:
{paste your C04 proof line showing which device kept the address and which was marked duplicate}

Reasoning:
{one sentence explaining why the outcome was not symmetric}
```

---

## 4. Evidence Limits Statement

Complete the statements using your Lab 03 evidence.

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
This lab proves local IPv6 discovery evidence: an RA drove SLAAC, NS/NA populated the neighbour cache
and NUD states tracked how current it was, and DAD rejected a duplicate address.

This lab does not prove DNS, application reachability, firewall policy, or full end-to-end enterprise
routing.
```

---

## 5. Reflection Questions

Answer briefly. Use one command or log line in each answer.

### Q1 — RA to SLAAC

What in EDGE's C02 log proves VM's address came from that RA specifically, and not from some other source?

```text
Command/log line:
What it proves:
```

---

### Q2 — NUD state you actually saw

Which NUD state did you capture in C03 (`REACH`, `STALE`, or a transition between them), and what action produced it?

```text
Answer:
Evidence:
```

---

### Q3 — DAD outcome

In C04, which device's address survived and which was marked duplicate — and why that one, not the other?

```text
Answer:
Evidence:
```

---

### Q4 — Evidence boundary, in your own words

Name one thing this week's evidence proves and one thing it does not — without reusing the lab's own closing line.

```text
Proves:
Does not prove:
```

---

### Q5 — First troubleshooting command

If a future lab's host comes up with no IPv6 address at all, what command will you run first, and why does this week's evidence make that your first move?

```text
Command:
Why this command first:
```

---

## 6. Ops Lexicon — New This Week

Complete each term in your own words.

| Term | My operational definition | Lab evidence that showed it |
|---|---|---|
| RA (Router Advertisement) |  |  |
| RS (Router Solicitation) |  |  |
| NS / NA |  |  |
| NUD |  |  |
| DAD |  |  |
| Neighbour cache |  |  |
