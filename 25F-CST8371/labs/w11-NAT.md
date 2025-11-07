# Lab 11: Public-to-Private Address Translation Techniques


## Overview

In this lab, you will deepen your understanding of IPv4 Network Address Translation (NAT) by configuring a single border router to translate traffic between private internal networks and the public Internet. You’ll work through three essential NAT scenarios:

1. **Port Address Translation (PAT)** on the router’s outside interface, allowing multiple inside hosts to share one public IP.
2. **Static port-forwarding**, mapping a specific TCP port on the public address to an internal server.
3. **Dynamic PAT using an address pool**, enabling inside hosts to translate to a small range of public IPs.

This hands-on exercise builds critical skills in conserving IPv4 address space and providing controlled inbound access—practical expertise for any enterprise network engineer.

---

## Learning Objectives

By the end of this lab, you will be able to:

- **Prepare interfaces for NAT** by assigning `ip nat inside` and `ip nat outside` on the correct router ports.
- **Create standard access-lists** to identify traffic for translation.
- **Define and apply a PAT overload** on the router’s public interface so multiple hosts share one IP.
- **Configure static port-forwarding** to permit inbound TCP connections on a specified public port.
- **Set up dynamic PAT with a pool** of public addresses for inside hosts.
- **Verify and troubleshoot NAT** using `show ip nat translations`, `show access-lists`, and end-to-end connectivity tests.
- **Document your work** by capturing configurations and command outputs for submission.
    
---

## Why This Lab Is Important

Network Address Translation (NAT) is a foundational technique in IPv4 networks, allowing organizations to:

- **Conserve scarce IPv4 addresses** by enabling multiple internal hosts to share a single public IP (PAT) or a small pool of addresses (dynamic PAT).
- **Control and secure inbound access** through static port-forwarding, exposing only necessary services to the Internet while keeping internal networks hidden.
- **Maintain address consistency** when renumbering or merging networks, since internal addressing can remain unchanged behind a NAT boundary.
    
By mastering PAT, static port-forwarding, and dynamic NAT pools, you’ll gain practical skills that are essential for designing, operating, and troubleshooting real-world enterprise networks under IPv4 constraints.

---

## Network Topology

![Lab 11 Topology](img/w11-topology.png)

## Addressing Table (IPv4)

Use the table below to configure your devices. Replace `U` with your assigned student ID number.

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

> 💡 **VM Network**  
> All students use **172.16.9.32/28** for their VM subnet.  
> Assign your VM **172.16.9.46/28**, identical across students, to illustrate how private networks overlap behind NAT.


---
## Partner Collaboration

You will need a partner for connectivity testing. If you don’t find a partner, ask your lab instructor to telnet into your RB.

- [ ]  You will each configure your own devices but work together to test connectivity. The topology diagram shows **only your own** devices; your partner will have the same topology on their end.
- [ ]  You need **one PC** and **one VM** each for testing.
- [ ] If you finish early, help your partner, but do **not** touch their keyboard.
- [ ] If your partner runs into a problem, guide them through troubleshooting. You **both** need full end-to-end connectivity for full marks.

---

## Initial Setup


### 0 — Prep: download automation tools (CO0)

**Goal:** Get the evidence-collection tools onto your Desktop before starting. You’ll use them in later tasks.

**Steps**
- [ ] Download **x-remote** from GitHub and place the script on your **Desktop** — https://github.com/ayalac1111/x-remote
  *(Open the repo → `src/x_remote.py` → download the raw file to your Desktop.)*
  *(Open the repo → `requirements.txt` → download the raw file to your Desktop.)*
- [ ] Install the requirements
```python
pip install -r requirements.txt
```
- [ ] Download `w11-ospf.yaml` [here](yaml/w11-ospf.yaml)  
- [ ] Extract files to your Desktop folder.  
- [ ] Open the YAML file and replace `{username}`and `{U}`variables with your information.
- [ ]  On your desktop, create **ONE** file: `11-NAT-{username}.txt`
### 1. Basic Configuration
- [ ] Use the provided base configuration file: [basic.cfg](../resources/basic.cfg)
- [ ] Enable real-time debug output display on all routers.
### 2. Addressing Configuration
- [ ] Configure addresses according to the topology diagram, paying attention to the network masks.
- [ ] Add a description to all Cisco interfaces.
- [ ] Ensure all interfaces are UP/UP before continuing.
- [ ] Console to **RA** and ssh into **RB** using the user `cisco/cisco`
### 3. Services
- [ ] Configure **RA** as the authoritative NTP master of stratum 4.
- [ ] Synchronize **RB**'s time with **RA**, using the IP address of their directly connected link.
- [ ] Prevent the router from using its own IP as a source when sending to TFTP.
- [ ] Direct `syslog` messages to the PC.  Note that **RA** messages will be sent once you configure OSPF.
	- [ ] Use the Gi0/0/0 interface as the syslog source.
	- [ ] Send informational (and higher) severity messages.
	- [ ] Include date/time on log entries

```bash
! --- SYSLOG Configuration ---
! Direct syslog messages to the PC
R(config)# logging host 10.U.18.14 transport udp port 514

! Use the Gi0/0/0 interface as the syslog source
R(config)# logging source-interface GigabitEthernet0/0/0

! Send informational (and higher) severity messages
R(config)# logging trap informational

! Include date/time on log entries
R(config)# service timestamps log datetime msec localtime
```

### 4. PC - Extra Configuration.
On your **PC** (using **TFTP64**):
- [ ] Launch **TFTP64** and open the **Settings** menu.
- [ ] Set the **Base Directory** (or “Home Directory”) to your **Desktop**.  
- [ ]  Enable the **TFTP** and the **Syslog** services.

> *Tip:* You can monitor real-time syslog messages in the **Syslog** tab while testing ping and routing events.

---
## Task 1 - OSPF Configuration

- [ ] **Process ID**: Use `U` as the OSPF process number.
- [ ] **Router-IDs**: Manually set each router’s ID under OSPF:
	- [ ] `RA`:  `U.0.0.17`
	- [ ] `RB`: `U.0.0.22`
- [ ] **Default Gateway Advertisement**: On **EDGE**, redistribute or originate a default route so that `0.0.0.0/0` is known across the OSPF domain
- [ ] **RA–RB DR Election**: Ensure **RB** becomes the DR by setting its priority to `U`. 
- [ ] Configure explicitly all interfaces **not directly connected to an OSPF neighbour** as passive.
- [ ] Set the OSPF **reference bandwidth** to `10000 Mbps` for accurate cost calculation on gigabit links.
- [ ] **Convergence Tuning:**
	- [ ] **RB-LoU**: Configure as a `point-to-point` OSPF network type.
	- [ ] On the `198.18.U.16/29` link, set the `hello 3` and `dead 6`.
	- [ ] **RB** interfaces towards the PC and the VM network should _never_ become a DR by setting its priority to `0`.
## CO1 – Verification and Collection of Information

Before collecting evidence, review your lab notes and the network specifications carefully.  
Ensure all configurations match the addressing plan, OSPF process settings, and service requirements described in this lab.  

Use your **Lab Book** and the **W11 network specifications** to determine which operational commands will confirm correct behaviour.  
Verify that all devices are reachable and that OSPF adjacencies, DR/BDR elections, timers, and passive interfaces are functioning as expected.  
Confirm that Syslog, NTP, and TFTP services are operating as configured.

Once you have verified that the network is stable and fully functional, execute the automated collector to gather your CO1 verification outputs:

```bash
python3 x-remote.py w11-ospf.yaml
```

This script connects to the routers, executes the required OSPF verification commands defined in the **`w11-ospf.yaml`** file, and saves the results to: **`w11-ospf-{username}.txt`**

### Submit Verification File

- [ ] Submit **`w11-ospf-{username}.txt`** to the TFTP server.
- [ ] Upload the syslog file to the TFTP server; label this file as `11-SYSLOG-{username}.txt`
- [ ] Upload router configurations to the PC's TFTP server.

> **NOTE**:  Do not include the curly brackets as part of your username; they represent that the {username} is a variable.

---

## Task 2 - NAT Rule #1: Translate to the Public Address of the Exit Interface

**Hosts in the VM Network 172.16.9.32/28 must share RA’s public IP when accessing the Internet.**

1. **Identify the NAT device**
    - Device: **RA** (the border router with the public‐facing interface)
2. **Set inside/outside interfaces**
    ```bash
    ! On RA
	interface GigabitEthernet0/0/1
	  ip nat inside
	interface GigabitEthernet0/0/0
	  ip nat outside
    ```
    
3. **Create a standard ACL (16) to match the inside network**
	```bash
	access-list 16 permit 172.16.9.32 0.0.0.15
	```
    
4. **Enable PAT to the exit interface’s public IP**
    ```bash
    ip nat inside source list 16 interface GigabitEthernet0/0/0 overload
    ```

	Command explanation:
	- **`ip nat inside source`**  
	    Specifies that we’re translating source addresses of packets coming from the **inside** network.
	- **`list 16`**  
	    Uses the **standard ACL 16** to identify which source addresses to translate (in our case, every host in `172.16.9.32/28`).
	- **`interface GigabitEthernet0/0/0`**  
	    Tells the router to use the IP address assigned to **Gi0/0/0** (the outside/public interface) as the translated address.
	- **`overload`**  
	    Enables **Port Address Translation (PAT)**, which lets **all** matched inside hosts share that single public IP by mapping their source port numbers.
	    Put simply, this one line says:

> For any packet sourced from `172.16.9.32/28`, translate its private source IP to the public IP on Gi0/0/0; and because of `overload`, allow many private hosts to reuse that one public IP by differentiating them via port numbers.

### Testing NAT with DNS and ICMP

1. **Set VM's DNS server** to 192.0.2.53 in your network settings.
2. From VM: **Perform a DNS lookup**
	```bash
	VM# nslookup www.cnap.cst
	Server:         192.0.2.53
	Address:        192.0.2.53#53

	Non-authoritative answer:
	Name:   www.cnap.cst
	Address: 192.0.2.80
	```
3. From VM: Perform a ping to www.cnap.cst
4. **Inspect the NAT translation table**
	```bash
	RA# show ip nat translations
	```
	
!-- Sample Output
	<table border="1" cellpadding="4" cellspacing="0">
  <thead>
    <tr>
      <th>Pro</th>
      <th>Inside Global</th>
      <th>Inside Local</th>
      <th>Outside Local</th>
      <th>Outside Global</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>UDP</td>
      <td>203.0.113.17:52345</td>
      <td>172.16.9.46:52345</td>
      <td>192.0.2.53:53</td>
      <td>192.0.2.53:53</td>
    </tr>
    <tr>
      <td>ICMP</td>
      <td>203.0.113.17:1</td>
      <td>172.16.9.46:1</td>
      <td>192.0.2.80:0</td>
      <td>192.0.2.80:0</td>
    </tr>
  </tbody>
</table>

6. **Verify ACL and statistics**
```bash
RA# show access-lists 16
RA# show ip nat statistics
```
- **ACL 16** should report ≥ 2 hits.
- **Total active translations** should be ≥ 2.
### CO2 – Verification and Collection of Information

In your `11-NAT-{username}.txt` file, create a section labelled:

```diff
=== CO2 – PAT to Exit Interface Verification ===
```

Under this header, perform the following steps and include the outputs as described.


| Step                         | Command(s)                 | What to Include                                                                                        |
| ---------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------ |
| **1. Show NAT translations** | `show ip nat translations` | - Full translation table, showing an entry for ICMP from `172.16.9.46` → `203.0.113.U` to `192.0.2.80` |
| **2. Verify ACL hits**       | `show ip access-lists 16`  | - The ACL 16 permit statement  <br>- Hit count ≥ 1 indicating the subnet was matched                   |
| **3. Show NAT statistics**   | `show ip nat statistics`   | - Total active translations ≥ 1  <br>- Hits/Misses summary for inside source translations              |

#### Sample Output Block

```bash
=== CO2 – PAT to Exit Interface Verification ===
!-- PAT to exit interface functioning; ICMP translations and ACL verified.

ayalac-RA# show ip nat translations
Pro  Inside global        Inside local         Outside local       Outside global
ICMP 203.0.113.17:1       172.16.9.46:1        192.0.2.80:0         192.0.2.80:0

ayalac-RA# show ip access-lists 16
Standard IP access list 16
    permit 172.16.9.32 0.0.0.15 (hit count: 2)

ayalac-RA# show ip nat statistics
Total active translations: 1 (0 static, 0 extended)
Outside interfaces: 1
  GigabitEthernet0/0/0
Hits: 2  Misses: 0

```


---

## Task 3 -  NAT Rule #2: Static Port‐Forwarding

**“Allow Internet users to Telnet to our internal RB at 172.16.9.33:23 via RA’s public IP on TCP 2323.”**

- [ ] Code a **port-forwarding** rule so that external hosts can connect to the telnet server on the RB. 
    ```
    ip nat inside source static <tcp/udp> <in_addr port> <out_add port>
    ```
    - [ ] Telnet is a TCP protocol
    - [ ] Internally, the RB telnet server is at `172.16.9.33` port `23`.
    - [ ] Outside hosts should connect to `203.0.113.U` port `2323`.

**Testing:**
- [ ] From PC, telnet to your partner's router `203.0.113.P` and login.  Keep the connection open, 
- [ ] From PC, telnet to your partner's RB `203.0.113.P 2323` and login.  Keep the connection open. 
- [ ] From your partner PC, telnet to your router `203.0.113.U` and login.  Keep the connection open
- [ ] From your partner PC, telnet to your RB ``203.0.113.U 2323` and login.  Keep the connection open.

> `P`:  Your partner's `U`
> You are testing your partner's configuration.  They are testing your configuration.

### CO3 – Port-Forwarding Verification

In your `11-NAT-{username}.txt` file, under the label:

```diff
=== CO3 – Static Port Forwarding Verification ===
```

Copy and paste the outputs (including device prompts) of the following commands from **RA** / **RB**, then add your confirmation comment below the header:

| Requirement            | Details                                                                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| NAT translations       | `RA:  show ip nat translations`  <br>Confirm a static entry mapping `203.0.113.U:2323` → `172.16.9.33:23`                             |
| TCP listener check     | `RB: show tcp brief`  <br>Ensure there is a LISTEN or ESTAB on local port 2323                                                        |
| Active Telnet sessions | `RB: show users`  <br>Confirm at least one `vty` line with protocol `telnet` from your PC or partner’s PC to RA and to the RB is open |

 **What to Include:**

| Requirement             | Details                                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Device prompt & command | Include device name and exact command,                                                                                     |
| Full command output     | Capture the entire output of each command without truncation                                                               |
| NAT translation entry   | In `show ip nat translations`, confirm a static entry mapping `220.0.0.U:2323` → `172.16.9.33:23`                          |
| TCP listener            | In `show tcp brief` ensure there is a `LISTEN` (or `ESTAB`) on local port 2323 on RA                                       |
| Active Telnet sessions  | In `show users`, confirm at least one `vty` line with protocol `telnet` from your PC (or partner’s PC) to RA and to the RB |
| Comment                 | e.g., `!-- Static port-forward for Telnet to 172.16.9.33:23 verified; TCP listener, and sessions confirmed.`               |

#### Sample Output Block

```bash
=== CO3 – Port-Forwarding Verification ===
!-- Static port-forward for Telnet to 172.16.9.33:23 verified; TCP listener and sessions present.

ayalac-RB# show ip nat translations
Pro Inside global          Inside local         Outside local        Outside global
tcp 220.0.0.U:2323        172.16.9.33:23       10.P.18.14:54321  10.P.18.14:54321

ayalac-RB# show tcp brief | include 2323
TCB     Local Address         Foreign Address      (state)
0xABC   220.0.0.U.2323       10.P.18.14.56789     LISTEN

ayalac-RB# show users
    Line       User       Host(s)              Idle       Location
   *  0 vty 2  telnet    idle                 00:00:12  10.P.18.14

```

> Be sure you leave each Telnet session open while you capture these outputs.


---

## Task 4 - NAT Rule #3: Dynamic PAT with Address Pool

**Enable dynamic PAT for your PC network (`10.U.18.0/28`) using a pool of public addresses on RA**.

- [ ] Configure NAT with overload for the network `10.U.18.0/28` to an address from the pool 
    - [ ] Identify the nat inside and outside interfaces.
    - [ ] Code access-list `18` to permit the inside private addresses `10.U.18.0/28`.
    - [ ] Set up the NAT pool called `NAT_POOL` using addresses `209.10.U.2 - 209.10.U.6 / 29`.
    - [ ] Code the translation rule with an overload to translate host addresses permitted by access-list 18 to an address from the NP NAT pool.

### CO4 – Dynamic PAT Pool Verification

In your `11-NAT-{username}.txt` file, create a section labelled:

```diff
=== CO4 – Dynamic PAT Pool Verification ===
```

Generate two types of traffic from your **PC (10.U.18.14)**:

1. **ICMP test**: Ping to 192.0.2.69
2. **TFPT test**:  Upload a file to the TFTP server; it could be your current `11-NAT-{username}.txt"

On **RA**:
```bash
show ip nat translations
show ip nat statistics
show access-lists 18
```

**What to Include:**

| Requirement             | Details                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Device prompt & command | Include device name and exact command for each, e.g., `ayalac-RA# show ip nat translations`                         |
| Full command output     | Copy the entire output of each command without truncation                                                           |
| Translation entries     | In `show ip nat translations`, confirm:<br> • An ICMP entry for `192.0.2.53` <br>• A UDP entry for port `69` (TFTP) |
| ACL hit count           | From `show access-lists 18`:<br>verify `permit 10.U.18.0 0.0.0.15` shows a matches count ≥ 2                        |
| Comment                 | e.g., `!-- Dynamic PAT pool NAT_POOL functioning; ICMP and TFTP translations verified.`                             |
> This confirms that inside hosts in `10.U.18.0/28` are translating via addresses in pool `NAT_POOL`, for both standard ICMP and UDP/TFTP traffic.

### Submit Verification File

- [ ] Submit `11-NAT-{username}.txt` to the TFTP server.
- [ ] Include all `CO` sections:  CO2-CO4
- [ ] Upload the syslog file to the TFTP server; label this file as `11-NAT-SYSLOG-{username}.txt`
- [ ] Upload router configurations to the PC's TFTP server - For you to keep a copy.
