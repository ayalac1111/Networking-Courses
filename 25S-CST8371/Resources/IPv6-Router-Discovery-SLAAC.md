
# 🧩 Guided Wireshark Activity: IPv6 Router Discovery and SLAAC

⏱️ **Duration**: 20 minutes  
🛠️ **Tools**: Wireshark + Provided PCAP file: `IPv6-Router-Discovery.pcap`  
🎯 **Goal**: Understand how a host discovers a router and forms an IPv6 address using SLAAC.

---

## 📁 Part 1 – Open the Packet Capture

[Download PCAP file](https://https://github.com/ayalac1111/Networking-Courses/raw/main/25S-CST8371/Resources/ipv6-Router-Discovery.pcapng)

Open the file: `IPv6-Router-Discovery.pcap` in Wireshark.

You should see 3 key packets listed in the capture pane:

- 1️⃣ **Neighbor Solicitation (NS)**
- 2️⃣ **Router Solicitation (RS)**
- 3️⃣ **Router Advertisement (RA)**

Look at the **Info** column or **ICMPv6 Protocol** field to identify each packet.

---

## 🧭 Part 2 – Step-by-Step Analysis

In this analysis, you'll review how a host uses **Neighbor Discovery Protocol (NDP)** to safely configure an IPv6 address without manual input. Start by examining how the host checks if an address is in use before trying to communicate.

### Step 1 – Duplicate Address Detection (DAD)

When a host autoconfigures an IPv6 address, it must ensure that the address is not already in use. This verification is done through **Duplicate Address Detection (DAD)** using a **Neighbor Solicitation (NS)** message.

In this case, the host is testing its tentative **link-local address** by sending an NS packet with:
- `::` as the **source address** (unspecified, since it doesn't yet have a usable address).
- A **multicast destination** derived from the address being verified.

### 1️⃣ **Locate**: Packet 1
### 🔍 Look inside:
- [ ]  `Internet Protocol Version 6`
- [ ]  `ICMPv6 Neighbor Solicitation`
#### **✏️ Answer** the following questions:

##### **(A)** **[ICMPv6]** What is the **Target Address** being verified for uniqueness in this NS packet?

> The **Target Address** in the NS packet is the **tentative IPv6 address** the host intends to use.

```
Target Address: ___________________________________________________
```

##### **(B)** **[IPv6]** What is the source IPv6 address?

>It should be `::`, because the host is testing an address it does not yet consider valid

```
Source IPV6 Address: ____________________________________________________  
```

##### **(C)** **[IPv6]** What multicast address is used as the destination address?

> This multicast address is derived from the last _6 hexadecimal characters_ of the Target Address (equivalent to the last 24 bits). You can confirm this by comparing the ending of the Target Address and the **solicited-node** multicast address.

```
Destination IPV6 Address: ___________________________________________  
```


---

### Step 2 – Router Solicitation (RS)

After validating its link-local address, the host must **discover if any routers are present** on the network. It sends a **Router Solicitation (RS)** message to the **all-routers multicast address** (`ff02::2`), using its **link-local address** as the source.

### 2️⃣ **Locate**: Packet 2
### 🔍 Look inside:
- [ ]  `Internet Protocol Version 6`
- [ ]  `ICMPv6 Router Solicitation`
#### **✏️ Answer** the following questions:

##### **(A)** **[IPv6]** What is the **source IPv6 address** of the host sending the RS?
 
> The source is the host’s **link-local address**, validated after Duplicate Address Detection (DAD).

```
Source IPV6 Address: ____________________________________________________  
```

##### **(B)** **[IPv6]** What is the **destination IPv6 address**?

> The RS is sent to `ff02::2`, which is the **all-routers multicast address**, scoped to the local link.

```
Destination IPV6 Address: ____________________________________________________  
```

##### **(C)**  **[Reflection]** Why does the host use a multicast address instead of broadcasting?

> Multicast allows efficient communication by targeting only **router devices** that have joined the `ff02::2` group, instead of all nodes on the network.

```



```


---

### Step 3 – Router Advertisement (RA)

A router replies to the RS with a **Router Advertisement (RA)**. This message includes important parameters: IPv6 prefix, router lifetime, flags, and MAC address info. It enables the host to complete **SLAAC** configuration.

### 3️⃣ **Locate**: Packet 3

### 🔍 Look inside:
- [ ]  `Internet Protocol Version 6`
- [ ] `ICMPv6 Router Advertisement`
- [ ] `Prefix Information`
- [ ] `Source MAC address` (Ethernet II)

#### **✏️ Answer** the following questions:

##### **(A)**  **[ICMPv6]** What IPv6 **prefix** was advertised?

> This prefix is combined with the host’s interface ID to generate a global IPv6 address.

```
Advertised Prefix: _____________________________________________________
```

##### **(B)** **[IPv6]** What IPv6 address sent the RA?

>This is the router’s source address for the RA message — typically a **link-local address**.

```
Source IPV6 Address: ____________________________________________________  
```

##### **(C)** **[Ethernet II]** What MAC address belongs to the router?

> This is the **Layer 2** address of the interface that sent the RA.

```
Router MAC Address: ____________________________________________________  
```

##### **(D)** **[Reflection]** How does this help the host configure a full IPv6 address?

>The host uses the advertised **prefix** and its own interface ID to form a **globally unique address** via SLAAC.

```


```

---
## 🔎 Final Reflection

- What role does **multicast** play in the IPv6 discovery process?
- Why does the host start with the source address `::`?
- How does SLAAC ensure the host receives a unique and routable IPv6 address?

---
For a more detailed explanation of this packet sequence, read:  
🔗 [How to: IPv6 Neighbor Discovery (APNIC)](https://blog.apnic.net/2019/10/18/how-to-ipv6-neighbor-discovery/)

---
<p style="font-size: 0.8em; text-align: center; color: gray;">
25S-CST8371 - C. Ayala 
</p>