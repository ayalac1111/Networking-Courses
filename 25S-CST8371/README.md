
# 📘 25S-CST8371 – Enterprise Networking Labs

Welcome to the lab repository for **CST8371: Introduction to Enterprise Networking**  
This course focuses on scalable network design, IPv6, NAT, routing protocols, ACLs, and network management using Cisco equipment and simulation tools.

---

## 📅 Week by Week Activity Index

| Week   | Title                                                                        | Type         | Description                                                                                                                                                                         | Due        |
| ------ | ---------------------------------------------------------------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| 01     | [Static Routing, SVIs, and Review](./01-Lab-Review/README.md)                | ✅ Lab        | Configure inter-VLAN routing, static routes, and management access on multilayer switches.                                                                                          | **May 7**  |
| 01     | [IPv4 Troubleshooting Topology](./Examples/01-Example/01-Example.md)         | 👩🏽‍🏫 Demo | CML-based walkthrough of common mistakes in IPv4 configuration during the 01-Lab.                                                                                                   | __         |
| 02     | [IPv6 Static Routing](./02-Lab-IPv6/02-Lab-IPv6.md)                          | ✅ Lab        | Configure IPv6 LLA, GUA, ULA; test static/default routes using different next-hop formats.                                                                                          | **May 14** |
| 02     | [IPv6 EUI-64 Addressing](./Resources/ipv6-eui64-student.md)                  | ✏️ Work      | Practice generating interface identifiers with EUI-64.                                                                                                                              | __         |
| 02     | [IPv6 Address Shortening](./Resources/ipv6-shorten-student.md)               | ✏️ Work      | Learn how to abbreviate IPv6 addresses per RFC rules.                                                                                                                               | __         |
| 02     | [IPv6 Shortening (Advanced)](./Resources/ipv6-shorten-student-tricky.md)     | ✏️ Work      | Complex shortening examples to test student accuracy.                                                                                                                               | __         |
| 03     | [Dual-Stack Network](./03-PT-Dual-Stack/03-PT-Dual-Stack.md)                 | ✅ Lab        | Deploy a dual-stack network with static and floating routes.                                                                                                                        | **May 23** |
| 03     | [IPv6 Router Discovery](Resources/IPv6-Router-Discovery-SLAAC.md)            | ✏️ Work      | Understand how a host discovers a router and forms an IPv6 address using SLAAC.                                                                                                     | __         |
| 04     | [OSPF Basic Configuration](./04-Basic-OSPF/04-OSPF-Basic.md)                 | ✅ Lab        | Configure and verify basic OSPFv2 on a multi-router topology. Includes router ID selection, interface-based OSPF activation, default route redistribution, and convergence testing. | **May 28** |
| 04     | [OSPF Basic Configuration Init Network](Resources/Example_-_Basic_OSPF.yaml) | 👩🏽‍🏫 Demo | CML-based walkthrough basic OSPF Configuration - SBA Topology                                                                                                                       | __         |
| 04     | [No Quiz Sample](Resources/route_not_present.md)                             | ✏️ No Quiz   | Sample Quiz                                                                                                                                                                         | __         |
| 05     | [OSPF Tuning and Advanced Features](./05-OSPF-Tuning/05-OSPF-Tuning.md)      | ✅ Lab        | Tune OSPF behaviour: router IDs, network types, DR elections, Hello/Dead timers, passive interfaces, and reference bandwidth.                                                       | **June 4** |
| 05     | [Sample 01 SBA](Resources/Sample-01-SBA.md)                                  | 🟡 Optional  | Use to practice your SBA skills                                                                                                                                                     | __         |
| 05<br> | [OSPF Demo – Full Config](Examples/02-OSPF/OSPF-Demo.md)                     | 👩🏽‍🏫 Demo | In-class demo of OSPF configuration and tuning with reference topology and YAML import for CML.                                                                                     | __         |

##### **Legend**: 
✅ Required & Graded  
👩🏽‍🏫 In-Class Example
✏️ In-Class Participation  
🟡 Optional / Self-Guided

---

### 📁 Folder Structure:

```
25S-CST8371/
├── 0X-Lab/               # Week X lab
├── Examples/             # Sample configs, instructor demos
├── Resources/            # Packet Tracer files, templates, diagrams
└── README.md             # This file
```

> More activities will be added weekly as the course progresses.

---
<p style="font-size: 0.8em; text-align: center; color: gray;">
25S-CST8371 - C. Ayala
</p>
