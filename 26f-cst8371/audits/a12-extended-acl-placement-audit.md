# A12 — Extended ACL Placement and Matching Audit

**Course:** CST8371
**Week:** 12
**Activity type:** Apply / Audit
**Points:** 4
**Submission file:** `a12-<username>.txt`

---

## Audit Scope

This audit evaluates extended ACL statement order, protocol/port matching, and placement (interface + direction) from operational evidence.

- You are not configuring devices.
- You are not fixing faults.
- You are identifying why the observed permit/deny behaviour occurred, using only the evidence provided.

Use only the topology and numbered outputs provided. Each scenario is an independent finding.

---

## Rules

- Fill in every keyword field listed in each scenario, using the exact keyword names.
- Keep each answer field to the value only: no paragraphs and no configuration commands.
- When an answer asks YES/NO, answer `YES` or `NO` only.
- When an answer asks for a line number, give the ACE sequence number only.
- When an answer asks you to choose a word from a list, write exactly one of the listed words.
- When an answer asks for a range, write it as network/wildcard (for example `10.0.0.0 0.0.0.255`) or network/prefix (for example `10.0.0.0/24`).
- The `REASONING_n` fields are the only place for sentences: 1-3 sentences each.

Example answer format (these example fields are not part of this audit):

```text
EXAMPLE_LINE: 30
EXAMPLE_ACTION: DENY
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
| R1 | Accounting LAN edge router — `ACCT-OUT` is configured here |
| R2 | Core / transit router |
| R3 | Engineering LAN edge router — hosts the department's management host (SSH and TFTP services) and web server |
| R4 | Sales LAN edge router — `SALES-OUT` is configured here (Scenario 4) |

### Link and Interface Reference

| Link | Network | Router A Interface / IP | Router B Interface / IP |
|---|---|---|---|
| R1-R2 | `10.12.0.0/29` | R1 G0/0 — `10.12.0.1/29` | R2 G0/0 — `10.12.0.2/29` |
| R2-R3 | `10.23.0.0/29` | R2 G0/1 — `10.23.0.2/29` | R3 G0/0 — `10.23.0.3/29` |
| R2-R4 | `10.24.0.0/29` | R2 G0/2 — `10.24.0.2/29` | R4 G0/0 — `10.24.0.4/29` |

### End Networks

| LAN | Router Interface | Network |
|---|---|---|
| LAN-R1 (Accounting) | R1 G0/1 — `192.168.1.1/24` | `192.168.1.0/24` |
| LAN-R3 (Engineering) | R3 G0/1 — `192.168.3.1/24` | `192.168.3.0/24` — management host `192.168.3.10` (SSH and TFTP), web server `192.168.3.20` |
| LAN-R4 (Sales) | R4 G0/1 — `192.168.4.1/24` | `192.168.4.0/24` |

**ACL naming:** an ACL name describes the traffic from the LAN's side (`ACCT-OUT` is traffic leaving the Accounting LAN). `in` and `out` describe the router interface's side (traffic entering or leaving that interface). Both can describe the same traffic.

**Intended policy:** Accounting hosts (`192.168.1.0/24`) must be denied SSH (TCP/22) to Engineering's management host `192.168.3.10`. All other traffic — including Accounting's access to Engineering's web server and to Sales — should remain permitted.

---

## Scenario 1 — Statement Order

R1 has extended ACL `ACCT-OUT` intended to deny Accounting-to-Engineering SSH while permitting everything else.

Use **Output 1 — ACL Order Evidence** to determine why the SSH deny never takes effect.

### Output 1 — ACL Order Evidence

```plaintext
R1# show ip access-lists ACCT-OUT

Extended IP access list ACCT-OUT
    10 permit ip any any (188 matches)
    20 deny tcp 192.168.1.0 0.0.0.255 host 192.168.3.10 eq 22 (0 matches)
```

### Scenario 1 Answers

| Keyword | Question |
|---|---|
| `FIRST_MATCH_LINE_1` | Which sequence number is evaluated first for Accounting-to-`192.168.3.10` SSH traffic? |
| `EFFECTIVE_ACTION_1` | What action is actually applied to that traffic? |
| `SSH_CURRENTLY_BLOCKED_1` | Is SSH to `192.168.3.10` currently being blocked, as configured? Answer `YES` or `NO`. |
| `REASONING_1` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 2 — Placement: Interface Binding

The statement order in `ACCT-OUT` has been corrected (deny before permit), but Accounting hosts can still reach `192.168.3.10` over SSH.

Use **Output 2 — Interface Binding Evidence** to determine why.

### Output 2 — Interface Binding Evidence

```plaintext
R1# show ip access-lists ACCT-OUT

Extended IP access list ACCT-OUT
    10 deny tcp 192.168.1.0 0.0.0.255 host 192.168.3.10 eq 22 (0 matches)
    20 permit ip any any (204 matches)

R1# show ip interface GigabitEthernet0/0 | include access list
  Inbound  access list is ACCT-OUT

R1# show ip interface GigabitEthernet0/1 | include access list
  Inbound  access list is not set
```

### Scenario 2 Answers

| Keyword | Question |
|---|---|
| `BOUND_INTERFACE_2` | Which interface currently has `ACCT-OUT` applied inbound? |
| `CORRECT_INTERFACE_2` | Which interface should `ACCT-OUT` be applied to inbound, so it sees Accounting's traffic as it enters the router? |
| `TRAFFIC_FILTERED_2` | As currently bound, does `ACCT-OUT` ever inspect Accounting-to-Engineering SSH traffic? Answer `YES` or `NO`. |
| `REASONING_2` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 3 — Unexpected Blocking

`ACCT-OUT` now has its SSH deny ahead of the `permit ip any any`, and it is bound where it sees Accounting traffic. Engineering reports a new problem: Accounting can no longer reach Engineering's web server (`192.168.3.20`) either, even though the policy never asked for that.

Use **Output 3 — Full ACL Evidence** to determine the cause.

### Output 3 — Full ACL Evidence

```plaintext
R1# show ip access-lists ACCT-OUT

Extended IP access list ACCT-OUT
    10 deny tcp 192.168.1.0 0.0.0.255 host 192.168.3.10 eq 22 (14 matches)
    20 deny ip 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255 (96 matches)
    30 permit tcp 192.168.1.0 0.0.0.255 host 192.168.3.20 eq 80 (0 matches)
    40 permit ip any any (12 matches)
```

### Scenario 3 Answers

| Keyword | Question |
|---|---|
| `UNINTENDED_DENY_LINE_3` | Which sequence number is denying more traffic than the policy requires? |
| `BLOCKED_SCOPE_3` | What destination range does that line deny? Give network/wildcard or network/prefix (see Rules). |
| `LINE_20_MATCHES_HTTP_3` | Does line 20 match Accounting's HTTP traffic to `192.168.3.20` before line 30 is evaluated? Answer `YES` or `NO`. |
| `REASONING_3` | Explain how the output evidence supports your answers. This field is not graded. |

---

## Scenario 4 — Matching a Service

R4 has extended ACL `SALES-OUT`, bound inbound on its Sales LAN interface. Its policy: Sales hosts (`192.168.4.0/24`) must be denied TFTP to Engineering's management host `192.168.3.10`, and all other traffic remains permitted.

Sales users report that TFTP transfers to `192.168.3.10` still succeed.

Use **Output 4 — Matching Evidence** to determine why.

### Output 4 — Matching Evidence

```plaintext
R4# show ip access-lists SALES-OUT

Extended IP access list SALES-OUT
    10 deny tcp 192.168.4.0 0.0.0.255 host 192.168.3.10 eq 69 (0 matches)
    20 permit ip any any log (143 matches)

R4# show ip interface GigabitEthernet0/1 | include access list
  Inbound  access list is SALES-OUT

R4# show logging | include SALES-OUT
%SEC-6-IPACCESSLOGP: list SALES-OUT permitted udp 192.168.4.25(51234) -> 192.168.3.10(69), 1 packet
```

### Scenario 4 Answers

| Keyword | Question |
|---|---|
| `TFTP_MATCHED_LINE_4` | Which sequence number matched the Sales-to-`192.168.3.10` TFTP traffic shown in Output 4? |
| `MISMATCHED_FIELD_4` | Which field of line 10 does not match that traffic? Answer `PROTOCOL`, `SOURCE`, `DESTINATION` or `PORT`. |
| `CORRECT_PROTOCOL_4` | Which protocol keyword must line 10 use to match that traffic? |
| `REASONING_4` | Explain how the output evidence supports your answers. This field is not graded. |

## Submission Format

Create a file named:

```text
a12-<username>.txt
```

Use this exact format:

```text
=== SCENARIO 1 ===
FIRST_MATCH_LINE_1:
EFFECTIVE_ACTION_1:
SSH_CURRENTLY_BLOCKED_1:

REASONING_1:

=== SCENARIO 2 ===
BOUND_INTERFACE_2:
CORRECT_INTERFACE_2:
TRAFFIC_FILTERED_2:

REASONING_2:

=== SCENARIO 3 ===
UNINTENDED_DENY_LINE_3:
BLOCKED_SCOPE_3:
LINE_20_MATCHES_HTTP_3:

REASONING_3:

=== SCENARIO 4 ===
TFTP_MATCHED_LINE_4:
MISMATCHED_FIELD_4:
CORRECT_PROTOCOL_4:

REASONING_4:
```

---

## End of A12 — Extended ACL Placement and Matching Audit
