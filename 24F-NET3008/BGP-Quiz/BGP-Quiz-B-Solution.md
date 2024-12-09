
# BGP Quiz Solutions (Version 2)

## **[3 Points] eBGP Neighboring R1 – R2**

### **Configuration (2 Points):**
1. Configure eBGP on R1:
    ```shell
    router bgp 65501
    neighbor 2.2.2.2 remote-as 65502
    neighbor 2.2.2.2 update-source Loopback0
    neighbor 2.2.2.2 ebgp-multihop 2
    network 10.1.12.0 mask 255.255.255.0
    ```
2. Ensure reachability of the loopback address:
    ```shell
    ip route 2.2.2.2 255.255.255.255 <next-hop-IP>
    ```

### **Explanation (1 Point):**
To verify the eBGP peering:
- Use the command:
    ```shell
    show ip bgp summary
    ```
- Check that the state shows **"Established"** and confirm BGP neighbor details like AS number and received routes.

---

## **[6 Points] BGP Next-Hop and Route Validity**

### **1. Path Analysis (2 Points):**
- If R1 sends a traceroute to 10.1.4.5, it will take the path via `2.2.2.2`, as it has the lowest metric of **5** compared to the alternative via `3.3.3.3` (metric 15).

### **2. Next-Hop Significance (2 Points):**
- For the prefix 10.1.1.64/26, the `Next Hop` is `0.0.0.0`. This indicates that the prefix is locally originated, meaning R1 advertises this prefix as part of its own BGP table.

### **3. Troubleshooting (2 Points):**
- The `?` symbol for the prefix 10.1.3.0/24 indicates an **incomplete origin** (manually redistributed into BGP without specifying an origin attribute).
- **Resolution:**
    Add a proper origin attribute:
    ```shell
    router bgp 65501
    network 10.1.3.0 mask 255.255.255.0
    ```

---

## **[6 Points] iBGP Behaviour and Advertisement**

### **1. Behavior Analysis (2 Points):**
- R5 learned the 10.1.1.28/26 prefix from R3 via iBGP. It appears with a non-preferred (*) status because the next-hop `10.1.12.1` is unreachable from R5.

### **2. Routing Table Analysis (2 Points):**
- R5 will not install the prefix because BGP requires the next-hop to be reachable. Since the next-hop is not reachable, the route remains inactive.

### **3. Configuration Update (2 Points):**
- On R3, configure the `next-hop-self` command:
    ```shell
    router bgp 65501
    neighbor 5.5.5.5 next-hop-self
    ```
- **Effect:** R3 updates the next-hop for all advertised routes to its own IP, ensuring R5 can reach the prefixes.

---

## **[6 Points] Analyzing and Troubleshooting eBGP Peering**

### **1. Problem Identification (2 Points):**
- The BGP state on R1 is `Idle`, indicating that the peer `3.3.3.3` is not reachable. Possible causes:
  - Incorrect neighbor IP or AS configuration.
  - Missing route to `3.3.3.3`.

### **2. Resolution Steps (2 Points):**
1. Verify and correct the neighbour configuration:
    ```shell
    router bgp 65501
    neighbor 3.3.3.3 remote-as 65503
    ```
2. Ensure reachability to the neighbour IP:
    ```shell
    ip route 3.3.3.3 255.255.255.255 <next-hop-IP>
    ```

### **3. Verification (2 Points):**
- Use:
    ```shell
    show ip bgp summary
    ```
- Confirm the state is **"Established"** and validate with:
    ```shell
    show ip bgp neighbor 3.3.3.3
    ```

---

