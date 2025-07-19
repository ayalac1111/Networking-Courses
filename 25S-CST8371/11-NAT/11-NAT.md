# Lab 11: Public-to-Private Address Translation Techniques


## 🧭 Overview

In this lab, you will deepen your understanding of IPv4 Network Address Translation (NAT) by configuring a single border router to translate traffic between private internal networks and the public Internet. You’ll work through three essential NAT scenarios:

1. **Port Address Translation (PAT)** on the router’s outside interface, allowing multiple inside hosts to share one public IP.
2. **Static port-forwarding**, mapping a specific TCP port on the public address to an internal server.
3. **Dynamic PAT using an address pool**, enabling inside hosts to translate to a small range of public IPs.

This hands-on exercise builds critical skills in conserving IPv4 address space and providing controlled inbound access—practical expertise for any enterprise network engineer.

---

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

- **Prepare interfaces for NAT** by assigning `ip nat inside` and `ip nat outside` on the correct router ports.
- **Create standard access-lists** to identify traffic for translation.
- **Define and apply a PAT overload** on the router’s public interface so multiple hosts share one IP.
- **Configure static port-forwarding** to permit inbound TCP connections on a specified public port.
- **Set up dynamic PAT with a pool** of public addresses for inside hosts.
- **Verify and troubleshoot NAT** using `show ip nat translations`, `show access-lists`, and end-to-end connectivity tests.
- **Document your work** by capturing configurations and command outputs for submission.
    
---

## 💡Why This Lab Is Important

Network Address Translation (NAT) is a foundational technique in IPv4 networks, allowing organizations to:

- **Conserve scarce IPv4 addresses** by enabling multiple internal hosts to share a single public IP (PAT) or a small pool of addresses (dynamic PAT).
- **Control and secure inbound access** through static port-forwarding, exposing only necessary services to the Internet while keeping internal networks hidden.
- **Maintain address consistency** when renumbering or merging networks, since internal addressing can remain unchanged behind a NAT boundary.
    
By mastering PAT, static port-forwarding, and dynamic NAT pools, you’ll gain practical skills that are essential for designing, operating, and troubleshooting real-world enterprise networks under IPv4 constraints.

---

## 🗺️ Network Topology

![Lab 11 Topology](img/11-Topology.png)

## 📘  Addressing Table (IPv4)

Use the table below to configure your devices. Replace `<U>` with your assigned student ID number.

| Device             | Interface | IP Address (CIDR)  | Description                                 |
| ------------------ | --------- | ------------------ | ------------------------------------------- |
| **RA**             | Gi0/0/0   | `203.0.113.U/24`   | Outside interface to Internet               |
| **RA**             | Gi0/0/1   | `198.18.U.17/29`   | Inside interface, OSPF neighbor to RB       |
| **RB**             | Gi0/0/0   | `10.U.18.1/28`     | Inside gateway for dynamic NAT pool hosts   |
| **RB**             | Gi0/0/1   | `198.18.U.22/29`   | OSPF neighbour to RA                        |
| **RB**             | Gi0/0/2   | `172.16.9.33/28`   | Host receiving static port‐forward (Telnet) |
| **RB**             | LoU       | `192.168.U.1/24`   | Private network                             |
| **VM (PAT host)**  | Gi0/0/2   | `172.16.9.46/28`   | Inside host for PAT translation tests       |
| **PC (Pool host)** | Gi0/0/0   | `10.U.18.14/28`    | Inside host for dynamic NAT pool tests      |
| **Remote**         | —         | `203.0.113.254/24` | External tester for Internet connectivity   |

> **VM Network Note**:
> **All students will use the same VM network (172.17.9.32/28) and configure identical IP addresses on their VMs.**  
> This demonstrates how overlapping private networks can coexist behind NAT: although every student’s VM will have the same internal address, the NAT on their border router will translate traffic uniquely to the public Internet.

---
## Partner Collaboration

You will need a partner for connectivity testing. If you don’t find a partner, ask your lab instructor to telnet into your switch.

- [ ]  You will each configure your own devices but work together to test connectivity. The topology diagram shows **only your own** devices; your partner will have the same topology on their end.
- [ ]  You need **one PC** and **one VM** each for testing.
- [ ] If you finish early, help your partner, but do **not** touch their keyboard.
- [ ] If your partner runs into a problem, guide them through troubleshooting. You **both** need full end-to-end connectivity for full marks.

---

## 🛠️ Initial Setup

### 0. Create submission file
- [ ]  On your desktop, create **three** files:
	- [ ] `11-OSPF-username.txt`
	- [ ] `11-NAT-username.txt`
	- [ ] `11-ACL-username.txt`
- [ ] You will later copy these files to the REMOTE TFTP server as proof of completion.
### 1. Basic Configuration
- [ ] Use the provided base configuration file: [basic.cfg](../Resources/basic.cfg)
- [ ] Enable real-time debug output display on all routers.
### 2. Addressing Configuration
- [ ] Configure addresses according to the topology diagram, paying attention to the network masks.
- [ ] Add a description to all Cisco interfaces.
- [ ] Ensure all interfaces are UP/UP before continuing.

---

## OSPF Configuration

- [ ] **Process ID**: Use `U` as the OSPF process number.
- [ ] **Router-IDs**: Manually set each router’s ID under OSPF:
	- [ ] `RA`:  `U.0.0.17`
	- [ ] `RB`: `U.0.0.22`
- [ ] **Default Gateway Advertisement**: On **EDGE**, redistribute or originate a default route so that `0.0.0.0/0` is known across the OSPF domain
- [ ] **CORE–EDGE DR Election**: **EDGE** should _not_ participate in the DR election.
- [ ] Configure explicitly all interfaces **not directly connected to an OSPF neighbour** as passive.
	- [ ] [-1 Point] deduction if using the `network` command.
- [ ] Set the OSPF **reference bandwidth** to `10000 Mbps`.
- [ ] Ensure all loopback interfaces report the **configured mask**, not a default host mask.
- [ ] On the `198.18.U.16/29` network, configure OSPF for **fast convergence** by setting:
	  - [ ] `Hello` interval: 3 seconds
	  - [ ] `Dead` interval: 5 seconds

## 🔍 CO1 - Verification and Collection of Information

📝 In your `11-OSPF-username.txt` file, create a section labelled:

```diff
=== CO1 – OSPF Verification ===
```

From **RA** and **RB**:
```bash
show ip protocols
show ip ospf interface brief
show ip ospf
show ip ospf neighbor
show ip ospf interface GigabitEthernet0/0/1
show ip route ospf | begin Gateway
```

✅ **What to Include:**

| ✅ Requirement                | Description                                                                      |
| ---------------------------- | -------------------------------------------------------------------------------- |
| 🖥️ Device Prompt & Commands | Include the device name and command for both commands                            |
| 📜 Full Output               | Copy **complete output** from both commands                                      |
| 👥 Neighbour Check           | Confirm **RA** has formed adjacencies with **RB**<br>**RB** is the **DR** router |
| 📌 Route Check               | Confirm that the OSPF routes                                                     |
| OSPF Interfaces              | Confirm **passive** interfaces<br>Confirm loopback mask                          |
| ⏱️ Timer Validation          | Confirm that **Hello = 3**, **Dead = 5** on `Gi0/0/1`                            |
| 🗒️ Comment                  | Add a verification note, e.g.:                                                   |
|                              | `!-- OSPF configuration check as per specs`                                      |


---

## <span style="color: red; font-weight: bold;">NEW</span> Services Configuration

### PC Service Configuration

On your PC (using TFTP64):

1. Launch **TFTP64** and open the **Settings** menu.
2. Enable the **TFTP** service and the **Syslog** service.
3. Set the **Base Directory** (or “Home Directory”) to your **Desktop** so that any transferred files and log files are easy to locate.

### Routers Configuration

On **both** RA and RB, enter global configuration mode and apply the following:

```bash
! 1. Synchronize time via NTP
ntp server 203.0.113.254

! 2. Send syslog messages to your PC’s Syslog server
logging trap informational
! (Optionally adjust trap level: debug, informational, warnings, errors, etc.)

! Specify which source interface’s IP shows up on the syslog messages
logging source-interface GigabitEthernet0/0/0

! Point syslog to the PC
logging host 10.U.18.14 transport udp port 514

! (Optional) Include timestamps in the log messages
service timestamps log datetime msec localtime
```

- **`ntp server 203.0.113.254`** ensures your router clocks are synchronized with the remote time source.
- **`logging trap informational`** sets the minimum severity level of messages sent to your
- **`logging source-interface`** picks the outbound interface whose IP is used as the syslog source address.
- **`logging host … transport udp port 514`** directs syslog packets to your PC on the standard syslog port.
- **`service timestamps`** adds human‐readable timestamps to each log entry.

On **RA**, enable the built-in HTTP and HTTPS web interface for management:

```bash
! Enable the HTTP server (clear-text)
ip http server
ip http authentication local

! Enable the HTTPS (secure) server
ip http secure-server
```

- `ip http server` turns on the HTTP daemon.
- `ip http secure-server` turns on the HTTPS daemon (uses TLS).
- `ip http authentication local` tells the router to check the local username database for web logins.


### 🔍 CO2 – Verification and Collection of Information

📝 In your `11-OSPF-<username>.txt` file, create four sections labelled:

```diff
=== CO2 – Services Verification ===
```

**Testing Preparation:**
1. **Console into RA** (your workstation) and  `ssh` into RB.
2. From your PC browser, open an HTTPS session to RA and log in with your credentials.

Copy the output of the commands listed below for **RA**:

```bash
show ssh
show ip ssh
show ip http server status
show ip http secure-server status
show users
show tcp brief
show running-config | section logging
show logging
show ntp status
show ntp associations detail
show clock
```

✅ **What to Include:**

| Requirement             | Details                                                                                           |
| ----------------------- | ------------------------------------------------------------------------------------------------- |
| 🖥️ Device prompt       | Include device name and command, e.g., `ayalac-RA# show ip ssh`                                   |
| 📜 Full command output  | Show the entire output of each command                                                            |
| 🔍 SSH status           | Verify `SSH Enabled - version 2.0`                                                                |
| 🔍 Active session       | In `show users`, confirm an active SSH session listed on RA                                       |
| 🔍 Test syslog entry    | Confirm syslog messages appears in PC’s Syslog window                                             |
| 🔍 HTTP status          | Confirm `HTTP server status: Enable`<br>Confirm `HTTP Secure server status: Enabled` (or similar) |
| 🔍 Active HTTPS session | In `show tcp brief                                                                                |
| 🗒️ Comment             | e.g., `!-- Services configured and verifed.`                                                      |

📘 **Sample Output Block** (partial):

```bash

!-- HTTPS session active; web management reachable.

ayalac-RA# show ip http server status
HTTP server status: Enabled

ayalac-RA# show ip http secure-server status
HTTP secure server status: Enabled

ayalac-RA# show tcp brief | include 443  
TCB       Local Address          Foreign Address        (state)
0x123456  198.18.U.17.443        10.U.18.14.51514       ESTAB

ayalac-RA# show users
    Line       User       Host(s)              Idle       Location
  *  0 vty 0  ssh     idle                 00:01:23  10.U.18.14 
  *  0 vty 1  http    idle                 00:00:10  10.U.18.14

!-- Syslog test message received at PC; RA logging config verified.

ayalac-RA# show logging
Syslog logging: enabled (console, host 10.U.18.14)
    Facility: daemon, Level: informational
  Timestamp logging: msec localtime
    No active filter modules.
  Console logging: disabled
    Buffer logging: level debugging, 50 messages logged
    Host 10.U.18.14:514, 10 messages dropped, 0 rate-limited


!-- NTP synchronized to 203.0.113.254; clock verified.

ayalac-RA# show ntp status
Clock is synchronized, stratum 2, reference is 203.0.113.254
nominal frequency is 250.0000 Hz, precision is 2**18
reference time is D4ED:86B3:8D3F.C000 (04:15:51.763 UTC Tue Jul 15 2025)

RA# show ntp associations detail

  address         ref clock       st  when  poll  reach  delay  offset    disp    jitter
=================================================================================
*~203.0.113.254   .REFCLK.        1   15    64    377   0.023   -0.001    0.040     0.005

associd=17 status=0124, leap_none, sync_ntp, auth_disable, port_ok
version=4, remote=203.0.113.254, local=198.18.<U>.17
delay=0.023, offset=-0.001, dispersion=0.040, jitter=0.005


```


