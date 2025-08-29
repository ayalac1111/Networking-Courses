# W01-LAB - Getting to Know Your Lab Environment

---
## Lab Overview
This Packet Tracer lab gets your toolkit working and establishes a **baseline router configuration** you’ll reuse all term. You will bring up the CORE–EDGE topology, apply minimal but essential settings, and verify each step with one clear command.

**You will:**
- Apply a sane base config on **CORE** and **EDGE**.
- Address interfaces from the provided plan (replace `U` with your assigned number).
- Build **reachability with static routes**.
- Secure remote access (**SSHv2** only, local user).
- Configure **NTP** (time), **Syslog** (remote logging), and **CDP** (neighbour discovery).
- Capture concise **evidence** for each checkpoint (C01–C07) in a small text file.

**What you submit:**  
`w01-pt-base-<username>.txt` containing evidence for C01–C07 (tag + one verification per item).  
*Keep it minimal—no full running-configs.*
**Estimated time:** 60–90 minutes (including verification and evidence capture).

---
## Learning Objectives
By the end of this lab, you should be able to:

1. **Interpret a topology & addressing plan** and apply IP settings to the correct interfaces.
2. **Establish basic reachability** using static routes and confirm with `ping`/`traceroute`.
3. **Harden management access** by enabling **SSHv2** only with a local admin account.
4. Configure management-plane services: **NTP**, **Syslog**, and **CDP**, using the appropriate **source interface**.
5. **Verify** each configuration with targeted `show` commands and **record minimal evidence** suitable for grading and troubleshooting.
6. Follow a **find → minimal fix → verify** habit you’ll reuse in audits and challenges.

---
## Why This Lab Is Important
- It creates a **repeatable baseline** so future labs (routing, ACLs, NAT, IPv6) don’t start from scratch.
- Time and logs (**NTP/Syslog**) make your troubleshooting **trustworthy and traceable**.
- Secure VTY access (**SSH-only**) is the foundation for safe remote management.
- Practicing **minimal configuration + one-line verification** builds the evidence discipline you’ll need for **audits** and **challenges**.
- Static routing here removes protocol complexity so you can **focus on essentials** first; dynamic routing arrives later.

### Topology
![W01 Topology](./img/w01-topology.png)  

*Addresses in red show network blocks; use your `U` value wherever shown.*

### Addressing Plan

| Device      | Interface  | Network        | IP/Mask              | Notes                        |
| ----------- | ---------- | -------------- | -------------------- | ---------------------------- |
| **CORE**    | G0/0/0     | 10.U.1.0/28    | **10.U.1.2/28**      | PC LAN — default gateway     |
| **CORE**    | G0/0/2     | 10.U.2.0/26    | **10.U.2.2/26**      | VM LAN — default gateway     |
| **CORE**    | G0/0/1     | 198.18.U.0/29  | **198.18.U.2/29**    | To EDGE (internal link)      |
| **CORE**    | LoopbackU  | 10.U.3.0/24    | **10.U.3.2/24**      | Loopback for tests/router-ID |
| **EDGE**    | G0/0/1     | 198.18.U.0/29  | **198.18.U.1/29**    | To CORE (internal link)      |
| **EDGE**    | G0/0/0     | 203.0.113.0/24 | **203.0.113.U/24**   | To REMOTE network            |
| **REMOTE**  | —          | 203.0.113.0/24 | **203.0.113.254/24** | Remote gateway (given)       |
| **Servers** | —          | 192.0.2.0/24   | —                    | Reachable via REMOTE         |
| **PC**      | NIC        | 10.U.1.0/28    | **10.U.1.10/28**     | DG = 10.U.1.2                |
| **VM**      | NIC        | 10.U.2.0/26    | **10.U.2.10/26**     | DG = 10.U.2.2                |
> Replace `U` with your assigned number.

---
## The Network Explained

**Roles**
- **CORE** — internal router: PC/VM LANs and the internal link to EDGE.
- **EDGE** — border router: connects the lab to the **REMOTE** network.

**Addressing pattern**
- Replace `U` with your assigned number. Examples: `10.U.1.0/28`, `198.18.U.0/29`, `203.0.113.U/24`.
- A small /29 between CORE↔EDGE keeps the internal link simple and point-to-point.

**Routing for Week 1 (static only)**
- **EDGE**: default route to the REMOTE gateway `203.0.113.254`.
- **CORE**: default route to **EDGE** (`198.18.U.1`).
- That’s enough to reach course services (NTP/Syslog) behind REMOTE without introducing OSPF yet.

**Management plane**
- **SSH only** on VTY (no Telnet).
- **NTP** to `192.0.2.123`; **Syslog** to `192.0.2.250`.  
  On EDGE, source management traffic from `G0/0/0` (the 203.0.113.0/24 side) for consistent reachability.
- **CDP** stays on for the CORE↔EDGE and EDGE↔REMOTE links so you can verify neighbours.

---

## Building the Network

> Replace every `U` with your assigned number. Keep configurations **minimal**.  
> After each Task, complete the matching **🔍 C0x — Verification & Collection of Information**.

### Create submission file
- [ ] On your desktop, create **`w01-pt-base-<username>.txt`**. You will submit this file to Brightspace.
### Task  0- — Basic Configuration (CORE & EDGE)
- [ ] Set **hostname** (CORE / EDGE) as `username-<devicename>` (eg: *ayal0014-EDGE*)
- [ ] Disable **DNS lookup**.
- [ ] Protect privileged exec with password `class` stored with strong encryption.
- [ ] Configure the console line to minimize disruptions caused by log messages.
### Task 1 — IP Addressing & Interface State
- [ ] Assign IPs per the **Addressing Plan/Topology** (replace `U` with your number).
- [ ] Add clear **descriptions** to each interface you configure.
- [ ] Ensure required interfaces are **up/up** (CORE↔EDGE link and EDGE↔REMOTE).

#### 🔍 C01 — Verification & Collection of Information
📝 In your `w01-pt-base-<username>.txt` file, create a section labelled:

```diff
=== CO1 – IP Addressing & Interface State ===
```

Copy the output of these commands from **EDGE** and **CORE**:

```bash
show ip interface brief 
```

✅ **What to Include:**

| Requirement              | Details                                                                       |
| ------------------------ | ----------------------------------------------------------------------------- |
| 🖥️ Device prompt        | Include device name and command, e.g., `ayalac-EDGE# show ip interface brief` |
| 📜 Full command output   | Paste the full `show ip interface brief` for **both** routers                 |
| 🔌 Interface state check | Required links show **up/up**                                                 |
| 🧭 Addressing check      | IPs match the addressing plan (`U` replaced with your assigned number)        |
| 🗒️ Comment              | Add a confirmation line, e.g.: `!-- IPs/state verified; link up both ways.`   |
📘 **Sample Output Block** _(example uses `U=7`; your values will differ)_

```
=== CO1 – IP Addressing & Interface State ===
!-- IPs/state verified; link up both ways

ayalac-EDGE#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     203.0.113.7     YES manual up                    up
GigabitEthernet0/1     198.18.7.1      YES manual up                    up

ayalac-CORE#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/1     198.18.7.2      YES manual up                    up
GigabitEthernet0/0     10.7.1.2        YES manual up                    up
```

---

### Task 2 — Static Routing (baseline reachability)

- [ ] On **EDGE**: install a **fully specified default route** **to REMOTE** (via `203.0.113.254`).
- [ ] On **CORE**: install a **fully specified default route** **to EDGE** (via `198.18.U.1`).
- [ ] On **EDGE**: install a **fully specified summary route** for **all internal networks** behind CORE.
- [ ] On **REMOTE**: install **fully specified return routes** **back via EDGE** so REMOTE can reach internal subnets:
  - `198.18.U.0/24` (internal link space)
  - `10.U.0.0/16`  (PC/VM/Loopback summary)

**Why the summary on EDGE?**  
EDGE must reach three internal networks that sit behind CORE:
- PC LAN `10.U.1.0/28`, VM LAN `10.U.2.0/26`, and LoopbackU `10.U.3.0/24`.  
A single summary **`10.U.0.0/16`** covers all of them → fewer routes, same reachability.

**Commands (replace `U` and interfaces to match your diagram):**
```plaintext
! ----- EDGE -----
ip route 0.0.0.0 0.0.0.0 203.0.113.254 GigabitEthernet0/0
ip route 10.U.0.0 255.255.0.0 198.18.U.2 GigabitEthernet0/1
!                             ^ next-hop ^ exit-if (fully specified)

! ----- CORE -----
ip route 0.0.0.0 0.0.0.0 198.18.U.1 GigabitEthernet0/1

! ----- REMOTE -----
! Return paths back via EDGE (EDGE’s WAN IP is 203.0.113.U)
ip route 198.18.U.0 255.255.255.0 203.0.113.U GigabitEthernet0/0
ip route 10.U.0.0    255.255.0.0   203.0.113.U GigabitEthernet0/0
!               ^ summaries back toward EDGE’s WAN
```

> **Full marks require “fully specified” static routes** (provide **next-hop IP** _and_ **exit interface**).

#### 🔍 C02 — Verification & Collection of Information
📝 In your `w01-pt-base-<username>.txt` file, create a section labelled:

```diff
=== CO2 – Static Routing & REMOTE Reachability ===
```

Run these commands on **EDGE** and **CORE**, then from the **PC**:

```bash
# Prove defaults (EDGE & CORE)
EDGE# show ip route | begin Gateway
CORE# show ip route | begin Gateway

# Prove internal summary on EDGE
EDGE# show ip route 10.U.0.0

# Prove return routes on REMOTE
REMOTE# show ip route 198.18.U.0
REMOTE# show ip route 10.U.0.0

# Connectivity tests
PC> ping 192.0.2.69                    # TFTP server behind REMOTE (end-to-end)
EDGE# ping 10.U.2.10 source g0/0/1     # to VM (validates EDGE→CORE→VM)
# (optional)
REMOTE# ping 10.U.3.2                  # to CORE LoopbackU (validates return paths)
CORE# traceroute 192.0.2.69            # should traverse EDGE, then REMOTE

```

✅ **What to Include:**

| Requirement               | Details                                                                                            |
| ------------------------- | -------------------------------------------------------------------------------------------------- |
| 🖥️ Device prompts        | Include device + command (e.g., `ayalac-EDGE# show ip route \| begin Gateway`)                     |
| 🌐 Defaults (EDGE & CORE) | Snippet showing **S*** `0.0.0.0/0` as **fully specified** on **both** routers                      |
| 🧭 EDGE internal summary  | `show ip route 10.U.0.0` showing **S 10.U.0.0/16 via 198.18.U.2, <internal-if>**                   |
| 🔁 REMOTE return routes   | `show ip route 198.18.U.0` and `show ip route 10.U.0.0` showing **next-hop 203.0.113.U**           |
| 💻 PC → REMOTE test       | PC ping to **192.0.2.69** (paste result; should succeed with return routes in place)               |
| 🔧 EDGE → VM test         | EDGE ping to **10.U.2.10** (source **g0/0/1**)                                                     |
| 🗒️ Comment               | Add: `!-- Fully specified defaults + EDGE /16 summary + REMOTE return routes verified; tests run.` |

📘 **Sample Output Block** _(example uses `U=7`; your values will differ)_

```bash
=== CO2 – Static Routing & REMOTE Reachability ===
!-- Fully specified defaults + EDGE /16 summary + REMOTE return routes verified; tests run.

ayalac-EDGE#show ip route | begin Gateway
Gateway of last resort is 203.0.113.254 to network 0.0.0.0
S*   0.0.0.0/0 [1/0] via 203.0.113.254, GigabitEthernet0/0

ayalac-CORE#show ip route | begin Gateway
Gateway of last resort is 198.18.7.1 to network 0.0.0.0
S*   0.0.0.0/0 [1/0] via 198.18.7.1, GigabitEthernet0/1

ayalac-EDGE#show ip route 10.7.0.0
Routing entry for 10.7.0.0/16
  Known via "static", distance 1, metric 0
  * 198.18.7.2, GigabitEthernet0/1

ayalac-REMOTE#show ip route 198.18.7.0
Routing entry for 198.18.7.0/24
  Known via "static", distance 1, metric 0
  * 203.0.113.7, GigabitEthernet0/0

ayalac-REMOTE#show ip route 10.7.0.0
Routing entry for 10.7.0.0/16
  Known via "static", distance 1, metric 0
  * 203.0.113.7, GigabitEthernet0/0

PC> ping 192.0.2.69
Pinging 192.0.2.69 with 32 bytes of data:
Reply from 192.0.2.69: bytes=32 time<1ms TTL=63
Reply from 192.0.2.69: bytes=32 time<1ms TTL=63
Reply from 192.0.2.69: bytes=32 time<1ms TTL=63
Reply from 192.0.2.69: bytes=32 time<1ms TTL=63
Ping statistics for 192.0.2.69:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

ayalac-EDGE#ping 10.7.2.10 source g0/0/1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.7.2.10, timeout is 2 seconds:
Packet sent with a source address of 198.18.7.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms

ayalac-REMOTE#ping 10.7.3.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.7.3.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/4 ms

ayalac-CORE#traceroute 192.0.2.69
Type escape sequence to abort.
Tracing the route to 192.0.2.69
  1  198.18.7.1  1 msec  1 msec  1 msec
  2  203.0.113.254  2 msec  2 msec  2 msec
  3  192.0.2.69  3 msec  3 msec  3 msec

```


---
### Task 3 — Secure Remote Access (SSHv2)
- [ ] Create a local **admin** user (privileged), with password **cisco**
- [ ] Set a **domain name** to `cnap.cst` (for key generation)
- [ ] Generate an RSA key modulus 1024 bits. 
- [ ] Ensure **SSHv2** is enabled (keys generated).
- [ ] **VTY** uses **login local** and **SSH-only** (no Telnet).
- [ ] Set a reasonable **exec-timeout** on VTY.

#### 🔍 C03 — Verification & Collection of Information
📝 In your `w01-pt-base-<username>.txt` file, create a section labelled:

```diff
=== CO3 – Secure Remote Access (SSHv2) ===
```

Run these checks:
```bash
# From the Packet Tracer PC to CORE (replace U):
ssh -l admin 198.18.U.2

# On CORE (verify the session landed)
show ssh
show tcp brief

# On EDGE (prove SSH server settings)
show ip ssh
```

✅ **What to Include:**

| Requirement             | Details                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| 🖥️ Device prompt       | Include device name + command (e.g., `ayalac-CORE# show ssh`)                                    |
| 🔐 Version proof        | From **EDGE**: `show ip ssh` showing **SSH Enabled – version 2.0**                               |
| 🌐 Domain name          | From **EDGE**: `show ip ssh` showing domain-name cnap.cst`                                       |
| 🔑 Key size             | From **EDGE**: `show ip ssh` showing  → include the **RSA modulus/key size**                     |
| 📡 Live session on CORE | From **CORE**: `show ssh` and `show tcp brief` showing the **active SSH connection**             |
| 💻 PC proof             | From **PC**: the `ssh -l admin 198.18.U.2` attempt (username prompt and success banner if shown) |
| 🗒️ Comment             | Add: `!-- SSHv2 enforced, domain & RSA key present, live session verified.`                      |

> **Full marks require**: SSH **version 2 only**, **domain name set**, **RSA keys present**, **VTY is SSH-only**, and a **successful SSH test** from the PC to CORE.

📘 **Sample Output Block** _(example uses `U=7`; your values will differ)_
=== CO3 – Secure Remote Access (SSHv2) ===
!-- SSHv2 enforced, domain & RSA key present, live session verified.

```bash
PC> ssh -l admin 198.18.7.2
Open
Password: ******
ayalac-CORE>           # (login success banner may vary)

ayalac-CORE#show ssh
Connection Version Mode  Encryption  Hmac        State        Username
0          2.0     IN    aes128-cbc  hmac-sha1   SESSION_OK   admin
%No SSHv1 server connections running.

ayalac-CORE#show tcp brief
TCP    10.7.1.10:51124    198.18.7.2:22       ESTAB

ayalac-EDGE#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
Minimum expected Diffie-Hellman key size : 1024 bits


ayalac-EDGE#show crypto key mypubkey rsa
% Key pair was generated at: 12:34:56 UTC Aug 28 2025
Key name: TP_self_signed_rsa_key
Storage: non-volatile
Key type: RSA KEYS
****    Key size: 1024 bits    ****
```

---

### Task 4 — NTP (time synchronization)
- [ ] On **EDGE**: act as an **NTP server of stratum 4**.
- [ ] On **CORE**: **sync time from EDGE**.
- [ ] Use the **internal link** as the NTP source on both devices.
- [ ] First, **set the clock** on EDGE so clients have a sane initial time.

**Suggested commands (copy/paste with your U value):**

```plaintext
! ----- EDGE -----
clock set 14:00:00 28 Aug 2025        ! set the current time
! (optional) clock timezone EST -5 0   ! or your local TZ

ntp master 4                           ! serve as stratum-4 master
ntp source g0/0/1                      ! source NTP from the CORE-facing link

! ----- CORE -----
ntp server 198.18.U.1                  ! EDGE's CORE-facing IP
ntp source g0/0/1                      ! source from the EDGE-facing link
```

> ⏱️ **Tip:** NTP may take ~10–60s to show as reachable/selected. In Packet Tracer it can be quicker, but give it a moment.

#### 🔍 CO4 — Verification & Collection of Information

📝 In your `w01-pt-base-<username>.txt` file, create a section labelled:

`=== CO4 – NTP (time synchronization) ===`

Run these checks:

```bash
# On EDGE (prove it is the server/master)
show ntp status
show ntp associations

# On CORE (prove it syncs to EDGE)
show ntp associations
show clock detail
```

✅ **What to Include:**

|Requirement|Details|
|---|---|
|🖥️ Device prompt|Include device name + command (e.g., `ayalac-EDGE# show ntp status`)|
|🕰️ EDGE master proof|From **EDGE**: `show ntp status` indicating **master clock** (stratum **4**)|
|🔗 Association view|From **EDGE**: `show ntp associations` (reference clock “.MASTER.” or reach > 0)|
|📡 CORE peer to EDGE|From **CORE**: `show ntp associations` where EDGE appears as **sys.peer** or reachable|
|⏱️ CORE clock source|From **CORE**: `show clock detail` showing **Time source is NTP**|
|🗒️ Comment|Add: `!-- EDGE serving stratum-4; CORE synced to EDGE (reach/peer confirmed).`|

📘 **Sample Output Block** _(example uses `U=7`; your values will differ)_

```bash
=== CO4 – NTP (time synchronization) ===
!-- EDGE serving stratum-4; CORE synced to EDGE (reach/peer confirmed).

ayalac-EDGE#show ntp status
Clock is synchronized, stratum 4, reference is .MASTER.
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz
reference time is DFB4A1A1.3A2B1C1C (14:00:33.226 UTC Thu Aug 28 2025)

ayalac-EDGE#show ntp associations
  address         ref clock     st  when  poll reach  delay  offset   disp
*~127.127.1.1     .MASTER.       4     4    64   377  0.0    0.00     1.0
 * master (synced), ~ configured, .MASTER. indicates local master

ayalac-CORE#show ntp associations
  address         ref clock     st  when  poll reach  delay  offset   disp
*198.18.7.1       .MASTER.       5     6    64   377  1.0    0.12     1.0
 * sys.peer = EDGE (reachable)

ayalac-CORE#show clock detail
14:00:40.123 UTC Thu Aug 28 2025
Time source is NTP

```


---

### Task 5 — Syslog (remote logging)
- [ ] Use the **VM (Packet Tracer Server)** as the Syslog server.
- [ ] Send Syslog from **CORE** and **EDGE** to the **VM**.
- [ ] **Source interface** for Syslog:
  - **CORE:** `LoopbackU`
  - **EDGE:** `GigabitEthernet0/0/0` (WAN/REMOTE)

**A) Prepare the VM (Packet Tracer Server)**
- Services → **Syslog** → **On** (no filters needed)

**B) Configure Syslog on routers**

```plaintext
! ----- CORE -----
logging host 10.U.2.10                       ! VM IP
logging trap informational
logging source-interface LoopbackU           ! LoopbackU (created earlier)
!

! ----- EDGE -----
logging host 10.U.2.10                       ! VM IP (reachable via static route above)
logging trap informational
logging source-interface GigabitEthernet0/0  ! use WAN/REMOTE as source per spec
!
```

>💡 **Tip:** To generate test events, `shutdown /no shutdown` an interface or do a single `login on-success` via console/VTY.  
> In PT, open the VM’s **Syslog** service to watch messages arrive.

#### 🔍 CO5 — Syslog (Verification & Collection of Information)

📝 In your `w01-pt-base-<username>.txt` file, create a section labelled:

```plaintext
=== CO5 – Syslog (remote logging) ===
```

Run these checks:

```plaintext
# On CORE
show logging | include 10\.|Trap|source-interface
show running-config | include logging host|logging source-interface

# On EDGE
show logging | include 10\.|Trap|source-interface
show running-config | include logging host|logging source-interface


# (Optional) On VM's Syslog pane
# confirm you see entries from CORE and EDGE
```

✅ **What to Include:**

|Requirement|Details|
|---|---|
|🖥️ Device prompt|Include device + command (e.g., `ayalac-CORE# show logging \| include ...`)|
|🎯 Remote host line|Show the **logging host** = `10.U.2.10` on **both** routers|
|🔊 Trap level|Show the **trap level = informational** on **both** routers|
|🧭 Source interface|Show **CORE → Loopback0** and **EDGE → G0/0/0** as the **logging source-interface**|
|🛣️ Edge reachability note|Include the `show ip route 10.U.2.0` from **EDGE** (route via CORE present)|
|🗒️ Comment|Add: `!-- Syslog to VM configured; source-if per spec; EDGE route to VM LAN confirmed.`|

📘 **Sample Output Block** _(example uses `U=7`; your values will differ)_
```plaintext
=== CO5 – Syslog (remote logging) ===
!-- Syslog to VM configured; source-if per spec; EDGE route to VM LAN confirmed.

ayalac-CORE#show logging | include 10\.|Trap|source-interface
Syslog logging: enabled (0 messages dropped, 0 messages rate-limited)
Logging to 10.7.2.10
Trap logging: level informational, 102 messages logged
Source interface is Loopback0

ayalac-CORE#show running-config | include logging host|logging source-interface
logging host 10.7.2.10
logging source-interface Loopback0

ayalac-EDGE#show logging | include 10\.|Trap|source-interface
Syslog logging: enabled (0 messages dropped, 0 messages rate-limited)
Logging to 10.7.2.10
Trap logging: level informational, 88 messages logged
Source interface is GigabitEthernet0/0

ayalac-EDGE#show running-config | include logging host|logging source-interface
logging host 10.7.2.10
logging source-interface GigabitEthernet0/0

ayalac-EDGE#show ip route 10.7.2.0
Routing entry for 10.7.2.0/26
  Known via "static", distance 1, metric 0
  * 198.18.7.2

ayalac-EDGE#ping 10.7.2.10 source g0/0/1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.7.2.10, timeout is 2 seconds:
Packet sent with a source address of 198.18.7.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms

```


---

### Task 6 — CDP Neighbour Discovery (internal link)
- [ ] Ensure **CDP** is running globally.
- [ ] Confirm CDP on the **CORE ↔ EDGE** interfaces.

> ℹ️ **Nothing to configure:** CDP is **enabled by default**. If someone disabled it, re-enable globally with `cdp run` (no need to include that in evidence).


#### 🔍 CO6 — Verification & Collection of Information

📝 In your `w01-pt-base-<username>.txt` file, create a section labelled:

```diff
=== CO6 – CDP Neighbour Discovery ===
```

Run this command **on EDGE only**:

```plaintext
show cdp neighbors
```


✅ **What to Include:**

|Requirement|Details|
|---|---|
|🖥️ Device prompt|Include device + command, e.g., `ayalac-EDGE# show cdp neighbors`|
|👥 Neighbour present|Table shows **CORE** as a neighbour|
|🔌 Ports match|**Local** = EDGE’s internal link (e.g., **Gi0/0/1**), **Port ID** = CORE’s matching port|
|⏳ Timing note|CDP entries can take up to ~60s to appear in PT—wait briefly if the table is empty|
|🗒️ Comment|Add: `!-- CDP up on internal link; CORE visible from EDGE as expected.`|

📘 **Sample Output Block** _(your interfaces may differ)_
```plaintext
=== CO6 – CDP Neighbour Discovery ===
!-- CDP up on internal link; CORE visible from EDGE as expected.

ayalac-EDGE#show cdp neighbors
Capability Codes: R - Router, S - Switch, H - Host, I - IGMP, r - Repeater, B - Bridge
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
CORE             Gig 0/0/1         153        R           ISR       Gig 0/0/1

```


---

## 📤 Submission Checklist (Brightspace)

- [ ] **File name:** `w01-pt-base-<username>.txt` (plain text, UTF-8)
- [ ] **Include all CO sections (in order):**
  - `=== CO1 – IP Addressing & Interface State ===`
  - `=== CO2 – Static Routing & REMOTE Reachability ===`
  - `=== CO3 – Secure Remote Access (SSHv2) ===`
  - `=== CO4 – NTP (time synchronization) ===`
  - `=== CO5 – Syslog (remote logging) ===`
  - `=== CO6 – CDP Neighbour Discovery ===`
- [ ] **Evidence style:** device prompt + command + minimal output only (no full running-configs, no screenshots)
- [ ] **CO2 note:** default routes must be **fully specified** (next-hop **and** exit interface) for full marks; include PC → `192.0.2.69` ping result
- [ ] **CO3 note:** prove SSHv2, domain `cnap.cst`, RSA key size, and a live SSH session from PC → CORE
- [ ] **CO4 note:** EDGE shows **stratum 4** master; CORE shows **peer** to EDGE and `Time source is NTP`
- [ ] **CO5 note:** both routers log to VM (`10.U.2.10`); **CORE source = LoopbackU**, **EDGE source = G0/0/0**
- [ ] **CO6 note:** `show cdp neighbors` from **EDGE** lists **CORE** with matching ports
- [ ] **Replace all `U`** with your assigned number throughout
- [ ] **Submit to Brightspace:** *Assignments → W01 PT Base Proof (5 pts)* by **Friday 3:30 PM**
