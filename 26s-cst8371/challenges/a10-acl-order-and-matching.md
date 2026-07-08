# A10 — Standard ACL Order and Matching Audit

**Course:** CST8371
**Week:** 10
**Activity type:** Apply / Audit
**Points:** 4
**Submission file:** `a10-<username>.txt`

---

## Audit Scope

This audit evaluates standard ACL statement order, wildcard-mask matching, and implicit-deny behaviour from operational evidence.

- You are not configuring devices.
- You are not fixing faults.
- You are identifying how ACL statement order and matching logic produced the observed behaviour.

Use only the topology and numbered outputs provided.

---

## Rules

- Submit only the answer keywords listed in each scenario.
- Use the exact keyword names.
- Do not include paragraphs.
- Do not include configuration commands.
- When an answer asks for a line number, give the ACE sequence number only.
- When an answer asks for a command, include the device prompt.

Example answer format:

```text
MATCH_LINE_1: 20
ACTION_1: DENY
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
| R1 | Edge router, ACL applied on LAN-facing interface |
| R2 | Core / transit router |
| R3 | Protected destination network (LAN-R3) |
| R4 | Unrelated destination network (LAN-R4) |

### Link and Interface Reference

| Link | Network | Router A Interface / IP | Router B Interface / IP |
|---|---|---|---|
| R1-R2 | `10.12.0.0/29` | R1 G0/0 — `10.12.0.1/29` | R2 G0/0 — `10.12.0.2/29` |
| R2-R3 | `10.23.0.0/29` | R2 G0/1 — `10.23.0.2/29` | R3 G0/0 — `10.23.0.3/29` |
| R2-R4 | `10.24.0.0/29` | R2 G0/2 — `10.24.0.2/29` | R4 G0/0 — `10.24.0.4/29` |

### End Networks

| LAN | Router Interface | Network |
|---|---|---|
| LAN-R1 | R1 G0/1 — `192.168.1.1/24` | `192.168.1.0/24` |
| LAN-R3 | R3 G0/1 — `192.168.3.1/24` | `192.168.3.0/24` |
| LAN-R4 | R4 G0/1 — `192.168.4.1/24` | `192.168.4.0/24` |

---

## Scenario 1 — Statement Order Determines Outcome

R1 has a standard ACL named `LIMIT-LAN1` bound outbound on `G0/1`, intended to deny host `192.168.1.50` while permitting all other LAN-R1 traffic.

Use **Output 1 — ACL Order Evidence** to determine why the deny never takes effect.

### Output 1 — ACL Order Evidence

```plaintext
R1# show ip access-lists LIMIT-LAN1

Standard IP access list LIMIT-LAN1
    10 permit any (42 matches)
    20 deny host 192.168.1.50 (0 matches)
```

### Scenario 1 Answers

| Keyword | Question |
|---|---|
| `FIRST_MATCH_LINE_1` | Which sequence number is evaluated first for traffic from `192.168.1.50`? |
| `EFFECTIVE_ACTION_1` | What action is actually applied to traffic from `192.168.1.50`? |
| `UNREACHABLE_LINE_1` | Which sequence number can never be matched as currently ordered? |
| `REASONING_1` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 2 — Wildcard Mask Matching

R3 has a standard ACL named `RESTRICT-R3` applied inbound on `G0/0`.

Use **Output 2 — Wildcard Mask Evidence** to determine which source addresses match the permit line.

### Output 2 — Wildcard Mask Evidence

```plaintext
R3# show ip access-lists RESTRICT-R3

Standard IP access list RESTRICT-R3
    10 permit 192.168.1.0 0.0.0.15 (18 matches)
    20 deny any log (6 matches)
```

### Scenario 2 Answers

| Keyword | Question |
|---|---|
| `RANGE_START_2` | What is the first usable address in the permitted range? |
| `RANGE_END_2` | What is the last address in the permitted range? |
| `HOST_192_168_1_50_MATCH_2` | Does `192.168.1.50` match line 10? Answer `YES` or `NO`. |
| `REASONING_2` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 3 — Implicit Deny Behaviour

R4 has a standard ACL named `LAN4-IN` applied inbound on `G0/1`. No explicit `deny` statement was configured.

Use **Output 3 — ACL Binding and Log Evidence** to determine what happened to unmatched traffic.

### Output 3 — ACL Binding and Log Evidence

```plaintext
R4# show ip access-lists LAN4-IN

Standard IP access list LAN4-IN
    10 permit 192.168.4.0 0.0.0.255 (30 matches)

R4# show ip interface GigabitEthernet0/1 | include access list
  Inbound  access list is LAN4-IN

R4# show logging | include ACCESS-LIST
*Jul  6 10:12:03.221: %SEC-6-IPACCESSLOGP: list LAN4-IN denied ip 10.24.0.2 (GigabitEthernet0/1) -> 192.168.4.5
```

### Scenario 3 Answers

| Keyword | Question |
|---|---|
| `EXPLICIT_DENY_PRESENT_3` | Is there an explicit deny statement in `LAN4-IN`? Answer `YES` or `NO`. |
| `DENIED_SOURCE_3` | What source address was denied according to the log? |
| `DENY_RULE_NAME_3` | What is the name of the rule that produced this denial, even though it is not shown in `show ip access-lists`? |
| `REASONING_3` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Submission Format

Create a file named:

```text
a10-<username>.txt
```

Use this exact format:

```text
=== SCENARIO 1 ===
FIRST_MATCH_LINE_1:
EFFECTIVE_ACTION_1:
UNREACHABLE_LINE_1:

REASONING_1:

=== SCENARIO 2 ===
RANGE_START_2:
RANGE_END_2:
HOST_192_168_1_50_MATCH_2:

REASONING_2:

=== SCENARIO 3 ===
EXPLICIT_DENY_PRESENT_3:
DENIED_SOURCE_3:
DENY_RULE_NAME_3:

REASONING_3:
```

---

## End of A10 — Standard ACL Order and Matching Audit
