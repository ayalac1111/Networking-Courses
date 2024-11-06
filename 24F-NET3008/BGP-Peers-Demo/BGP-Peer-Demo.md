# BGP Peering and Network Advertising Demo

This demo walks through configuring eBGP on two routers to establish neighbour relationships and advertise networks. We start with directly connected interfaces and then move to using loopback interfaces for peering.

## Topology

**Routers**:
- **Router A** (AS 65001)
- **Router B** (AS 65002)

### IP Addressing
Using public documentation IP addresses for the lab to simulate a real-world setup.

| Router   | Interface   | IP Address        | ASN     |
| -------- | ----------- | ----------------- | ------- |
| Router A | `G0/0`      | `198.51.100.1/24` | `65001` |
| Router B | `G0/0`      | `198.51.100.2/24` | `65002` |
| Router A | `Loopback0` | `192.0.2.1/32`    | `65001` |
| Router B | `Loopback0` | `203.0.113.2/32`  | `65002` |

---

## Part 1: Basic eBGP Peering (Directly Connected Interfaces)

### Objective
Establish an eBGP neighbour relationship between Router A and Router B using their directly connected interfaces. Then, advertise a network from each router to verify that routes are exchanged.

### Key BGP Concepts

1. **Administrative Distance (AD)**: BGP has an **Administrative Distance (AD) of 20** for external routes. This low AD makes BGP routes highly preferred in the routing table when compared to other protocols like OSPF (AD 110) or EIGRP (AD 90), allowing BGP routes to be selected first when multiple routes to a destination exist.
    
2. **eBGP Peer Relationship via TCP**: eBGP peers establish their relationship over a **TCP connection on port 179**. Unlike IGP protocols like EIGRP and OSPF, which use their specific protocols, BGP’s reliance on TCP provides reliability and ordered packet delivery, crucial for maintaining stable BGP sessions.
    
3. **TTL of 1 for eBGP Sessions**: eBGP uses a **TTL (Time-To-Live) of 1** by default to limit neighbour relationships to directly connected peers. This setting prevents unintended BGP relationships with distant routers, helping contain BGP peering within the intended network boundaries unless explicitly configured otherwise (e.g., for multi-hop eBGP).
    
4. **Network Advertisement Requirement**: BGP allows advertising networks that are not directly connected. However, for a network to be advertised, it must be **present in the routing table**. This means a route can originate from any protocol (static, connected, OSPF, etc.), as long as it appears in the routing table for BGP to advertise it to other BGP peers.

5. **Loop Prevention**: BGP includes mechanisms such as `AS Path` to prevent routing loops, vital in larger networks where multiple ASes may have overlapping routes.

### Steps

1. **Configure eBGP on Router A**:
   - Set the router-id.
   - Set up eBGP and configure Router B as a neighbour.
   - Advertise the network `192.0.2.0/24` to Router B.
   
   ```bash
   # Router A Configuration
   router bgp 65001
     bgp router-id 1.0.0.1
     neighbor 198.51.100.2 remote-as 65002
     network 192.0.2.0 mask 255.255.255.0
	```

2. **Configure eBGP on Router B**:
    - Set the router-id.
    - Set up eBGP and configure Router A as a neighbour.
    - Advertise the network `203.0.113.0/24` to Router A.
    
     ```bash
    
    # Router B Configuration 
    router bgp 65002   
      bgp router-id 2.0.0.2
      neighbor 198.51.100.1 remote-as 65001   
      network 203.0.113.0 mask 255.255.255.0
    ```
    
3.  **Verify BGP Neighbour Relationship**:
    - Use the `show ip bgp summary` command to confirm that the eBGP session is **established**.
    
4. **Verify Route Propagation**:
	- Check that each router has received the advertised network from its neighbour.
	    - On **Router A**, check for `203.0.113.0/24` in the BGP table.
	    - On **Router B**, check for `198.51.100.0/24` in the BGP table.

5. **Other Verification commands:**
	- Check the bgp table with the command `show ip bgp`

---
```bash 

RA#show ip bgp summary
BGP router identifier 1.0.0.1, local AS number 65001
BGP table version is 7, main routing table version 6
2 network entries using 264 bytes of memory
2 path entries using 104 bytes of memory
1/1 BGP path/bestpath attribute entries using 184 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 1) using 32 bytes of memory
BGP using 632 total bytes of memory
BGP activity 2/0 prefixes, 2/0 paths, scan interval 60 secs

Neighbor V AS MsgRcvd MsgSent TblVer InQ OutQ Up/Down State/PfxRcd
198.51.100.2 4 65002 23 22 7 0 0 00:02:18 4

RA#show ip route bgp
B 203.0.113.0/24 [20/0] via 198.51.100.2, 00:00:00
  
RA#show ip bgp
BGP table version is 7, local router ID is 1.0.0.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

Network           Next Hop       Metric   LocPrf Weight     Path
*> 192.0.2.0/24   0.0.0.0        0        0       32768     i
*> 203.0.113.0/24 198.51.100.2   0        0       0         65002 i
```

### **BGP table** 
A database on a router that stores information about BGP routes received from neighbours, routes originated by the router itself, and any routes redistributed into BGP. Each entry in the BGP table contains various attributes that help BGP determine the best path to a destination.
#### Column Breakdown

1. **Network**:
    - This column lists the destination network prefixes known to the BGP process. These prefixes are advertised by BGP neighbours or originated by the router itself.
    - In this example:
        - `192.0.2.0/24` and `203.0.113.0/24` are destination networks in the BGP table.
2. **Next Hop**:
    - This column shows the IP address of the **next hop** router to reach each destination.
    - If the next hop is `0.0.0.0`, it indicates that the route was **originated by the local router** itself (as in the case of `192.0.2.0/24`).
    - For `203.0.113.0/24`, the next hop is `198.51.100.2`, meaning this route was learned from a neighbour with IP address `198.51.100.2`.
3. **Metric**:
    - This field is used in **multi-protocol BGP (MP-BGP)** to track additional route characteristics, but it often shows `0` in traditional IPv4 BGP.
    - It typically doesn’t play a significant role in standard BGP path selection; however, it may be used in certain advanced BGP configurations.
4. **LocPrf (Local Preference)**:
    - Local Preference (LocPrf) indicates the preference for a route **within the local AS**. A higher LocPrf value is preferred over a lower one.
    - In this example, both routes have a LocPrf of `0`, the default value when Local Preference isn’t explicitly set.
5. **Weight**:
    - The **Weight** attribute is a Cisco-proprietary value that is only relevant on the local router and is not shared with other routers.
    - Routes with higher Weight are preferred over routes with lower Weight.
    - In this example, `192.0.2.0/24` has a Weight of `32768`, indicating it’s a **locally originated route** (Cisco automatically sets Weight to `32768` for locally originated routes). The route learned from a peer (`203.0.113.0/24`) has a default Weight of `0`.
6. **Path**:
    - The **AS Path** column lists the Autonomous Systems that the route traversed to reach the destination, with the most recent AS listed at the end.
    - The `i` (internal) at the end of each Path indicates whether the route is internal or external to the AS:
        - `i` signifies that the route is **internal** (originated within the same AS).
        - `e` would signify an **external** route (received from a different AS).
    - In this example:
        - `192.0.2.0/24` has `i`, indicating it was originated by the router itself.
        - `203.0.113.0/24` has `65002 i`, indicating it was learned from AS `65002` and is treated as an internal route in the local AS.

#### Understanding Symbols and Path Selection

- **Symbols (`*>`)**:
    - `*`: Indicates that this route is valid and usable.
    - `>`: Indicates that this route is the **best path** and will be used in the routing table.
    - In the table above, both routes (`192.0.2.0/24` and `203.0.113.0/24`) have the `*>` symbol, meaning they’re valid and selected as the best paths.
- **Best Path Selection**:
    - BGP uses a selection process to determine the best path for each destination, considering attributes like Weight, Local Preference, AS Path, Origin, and MED. In this example, since both routes have only one valid path, each route is selected as the best.

---

# Part 2: eBGP Peering Using Loopback Interfaces

## Objective
Modify the configuration to use the loopback interfaces for BGP peering. This approach is commonly used for stability and resilience in BGP connections, especially in larger networks.

### Updated Topology for Loopback Peering
In this configuration:
- **Router A (RA)** uses `192.0.2.1/32` as its loopback for peering.
- **Router B (RB)** uses `203.0.112.2/32` as its loopback for peering.

To make the loopbacks reachable, we need to configure static routes pointing to the directly connected interfaces between RA and RB.

---

## Steps

1. **Add Static Routes for Loopback Reachability**:
   - On **Router A (RA)**, add a static route to reach Router B’s loopback.
     ```bash
     ip route 203.0.112.2 255.255.255.255 198.51.100.2
     ```
   - On **Router B (RB)**, add a static route to reach Router A’s loopback.
     ```bash
     ip route 192.0.2.1 255.255.255.255 198.51.100.1
     ```

2. **Update BGP Configuration on Router A (RA)**:
   - Change the BGP neighbour IP to Router B's loopback IP `203.0.112.2`.
   - Set the `update-source` to use Router A's loopback interface.
   - Enable `ebgp-multihop` to allow BGP packets to traverse more than one hop.

   ```bash
   # Router A Configuration for Loopback Peering
   router bgp 65001
     neighbor 203.0.112.2 remote-as 65002
     neighbor 203.0.112.2 update-source Loopback0
     neighbor 203.0.112.2 ebgp-multihop 2
```

3. **Update BGP Configuration on Router B (RB)**:
    - Change the BGP neighbour IP to Router A's loopback IP `192.0.2.1`.
    - Set the `update-source` to use Router B's loopback interface.
    - Enable `ebgp-multihop` similarly.

		```bash
    # Router B Configuration for Loopback Peering 
    router bgp 65002   
      neighbor 192.0.2.1 remote-as 65001   
      neighbor 192.0.2.1 update-source Loopback0   
      neighbor 192.0.2.1 ebgp-multihop 2
    ```
    
4. **Verify BGP Neighbor Relationship**:
    - Use the `show ip bgp summary` command on both routers to confirm the BGP session over the loopback addresses.
    - The neighbour should be established with the loopback IPs now.
5. **Verify Route Propagation**:
    - Confirm that each router still receives the advertised network from its peer over the loopback peering.

---

## Key Points

1. **Loopback Peering for Stability**: Using loopback interfaces for peering adds stability, as it allows BGP to maintain the session as long as there is an available path between the loopbacks, even if a specific physical link goes down.
    
2. **Use of `update-source` Command**:
    - The `update-source` command is essential when peering with a neighbor's loopback interface. By default, BGP uses the IP address of the outgoing interface as the source IP for establishing the TCP session on port 179.
    - Specifying `update-source Loopback0` instructs BGP to use the **loopback IP** instead, ensuring sessions are established with the intended address. This consistency is crucial in networks with multiple paths, where the physical interface IP might vary.
    
3. **TTL and `ebgp-multihop` Requirement**: Since loopback-to-loopback peering generally involves more than one hop, the **TTL (Time-To-Live)** must be adjusted. The `ebgp-multihop` command allows BGP packets to traverse additional hops to reach the neighbour, in this case set to 2.