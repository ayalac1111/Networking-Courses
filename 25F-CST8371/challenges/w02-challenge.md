# W02 Challenge — IPv6 “Find it, Fix it”

> **Goal:** In ≤ 30 minutes, diagnose and repair two IPv6 issues in a tiny topology so that **PC ↔ Server** connectivity works end‑to‑end. Submit a short **evidence** text file **w02-challenge-username.txt**.

---

## 🔎 Scenario (student view)

You’re handed a prebuilt Packet Tracer file. Users report they can’t reach the HTTP server by IPv6. You must verify the addressing on each link, install the right static/default routes, and fix any mistakes so the **PC** can reach the **Server** at `2001:db8:192::69`.

---

## Baseline (already configured for you)

**You do not need to configure devices from scratch**—you will only diagnose and repair faults.

| Node          | Interface        | IPv6 Address                            | Notes                                    |
| ------------- | ---------------- | --------------------------------------- | ---------------------------------------- |
| **PC0**       | Fa0              | `2010:acad:10:aa::10/64`                | **Default GW:** `2010:acad:10:aa::1`     |
| **R1 (EDGE)** | Gi0/0/1 ↔ PC     | `2010:acad:10:aa::1/64`, LLA `fe80::1`  | `no shut`                                |
|               | Gi0/0/0 ↔ R2     | `2010:acad:10:bc::1/64`, LLA `fe80::1`  | `no shut`                                |
| **R2 (CORE)** | Gi0/0/0 ↔ R1     | `2010:acad:10:bc::2/64`, LLA `fe80::2`  | `no shut`                                |
|               | Gi0/0/1 ↔ Server | `2001:db8:192::254/64`, LLA `fe80::254` | `no shut`                                |
| **Server0**   | Fa0              | `2001:db8:192::69/64`                   | **Default GW:** `2001:db8:192::254` (R2) |

**Target state (reference)**
- Each router has reachability to the _opposite_ LAN via static/default routes.
- At least one static uses **link‑local next‑hop + exit interface**.
- IPv6 forwarding should be enabled on both routers (`ipv6 unicast-routing`).

> **Must‑remember rule:** If the next hop is a **link‑local** IPv6 address, your static route must include the **exit interface + LLA**.

---

## Your tasks
1. Systematically verify Layer‑1/2/3 on the path PC → R1 → R2 → Server.
2. Identify and repair the faults so end‑to‑end connectivity works.
3. Capture concise evidence (interfaces, neighbours, routes, and path tests).
    
**Helpful commands** _(use as you see fit)_

```text
show ipv6 interface brief
show ipv6 neighbors <interface>
show ipv6 route
show ipv6 route static
show running-config | include ^ipv6 route|ipv6 unicast-routing
ping ipv6 <addr>
ping ipv6 fe80::XX source <interface>
traceroute 2001:db8:192::69
```

---

## 🧩 Troubleshooting Tickets

There are **two independent tickets** in the file. Use the checklists to drive your investigation. Avoid random changes—work top‑down and prove each hop.

### Ticket 1 — PC can reach R1, but not R2

**Scenario:** Helpdesk reports that pings from **PC** reach **R1**, but anything beyond **R1** fails. The server is reported healthy and on the correct IPv6.

**Student checklist**
- [ ] Map the path and **name the interfaces** involved (PC↔R1, R1↔R2, R2↔Server).
- [ ] On each router, confirm **interfaces are up/up** with an IPv6 GUA and an LLA.
- [ ] Verify **neighbour discovery** on each transit link (check the expected LLA appears on the correct interface).
- [ ] Inspect **routing readiness** on both routers (global forwarding, presence of routes—do not assume!).
- [ ] From each hop, try **direct pings** to the next hop’s GUA and LLA (use `source <interface>` where appropriate).
- [ ] If the path still breaks, trace the hop where replies stop and **focus your diffs there**.
    
**Success criteria:** Traffic now progresses **past R1 to R2** (e.g., PC/R1 can `ping` R2’s GUA/LLA and `traceroute` reaches R2). _Full end‑to‑end may still fail until Ticket 2 is corrected._

---

### Ticket 2 — A static route exists, but traffic still dies

**Scenario:** The configuration on **R1** contains a static route toward the server’s LAN. However, PC‑to‑Server traffic still fails and the trace seems to loop or black‑hole.

**Student checklist**
- [ ] Identify the **exact static route line** on R1 that targets the server’s prefix.
- [ ] For routes that use **link‑local next‑hop**, correlate the **next‑hop LLA** with the **interface’s neighbour table**—they must match the same link.
- [ ] Confirm **prefix, next‑hop, and exit interface** align with the physical cabling.
- [ ] Validate that **R2** has return reachability to the PC LAN.
- [ ] Re‑test with `ping`/`traceroute` from each hop after adjustments.
    
**Success criteria:** End-to-end connectivity is successful; PC can now ping the Server.

---
### Evidence file template - **w02-challenge-username.txt**

Use this **troubleshooting log** format. Keep outputs **minimal** (use pipes like `| include`), and focus on what proves your diagnosis and fix. Do **not** paste full configs.

```bash
#-- W02 IPv6 Troubleshooting — Evidence 

[TICKET 1 — Users can reach R1, not the server]
# Root cause (one line):
- <e.g., forwarding disabled on device X>

# Change applied (minimal commands; no extra):
- <command 1>
- <command 2>

# Verification (proof after fix; minimal):
$ PC> ping 2010:acad:10:BC::2
<now progresses past R1 to R2>

---
[TICKET 2 — Static route present but traffic still dies]
# Root cause (one line):
- <e.g., incorrect static route>

# Change applied (minimal commands):
- <command to remove incorrect line>
- <command to add correct line>

# Verification (proof after fix; minimal):
$ R1# show ipv6 route
$ R1# ping 2001:db8:192::69

---
[End-to-end proof]
$ PC> tracert 2001:db8:192::69

# Result summary (one line): <Working path PC → R1 → R2 → Server>
[Post-mortem — 2 bullets]
- What caused Ticket 1 in one sentence.
- What caused Ticket 2 in one sentence.
```


---
## 📤 Submission
- Upload your file **w02-challenge-username.txt** to **W02-Challenge** on Brightspace.

