# IPv6 Addressing and SLAAC – VLAN 150 Review

A PC boots up in **VLAN 150** with the MAC address `AC-DE-48-76-54-32`. The router on that VLAN advertises the IPv6 prefix `2010:ACAD:U:150::/64`. The PC uses SLAAC (Stateless Address Autoconfiguration) to configure its IPv6 address using the EUI-64 method.

---

## Questions

1. **GUA Generation**  
   Using the EUI-64 method, what Global Unicast Address (GUA) will the PC assign to its interface?  
   - Show the steps for generating the **Interface ID**.  
   - Write the complete **IPv6 address**.

2. **Router Solicitation (RS)**  
   When the PC sends a **Router Solicitation (RS)** message upon boot, what is the **IPv6 destination address** of that message?

3. **Router Advertisement (RA)**  
   After the RS, the router sends a **Router Advertisement (RA)** in response. What is the **destination IPv6 address** of the RA when responding directly to the host?

4. **Source Address in RS**  
   What is the **source IPv6 address** used by the PC when it sends the RS, before it has assigned a Global Unicast Address?

5. **Prefix Advertisement**  
   Which **network device** is responsible for advertising the IPv6 prefix `2010:ACAD:U:150::/64` to the PC?

---
