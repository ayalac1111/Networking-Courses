
# BGP Quiz Solutions

## **[3 Points] eBGP Neighboring R1 – R3**

### **Configuration (1 Point):**
1. Enable BGP on R1:
    ```shell
    router bgp 65501
    neighbor 10.1.3.3 remote-as 65502
    network 10.1.3.0 mask 255.255.255.0
    ```

### **Explanation (2 Points):**
- **Establishing eBGP Peering:** eBGP peers require both ends to explicitly configure each other's AS numbers and IP addresses. 
  - R1 in AS 65501 and R3 in AS 65502 communicate over the 10.1.3.0/24 network.
  - BGP uses TCP port 179, ensuring reliable session establishment and route exchange.
- **Why AS Numbers and Interfaces are Critical:** AS numbers distinguish administrative domains. Directly connected interfaces or reachable addresses ensure a valid TCP connection.

---

## **[6 Points] Troubleshooting BGP Peering (R1 – R2)**

### **Problem Identification (2 Points):**
The BGP state is "Idle," indicating no active TCP session. Since 2.2.2.2 is a loopback interface, the possible issues are:
1. **TTL is not set to at least 2:** Loopbacks require TTL for multi-hop connections.
2. **Reachability Issue:** The loopback IP must be reachable from R1.
3. **Source Interface Not Configured:** The source must be set to R1's loopback.

### **Resolution (2 Points):**
1. **Increase TTL for multi-hop BGP:**
    ```shell
    router bgp 65501
    neighbor 2.2.2.2 ebgp-multihop 2
    ```
2. **Ensure reachability of 2.2.2.2:**
    - Verify the route:
      ```shell
      ping 2.2.2.2
      ```
    - Add a static route if missing:
      ```shell
      ip route 2.2.2.2 255.255.255.255 <next-hop>
      ```
3. **Set the source interface:**
    ```shell
    router bgp 65501
    neighbor 2.2.2.2 update-source Loopback0
    ```

### **Technical Explanation (2 Points):**
1. **TTL Configuration:** Default TTL for eBGP is 1. Increase it for multi-hop communication.
2. **Reachability:** The neighbor IP must be reachable for a TCP session to establish.
3. **Source Interface:** Ensures session stability when physical interfaces go down.

---

## **[6 Points] Understanding iBGP Behavior**

### **Behavior Explanation (2 Points):**
- R5 learned the 10.1.1.28/26 prefix from R3 via iBGP.
- The non-preferred (\*) status indicates a better path exists, likely via eBGP.

### **Routing Table Analysis (2 Points):**
- R5 will not install the prefix because iBGP relies on AS_PATH, and R3's configuration does not include next-hop-self.

### **Configuration Changes (2 Points):**
1. Apply next-hop-self on R3:
    ```shell
    router bgp 65501
    neighbor 5.5.5.5 next-hop-self
    ```
2. This ensures R5 uses R3 as the next hop, resolving reachability issues.

---

## **[6 Points] Analyzing the BGP Table on R1**

### **Path Analysis (2 Points):**
- R1 selects 2.2.2.2 for 10.1.2.5 because it has a lower metric (5 vs. 15).
### **Candidate Route Selection (2 Points):**
- For 10.1.4.0/24, the route via 2.2.2.2 is preferred due to the lower metric (5 vs. 15).
### **Path Information (2 Points):**
- The "?" symbol indicates an incomplete origin (e.g., a route manually redistributed into BGP), lowering its preference.

---