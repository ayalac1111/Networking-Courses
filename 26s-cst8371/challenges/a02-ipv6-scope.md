# W02 Audit — IPv6 Scope Configuration Analysis

## What is an Audit?

An **Audit** is a read / diagnose / plan activity.

You will analyze a static IPv6 configuration, identify what violates the Week 2 target state, prescribe the **minimal IOS fix**, and list one verification command for each issue.

This activity does **not** require live Packet Tracer devices.

---

## Week 2 Target State

For this audit, use the following IPv6 scope rules:

```text
LLA boundary = protocol scope boundary.
ULA boundary = routing-policy / administrative boundary.
GUA boundary = globally routable address scope.
```

The target state is:  
  
1. **Site A and Site B** must use ULA addressing:  
- Site A: `fd00:acad:100:a::/64`  
- Site B: `fd00:acad:100:b::/64`  
  
2. **R1–R2 transit** must use GUA addressing:  
- R1–R2 transit: `2010:acad:100:12::/64`  
  
3. **R1 router interfaces** must use:  
- LLA `fe80::1`  
- interface ID `::1`  
  
4. **R1 must not have external reachability routes**:  
- no default route `::/0`  
- no route to the External Server network `2010:acad:100:99::/64`  
- no route to Site C `fd00:acad:100:c::/64`

---

## What to Do

Read the embedded R1 running-config.

Identify exactly **three** violations of the Week 2 target state.

For each violation, provide:

- one finding
- one minimal IOS fix
- one verification command with the expected good result

---

## Deliverable

- **Filename:** `a02-username.txt`
- **Where:** Brightspace →02-Audit
- **Points:** 3 points total
  - 1 point per issue
  - each issue requires finding + minimal fix + verification

---

## Audit Submission Template

```text
findings1=<one clear issue>
fix1=<minimal IOS fix>
verify1=<command -> expected good>

findings2=<one clear issue>
fix2=<minimal IOS fix>
verify2=<command -> expected good>

findings3=<one clear issue>
fix3=<minimal IOS fix>
verify3=<command -> expected good>
```

### Field rules

| Field | Requirement |
|---|---|
| `findings#` | State what violates the target state |
| `fix#` | Use the fewest IOS commands needed |
| `verify#` | Include one command and one expected good result |

---

## Audit Configuration — `R1-AUDIT-W02`

Assume assigned value:

```text
U = 100
```

Analyze this configuration:

```plaintext
! running-config — Week 2 IPv6 Scope Audit
! Purpose: identify IPv6 scope and routing-policy violations.

version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname R1-AUDIT-W02
no ip domain-lookup
!
ipv6 unicast-routing
!
interface GigabitEthernet0/0/1
 description SITE-A-ULA-LAN
 ipv6 address FE80::1 link-local
 ipv6 address FD00:ACAD:100:A::1/64
 no shutdown
!
interface GigabitEthernet0/0/2
 description SITE-B-ULA-LAN
 ipv6 address FE80::1 link-local
 ipv6 address 2010:ACAD:100:B::1/64
 no shutdown
!
interface GigabitEthernet0/0/0
 description R1-R2-GUA-TRANSIT
 ipv6 address FE80::1 link-local
 ipv6 address FD00:ACAD:100:12::1/64
 no shutdown
!
ipv6 route ::/0 2010:ACAD:100:12::2
!
line console 0
 logging synchronous
!
end
```

---

## Expected Audit Focus

Look for violations related to:

```text
address type
scope boundary
interface identity
routing-policy boundary
```

Do not report cosmetic issues.

---

## Verification Command Bank

Use these examples when writing your verify lines.

```plaintext
show ipv6 interface brief
show ipv6 route static
show ipv6 route ::/0
```

Example format:

```text
verify1=show ipv6 interface brief -> FD00:ACAD:100:B::1 appears on Gi0/0/2
```

---
## End of W02 Audit

A correct audit does not rewrite the whole configuration.

It identifies the smallest change that restores the target state and names the command that proves the fix.
