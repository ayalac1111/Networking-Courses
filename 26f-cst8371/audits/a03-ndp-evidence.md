
# W03 Apply — NDP Evidence Audit (RA/RS, NA/NS, DAD)

## Purpose

This Apply activity checks whether you can identify **Router Solicitation/Advertisement (RS/RA)**, **Neighbor Solicitation/Advertisement (NS/NA)**, and **Duplicate Address Detection (DAD)** evidence from Cisco-style IPv6 NDP debug/log output.

Reading logs is a core network technician skill because logs show what the device observed, not what the configuration was supposed to do. A technician uses log evidence to identify when a host asked for network information, when a router or neighbor responded, and whether the observed process matches the expected behaviour.

---

## Context

The log excerpt came from a router named `EDGE` on a single IPv6 LAN.

Minimal context:

```text
EDGE LAN interface: GigabitEthernet0/0/1
EDGE LAN IPv6:      2010:acad:15:aa::11/64
EDGE link-local:    fe80::11
LAN prefix:         2010:acad:15:aa::/64
Hosts on LAN:       VM-A and VM-B
```

Use the context only to understand the log. The proof must come from the numbered evidence lines.

---

## Deliverable

**Filename**: `a03-{username}.txt`
**Where**: Brightspace
**Points**: 3 points total — A1 (RA/RS), A2 (NA/NS), and A3 (DAD) are worth 1 point each.

---

## Instructions

Read the numbered log excerpt.

For each required section:
1. Find the **FIRST** matching evidence pair for the process named in that section.
2. Copy each full line exactly.
3. Answer the guided questions using the required keyword lines.

Do not add extra prose outside the requested answer fields.

---

## Required Evidence File Format

Your file must contain these sections and fields exactly.

```text
=== A1 — RA/RS Evidence ===
A1_RS_LINE_NUMBER=
A1_RS_COPIED_LINE=
A1_RS_SOURCE_IPV6=
A1_RA_LINE_NUMBER=
A1_RA_COPIED_LINE=
A1_RA_DESTINATION=
A1_PROVES_HOST_HAS_GUA=
A1_REASON=

=== A2 — NA/NS Evidence ===
A2_NS_LINE_NUMBER=
A2_NS_COPIED_LINE=
A2_NS_TARGET_IPV6=
A2_NA_LINE_NUMBER=
A2_NA_COPIED_LINE=
A2_NA_DESTINATION_IPV6=
A2_PROVES_NEIGHBOR_REACHABLE=
A2_REASON=

=== A3 — DAD Evidence ===
A3_DAD_START_LINE_NUMBER=
A3_DAD_START_COPIED_LINE=
A3_TENTATIVE_ADDRESS=
A3_DUPLICATE_RESULT_LINE_NUMBER=
A3_DUPLICATE_RESULT_COPIED_LINE=
A3_ADDRESS_USABLE_ON_EDGE=
A3_REASON=
```

---

## Guided Questions

### A1 — RA/RS Evidence

Find the **FIRST** Router Solicitation and the Router Advertisement EDGE sent in reply.

Questions:

1. Which line is the first Router Solicitation evidence?
2. What full log line proves it?
3. What IPv6 address **sent** the Router Solicitation?
4. Which line is the Router Advertisement EDGE sent in reply?
5. What full log line proves it?
6. Which destination address did the RA use?
7. Does this RA/RS exchange prove that the host formed a global unicast address (GUA)? `(YES/NO)`
8. Reason.

Answer using:

```text
=== A1 — RA/RS Evidence ===
A1_RS_LINE_NUMBER=
A1_RS_COPIED_LINE=
A1_RS_SOURCE_IPV6=
A1_RA_LINE_NUMBER=
A1_RA_COPIED_LINE=
A1_RA_DESTINATION=
A1_PROVES_HOST_HAS_GUA=
A1_REASON=
```

---

### A2 — NA/NS Evidence

Find the **FIRST** Neighbor Solicitation/Neighbor Advertisement exchange that involves a **global unicast address** (not a link-local address).

Questions:

1. Which line is the first NS evidence for a global unicast target?
2. What full log line proves it?
3. Which IPv6 address is the NS asking about?
4. Which line is the NA EDGE sent in reply?
5. What full log line proves it?
6. Which destination address did the NA use?
7. Does this NS/NA exchange by itself prove the neighbor cache entry is in the REACHABLE state right now? `(YES/NO)`
8. Reason.

Answer using:

```text
=== A2 — NA/NS Evidence ===
A2_NS_LINE_NUMBER=
A2_NS_COPIED_LINE=
A2_NS_TARGET_IPV6=
A2_NA_LINE_NUMBER=
A2_NA_COPIED_LINE=
A2_NA_DESTINATION_IPV6=
A2_PROVES_NEIGHBOR_REACHABLE=
A2_REASON=
```

---

### A3 — DAD Evidence

Find the **FIRST** line where EDGE starts Duplicate Address Detection, and the line that shows the result.

Questions:

1. Which line is the first DAD start evidence?
2. What full log line proves it?
3. What tentative address is being checked?
4. Which line shows the DAD result for that address?
5. What full log line proves it?
6. Is the tentative address usable on EDGE? `(YES/NO)`
7. Reason.

Answer using:

```text
=== A3 — DAD Evidence ===
A3_DAD_START_LINE_NUMBER=
A3_DAD_START_COPIED_LINE=
A3_TENTATIVE_ADDRESS=
A3_DUPLICATE_RESULT_LINE_NUMBER=
A3_DUPLICATE_RESULT_COPIED_LINE=
A3_ADDRESS_USABLE_ON_EDGE=
A3_REASON=
```

---

## Audit Log Excerpt

```text
[01] EDGE# terminal monitor
[02] EDGE# debug ipv6 nd
[03] ICMP Neighbor Discovery events debugging is on
[04] *Sep 19 13:41:02.112: %LINK-3-UPDOWN: Interface GigabitEthernet0/0/1, changed state to up
[05] *Sep 19 13:41:03.118: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
[06] *Sep 19 13:41:06.420: ICMPv6-ND: Received RS from FE80::0:FF:FE00:150A on GigabitEthernet0/0/1
[07] *Sep 19 13:41:06.421: ICMPv6-ND: Sending solicited RA on GigabitEthernet0/0/1 to FF02::1
[08] *Sep 19 13:41:06.421: ICMPv6-ND:   MTU = 1500
[09] *Sep 19 13:41:06.422: ICMPv6-ND:   prefix 2010:ACAD:15:AA::/64, valid lifetime 2592000, preferred lifetime 604800
[10] *Sep 19 13:41:06.423: ICMPv6-ND:   source link-layer address 5254.0011.0001
[11] *Sep 19 13:41:09.914: ICMPv6-ND: Received NS for FE80::11 from FE80::0:FF:FE00:150A on GigabitEthernet0/0/1
[12] *Sep 19 13:41:09.915: ICMPv6-ND: Sending NA for FE80::11 to FE80::0:FF:FE00:150A on GigabitEthernet0/0/1
[13] *Sep 19 13:41:10.226: ICMPv6-ND: Received NS for 2010:ACAD:15:AA::11 from 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1
[14] *Sep 19 13:41:10.227: ICMPv6-ND: Sending NA for 2010:ACAD:15:AA::11 to 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1
[15] *Sep 19 13:41:10.881: ICMPv6-ND: Neighbor FE80::0:FF:FE00:150A on GigabitEthernet0/0/1 changed state to REACHABLE
[16] *Sep 19 13:41:12.316: ICMPv6-ND: Received NS for 2010:ACAD:15:AA::22 from FE80::0:FF:FE00:150A on GigabitEthernet0/0/1
[17] *Sep 19 13:41:12.317: ICMPv6-ND: No neighbor advertisement sent; target address not local to this router
[18] *Sep 19 13:41:14.013: ICMPv6-ND: Received RS from FE80::0:FF:FE00:220B on GigabitEthernet0/0/1
[19] *Sep 19 13:41:14.014: ICMPv6-ND: Sending solicited RA on GigabitEthernet0/0/1 to FF02::1
[20] *Sep 19 13:41:15.660: ICMPv6-ND: Received NS for 2010:ACAD:15:AA::11 from 2010:ACAD:15:AA:0:FF:FE00:220B on GigabitEthernet0/0/1
[21] *Sep 19 13:41:15.661: ICMPv6-ND: Sending NA for 2010:ACAD:15:AA::11 to 2010:ACAD:15:AA:0:FF:FE00:220B on GigabitEthernet0/0/1
[22] *Sep 19 13:41:21.004: ICMPv6-ND: Neighbor 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1 changed state to STALE
[23] *Sep 19 13:41:23.772: ICMPv6-ND: Neighbor 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1 changed state to DELAY
[24] *Sep 19 13:41:28.774: ICMPv6-ND: Neighbor 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1 changed state to PROBE
[25] *Sep 19 13:41:28.775: ICMPv6-ND: Sending NS for 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1
[26] *Sep 19 13:41:28.781: ICMPv6-ND: Received NA from 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1
[27] *Sep 19 13:41:28.782: ICMPv6-ND: Neighbor 2010:ACAD:15:AA:0:FF:FE00:150A on GigabitEthernet0/0/1 changed state to REACHABLE
[28] *Sep 19 13:41:31.400: ICMPv6-ND: DAD: Sending NS for 2010:ACAD:15:AA::10 on GigabitEthernet0/0/1
[29] *Sep 19 13:41:31.402: ICMPv6-ND: DAD: Received NA for 2010:ACAD:15:AA::10 from FE80::0:FF:FE00:150A on GigabitEthernet0/0/1
[30] *Sep 19 13:41:31.403: %IPV6_ND-4-DUPLICATE: Duplicate address 2010:ACAD:15:AA::10 on GigabitEthernet0/0/1
[31] *Sep 19 13:41:31.404: ICMPv6-ND: DAD: 2010:ACAD:15:AA::10 is duplicate, address not usable on GigabitEthernet0/0/1
[32] *Sep 19 13:41:33.044: %SYS-5-CONFIG_I: Configured from console by console
[33] *Sep 19 13:41:35.500: ICMPv6-ND: Sending unsolicited RA on GigabitEthernet0/0/1 to FF02::1
[34] *Sep 19 13:41:35.501: ICMPv6-ND:   prefix 2010:ACAD:15:AA::/64, valid lifetime 2592000, preferred lifetime 604800
[35] EDGE# show ipv6 neighbors
[36] IPv6 Address                              Age Link-layer Addr State Interface
[37] FE80::0:FF:FE00:150A                       0 0200.0000.150a  REACH Gi0/0/1
[38] 2010:ACAD:15:AA:0:FF:FE00:150A             0 0200.0000.150a  REACH Gi0/0/1
[39] FE80::0:FF:FE00:220B                       1 0200.0000.220b  STALE Gi0/0/1
[40] 2010:ACAD:15:AA:0:FF:FE00:220B             1 0200.0000.220b  STALE Gi0/0/1
[41] EDGE# show ipv6 interface GigabitEthernet0/0/1 | include IPv6|link-local|Joined|FF02|2010|DUP
[42] GigabitEthernet0/0/1 is up, line protocol is up
[43]   IPv6 is enabled, link-local address is FE80::11
[44]   Global unicast address(es):
[45]     2010:ACAD:15:AA::11, subnet is 2010:ACAD:15:AA::/64
[46]     2010:ACAD:15:AA::10, subnet is 2010:ACAD:15:AA::/64 [DUP]
[47]   Joined group address(es):
[48]     FF02::1
[49]     FF02::2
[50]     FF02::1:FF00:10
[51]     FF02::1:FF00:11
[52] EDGE# undebug all
[53] All possible debugging has been turned off
```

---
