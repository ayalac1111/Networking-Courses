# A05 — OSPF Neighbours

## Scenario

A three-router OSPF topology has been partially configured. The physical links are assumed to be connected and administratively up.

The configuration contains exactly **three** problems that prevent all expected OSPF neighbours from forming.

Your task is to identify the three problems, cite the numbered config lines that prove each problem, write the smallest safe fix, and state how success would be verified.

---

## Target State

| Category | Required State |
|---|---|
| OSPF neighbour formation | OSPF neighbours form on all three point-to-point links |
| OSPF area | All transit links use area `0` |
| OSPF activation method | OSPF is enabled under each transit interface |
| Network statements | Do not use `network` statements |
| Point-to-point masks | Each transit pair uses matching `/30` masks |
| Timers | All link timers use OSPF default values: hello `10`, dead `40` |
| Expected result | Each router can show the correct OSPF neighbour in `FULL` state |

---

## Topology / Addressing / Context

| Link | Devices / Interfaces | Expected Network |
|---|---|---|
| Link-A | RTR-NEIGH-1 Gi0/0 ↔ RTR-NEIGH-2 Gi0/0 | `10.0.12.0/30` |
| Link-B | RTR-NEIGH-2 Gi0/1 ↔ RTR-NEIGH-3 Gi0/1 | `10.0.23.0/30` |
| Link-C | RTR-NEIGH-3 Gi0/2 ↔ RTR-NEIGH-1 Gi0/2 | `10.0.31.0/30` |

---

## Evidence

Review the numbered configurations below.

### RTR-NEIGH-1

```text
[R1-1] version 15.4
[R1-2] hostname RTR-NEIGH-1
[R1-3] interface GigabitEthernet0/0
[R1-4]  description Link-A to R2
[R1-5]  ip address 10.0.12.1 255.255.255.252
[R1-6]  ip ospf 1 area 0
[R1-7] !
[R1-8] interface GigabitEthernet0/2
[R1-9]  description Link-C to R3
[R1-10] ip address 10.0.31.1 255.255.255.252
[R1-11] ip ospf 1 area 0
[R1-12] ip ospf hello-interval 10
[R1-13] ip ospf dead-interval 40
[R1-14] !
[R1-15] interface Loopback0
[R1-16] ip address 1.1.1.1 255.255.255.255
[R1-17] !
[R1-18] router ospf 10
[R1-19]  router-id 1.1.1.1
[R1-20] end
```

### RTR-NEIGH-2

```text
[R2-1] version 15.4
[R2-2] hostname RTR-NEIGH-2
[R2-3] interface GigabitEthernet0/0
[R2-4]  description Link-A to R1
[R2-5]  ip address 10.0.12.2 255.255.255.0
[R2-6]  ip ospf 1 area 0
[R2-7] !
[R2-8] interface GigabitEthernet0/1
[R2-9]  description Link-B to R3
[R2-10] ip address 10.0.23.2 255.255.255.252
[R2-11] ip ospf 1 area 1
[R2-12] !
[R2-13] interface Loopback0
[R2-14] ip address 2.2.2.2 255.255.255.255
[R2-15] !
[R2-16] router ospf 10
[R2-17]  router-id 2.2.2.2
[R2-18] end
```

### RTR-NEIGH-3

```text
[R3-1] version 15.4
[R3-2] hostname RTR-NEIGH-3
[R3-3] interface GigabitEthernet0/1
[R3-4]  description Link-B to R2
[R3-5]  ip address 10.0.23.3 255.255.255.252
[R3-6]  ip ospf 1 area 0
[R3-7] !
[R3-8] interface GigabitEthernet0/2
[R3-9]  description Link-C to R1
[R3-10] ip address 10.0.31.3 255.255.255.252
[R3-11] ip ospf 1 area 0
[R3-12] ip ospf hello-interval 5
[R3-13] ip ospf dead-interval 20
[R3-14] !
[R3-15] interface Loopback0
[R3-16] ip address 3.3.3.3 255.255.255.255
[R3-17] !
[R3-18] router ospf 100
[R3-19]  router-id 3.3.3.3
[R3-20] end
```

---

## Task

Submit exactly **three** findings.

For each finding, identify:

1. the fault code
2. the numbered evidence lines
3. the expected value
4. the minimal fix
5. the complete Cisco command or commands that apply the fix
6. where verification should be run
7. the complete verification command
8. the expected output keywords

---

## Fault Codes

Use one fault code per finding.

```text
MASK_MISMATCH
AREA_MISMATCH
TIMER_MISMATCH
ROUTER_ID_MISMATCH
PASSWORD_MISMATCH
PROCESS_ID_MISMATCH
MISSING_OSPF_INTERFACE
WRONG_INTERFACE_DESCRIPTION
MISSING_LOOPBACK
```

---

## Required Submission Template

Copy this template into `a05-{username}.txt`.

```text
student=
username=

=== FINDING 1 ===
finding1=
fault_code_1=
evidence_1_lines=
evidence_1_expected_value=
fix1=
fix1_command=
verify_1_command=
verify_1_expected_keywords=

=== FINDING 2 ===
finding2=
fault_code_2=
evidence_2_lines=
evidence_2_expected_value=
fix2=
fix2_command=
verify_2_command=
verify_2_expected_keywords=

=== FINDING 3 ===
finding3=
fault_code_3=
evidence_3_lines=
evidence_3_expected_value=
fix3=
fix3_command=
verify_3_command=
verify_3_expected_keywords=
```

---

## Field Rules

| Field                        | Required Content                                                                                                                         |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `finding#`                   | Short explanation in your own words                                                                                                      |
| `fault_code_#`               | One code from the fault-code list                                                                                                        |
| `evidence_#_lines`           | Copy the numbered config lines that prove the incorrect value, mismatch, or missing requirement; only the numbers, no the complete lines |
| `evidence_#_expected_value`  | Write the value required for the target state                                                                                            |
| `fix#`                       | Short explanation in your own words                                                                                                      |
| `fix#_command`               | Complete Cisco command or commands needed to correct the issue; identify the devices and the line that should be changed                 |
| `verify_#_command`           | Complete Cisco verification command; identify the device where the command should be run                                                 |
| `verify_#_expected_keywords` | Output keywords, neighbour state, interface, IP address, or value that proves success                                                    |

---

## Command and Verification Rules

Use complete commands. Do not use abbreviated commands.

If multiple commands are required in one fix field, separate them with a semicolon `;`.

Acceptable format:

```
fix1_command=On SW1, replace line [SW1-14] with: switchport access vlan 20
```


A verification command must include where the command is run and the complete Cisco command.

Examples:

```text
verify_1_command=On RTR-EXAMPLE-1, run: show ip ospf neighbor
verify_1_expected_keywords=FULL, 4.4.4.4, GigabitEthernet0/3
```

---

## Deliverable

| Item | Requirement |
|---|---|
| Filename | `a05-{username}.txt` |
| Where | Brightspace |
| Points | 3 points total — each problem/fix is 1 point |
