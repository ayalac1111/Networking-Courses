# 🧭 Midterm Objectives – IPv6, Static Routing, OSPF

This document outlines the learning objectives for each topic covered in the Week 7 Review Countdown. Each objective is categorized as:

- ✅ **Essential** – Core knowledge; must-know for success on the midterm
- 🟡 **Good to Know** – Helpful for deeper understanding and confidence
- 🔵 **Beyond** – Advanced knowledge; may appear in challenging scenarios

---

## Day 1 – IPv6 Addressing

- ✅ **Identify and classify IPv6 address types (GUA, LLA, multicast)**  
    → _Given an interface or routing table, determine the type and scope of each IPv6 address and explain its purpose._
    
- ✅ **Describe and perform IPv6 EUI-64 address generation**  
    → _Generate an IPv6 address using the EUI-64 method from a MAC address, and verify it on an interface using CLI._
    
- ✅ **Manually assign IPv6 addresses and verify with `show ipv6 interface` / `show ipv6 route`**  
    → _Configure static IPv6 addressing and confirm interface and routing behaviour using verification commands._
    
- 🟡 **Configure and verify IPv6 SLAAC operation, including observing RS/RA messages and checking autoconfigured addresses and prefixes**
    → _Enable SLAAC on a router and verify a host has autoconfigured its IPv6 address and default gateway correctly._
    
- 🟡 **Understand DAD (Duplicate Address Detection) process**  
    → _Observe DAD using debug commands and explain what happens when a duplicate address is detected._
    
- 🔵 **Explain why the same link-local address can appear on multiple interfaces of the same router**  
    → _Compare link-local addressing behaviour across multiple router interfaces and validate scope limitations._
    
- 🔵 **Describe how IPv6 address resolution works using Neighbor Discovery (NS/NA)**  
    → _Trace and interpret Neighbor Solicitation/Advertisement messages during communication between two IPv6 hosts._


---

## Day 2 – Basic Routing Concepts

- ✅ **Explain the purpose and syntax of a static route**  
    → _Configure static routes for both directly connected and next-hop scenarios, and explain their purpose based on topology needs._
    
- ✅ **Understand routing decisions based on the longest prefix match**  
    → _Analyze a routing table and determine which route will be selected for a specific destination IP based on prefix length._
    
- ✅ **Explain how administrative distance affects route selection**  
    → _Configure a static route with a custom AD, then compare it with a dynamic route to observe which one is installed._
    
- ✅ **Understand the difference between directly connected, static, and dynamic routes**  
    → _Identify route sources (`C`, `S`, `O`, etc.) in a routing table and explain how each was learned._
    
- 🟡 **Describe recursive route resolution**  
    → _Given a static route with only a next-hop address, use `show ip route` to verify the recursive lookup path._
    


---

## Day 3 – The Routing Table

- ✅ **Interpret and troubleshoot missing routes in `show ip route`**  
    → _Given a topology and config, use `show ip route` and interface status to determine why a route is missing._
    
- ✅ **Interpret `show ip route / show ipv6 route` and route types (`L`, `C`, `S`, `O`, `*`, `O*E2`)**
    → _Analyze routing table entries to identify route sources, next hops, and forwarding behaviour._
    
- ✅ **Explain how a route is selected for the routing table**  
    → _Compare multiple candidate routes (e.g., static vs OSPF) and explain which one is installed and why._
    
- ✅ **Understand how `next-hop` reachability affects static route installation**  
    → _Configure a static route and verify whether it is installed or rejected based on the reachability of its next-hop._
    
- 🟡 **Understand how to propagate a default route in OSPF to other routers in the domain**
	→ _Identify the configuration needed to inject a default route into OSPF and explain how it propagates through the domain._

---

## Day 4 – Static and Default Route Configuration

- ✅ **Configure and verify IPv4 and IPv6 static routes**  
    → _Use `ip route` and `ipv6 route` commands to create static routes, then verify installation using `show ip/ipv6 route` and `ping`._
    
- ✅ **Understand that when using link-local addresses (LLA) in static routes, the route must be fully specified (next-hop **and** exit interface)**  
    → _Configure a static IPv6 route with a link-local next-hop and validate its presence in the routing table._
    
- ✅ **Configure default routes with `0.0.0.0/0` and `::/0`**  
    → _Apply default static routes in both IPv4 and IPv6 environments and confirm they are used for unknown destinations._
    
- 🟡 **Configure floating static routes with adjusted AD**  
    → _Create a backup static route with a higher administrative distance and verify failover when the primary route fails._
    
- 🟡 **Compare next-hop only vs exit-interface static route forms**  
    → _Test both types of static routes and analyze their effect on recursive lookups and forwarding behaviour._
    
- 🟡 **Determine when to use a link-local vs global unicast address as the next-hop in an IPv6 static route**
	→ _Compare both approaches in lab scenarios and explain their requirements and impact on route resolution and interface dependency._


---

## Day 5 – OSPF Operation

- ✅ **Identify fields in OSPF Hello packets**  
    → _Use `debug ip ospf hello` or packet captures to identify fields like router ID, hello/dead timers, and priority._
    
- ✅ **List parameters required for OSPF neighbour adjacency (area, timers, mask, etc.)**  
    → _Configure matching parameters on two routers and verify that mismatches prevent neighbour formation._
    
- ✅ **Use `show ip ospf neighbor` to verify adjacency state**  
    → _Interpret output to confirm adjacency state (FULL, 2-WAY, etc.) and identify which neighbour is DR/BDR._
    
- 🔵 **Interpret INIT and 2-WAY states in adjacency troubleshooting**  
	→ _Identify causes of stuck states like INIT or 2-WAY using interface configs and `show ip ospf interface`._
	
- 🔵 **Explain multicast addresses used in OSPFv2 (224.0.0.5 and 224.0.0.6)**  
	→ _Describe which addresses are used for all OSPF routers vs. DR/BDR communications and validate them in Wireshark or debug output._

---

## Day 6 – OSPF Implementation

- ✅ **Configure OSPF by enabling it directly on interfaces**  
	→ _Use interface mode configuration (`ip ospf 1 area 0`) to assign interfaces to OSPF and verify participation with `show ip ospf interface`._
	
- ✅ **Verify OSPF-enabled interfaces with `show ip ospf interface`**  
    → _Use the command to confirm which interfaces are participating in OSPF, along with their area and hello/dead intervals._
    
- ✅ **Explain Router ID selection logic**  
    → _Predict or influence the Router ID selection by configuring loopback interfaces or using manual router-id commands._

- 🟡 **Configure loopbacks and passive interfaces**  
    → _Use loopbacks to ensure Router ID stability and configure passive interfaces to suppress OSPF hellos on non-routing links._
    
- 🟡 **Analyze a network topology to decide which interfaces should participate in OSPF**
    → _Determine OSPF-relevant interfaces based on connectivity and routing requirements._
    
- 🔵 **Understand that OSPF Router ID is non-preemptive and requires process reset**
    → _Use `clear ip ospf process` to force re-evaluation of the Router ID after changes to interface IPs or loopbacks._

---

## Day 7 – OSPF Tuning and Troubleshooting

- ✅ **Explain and influence DR/BDR elections**  
    → _Predict or influence which router becomes DR/BDR using priority settings and validate results with `show ip ospf neighbor`._
    
- ✅ **Understand Hello/Dead interval tuning and impact**  
    → _Modify Hello/Dead timers to control neighbour detection sensitivity and explain effects on adjacency formation._

- ✅ Understand how OSPF calculates cost to a network based on reference bandwidth  
  → *Manually calculate the cost of a route based on interface bandwidth and verify cost in `show ip ospf` output.*

- 🟡 **Use `debug ip ospf adj` and interpret the output**  
	→ _Run the debug command to trace OSPF state changes and understand the adjacency formation process._

- 🟡 **Explain the purpose of the OSPF reference bandwidth setting**
	→ _Use `show ip ospf` to verify the reference bandwidth across routers. Modify it using `auto-cost reference-bandwidth` and ensure it is consistently applied to all OSPF routers in the domain._

- 🔵 **Understand how to propagate a default route in OSPF to other routers in the domain**  
    → _Configure `default-information originate` and verify that OSPF neighbours receive and install the default route._
    
- 🔵 **Analyze OSPF log messages and state changes (`FULL to DOWN`)**  
    → _Interpret syslog or debug output to diagnose causes of adjacency flaps and state transitions._
