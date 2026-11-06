# A10 — NAT Translation Audit

**Course:** CST8371
**Week:** 10
**Activity type:** Apply / Audit
**Points:** 4
**Submission file:** `a10-<username>.txt`

---

## Audit Scope

This audit evaluates NAT interface roles, ACL-based traffic selection for NAT, dynamic pool behaviour, and address translation from operational evidence.

- You are not configuring devices.
- You are not fixing faults.
- You are identifying why the observed NAT behaviour occurred, using only the evidence provided.

Use only the topology and numbered outputs provided. Each scenario is an independent finding worth 1 point (4 points in total).

---

## Rules

- Submit only the answer keywords listed in each scenario.
- Use the exact keyword names.
- Do not include paragraphs.
- Do not include configuration commands.
- When an answer asks YES/NO, answer `YES` or `NO` only.
- When an answer asks for an interface, the full name or the usual short form is accepted (for example `GigabitEthernet0/2` or `Gi0/2`).
- When an answer asks for an address or a port, give only the value requested.
- Outputs are shortened to the relevant lines. The numbers at the left of each output line are for reference only and are not part of the device output. Blank lines are not numbered.

Example answer format:

```text
EXAMPLE_ROLE_1: outside
EXAMPLE_ANSWER_1: NO
```

---

## Reasoning Section

Each scenario includes a `REASONING_n` field.

- Provide 1–3 sentences explaining how you determined your answers from the provided evidence. Cite output line numbers where you can.
- This section is not graded and does not affect your score.
- It is collected to help improve future course materials and identify common reasoning patterns.

---

## Topology

### Router IDs and Roles

| Router | Role |
|---|---|
| R1 | Border router performing NAT — sits at the inside/outside boundary |
| R2 | Internal core router — LAN-side, no NAT configured |
| ISP | Simulated outside next hop (not a course-managed device) |

### Link and Interface Reference

| Link | Network | Router A Interface / IP | Router B Interface / IP |
|---|---|---|---|
| R2-R1 | `10.12.0.0/29` | R2 G0/0 — `10.12.0.2/29` | R1 G0/0 — `10.12.0.1/29` |
| R1-ISP | `203.0.113.0/30` | R1 G0/1 — `203.0.113.1/30` | ISP — `203.0.113.2/30` |

### End Networks

| LAN | Router Interface | Network |
|---|---|---|
| LAN-R2 | R2 G0/1 — `192.168.10.1/24` | `192.168.10.0/24` |

**Intended design:** R1 translates LAN-R2 traffic bound for the ISP. A NAT ACL named `NAT-SELECT` selects the traffic to translate, using a pool named `NATPOOL`.

---

## Scenario 1 — Interface Role Assignment

R1's NAT interface roles were configured, but hosts on `192.168.10.0/24` report no Internet reachability.

Use **Output 1 — Interface Role Evidence** to determine what is wrong.

### Output 1 — Interface Role Evidence

```plaintext
 1 | R1# show ip nat statistics
 2 | Total active translations: 0 (0 static, 0 dynamic; 0 extended)
 3 | Outside interfaces:
 4 |   GigabitEthernet0/0
 5 | Inside interfaces:
 6 |   GigabitEthernet0/1
 7 | Hits: 0  Misses: 0
 8 | Dynamic mappings:
 9 | -- Inside Source
10 | [Id: 1] access-list NAT-SELECT pool NATPOOL refcount 0
11 |  pool NATPOOL: netmask 255.255.255.252
12 |         start 203.0.113.5 end 203.0.113.6
13 |         type generic, total addresses 2, allocated 0 (0%), misses 0

14 | R1# show ip nat translations

15 | R1#
```

### Scenario 1 Answers

| Keyword | Question |
|---|---|
| `WRONG_INSIDE_INTERFACE_1` | Which interface is currently (incorrectly) marked `inside`? |
| `CORRECT_ROLE_G0_0_1` | What role should `GigabitEthernet0/0` have, based on the topology? |
| `EXPECTED_AFTER_FIX_1` | Once the roles are corrected, would `show ip nat translations` show entries for LAN-R2 traffic destined to the Internet? Answer `YES` or `NO`. |
| `REASONING_1` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 2 — ACL Selection for NAT

R1's NAT pool and interface roles are correctly configured, but LAN-R2 hosts (`192.168.10.0/24`) still show no translations.

Use **Output 2 — NAT ACL Evidence** to determine why this subnet is never selected for translation.

### Output 2 — NAT ACL Evidence

```plaintext
 1 | R1# show access-lists NAT-SELECT

 2 | Standard IP access list NAT-SELECT
 3 |     10 permit 192.168.20.0, wildcard bits 0.0.0.255

 4 | R1# show ip nat statistics

 5 | Total active translations: 0 (0 static, 0 dynamic; 0 extended)
 6 | Hits: 0  Misses: 0
 7 | Dynamic mappings:
 8 | -- Inside Source
 9 | [Id: 1] access-list NAT-SELECT pool NATPOOL refcount 0
10 |  pool NATPOOL: netmask 255.255.255.252
11 |         start 203.0.113.5 end 203.0.113.6
12 |         type generic, total addresses 2, allocated 0 (0%), misses 0
```

### Scenario 2 Answers

| Keyword | Question |
|---|---|
| `PERMITTED_SUBNET_2` | Which subnet does `NAT-SELECT` line 10 actually permit? |
| `INTENDED_SUBNET_2` | Which subnet should `NAT-SELECT` be permitting, based on the topology's LAN-R2 network? |
| `TRANSLATION_EXPECTED_2` | As currently configured, will a host at `192.168.10.10` be selected for translation? Answer `YES` or `NO`. |
| `REASONING_2` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 3 — Dynamic Pool Behaviour

The NAT ACL and interface roles have both been corrected. Five hosts on LAN-R2 are active, but not all of them have Internet reachability.

Use **Output 3 — Pool Statistics Evidence** to determine why.

### Output 3 — Pool Statistics Evidence

```plaintext
 1 | R1# show ip nat statistics

 2 | Total active translations: 4 (0 static, 4 dynamic; 2 extended)
 3 | Hits: 8  Misses: 3
 4 | Dynamic mappings:
 5 | -- Inside Source
 6 | [Id: 1] access-list NAT-SELECT pool NATPOOL refcount 4
 7 |  pool NATPOOL: netmask 255.255.255.252
 8 |         start 203.0.113.5 end 203.0.113.6
 9 |         type generic, total addresses 2, allocated 2 (100%), misses 3

10 | R1# show ip nat translations

11 | Pro  Inside global         Inside local          Outside local         Outside global
12 | icmp 203.0.113.5:1         192.168.10.10:1       198.51.100.10:1       198.51.100.10:1
13 | ---  203.0.113.5           192.168.10.10         ---                   ---
14 | icmp 203.0.113.6:2         192.168.10.11:2       198.51.100.10:2       198.51.100.10:2
15 | ---  203.0.113.6           192.168.10.11         ---                   ---
```

### Scenario 3 Answers

| Keyword | Question |
|---|---|
| `POOL_SIZE_3` | How many total addresses are in `NATPOOL`? |
| `POOL_STATUS_3` | Is the pool exhausted or does it have addresses available? Answer `EXHAUSTED` or `AVAILABLE`. |
| `NEW_SESSION_RESULT_3` | What happens to a translation attempt from another inside host while the pool is in this state? Answer `FAILS` or `TRANSLATED`. |
| `REASONING_3` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 4 — Address Translation

R1's interface roles, NAT ACL and pool are correctly configured. R1 also publishes selected services from LAN-R2 on its ISP-facing interface.

Packet 1 arrives at R1 from the ISP side, as shown below, before R1 translates it. Use **Output 4 — Translation Table Evidence** to determine what R1 does to Packet 1.

### Packet 1 — as received by R1 on GigabitEthernet0/1, before translation

```plaintext
Protocol:     TCP
Source:       198.51.100.10:49152
Destination:  203.0.113.1:2323
```

### Output 4 — Translation Table Evidence

```plaintext
 1 | R1# show ip nat translations

 2 | Pro  Inside global         Inside local          Outside local         Outside global
 3 | tcp  203.0.113.1:2222      192.168.10.51:22      ---                   ---
 4 | tcp  203.0.113.1:2323      192.168.10.50:23      ---                   ---
```

### Scenario 4 Answers

| Keyword | Question |
|---|---|
| `TRANSLATED_DEST_ADDR_4` | What destination address does Packet 1 have after R1 translates it? |
| `TRANSLATED_DEST_PORT_4` | What destination port does Packet 1 have after R1 translates it? |
| `SOURCE_CHANGED_4` | Does R1 change the source address of Packet 1? Answer `YES` or `NO`. |
| `REASONING_4` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Submission Format

Create a file named:

```text
a10-<username>.txt
```

Use this exact format:

```text
=== SCENARIO 1 ===
WRONG_INSIDE_INTERFACE_1:
CORRECT_ROLE_G0_0_1:
EXPECTED_AFTER_FIX_1:

REASONING_1:

=== SCENARIO 2 ===
PERMITTED_SUBNET_2:
INTENDED_SUBNET_2:
TRANSLATION_EXPECTED_2:

REASONING_2:

=== SCENARIO 3 ===
POOL_SIZE_3:
POOL_STATUS_3:
NEW_SESSION_RESULT_3:

REASONING_3:

=== SCENARIO 4 ===
TRANSLATED_DEST_ADDR_4:
TRANSLATED_DEST_PORT_4:
SOURCE_CHANGED_4:

REASONING_4:
```

---

## End of A10 — NAT Translation Audit
