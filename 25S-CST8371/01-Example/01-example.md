# 🧪 Example Walkthrough: Troubleshooting IPv4 Lab Topology

This document uses the same topology and IP addressing from [Lab 01](../01-Lab-Review/README.md). It walks through common misconfigurations and troubleshooting scenarios that students may encounter during or after completing the lab.

---
## 🖥️ TFTP Server Uses Alpine Linux

The TFTP server runs on Alpine and must be statically configured with an IPv4 address.
### 🛠️ Configuring a Static IP in Alpine:

Edit `/etc/network/interfaces` or manually set the interfaces:

```bash
ifconfig eth0 192.0.2.69 netmask 255.255.255.0 up
route add default via 1192.0.2.254 dev eth0
```

During this course, we will use Alpine Linux.
## 🛰️ Understanding REMOTE Configuration

The REMOTE switch connects to the wider network and includes the TFTP server. However, it must also be able to reach back into the student network.
### ❗ Problem 1:  REMOTE cannot ping student routers or subnets
#### 🧭 Why:
- REMOTE must have route backs to all students' networks.  
#### ✅ Solution:
- Add static routes using a supernet in the `/24`.
```bash
ip route 198.18.U.0 255.255.255.0 vlan203 203.0.113.U
```

### 🧠 Concept 1 - Directly Connected Networks
When you assign an IP address to an interface and bring it **up/up**, the router automatically adds that network to its **routing table** as a **directly connected route**. This means the router doesn’t need a static or dynamic route to reach it, it’s already local.

Shutdown the interface (e.g., `GigabitEthernet0/3`), and the directly connected network **disappears** from the routing table.

### 🧠 Concept 2 - Unknown packets in REMOTE
When **REMOTE** receives a packet destined for a network it doesn't recognize, one that isn’t in its routing table, the switch has no instructions on where to send it. Cisco switches will simply **drop the packet** silently. It's critical to have a **default route** or static entries that tell REMOTE how to reach internal networks. If that information is missing, any return packets, will be discarded without notification, leading to timeouts or failed uploads.

### 🧠 Concept 3 - Private addresses in Public networks
Private IP address ranges (like `192.168.0.0/16`, `10.0.0.0/8`, and `172.16.0.0/12`) are **not routable on the public Internet**.  They’re intended for use within internal networks only. If a device like **REMOTE** receives a packet destined for one of these ranges and no specific route exists, it will drop the packet. 
Even with a default route in place, many upstream networks (or REMOTE itself, if **configured** that way) may be configured to **filter out private addresses** for security reasons.


---

## 🌐 MLS – Inter-VLAN Routing with SVIs

MLS provides Layer 3 routing between VLANs (e.g., VLAN10, VLAN20).

Layer 3 switches provide **hardware-based routing between VLANs**, allowing inter-VLAN traffic to be handled internally without needing to send packets to an external router. This results in **faster performance**, lower latency, and **simpler cabling** — no trunk link congestion or single point of failure. Unlike ROAS (Router-on-a-Stick), which uses a single routed link and subinterfaces on a router, a Layer 3 switch can route directly between VLAN interfaces (SVIs) using its internal switching fabric. This makes Layer 3 switches ideal for **enterprise access and distribution layers**, where **scalability and throughput** are key.
### ❗ Problem 1: SVI Shows `down/down` Even Though Port Is Connected

You’ve configured your SVI (`Vlan10` or `Vlan20`) with an IP address and issued `no shutdown`, and your switchport is `connected` and assigned to the correct VLAN, but the **SVI is still stuck in `down/down`**.

#### ✅ Solution:
```bash
interface vlan 20
 shutdown
 no shutdown

```
#### Why This Works

SVIs depend on at least one **active Layer 2 port** in their VLAN to transition to `up/up`. In some cases, especially after VLAN or port configuration changes, IOS doesn’t immediately recognize that a port is ready to support the SVI. Doing a `shutdown` followed by `no shutdown` on the access port resets the interface state and forces the switch to:

- Reprocess **VLAN membership**
- Re-run **Spanning Tree Protocol (STP)**
- **Re-evaluate Layer 2 reachability** for the SVI

This triggers the SVI to detect the VLAN as active and bring it up.

### ❗ Problem 2: Inter-VLAN Routing Doesn’t Work

**Symptoms:**

- Devices in different VLANs (e.g., VM and PC) can’t ping each other
- SVIs are `up/up`
- Devices are correctly configured with IP addresses and default gateways.

**Cause:**
- MLS may be missing the `ip routing` command, which is required for inter-VLAN routing

#### ✅ Solution:
```
ip routing
```

### ❗ Problem 3: SVIs Remain `down/down` Despite Configuration

**Symptoms:**
- `show ip interface brief` shows SVIs like `Vlan10` and `Vlan20` in `down/down` state
- IP addresses are configured and `no shutdown` has been applied
    
**Cause:**
- No active switchport assigned to the VLAN, or connected devices are offline or shut down
#### ✅ Solution:
- Ensure that at least one physical port in the VLAN is `up`
- Recheck the VLAN assignment and the physically connected devices
- Use `shutdown` / `no shutdown` on the interface to refresh Layer 2 state


---

## 🚦 MLS – Default Route to REMOTE

### ❗ Problem 1: Packets are routed, but to the wrong place

- Routing to the wrong IP
``` bash
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/3 203.0.113.1
```
- Using the wrong interface, wrong address
``` bash
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/1 203.0.113.1
```
- Creating a routing loop
```bash
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/1 198.18.100.2
```


---

## 5. 🧱 MLS – Static Routes to R1 and R2 Loopbacks

Loopbacks are often used for testing reachability.

### ❗ Problem: Ping fails from PC to loopback
- No return route or wrong subnet mask in MLS route.

---

## 6. 🔍 ICMP Ping Response Symbols

| Symbol | Meaning                      | Cause / Explanation                                           |
| ------ | ---------------------------- | ------------------------------------------------------------- |
| `!`    | **Success**                  | ICMP Echo Reply received — the host is reachable.             |
| `.`    | **Timeout / No Reply**       | Packet dropped silently — no ICMP reply received.             |
| `!H`   | **Host Unreachable**         | No route to host — usually from intermediate device.          |
| `!N`   | **Network Unreachable**      | No route to the destination network.                          |
| `!U`   | **Destination Unreachable**  | ICMP error — often due to a firewall or missing return route. |
| `U`    | **Unreachable (Short Form)** | Seen in Linux; like `!U`, just abbreviated.                   |
| `?`    | **Unknown response**         | Unexpected code — often due to malformed packets or filters.  |

---

> Example will be updated after the class.