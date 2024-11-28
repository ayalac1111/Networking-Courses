# **SBA Demo for 24F-NET3008**

## **Overview**
During this demo, I will guide you through the **key tasks and configurations** you will encounter in the SBA. The objective is to ensure you are familiar with the topology, the requirements, and the types of configurations expected.

### **Demo Objectives**
1. Understand the **network topology** and its components.
2. Review the initial setup, including **IP addressing** and connectivity verification.
3. Demonstrate basic configurations for:
   - **EIGRP**
   - **OSPF**
   - **BGP**
4. Provide an overview of **redistribution** and **path control** tasks.
5. Answer your questions to clarify SBA expectations.

---

## **1. Topology Overview**
![Network Topology](SBA-Topology.png)

### **Key Points in the Topology**
- **EIGRP (AS123)** is configured between **EIGRP-1** and **RDIS-2**.
- **OSPF (Process ID 200)** is configured between **RDIS-2** and **ASB-3**.
- **BGP (AS34)** is configured between **ASB-3** and **BGP-4**, using both directly connected interfaces and loopbacks.
- Links between routers:
  - **EIGRP-1 ↔ RDIS-2**: `10.200.12.0/24` and `10.200.21.0/24`
  - **RDIS-2 ↔ ASB-3**: `10.200.23.0/24`
  - **ASB-3 ↔ BGP-4**: `10.200.34.0/24` and `10.200.43.0/24`
  - Initial configuration for basic IP addressing will be given.
  - You will have to configure extra loopback devices to emulate routes throughout the SBA.

---
## **2. Initial Setup** (Estimated Time:  20 minutes)
### Tasks:
1. **Verify IP Addressing**:
   - Show the IP addresses for each router using:
    ```bash
     show ip interface brief
     ```
   - Confirm that interfaces are up and correctly assigned.

2. **Verify Basic Connectivity**:
   - Use **ping** to ensure Layer 3 connectivity between directly connected routers.

---

## **3. Routing Protocol Configuration** (Estimated Time: 1 hour)
### **Total Marks: 8 Points**

### [ 2 Points ] **EIGRP Configuration**
1. [ No Points given for basic configuration ] **AS123 Configuration**: 
   - Enable EIGRP on **EIGRP-1** and **RDIS-2**.
   - Verify adjacencies:
     ```bash
     show ip eigrp neighbors
     ```

2. **Possible extras to configure**:
	- Inject an external network.
	- Adjust hello interval and hold timer.
	- Set router as stub routers.
	- Implement EIGRP summarization for the loopback routes.
	- Use all the feasible paths between the routers.

---

### [3 Points]**OSPF Configuration**
1. **Process ID 200**:
   - Enable OSPF on **RDIS-2** and **ASB-3**
   - Verify adjacencies:
     ```bash
     show ip ospf neighbor
     ```

2. **Possible extras to configure**:
	- Configure multi-area OSPF.
	- Adjust hello interval and hold timer.
	- Set an area as stub area or totally stubby area
	- Implement OSPF summarization for the loopback routes.

---

### [ 3 Points] **BGP Configuration**
1. **eBGP Configuration**:
   - Configure **eBGP** on **ASB-3** and **BGP-4**
	   - Using directly connected or loopback devices
1. **Verify BGP Neighbours**:
```bash
   show ip bgp summary
```

2. **Route Advertisement**:
    - Advertise network prefixes in BGP.

---

## **4.  Route Filtering, Redistribution, and Path Control** (Estimated Time: 1 hour)
### **Total Marks: 7 Points**

### [ 2 Points] **Route Filtering**
1. Create a set of loopback devices in a router.
2. Advertise the routes via a routing protocol.
3. Use a prefix-list or a distribution list to filter out the routes.
### [ 2 Points] **Redistribution**
1. Perform `redistribution` from one protocol to another, such as EIGRP to OSPF.
2. Use a `route-map` to apply a metric or change a value in the redistribution.
### [ 3 Points] **Path Control**
1. Use a `route-map` to modify attributes from BGP.
2. Apply PBR to specify the `next-hop` of a route.

---

## **Key Guidelines for the SBA**

1. **Basic Protocols Are Enough to Pass**
    - The SBA is designed so that you can pass by configuring only the **basic routing protocols** (EIGRP, OSPF, and BGP). Ensure these are fully functional before attempting advanced tasks.
2. **Summarization is Critical**
    - **Summary routes must be as small as possible** to optimize routing and prevent unnecessary advertisements. Review your summarization calculations carefully.
3. **EIGRP Variance**
    - The **variance** setting should be precise. Avoid using excessively large values, as they will not be accepted. Test your configuration to ensure the correct routes are included.
4. **Route Filtering and Control**
    - Review and practice using:
        - **Prefix lists**
        - **Distribute lists**
        - **Route maps**
    - These concepts form a major portion of the SBA and will be essential for tasks involving route filtering and redistribution.
5. **BGP Path Selection**
    - Be familiar with modifying the following attributes for BGP path selection:
        - **Local Preference**
        - **Multi-Exit Discriminator (MED)**
        - **Weight**
    - For **AS-PATH Prepending**, use the following command in Cisco
        `set as-path prepend 650014 650014 650014 650014`

---

## **SBA Arrival Instructions**

1. **Personal Items**
    - Place all your items at the front of the lab.
    - Do **not** keep your phones with you.
    - Do **not** bring or use a USB memory stick.
2. **Equipment and Materials**
    - You are allowed your **written** lab book.  If you don't have a lab book you are allowed a set of **written** notes on paper.  You must leave the papers behind.
    - It is considered **academic dishonesty** to use another student's note for your SBA.
    - You will be provided with **10 cables**. Use them wisely, as no extra cables will be given.
    - Do **not** turn off any equipment during the SBA. Ensure all devices remain powered on to avoid loss of progress or configurations.

---

## **Final Reminders**

- Plan your time effectively.
- Verify each configuration step as you proceed.
- If in doubt, refer to your lab notes.
- I'm there to support you in case of an equipment failure, but I will not be helping any student to remember commands or interpret the SBA.
