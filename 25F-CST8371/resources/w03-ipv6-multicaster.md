# Understanding the Basics of Multicasting in IPv6

> **Goal**: Understand what multicast is and why it's used in IPv6.

##  What is Multicasting?

In networking, **multicasting** is the method of sending a message **to a selected group of devices**, not everyone. It’s more efficient than **broadcasting**, which sends the message to **every device on the network**, even those that don’t care.

> ✋🏽 **Important:** IPv6 multicast is mostly used in **local or private networks**, not across the open Internet.  
> 
> It operates within the **link-local** or **site-local** scope, such as inside a LAN or enterprise.  

### Analogy: **Bluetooth Speaker Party Mode**

Imagine you're in a student lounge, and someone starts streaming music using a Bluetooth speaker in **Party Mode**.  Everyone who wants to listen can connect their headphones to the speaker, but only **if they join the stream**.

The music isn’t blasted out to the whole building. It’s not a private one-on-one stream either. It’s **one stream sent to a select group** of connected listeners, those who’ve opted in.

That’s how IPv6 **multicast** works: one message is sent to **all devices that have joined the group** and **only those** devices process it.

---
## Why Multicast Matters in IPv6

IPv6 completely **replaces broadcasting** with multicasting. Here’s why it’s critical:

- [ ] Reduces unnecessary network traffic (no flooding like in IPv4)
- [ ] Powers essential protocols like:
	 - **Neighbor Discovery Protocol (NDP)** → RA/RS, NS/NA
	  - **SLAAC** → Router Advertisements via multicast
	  - **DHCPv6** → Client requests
	  - **Routing protocols** like OSPFv3, RIPng, EIGRP for IPv6
- [ ] Makes networks more scalable and efficient

---
## Key Multicast Addresses to Know

| Address             | Purpose                                                |
| ------------------- | ------------------------------------------------------ |
| `FF02::1`           | All nodes on the local link                            |
| `FF02::2`           | All routers on the local link                          |
| `FF02::1:FFxx:xxxx` | Solicited-node multicast (used for address resolution) |

## Common Applications of Multicast

| Application               | Multicast Role                                      |
|---------------------------|-----------------------------------------------------|
| **Router Discovery (RA/RS)** | Routers send advertisements to `FF02::1` (all hosts) |
| **Address Resolution (NS/NA)** | Nodes use multicast to find MAC addresses (solicited-node) |
| **DHCPv6**                | Clients query multicast groups to find DHCPv6 servers |
| **Routing Protocols**     | OSPFv3 and EIGRPv6 send updates to multicast groups |
| **Streaming / IPTV**      | One source to many viewers (real-world multicast)   |

> Services like Netflix use **unicast with CDNs**, not multicast.
> 
> **CDN** or **Content Delivery Network** is a distributed server network that works together to **deliver content (like videos, websites, or software) quickly and reliably** to users based on their geographic location.
> 
> A **CDN** ensures that the content you’re trying to access (like a Netflix show or a YouTube 
> video) comes from a **server physically close to you**, not from the original data center far away.

---
<p style="font-size: 0.8em; text-align: center; color: gray;">
25S-CST8371 - C. Ayala 
</p>
