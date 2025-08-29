# OSPF Evaluation – Answer Key

This answer key provides model responses and explanations for each of the OSPF evaluation questions.

---

## 🔸 Question 1 – OSPF Route Analysis

1. OSPF routes will be listed with an "O" code. Cost to 192.0.2.0/24 depends on interface bandwidth; if using default 100 Mbps reference and Gigabit links, cost is 1 per link hop.
2. The path would be: RB → RA → E0/2 → REMOTE. Next-hop is RA’s IP on that segment.
3. Default reference bandwidth (100 Mbps) causes all Gigabit links to show the same cost (1), making path selection less accurate in high-speed networks.

---

## 🔸 Question 2 – Neighbourship Troubleshooting

1. The mismatched Hello/Dead timers prevent adjacency formation; both sides must match exactly.
2. Set the timers to match on both interfaces:
   ```bash
   ip ospf hello-interval 10
   ip ospf dead-interval 40
   ```
   or adjust both to 5/20.
3. If subnet masks differ, OSPF will not form an adjacency because it checks for matching subnet info on both ends of the link.

---

## 🔸 Question 3 – Default Route Propagation

1. Possible reasons:
   - RA doesn’t currently have a valid default route in its routing table.
   - RA needs `default-information originate always` if the default route is missing.
2. Add:
   ```bash
   default-information originate always
   ```
3. If RA continues to originate the default route without actually having one, it could cause routing loops or blackhole traffic at RB.

---

## 🔸 Question 4 – Router ID and Passive Interface

1. Router ID is selected from the highest loopback address if present. If none, it picks the highest active IPv4 address.
2. After configuring `router-id`, the OSPF process must be restarted:
   ```bash
   clear ip ospf process
   ```
3. Passive interfaces do not send OSPF Hello packets, preventing neighbor formation on non-routing links (e.g., loopbacks), while still advertising the network.

---

## 🔸 Question 5 – Cost Calculation and Path Preference

1. Cost = Reference Bandwidth (100 Mbps) / Interface Bandwidth
   - Cost per 100 Mbps link = 1
   - RA path cost = 1
   - RB via alternate path (if any) = higher cost depending on number of hops.
2. Set `ip ospf cost` on the undesired link to a higher value to force preferred path.
   Example:
   ```bash
   interface e0/1
     ip ospf cost 10
   ```
3. Changing reference bandwidth affects all interfaces network-wide; local cost changes override cost calculation **per interface** and are used for manual control.

