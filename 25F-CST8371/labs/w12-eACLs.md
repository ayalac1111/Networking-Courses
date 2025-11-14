# Lab 12 – Controlling Services with Extended ACLs

## Overview

In this lab, you will enforce service-level access controls by creating and applying **extended named ACLs** on selected network device interfaces. 

---
## Learning Objectives

By completing this lab, you will learn how to:

- Define and apply **named extended ACLs** to enforce service- and protocol-specific policies.  
- Choose the correct **interface** and **direction** for placement of extended ACLs (near the source).  
- Use **port numbers** and **protocols** in ACEs to permit or deny traffic precisely.  
- Enable and interpret **ACL hit-counters** and **logging** for troubleshooting and auditing.  
- Verify service access and confirm that unwanted traffic is blocked.

---
## Why This Lab Is Important

- **Granular Control:** Extended ACLs filter on source IP, destination IP, protocol, and port—essential for enforcing least-privilege access.  
- **Network Efficiency:** By dropping disallowed traffic close to the source, you reduce unnecessary load on upstream links and remote hosts.  
- **Auditability:** Logging denied ACEs provides visibility into unauthorized service attempts, aiding security monitoring.  
- **Real-World Relevance:** Extended ACLs are widely used on Internet edge routers and firewall devices to control which services internal users can access.

---
## 🗺️ Network Topology

![Lab 10 Topology](img/w12-topology.png)

| **Network**           | **Subnet**      | **Notes**                                            |
| --------------------- | --------------- | ---------------------------------------------------- |
| CORE–EDGE transit     | 198.18.U.0/30   | EDGE = 198.18.U.1 ; CORE = 198.18.U.2                |
| VM subnet             | 198.18.U.32/28  | VM hosts on last usable address                      |
| PC subnet & Mgmt VLAN | 198.18.U.128/26 | PC hosts on first usable; ALS SVI on 2nd-last usable |
| Remote cloud          | 192.0.2.0/24    | GW = .254 ; TFTP = .69 ; DNS = .53 ; WEB = .80       |

---
## Addressing Table

| Device       | Interface                     | IP Address           |
| ------------ | ----------------------------- | -------------------- |
| **EDGE**     | GigabitEthernet0/0/0 (REMOTE) | 203.0.113.U/24       |
|              | GigabitEthernet0/0/1 (CORE)   | 198.18.U.1/30        |
|              | GigabitEthernet0/0/2 (VM)     | 198.18.U.33/28       |
| **CORE**     | GigabitEthernet0/0/0 (EDGE)   | 198.18.U.2/30        |
|              | GigabitEthernet0/0/1 (ALS)    | 198.18.U.190/26      |
| **ALS (SW)** | Vlan1 (Mgmt)                  | 198.18.U.189/26      |
| **PC Host**  | —                             | 198.18.U.129 (first) |
| **VM Host**  | —                             | 198.18.U.46 (last)   |
| **Remote**   | — (Gateway)                   | 203.0.113.254        |
| **TFTP SVR** | —                             | 192.0.2.69           |
| **DNS SVR**  | —                             | 192.0.2.53           |
| **WEB SVR**  | —                             | 192.0.2.80           |

---

## Initial Setup

### 0 — Preparation
- [ ]  On your desktop, create **ONE** file: `w12-acl-{username}.txt`
### 1. Basic Configuration
- [ ] Use the provided base configuration file: [basic.cfg](../resources/basic.cfg)
- [ ] Enable real-time debug output display on all routers.
### 2. Addressing Configuration
- [ ] Configure addresses according to the topology diagram, paying attention to the network masks.
- [ ] Add a description to all Cisco interfaces.
- [ ] Ensure all interfaces are UP/UP before continuing.
- [ ] Console to **EDGE** and ssh into **CORE** using the user `cisco/cisco`
### 3. Services
- [ ] Prevent the router from using its own IP as a source when sending to TFTP.
- [ ] Enable the `http` and `https` servers on both routes.
	- [ ] `ip http secure-server` - `ip http server`
	- [ ] `ip http authentication local`
- [ ] Direct `syslog` messages to the PC.  Note that **EDGE** messages will be sent once you configure OSPF.
### 4. OSPF
- [ ] **Process ID**: Use `U` as the OSPF process number.
- [ ] **Router-IDs**: Manually set each router’s ID under OSPF:
	- [ ] `EDGE`:  `1.0.0.U`
	- [ ] `CORE`: `2.0.0.U`
- [ ] **Default Gateway Advertisement**: On **EDGE**, redistribute or originate a default route so that `0.0.0.0/0` is known across the OSPF domain
- [ ] Configure explicitly all interfaces **not directly connected to an OSPF neighbour** as passive.

---

## Policy #1 – POLICY-VM-SUBNET

Devices in the **VM subnet** are denied access to the following remote services:
- Web access (HTTP and HTTPS) to the remote web server at `192.0.2.80`
- TFTP access to the remote server at `192.0.2.69`
- SSH access to the remote server at `192.0.2.22`
- ICMP echo requests (pings) to devices in the PC subnet (`198.18.U.128/26`)

All other traffic is permitted, including access to internal infrastructure such as the CORE router.
### Goal:
Restrict VM subnet traffic to selected services, deny unnecessary or unauthorized access to remote resources, and permit legitimate internal and external access.
### Sample Configuration
```bash
ip access-list extended ACL-VM
  remark Block HTTP and HTTPS to remote web server
  deny tcp 198.18.U.32 0.0.0.15 host 192.0.2.80 eq 80
  deny tcp 198.18.U.32 0.0.0.15 host 192.0.2.80 eq 443
  remark Block TFTP to remote server
  deny udp 198.18.U.32 0.0.0.15 host 192.0.2.69 eq 69
  remark Block SSH to remote server
  deny tcp 198.18.U.32 0.0.0.15 host 192.0.2.22 eq 22
  remark Block ICMP echo to PC subnet
  deny icmp 198.18.U.32 0.0.0.15 198.18.U.128 0.0.0.63 echo
  remark Permit all other traffic
  permit ip any any
```

> Use `log`  at the end of each ACE to create a syslog message for testing.
### ACL Application
The ACL is applied **inbound** on `GigabitEthernet0/0/2`, the interface facing the VM subnet. This placement ensures that all traffic originating from the VMs is filtered **before it enters the router**, enforcing restrictions on web, SSH, TFTP, and ICMP traffic early in the path. This not only improves efficiency but also aligns with best practices for source-based filtering.

```bash
interface GigabitEthernet0/0/2
  ip access-group ACL-VM in
```
### Verification and Testing

1. **Clear counters**
	```bash
	clear access-list counters ACL-VM
    ```
    
2. **Test each ACL entry**
- `curl http://192.0.2.80` → ❌ Denied
- `curl -k https://192.0.2.80` → ❌ Denied
- `tftp -i 192.0.2.69 put <path-to-file>` → ❌ Denied
- `ssh cisco@192.0.2.22` → ❌ Denied
- `ping 198.18.U.129` → ❌ Denied
- `curl http://198.18.U.2` → ✅ Allowed

> **VM TFTP**: The TFTP client isn’t enabled by default. To turn it on:  
> `Control Panel > Programs > Turn Windows features on or off > TFTP Client`
> You could use a web browser or telnet with port 80 instead of `curl`

3. **Validate ACL hit-counters**

    ```bash
    show access-lists ACL-VM
    ```
    
> All lines should have matches
    
4. **Check syslog entries**
> To see `log` entries, you need to have configured `log` at the end of each entry in your ACL.
    
```bash
# show logging | include ACL-VM
```
    
- [ ] Look for `%IPACCESSLOGD` for the VM deny and `%IPACCESSLOGP` for each permit.

## CO1 – Verification and Collection of Information

In your `w12-acl-{username}.txt`, include **all** of the following:

```diff
=== CO1 – Policy #1 - ACL-VM Verification ===
```

1. **ACL Hit-Counters**  
```bash
   show ip access-lists ACL-VM
```
(This lists each ACE and its match count for all three policies.)

2. **ACL Binding**
```bash
show ip interface GigabitEthernet0/0/2 | include ACL-VM
```

---
## Policy #2 – POLICY-PC-SUBNET

### Description:
Devices in the PC subnet are allowed to access most services, except:
- DNS queries from host `198.18.U.129` to the DNS server at `192.0.2.53`
- Web (HTTP) access to the TFTP server address at `192.0.2.69`
- Secure web (HTTPS) access to the HTTP server address at `192.0.2.53`

All other traffic, including access to TFTP, SSH, and the main web server, is permitted.
### Goal:
Block DNS queries from a specific PC host, limit web access to only approved remote services, and allow all other communications.

### Sample Configuration
```bash
ip access-list extended ACL-PC
  remark Block DNS from specific host
  deny udp host 198.18.U.129 host 192.0.2.53 eq 53
  remark Block HTTP to TFTP server IP
  deny tcp 198.18.U.128 0.0.0.63 host 192.0.2.69 eq 80
  remark Block HTTPS to DNS server IP
  deny tcp 198.18.U.128 0.0.0.63 host 192.0.2.53 eq 443
  remark Permit all other traffic
  permit ip any any
```

### ACL Application
Determine the router, interface and direction to apply the `ACL-PC`extended ACL.
### Verification and Testing
#### DNS Server Details
The remote DNS server (`192.0.2.53`) is authoritative for the `cnap.cst` zone and hosts the following records:

| **Hostname**    | **IPv4 Address** | **Description**                    |
| --------------- | ---------------- | ---------------------------------- |
| `www.cnap.cst`  | `192.0.2.80`     | Web server for the cnap.cst domain |
| `ns.cnap.cst`   | `192.0.2.53`     | DNS master server                  |
| `tftp.cnap.cst` | `192.0.2.69`     | TFTP server                        |

> **Note:**  
> - You’ll use these names when testing DNS resolution (e.g. `ping www.cnap.cst`).  
> - The DNS server itself is at `192.0.2.53`, but all zone records reside on `ns.cnap.cst`.

**PC**:  Set your dns to `192.0.2.53`

1. **Clear counters**
	```bash
	clear access-list counters ACL-PC
    ```
    
2. **Test each ACL entry**
- `nslookup ns.cnap.cst` (from .129) → ❌ Denied
- `nslookup ns.cnap.cst` (from other PC) → ✅ Allowed
- `curl http://192.0.2.80` → ✅ Allowed
- `curl http://192.0.2.69` → ❌ Denied (HTTP not allowed to TFTP IP)
- `curl -k https://192.0.2.53` → ❌ Denied (HTTPS not allowed to DNS IP)
- `ssh cisco@192.0.2.22` → ✅ Allowed
- `tftp 192.0.2.69` → ✅ Allowed

3. **Validate ACL hit-counters**

    ```bash
    show access-lists ACL-PC
    ```
    
> All lines should have matches
    
4. **Check syslog entries**
> To see `log` entries, you need to have configured `log` at the end of each entry in your ACL.
    
```bash
# show logging | include ACL-PC
```
    
- [ ] Look for `%IPACCESSLOGD` for the VM deny and `%IPACCESSLOGP` for each permit.


---
## CO2 – Verification and Collection of Information

In your `w12-acl-{username}.txt`, include **all** of the following:

```diff
=== CO2 – Policy #2 - ACL-PC Verification ===
```

1. **ACL Hit-Counters**  
```bash
   show ip access-lists ACL-PC
```
(This lists each ACE and its match count for all three policies.)

2. **ACL Binding**
```bash
show ip interface GigabitEthernet0/0/1 | include ACL-PC
```


---

## 📤 Submission Checklist

- [ ] `w12-acl-{username}.txt` uploaded via TFTP.  
- [ ] `12-{username}-syslog.txt` uploaded via TFTP.
- [ ] Upload running configs to the TFTP server.
- [ ] Verify with:
```bash
ssh cisco@192.0.2.22
ls -l /var/tftp/*username*
```


