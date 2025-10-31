# Lab 10 – Securing Your Network with Standard ACLs

## Overview

In this lab, you will secure a two-router, one-switch network by implementing three named standard ACL policies.  You will:

1. **PROTECT-VM** – Ensure only hosts in the PC subnet may reach the VM network.  
2. **PROTECT-PC** – Prevent any hosts from spoofing internal PC-subnet addresses.  
3. **PROTECT-ALS** – Restrict SSH access to the switch management plane to PCs and a designated TFTP server.

All ACLs will be created as **named standard** lists, applied on the router or switch interface closest to the resource they protect.

Throughout the lab, you will also verify ACL hits, practice using `access-class` on VTY lines, and validate correct placement and logging.  

---

## Learning Objectives

By the end of this lab, you will be able to:

- Define and apply **named standard ACLs** to enforce source-based security policies.  
- Choose the correct **interface** and **direction** for standard ACL placement.  
- Enable and interpret **ACL hit-counters** and **logging** for both permit and deny entries.  
- Use the **`access-class`** command on switch VTY lines to harden the management plane.  
- Design and execute **verification tests** (sourced pings, spoof tests, SSH attempts) to confirm policy enforcement.  

---

## Why This Lab Is Important

- **Fundamental Security Control**: Standard ACLs provide a simple yet powerful method to enforce source-based access policies, a cornerstone of network segmentation and access control in enterprise environments.  
- **Defence in Depth**: By protecting each network segment (VM hosts, PC workstations, and switch management) separately, you limit the blast radius of misconfigurations or compromised devices.  
- **Operational Visibility**: Learning to enable and interpret ACL hit-counters and logs equips you with the tools to monitor attempted breaches, misrouted traffic, or policy violations.  
- **Management-Plane Hardening**: Applying `access-class` on VTY lines is a best practice for securing device access, an essential skill for protecting critical infrastructure components.  
- **Real-World Applicability**: Named standard ACLs are routinely used in campus and data-center networks; mastering their placement and verification prepares you for production network deployments.  

Mastering these ACL techniques will equip you to enforce policy, limit attack surfaces, and audit network behaviour in any campus or enterprise environment.  

---
## Network Topology

![Lab Topology](img/w10-Topology.png)

## Addressing Table

| Network                   | Subnet           |
| ------------------------- | ---------------- |
| **CORE–EDGE transit**     | 198.18.U.0 /30   |
| **VM subnet**             | 198.18.U.32 /28  |
| **PC subnet & Mgmt VLAN** | 198.18.U.128 /26 |
- [ ] **EDGE** always takes the **first usable** address in each lab subnet.
- [ ] **CORE** (when connected) takes the **last usable** address
- [ ] **VM** hosts live on the **last usable** of their subnet.
- [ ] **PC** hosts live on the **first usable** of their subnet.
- [ ] **ALS SVI** (the switch management interface on VLAN 1) uses the **second-last usable** address in the PC/Management VLAN.

---

## Configuration and Verification Commands for Cisco Standard ACLs

| Task                              | Command                                                         | Notes                                                         |
| --------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------- |
| Define a numbered standard ACL    | <code>access-list <1-99><br>permit\|deny [source] [wild]</code> | e.g.<br>`access-list 10`<br>`deny 10.0.0.0 0.255.255.255 log` |
| Define a named standard ACL       | <code>ip access-list standard [NAME]</code>                     |                                                               |
| Apply ACL to an interface         | <code>ip access-group # \| [NAME] in\|out</code>                | under `interface` config                                      |
| Apply ACL to VTY lines            | <code>access-class # \| [NAME] in</code>                        | under `line vty 0 4` config                                   |
| Enable ACL logging on interface   | `logging access-list`                                           | under same `interface` stanza as the ACL bind                 |
| View all ACLs and hit counts      | `show ip access-lists`                                          | EXEC mode                                                     |
| Show ACLs on a specific interface | `show ip interface <interface>`                                 | EXEC mode                                                     |
| Clear ACL counters                | `clear access-list counters`                                    | EXEC mode                                                     |
| Check ACL log entries             | <code>show logging \| include ACCESS-LIST</code>                | EXEC mode (pipe escaped as HTML entity)                       |

---

## Initial Setup

### 0. Create submission file
- [ ]  On your desktop, create a file `w10-username.txt`.  
- [ ] You will later copy this file to the REMOTE TFTP server as proof of completion.
### 1. Basic Configuration
- [ ] Use the provided base configuration file: [basic.cfg](../resources/basic.cfg)
### 2. Addressing Configuration
- [ ] Configure addresses according to the topology diagram, paying attention to the network masks.
- [ ] Add a description to all Cisco interfaces.
- [ ] Ensure all interfaces are UP/UP before continuing.
- [ ] **ALS** assign an IP address to VLAN 1 and configure the default gateway.
### 3. Routing Configuration
- [ ] **Process ID**: Use `U` as the OSPF process number.
- [ ] **Router-IDs**: Manually set each router’s ID under OSPF:
	- [ ] `EDGE`:  `U.0.0.0`
	- [ ] `CORE`: `0.0.0.U`
- [ ] **Default Gateway Advertisement**: On **EDGE**, redistribute or originate a default route so that `0.0.0.0/0` is known across the OSPF domain
- [ ] **CORE–EDGE DR Election**: Force **CORE** to win the DR election on the transit (/30) link by setting its interface priority to `U`
- [ ] **Passive Interfaces**: Mark all interfaces that do _not_ form OSPF adjacencies as passive
### 4. Verification and Testing
- [ ] **OSPF Neighbours**
    - Command: `show ip ospf neighbor`
    - Check on **EDGE** and **CORE** that each sees the other in FULL state.
- [ ]  **Router-ID Selection**
    - Command: `show ip ospf`
    - Verify each router’s **Router ID** matches the manually configured value (`EDGE = U.0.0.0`, `CORE = 0.0.0.U).
- [ ] **DR/BDR Election**
    - Command: `show ip ospf interface GigabitEthernet0/0/0`
    - On the **CORE–EDGE** /30 link, confirm **CORE** is DR and **EDGE** is BDR.
- [ ] **Default Route Propagation**
    - On **CORE**, verify a `0.0.0.0/0` route is installed, pointing to the EDGE router.
- [ ] **End-to-End Reachability**
    - Commands:
        - `ping 198.18.U.129` (PC SVI or PC host)
        - `ping 198.18.U.46` (VM host)
        - `ping 203.0.113.254` (REMOTE gateway)            
    - From **all** ALS, CORE and EDGE, confirm successful replies to every target.

---

## Policy 1 – Protect the VM Network

### 1. Security Policy Statement

**Policy #1 – Protect the VM Network**  
To enforce strict segmentation of our virtual-machine environment, only hosts in the PC subnet (`198.18.U.128/26`) may initiate traffic to the VM network (`198.18.U.32/28`).  All other sources shall be denied.

### 2. Policy Decomposition

Break the policy into its key elements:

| Component          | Details                                                  |
| ------------------ | -------------------------------------------------------- |
| **Policy Name**    | `PROTECT-VM`                                             |
| **ACL Type**       | Standard (filters on source IP only)                     |
| **Match Criteria** | Source IP in PC subnet: `198.18.U.128/26`              |
| **Actions**        | 1. **Permit** matching traffic<br>2. **Deny** all others |

### ACL Placement

![Figure 2: Policy-1 ACL Placement on EDGE --> VM](img/w10-Policy-1.png)


To decide exactly where and how to apply our **PROTECT-VM** standard ACL, follow these three steps:

1. **Trace the Traffic Path**  
   - **Source:** any PC in 198.18.U.128/26 → moves up to the CORE router (Gi0/0/1) → across the transit link (CORE Gi0/0/0 → EDGE Gi0/0/1) → exits EDGE toward the VM network (Gi0/0/2).  
   - **Destination:** any VM in 198.18.U.32/28.

2. **Choose the Closest Device to the Destination**  
   - Since this is a **standard ACL** (which matches only on source IP), best practice is to place it as **close to the destination** as possible.  
   - The device nearest the VM network is the **EDGE** router.

3. **Determine the Correct Interface & Direction**  
   - At the **EDGE** router, traffic to the VM segment _egresses_ via **GigabitEthernet0/0/2**.  
   - We must inspect packets **before** they enter the VM subnet—i.e. apply the ACL **outbound** on Gi0/0/2.

| Device | Interface             | Direction | Reason                                                    |
|--------|-----------------------|-----------|-----------------------------------------------------------|
| EDGE   | `GigabitEthernet0/0/2`| `out`     | Filters on PC-source IP just before packets hit the VM LAN; avoids collateral blocking of other traffic. |

> **Why “close to the destination” for Standard ACLs?**  
> Standard ACLs filter only on source address.  By placing them near the destination, you ensure that only traffic actually headed for the protected subnet is tested, and you avoid inadvertently blocking other traffic from the same source network that has different destinations elsewhere.  

### 4. Configuration

```shell
!-- Create and populate the ACL
configure terminal
 ip access-list standard PROTECT-VM
   permit 198.18.U.128 0.0.0.63 log   ! PC subnet
   deny any                           ! everything else
 exit

!-- Apply the ACL inbound on the VM-facing interface
interface GigabitEthernet0/0/2
 ip access-group PROTECT-VM out
 exit
```

> **Note on `log` keyword:**
> - Appending `log` to an ACE causes a syslog entry each time that line matches.
> - Log permits if you need visibility into allowed flows; log denies to audit or troubleshoot blocked traffic.
> - Be mindful of log volume—high-traffic networks may require rate-limiting (`logging rate-limit access 10 conform-action log exceed-action drop`).

### 5. Verification and Testing

**Preparation:** Clear ACL counters before you begin 

> _Only_ if necessary.  If you have tested multiple times, it is sometimes easy to `clear` the counters when you make adjustments to the ACLS.  Ensuring you are actually matching the new ACLs.

```bash 
EDGE# clear access-list counters PROTECT-VM
```

##### **Reachability Testing**

| Result | Test                | Command                                      | Expected Result                                         |
| ------ | ------------------- | -------------------------------------------- | ------------------------------------------------------- |
| ✅      | PC → VM             | `PC# ping 198.18.U.46`                       | 100% success; ICMP replies from the VM host             |
| ❌      | CORE → VM           | `CORE# ping 198.18.U.46`                     | 0% success; packets dropped by PROTECT-VM               |
| ✅      | CORE (Gi0/0/1) → VM | `CORE# ping 198.18.U.46 source 198.18.U.190` | 100% success; matches the `permit 198.18.U.128/26` line |
##### **ACL Counter Validation**
```bash
EDGE# show access-lists PROTECT-VM
```
    
- **Permit** line hits should equal the number of successful PC→VM (and spoofed) pings.
- **Deny** line hits should equal the number of failed CORE→VM pings.
##### **Logging Check (if enabled)**
```bash
show logging | include PROTECT-VM
```

>_Optional:_ Verify that denied matches were logged (if you left `log` on the `deny any` ACE).

### CO1 – Verification and Collection of Information

In your `09-username.txt` file, create a section labelled:

```diff
=== CO1 – Policy #1 - PROTECT-VM Verification ===
```

**CORE**:
```bash 
# ping 198.18.U.46                          !-- FAIL
# ping 198.18.U.46 source 198.18.U.190      !-- PASS
```

**EDGE**:
```bash
show ip access-lists PROTECT-VM 
show ip interface GigabitEthernet0/0/2 | include PROTECT-VM
show logging | include PROTECT-VM
```


**What to Include:**

| Requirement             | Details                                                                                           |
| ----------------------- | ------------------------------------------------------------------------------------------------- |
| Device prompt           | Include device name and command, e.g., `ayalac-EDGE# show ip access-lists PROTECT-VM`             |
| Full command output     | Show the entire ACL with hit counts, _and_ the interface binding with direction                   |
| ACL name & direction    | Verify the ACL name (`PROTECT-VM`) and that it’s bound `out` on `GigabitEthernet0/0/2`            |
| Hit counts for each ACE | Ensure the **permit** and **deny** lines both have non-zero **match counters** (after your tests) |
| Comment                 | Add a confirmation line, e.g.:                                                                    |
|                         | `!-- PROTECT-VM is applied outbound on Gi0/0/2 and both ACEs have hits as expected.`              |

📘 **Sample Output Block**:
!-- Your matches may be different depending on how many pings you perform.
!-- This sample does not include the output of the pings

```bash
=== CO1 – Policy #1 - PROTECT-VM Verification ===
!-- PROTECT-VM applied outbound on Gi0/0/2; permit and deny counters show traffic matches  

ayalac-EDGE# show ip access-lists PROTECT-VM 
Standard IP access list PROTECT-VM     
10 permit 198.18.U.128, wildcard bits 0.0.0.63  (4 matches)     
20 deny   any                                   (4 matches)  

ayalac-EDGE# show ip interface GigabitEthernet0/0/2  | include PROTECT-VM
Outgoing access list is PROTECT-VM

ayalac-EDGE#show logging | include PROTECT-VM
*Jul  4 14:05:22.123: %SEC-6-IPACCESSLOGP: list PROTECT-VM permitted ip 198.18.100.129 (Ethernet0/2) -> 198.18.100.46
*Jul  4 14:06:10.456: %SEC-6-IPACCESSLOGD: list PROTECT-VM denied    ip 203.0.113.100 (Ethernet0/2) -> 198.18.100.46
```

> Your logs may look different, as this output was taken from CML nor the actual router.

Use this section to demonstrate that your ACL is correctly placed and actively enforcing the policy.

---

## Policy 2 – Protect the PC Network
### 1. Security Policy Statement

**Policy #2 – Protect the PC Network**  
To prevent IP-spoofing attacks against the corporate PC subnet (`198.18.U.128/26`), any packet whose **source** IP claims to be within that subnet, but does **not** originate from the trusted PC segment, shall be dropped and logged.  All other traffic, including legitimate external or remote sources, shall be permitted to reach the PC network.

>Packets from the Internet or the VM segment **use destination addresses in 198.18.U.128/26**, but our ACL is **not concerned with destination addresses** — it examines **source addresses** instead.  
>The goal is to **stop packets arriving from EDGE** that **pretend to have a source inside 198.18.U.128/26**, since those would spoof the PC network.

### Spoof Protection – Why It Matters

When untrusted devices inject packets claiming to come from inside your PC subnet, they can bypass perimeter defences and blend in with legitimate traffic. This practice, known as **IP spoofing**, enables attackers to:
- **Evade detection** by appearing as trusted hosts  
- **Launch reflection/amplification attacks** (e.g. DNS or NTP spoofing)  
- **Bypass ACLs or firewall rules** that permit traffic from “inside” addresses  

Our **PROTECT-PC** ACL serves as a **spoof-protection** measure: it explicitly denies any packet whose **source IP** does not belong to the approved PC range (`198.18.U.128/26`). By applying this ACL **inbound** on the **EDGE→CORE** transit network, we ensure that:

1. **Only genuine PC hosts** can send traffic into the network  
2. **Malicious or misconfigured devices** cannot masquerade as internal machines  
3. **Network integrity and auditability** are preserved, since all dropped or logged packets reveal potential attacks or misconfigurations  

IP spoof-protection is a fundamental security control, it stops attackers from forging internal addresses and gives you confidence that “inside” source IPs really do belong to trusted PCs.  

## 2. Policy Decomposition

Break down **Policy #2 – Protect the PC Network** into its core components.  Fill in the matching criteria and actions based on the security policy statement.

| Component          | Details                                             |
| ------------------ | --------------------------------------------------- |
| **Policy Name**    | `PROTECT-PC`                                        |
| **ACL Type**       | Standard                                            |
| **Match Criteria** | _(source IP range to match)_                        |
| **Actions**        | _(permit or deny)_                                  |
| **Device**         | _(DEVICE)_                                          |
| **Interface**      | _(e.g. the interface facing the PC subnet)_         |
| **Direction**      | _(inbound or outbound)_                             |
| **Logging**        | _(yes/no – interface logging and/or `log` keyword)_ |

## 3. ACL Placement

![Figure 3: Policy-2 ACL Placement](img/w10-Policy-2.png)

Use the table below to plan exactly where to apply `PROTECT-PC`.  Then answer the reflection questions to confirm your choices.

| Device | Interface             | Direction | Reason                                                                                       |
|--------|-----------------------|-----------|----------------------------------------------------------------------------------------------|
| CORE   |                       |           |                                                                                              |

**Reflection Questions:**
1. Trace the path of a packet **from the Internet/EDGE** to a PC in 198.18.U.128/26—on which router/interface does it arrive before hitting the PC network?  
2. Why must the ACL be applied **before** packets reach the PCs but **after** normal routing?  
3. Based on the above, which **interface** and **direction** will ensure only packets destined for the PC network are checked?
## 4. Configuration

> _[Configuration commands for PROTECT-PC will go here — students to complete based on their decomposition and placement.]_

## 5. Verification and Testing

Before you begin, create a spoof-address on **EDGE** to test denial of unauthorized sources:

```shell
EDGE(config)# interface Loopback130
EDGE(config-if)# ip address 198.18.U.130 255.255.255.255
EDGE(config-if)# exit
```

**Preparation:** Clear ACL counters before you begin 

```bash 
clear access-list counters PROTECT-PC
```

##### **Reachability Testing**

| Result | Test                  | Command                                       | Expected Result                                              |
| ------ | --------------------- | --------------------------------------------- | ------------------------------------------------------------ |
| ✅      | PC → CORE(legitimate) | `PC# ping 198.18.U.190`                       | 100% success;                                                |
| ❌      | Spoof → PC            | `EDGE# ping 198.18.U.129 source 198.18.U.130` | 0% success; packets dropped by PROTECT-PC                    |
| ✅      | REMOTE → PC           | `EDGE# ping 198.18.U.129 source 203.0.113.U`  | 100% success; traffic from genuine external hosts is allowed |
##### **ACL Counter Validation**
```bash
show access-lists PROTECT-PC
```
    
- Permit line hits should equal the number of successful pings.
- Deny line hits should equal the number of spoof attempts.
##### **Logging Check (if enabled)**
```bash
show logging | include PROTECT-PC
```

_Optional:_ Verify that denied matches were logged (if you left `log` on the `deny any` ACE).

### 🔍  CO2 – Verification and Collection of Information

In your `w10-username.txt` file, create a section labelled:


```diff
=== CO2 – Policy #2 - PROTECT-PC Verification ===
```

Copy the command and output of your pings:
```bash
PC# ping 198.18.U.190                           !-- PASS
EDGE# ping 198.18.U.129 source 198.18.U.130     !-- FAIL
EDGE# ping 198.18.U.129 source 203.0.113.U      !-- PASS    
```

Copy the output of these commands from the device you applied the AC::
```bash
show ip access-lists PROTECT-PC 
show ip interface GigabitEthernetX | include PROTECT-PC
show logging | include PROTECT-PC
```

**What to Include:**

| Requirement             | Details                                                                                       |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| Device prompt           | Include device name and command                                                               |
| Full command output     | Show the entire ACL with hit counts, _and_ the interface binding with direction               |
| ACL name & direction    | Verify the ACL name (`PROTECT-PC`) and that it’s bound `out` or `in` on  the interface        |
| Hit counts for each ACE | Ensure the **permit** and **deny** lines both have non-zero match counters (after your tests) |
| Comment                 | Add a confirmation line, e.g.:                                                                |
|                         | `!-- PROTECT-PC is applied to prevent spoofing addresses.`                                    |

---

## Policy 3 – Protect the ALS

### 1. Security Policy Statement

**Policy #3 – Protect the Switch Management Plane**  
To secure the switch’s control plane, only hosts in the corporate PC subnet (`198.18.U.128/26`) and the TFTP server may establish SSH sessions to the switch management interface.  All other attempts to access the management plane shall be denied and logged.

> **Implementation Note:**  
> We will enforce this with a **standard ACL** applied as an `access-class` on the switch’s VTY lines (0–4) and bound **inbound** on the management SVI.  

## 2. Policy Decomposition

Break down **Policy #3 – Protect the Switch Management Plane** into its core components.  Fill in the matching criteria and actions based on the security policy statement.

| Component          | Details                                                  |
| ------------------ | -------------------------------------------------------- |
| **Policy Name**    | `PROTECT-ALS`                                            |
| **ACL Type**       | Standard                                                 |
| **Match Criteria** | _(source IP ranges to match: PC subnet and TFTP server)_ |
| **Actions**        | _(permit or deny)_                                       |
| **Device**         | Switch                                                   |
| **Interface**      | _(management SVI - Vlan1)_                               |
| **Direction**      | _(inbound on VTY lines and/or on the SVI)_               |
| **Logging**        | _(yes/no – interface logging and/or `log` keyword)_      |

---

## ACL Placement

Plan where to apply the `PROTECT-ALS` ACL.  Then answer the reflection questions to confirm your choices.

| Device | Interface | Direction | Reason |
| ------ | --------- | --------- | ------ |
| Switch |           |           |        |

**Reflection Questions:**
1. Which interface(s) carry SSH/Telnet traffic destined for the switch’s management plane?  
2. Should the ACL be applied on the VLAN interface (SVI) or directly on the VTY lines?  
3. What direction (`in` or `out`) inspects SSH session attempts before they are processed by the control plane?

---

## 4. Configuration

> _[Translate your decomposition and placement into IOS commands to create `PROTECT-ALS`, apply it to the switch’s SVI and VTY lines, and enable logging as required.]_

---

## 5. Verification and Testing

Clear any existing ACL counters before testing:

```shell
ALS# clear access-list counters PROTECT-ALS
```

| Test                             | Command                                                | Expected Result                                                                      |
| -------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| SSH from PC (allowed)            | `PC# ssh admin@198.18.U.189`                           | ✅ Successful login prompt                                                            |
| SSH from TFTP server (allowed)   | `TFTP# ssh admin@198.18.U.189`                         | ✅ Successful login prompt - Ask your teacher to reach out to your switch.            |
| SSH from VM host (denied)        | `VM# ssh admin@198.18.U.189`                           | ❌ Connection refused or timeout                                                      |
| SSH (CORE from loopback99 _new_) | `CORE# ssh -l admin 198.18.U.189` source `198.18.U.99` | ❌ Connection refused or timeout                                                      |
| ACL Counter Validation           | `ALS# show access-lists PROTECT-ALS`                   | Permit ACEs have hits for PC & TFTP tests; Deny ACE has hits for VM & spoof attempts |

> **Tip:** Verify first that your switch’s management SVI (`show ip interface brief`) is up/up and has the correct IP before testing SSH.

### 🔍  CO3 – Verification and Collection of Information

In your `w10-username.txt` file, create a section labelled:


From your **ALS**:

```bash
show ip access-lists PROTECT-ALS 
show ip interface Vlan1 | include PROTECT-ALS
show running-config | section line vty
```

**What to Include:**

| Requirement             | Details                                                                                         |
| ----------------------- | ----------------------------------------------------------------------------------------------- |
| Device prompt           | Include device name and command                                                                 |
| Full command output     | Capture the entire ACL with hit counts, the SVI binding, and the VTY line ACL application       |
| ACL name & binding      | Verify the ACL name (`PROTECT-ALS`), that it’s applied to `Vlan1` **in**, and on `line vty 0 4` |
| Hit counts for each ACE | Ensure the **permit** entries for PC & TFTP and the **deny** entry all show non-zero matches    |
| Comment                 | Add a confirmation line, e.g.:                                                                  |
|                         | `!-- PROTECT-ALS bound to Vlan99 and VTY lines; all ACEs have expected hit counts.`             |

---
## Submission Checklist

- [ ] Submit `w10-username.txt` to the TFTP server.
- [ ] Include all `CO` sections:  CO1-CO3
- [ ] Upload updated configs to the TFTP server.
- [ ] Verify with:
```bash
ssh cisco@192.0.2.69
ls -l /var/tftp/*username*
```

>**Note:** Be sure to **save a local copy** of your configurations.  
   w11 lab will use the **same topology**, and having your configs available will save you time during setup.