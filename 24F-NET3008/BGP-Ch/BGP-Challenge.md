# BGP Challenge Lab

## Topology

![Topology Diagram](img/BGP-Challenge-v2.png)


---
## Overview

This lab is designed to deepen your understanding of BGP by focusing on configuring and manipulating path attributes to influence route selection and advertisement. You will work with both eBGP and iBGP configurations, exploring the intricacies of neighbour relationships, AS path manipulation, MED configuration, and network advertisement. Additionally, you will integrate OSPF as an IGP within an autonomous system to facilitate iBGP connectivity.

You will learn how to apply BGP attributes to control route preferences and ensure efficient routing policies between and within autonomous systems. This knowledge is critical for managing large-scale, multi-AS environments and optimizing network traffic.

Key focus areas include:
1. Establishing eBGP and iBGP neighbour relationships using directly connected interfaces and loopbacks.
2. Manipulating BGP attributes such as `AS_PATH` and `MED` to influence routing decisions.
3. Advertising and filtering networks using prefix lists, AS-path access lists, and distribute lists.
4. Integrating OSPF with BGP to ensure reachability and seamless routing within an autonomous system.

By the end of this lab, you will have the foundational skills required to configure and troubleshoot BGP in real-world scenarios, as well as an appreciation for how BGP attributes shape the flow of traffic in enterprise and service provider networks.

---
## RouterOS Version Selection

### Recommended Version: RouterOS 6.49

For this lab, it is recommended that students use **MikroTik RouterOS 6.49** instead of the newer **RouterOS 7.x**. The decision to use RouterOS 6.45 is based on the following considerations:

### Why RouterOS 6.49?

1. **Alignment with Cisco's Configuration Style**:
   - RouterOS 6.45 features a BGP configuration process that is more closely aligned with Cisco's approach. This includes a hierarchical structure and similar command-line syntax, making it easier for students transitioning from Cisco devices to MikroTik.

2. **Simpler Learning Curve**:
   - Students who are familiar with Cisco's methods will find the configuration in RouterOS 6.49 more intuitive and easier to follow, minimizing potential confusion during the lab.

3. **Documentation and Examples**:
   - There is an abundance of documentation, examples, and community support for configuring BGP in RouterOS 6.49. In contrast, RouterOS 7.x, while more powerful and feature-rich, has limited command-line examples available.

---
## Initial Configuration

1. Use `eth1` in all devices to connect to a `host-only` management network using the address `192.168.100.0/24`.  Each device should use its ID as its IP address.  For example, T1, eth1 should be `192.168.100.1/24`
2. Use `internal network` for all other networks.
4. Set the hostnames of each device as shown in the topology and prepend your username.
5. Create all loopback devices using the **exact** names as shown in the topology. For example, the loopback 0 interfaces should be called `Lo0`
6. Configure the initial IP addresses for all routers as shown in the topology diagram.

---
## Configure OSPF 240 in AS 240

- **[1 Point]** Router IDs should follow the format `{U}.0.0.ID`, where `ID` represents the router number.
- **[1 Point]** Ensure Lo0 of T2 & T4 participate in OSPF for reachability of iBGP neighbours.

---
## eBGP Peering

- **[1 Point]** T1 & T2 should peer via eBGP using directly connected interfaces.
- **[1 Point]** T4 & T5 should peer via eBGP using the Lo0 of each router.
- **[1 Point]** T1 & T5 should peer via eBGP using the Lo0 of each router.
- **[1 Point]** Only static routes should be used to achieve connectivity to neighbours.
- **[1 Point]** Router IDs should follow the format `{U}.0.0.ID`, where `ID` represents the router number.

---
## iBGP Peering

- **[1 Point]** T2 & T4 should peer via iBGP using the Lo0 of each router.
- **[1 Point]** Ensure Lo0 of T2 & T4 participate in OSPF for reachability of iBGP neighbors.
- **[1 Point]** Ensure the `nexthop-choice` is correctly set.
