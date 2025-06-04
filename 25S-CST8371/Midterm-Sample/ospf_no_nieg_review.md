# OSPF Operation – Neighbour Adjacency
![Midterm Topology](img/midterm-sample-topology.png)

Routers **RA** and **RC** are directly connected via their `GigabitEthernet0/2` interfaces and are both configured for **OSPF in area 0**.

However, the OSPF neighbour relationship is **not forming** between the two routers.

---

## Routing Table Output – RA

```
RA# show ip route connected
C    10.U.10.0/24 is directly connected, GigabitEthernet0/0
L    10.U.10.1/32 is directly connected, GigabitEthernet0/0
C    10.U.20.0/24 is directly connected, GigabitEthernet0/1
L    10.U.20.1/32 is directly connected, GigabitEthernet0/1
C    10.U.40.0/28 is directly connected, GigabitEthernet0/2
L    10.U.40.1/32 is directly connected, GigabitEthernet0/2
```

---

## Routing Table Output – RC

```
RC# show ip route connected
C    10.U.50.0/24 is directly connected, GigabitEthernet0/0
L    10.U.50.1/32 is directly connected, GigabitEthernet0/0
C    10.U.40.0/24 is directly connected, GigabitEthernet0/2
L    10.U.40.2/32 is directly connected, GigabitEthernet0/2
```

---

## Questions

1. Why are RA and RC **not forming an OSPF neighbour adjacency**, based on the routing information provided?

2. OSPF routers exchange hello packets to determine neighbour compatibility. List **three OSPF interface parameters** that must match between two routers for a neighbour adjacency to form.

3. How does the **difference in connected route masks** affect the outcome of the **OSPF hello packet exchange**?

4. Which command would you use to confirm that RA is not seeing RC as a neighbour?

5. What change could be made to **resolve the adjacency issue** between RA and RC?

---
