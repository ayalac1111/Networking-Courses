title W04 — Reflect: IPv6 Reachability Troubleshooting Methodology and Evidence
activity reflect
deliverable reflections/w04-reflect-{username}.md
points 0

# Weekly Reflect — W04: IPv6 Reachability Troubleshooting Methodology and Evidence

Week: W04  
Date: {date}  
Student: {your_name}  

> Ops-only evidence policy: Use operational commands first: `show ipv6 route <destination>`, `show ipv6 interface brief | exclude una|down`, `show ipv6 neighbors`, `ping`, `tracert`, `traceroute6`. Use `show running-config` only when the ticket explicitly says to change configuration, never as evidence.

---

## Submission Contract

| Item | Requirement |
|---|---|
| Action | Answer each prompt using evidence from your Challenge 01 checkpoint files. |
| Evidence | Include at least one exact command/output proof line for each reflection question. |
| Success Indicator | Your answer names the methodology step, the evidence that supported it, and what that evidence proved. |
| Failure Signal | Generic answers, missing command output, or claims not matched by before/after evidence. |

---

## Always True Rules

### Always Rule 1 — A symptom must be captured before anything changes

Rule (one line):  
If you have not run and saved the ticket's test before touching the configuration, you have no baseline to prove the fault existed or that your repair fixed it.

In my own words (1–2 sentences):  
{write why CONFIRM comes before every other step, and what "before" evidence has to show}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ipv6 route 2001:db8:192::69 → {before-repair line}`
- `{PC-A}> tracert -6 <server> → {before-repair failure}`

If this breaks next week, first move:  
{run and save the exact test the ticket specifies before making any change}

---

### Always Rule 2 — The design tells you what "correct" looks like before you go looking for the fault

Rule (one line):  
If you have not identified the intended next hop, outgoing link, and PRIMARY/SECONDARY path from the topology and addressing table, you cannot recognize a wrong route when you see one.

In my own words (1–2 sentences):  
{write how ESTABLISH EXPECTATIONS turned the topology/addressing table into a specific expected route or next hop}

Proof lines (pick one, 1–2 lines each):
- `{expected next hop / outgoing interface from B4, before checking live output}`
- `{CORE}# show ipv6 route 2001:db8:192::69 → {compare against expectation}`

If this breaks next week, first move:  
{write down the expected path from the design before opening a device}

---

### Always Rule 3 — Forward path and return path are separate proofs, and separate faults

Rule (one line):  
If a ping fails, the forward route, the return route, or both could be at fault; only a route lookup on each direction's device tells you which.

In my own words (1–2 sentences):  
{write how you used the test's source address to decide whether a failure pointed at the forward path or the return path}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ipv6 route 2001:db8:192::69 → forward-path evidence`
- `{RU}# show ipv6 route <student prefix> → return-path evidence`
- `{PC-A}> tracert -6 <server> → path stops at {hop}`

If this breaks next week, first move:  
{run the route lookup on the device making the forwarding decision in each direction before assuming the fault is symmetric}

---

### Always Rule 4 — A repair is only proven by matched before/after evidence

Rule (one line):  
If the after-repair test does not use the same source, destination, and test conditions as the before-repair test, it does not prove the repair worked.

In my own words (1–2 sentences):  
{write what REPAIR and VERIFY each required, and why re-running the identical test matters more than the fix looking correct}

Proof lines (pick two, 1–2 lines each):
- `{CORE}# show ipv6 route 2001:db8:192::69 → {after-repair line, same destination as before}`
- `{PC-A}> tracert -6 <server> → {after-repair success, same source/destination as before}`

If this breaks next week, first move:  
{re-run the exact CONFIRM test again and compare it line-by-line against the before-repair evidence}

---

## CER Micro-Cards

> What is a CER micro-card? A tiny 3-line check: Claim → Evidence → Reasoning.

### CER 1 — Confirmed symptom

Claim:  
{the fault was present before any configuration change}

Evidence:  
`{DEVICE}# {command} → {paste one exact before-repair line}`

Reasoning:  
{explain what this line proves was broken and how you know it predates your fix}

---

### CER 2 — Fault located by direction

Claim:  
{the fault was on the forward path / the return path — name which}

Evidence:  
`{DEVICE}# show ipv6 route <destination> → {paste one exact line}`

Reasoning:  
{explain how this route lookup, and not the other direction's, pointed at the fault}

---

### CER 3 — Verified repair

Claim:  
{the repair restored the intended path and reachability}

Evidence:  
`{DEVICE}# {same command as CER 1} → {paste one exact after-repair line}`

Reasoning:  
{explain why this after-repair line, compared to CER 1's before-repair line, proves the fix and nothing else}

---

## Ops Lexicon — New This Week

CONFIRM  
The step that captures the symptom, with a saved command and output, before any configuration change.

ESTABLISH EXPECTATIONS  
The step that uses the topology and addressing table to determine the intended path before comparing it to live evidence.

INVESTIGATE  
The step that compares live route, interface, and neighbour evidence against the design to locate the discrepancy.

REPAIR  
The smallest configuration change, within the ticket's permitted scope, supported by the INVESTIGATE evidence.

VERIFY  
The step that repeats the original CONFIRM test, under the same conditions, to prove the repair.

Fault domain  
The part of the network narrowed down as the likely location of a problem, based on evidence rather than guesswork.

Before/after evidence  
Matched proof — same test, same source, same destination — captured once before a repair and once after it.

---

## Reflection Questions

1. CONFIRM: Which command and output captured your ticket's symptom before you changed anything, and what exactly did it prove was broken?  
{2–3 sentences + one proof line}

2. ESTABLISH EXPECTATIONS: What did the topology and addressing table tell you the correct next hop or path should be, before you looked at live evidence?  
{2–3 sentences}

3. INVESTIGATE — forward vs return path: Which evidence told you whether the fault was on the forward path, the return path, or both? How did the test's source address shape that judgment?  
{2–3 sentences + two proof lines}

4. REPAIR and VERIFY: What was the smallest change you made, and what matched before/after evidence proves it repaired the fault without changing anything else?  
{2–3 sentences + two proof lines}

5. Reusable knowledge: Of the five steps (CONFIRM, ESTABLISH EXPECTATIONS, INVESTIGATE, REPAIR, VERIFY), which one would you be most tempted to skip under time pressure, and what would that cost you? What is the first thing you would check next time a device reports "no reachability"?  
{2–3 sentences}

---

File naming suggestion: `reflections/w04-reflect-{username}.md`
