# 00-LAB - Getting to Know Your Lab Environment

---

## Lab at Glance

**Platform**:  PacketTracer 8.2.2
**Format**:  At Home with BrightSpace Submission
**Due Date**:  Friday, September 5th @ 4 PM
**Points**:  5 Points

> **Note**: all packet tracer activities are graded up to 5 points.

---

## 🧭 Lab Overview
-- What are students to do in the lab.


---
## 🎯 Learning Objectives


---
## 💡 Why This Lab is Important

understanding the lab topology you are going to be working on during the semester
will allow you to troubleshoot better in case you encounter problems as you are completing
the labs.

As well as allowing you to maintain a basis script to use at the beginning of all labs; I will make it
easier to understand the deliveries and expectations for each lab from now on.

---


## 🗺️ Network Topology

![Lab 11 Topology](img/w01-topology.png)

## 📘  Addressing Table (IPv4)

Use the table below to configure your devices. Replace `U` with your assigned student ID number.

| Device     | Interface | IP Address (CIDR)  | Description                                 |
| ---------- | --------- | ------------------ | ------------------------------------------- |
| **EDGE**   | Gi0/0/0   | `203.0.113.U/24`   | Outside interface to the Remote Network     |
| **EDGE**   | Gi0/0/1   | `198.18.U.17/29`   | Inside interface towards CORE               |
| **CORE**   | Gi0/0/0   | `10.U.18.1/28`     | Inside gateway for dynamic NAT pool hosts   |
| **CORE**   | Gi0/0/1   | `198.18.U.22/29`   | Inside interface toward EDGE                |
| **CORE**   | Gi0/0/2   | `172.16.9.33/28`   | Host receiving static port‐forward (Telnet) |
| **CORE**   | LoU       | `192.168.U.1/24`   | Private network                             |
| **VM**     | Gi0/0/2   | `172.16.9.46/28`   | Inside host for PAT translation tests       |
| **PC**     | Gi0/0/0   | `10.U.18.14/28`    | Inside host for dynamic NAT pool tests      |
| **Remote** | —         | `203.0.113.254/24` | External tester for Internet connectivity   |

---

# The Network Explained

## 1.  Find Your Unique Network Octet - U

--- Create a simple python program; upload student information as a csv file; use usernames and student number; 
ask for the username and the student number and then show the U number for that student.

Post the information as well in BrightSpace.

The `U`number will be used during the course as part of your network address. It will identify your networks from the other students' network.  It will be use as part of the IPv4 and the IPv6 address will configured.  

If you don't have a `U`number associated to you, email your theory professor to get one.

---
## 2. The Remote Network

![Remote Network](img/Remote.png)


**RemoteSW** is a 48 ports L3 switch running -- insert version number here --.  It uses the `.254`ip in all its L3 interfaces.

Switchport `Gi0/0/1`connects to Station 1; `Gi0/0/2`connects to Station 2 and so on.

Switchports Gi0/0/1 - Gi0/0/25 are access ports in  VLAN 203.  The SVI VLAN 203 uses the network address of `203.0.113.0/24`;  and the IP address of `203.0.113.254/24`

Servers are located in the the network `192.0.2.0/24`; all servers are Raspberry Pi.  Servers use the as their IP the port number of the application it servers.  For example, the TFTP server uses the `192.0.2.69/24` address and the http server the `192.0.2.80/24` address.


---

## 3. Connection to the REMOTE Network

Link a photo to the students' rack; use the white RJ-45 connector.

-- Use Rich's patch panel drawing 

All students connect to the Remote Switch using the network `203.0.113.0/24`.  You are to use your `U` as the last octet in your IP address.  

For example, if you `U=100`; your IP address in the port facing Remote will be `203.0.113.100/24`

From the student station, connect your EDGE Router to Remote using the While connector.  If using a router, you will always connect your Router `Gi0/0/0` to Remote.

---

## 4.  The Student Network

The IP address we'll work with will change throughout the semester, but I will try to use the same network design as much as possible.

You will be using at least two routers and one L3 switch to create a in-line or a triangle topology.  This configuration will allow you to see how routing works with the less amount of devices.   Working with L3 switches will allow to understand Switch Virtual Interfaces (SVIs) as well a SVI routing.

For testing purposes, you will be using two hosts machines, a Windows X and a Linux one.  During some labs, you will use an Alpine Linux host, a small and compact Linux distro and in others you will use Khali, a powerful distro used in security settings.

---

# Building the Network

## **Task 1**: Initial Configuration

### 0. Create submission file
- [ ] On your desktop, create a file `00-username.txt`.  This is the file you will be submitting at the end of the lab to the tftp server.
- [ ] Only the `00-username.txt`will be graded even if the running configs are submitted.

> Note about filename; importance; if using notepad, the .txt extension is used by default.
### 1. Initial Setup for all Devices
- [ ] Hostnames: `username-<devicename>R` (eg: *ayal0014-R1*)
- [ ] Protect privileged exec with password `class` stored with strong encryption
- [ ] Disable dns lookups
- [ ] Configure the console line to minimize disruptions caused by log messages.
- [ ] On the vty lines: enable SSHv2 access using the domain `cnap.cst` authenticating with `admin / cisco`.
- [ ] Switches must be configured with `vtp mode transparent`
- [ ] Routers must be configured with `(config)# no ip tftp source-interface`
### 2. MLS: VLAN, SVIs and Port Configuration
- [ ] Create VLAN 10, 20, and 666, use VLAN names as `username-<VLANID>`(eg: `ayal0014-VLAN10`).  All VLANs should have your `username` pre-appended to the device name!
- [ ] Assign SVIs IP addresses  to VLAN 10 and VLAN 20.
- [ ] Place PC on VLAN 10 and VM on VLAN 20.
- [ ] Shutdown all interfaces that are not in use and move them to VLAN 666.  
- [ ] No port should be in VLAN 1
### 3. Addressing Configuration
- [ ] Configure addresses according to the topology diagram, paying attention to the network masks.
- [ ] Add a description to all Cisco interfaces, using `notes` from the network addresses table.
- [ ] Ensure all interfaces are UP/UP before continuing.

