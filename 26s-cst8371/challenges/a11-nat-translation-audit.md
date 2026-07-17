# A11 — NAT Translation Audit

**Course:** CST8371
**Week:** 11
**Activity type:** Apply / Audit
**Points:** 3
**Submission file:** `a11-{username}.txt`

---

## Audit Scope

This audit evaluates NAT interface roles, ACL-based traffic selection for NAT, and dynamic pool behaviour from operational evidence.

- You are not configuring devices.
- You are not fixing faults.
- You are identifying why the observed NAT behaviour occurred, using only the evidence provided.

Use only the topology and numbered outputs provided. Each scenario is an independent finding worth 1 point.

---

## Rules

- Submit only the answer keywords listed in each scenario.
- Use the exact keyword names.
- Do not include paragraphs.
- Do not include configuration commands.
- When an answer asks YES/NO, answer `YES` or `NO` only.
- When an answer asks for a command, include the device prompt.

Example answer format:

```text
WRONG_INSIDE_INTERFACE_1: GigabitEthernet0/0
EXPECTED_AFTER_FIX_1: YES
```

---

## Reasoning Section

Each scenario includes a `REASONING_n` field.

- Provide 1–3 sentences explaining how you determined your answers from the provided evidence.
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

**Intended design:** R1 G0/0 (facing R2/LAN) should be `ip nat inside`; R1 G0/1 (facing ISP) should be `ip nat outside`. A NAT ACL should permit `192.168.10.0/24` for translation through a pool named `NATPOOL`.

---

## Scenario 1 — Interface Role Assignment

R1's NAT interface roles were configured, but hosts on `192.168.10.0/24` report no Internet reachability.

Use **Output 1 — Interface Role Evidence** to determine what is wrong.

### Output 1 — Interface Role Evidence

```plaintext
R1# show ip nat interfaces

Interface        Role
GigabitEthernet0/0    outside
GigabitEthernet0/1    inside

R1# show ip nat translations

R1#
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
R1# show access-lists NAT-SELECT

Standard IP access list NAT-SELECT
    10 permit 192.168.20.0 0.0.0.255

R1# show ip nat statistics

Total active translations: 0 (0 static, 0 dynamic; 0 extended)
Pool NATPOOL: netmask 255.255.255.252
        start 203.0.113.5 end 203.0.113.6
        total addresses 2, allocated 0 (0%), misses 0
```

### Scenario 2 Answers

| Keyword | Question |
|---|---|
| `PERMITTED_SUBNET_2` | Which subnet does `NAT-SELECT` line 10 actually permit? |
| `INTENDED_SUBNET_2` | Which subnet should `NAT-SELECT` be permitting, based on the topology's LAN-R2 network? |
| `TRANSLATION_EXPECTED_2` | As currently configured, will a host at `192.168.10.10` be selected for translation? Answer `YES` or `NO`. |
| `REASONING_2` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 3 — Dynamic Pool Exhaustion

The NAT ACL and interface roles have both been corrected. LAN-R2 has 5 hosts active, but only the first 2 to start a session reach the Internet — the other 3 fail.

Use **Output 3 — Pool Statistics Evidence** to determine why.

### Output 3 — Pool Statistics Evidence

```plaintext
R1# show ip nat statistics

Total active translations: 2 (0 static, 2 dynamic; 0 extended)
Pool NATPOOL: netmask 255.255.255.252
        start 203.0.113.5 end 203.0.113.6
        total addresses 2, allocated 2 (100%), misses 3

R1# show ip nat translations

Pro Inside global      Inside local        Outside local      Outside global
--- 203.0.113.5        192.168.10.10       ---                ---
--- 203.0.113.6        192.168.10.11       ---                ---
```

### Scenario 3 Answers

| Keyword | Question |
|---|---|
| `POOL_SIZE_3` | How many total addresses are in `NATPOOL`? |
| `POOL_STATUS_3` | Is the pool exhausted or does it have addresses available? Answer `EXHAUSTED` or `AVAILABLE`. |
| `NEW_SESSION_RESULT_3` | What happens to a 6th host's translation attempt while the pool is in this state? Answer `FAILS` or `TRANSLATED`. |
| `REASONING_3` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Submission Format

Create a file named:

```text
a11-{username}.txt
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
```

---

## End of A11 — NAT Translation Audit
