# W01 — Be Ready: Alpine Linux Tool Readiness

**Course:** CST8371 — Introduction to Enterprise Networking  
**Week:** 01  
**Activity type:** Readiness / lab-book setup  
**Environment:** Windows host + Alpine Linux VM  
**Status:** Ungraded  
**Submission:** None, unless your instructor requests a lab-book check  

---

## Why this activity exists

You will use Alpine Linux during the course. The server environment used in the course also runs Alpine Linux.

This readiness activity is not a formal graded lab. The goal is to prepare your lab book with command evidence that you will reuse later in all labs.

You should be able to work from both sides:

```text
Windows host  <-->  Alpine Linux VM
Alpine Linux VM  <-->  Windows host
```

For each tool, record the command syntax, the platform used, the target, success indicators, common failure signals, and the next command you would run to troubleshoot.

---

## Lab book reference

Use the lab book model as you complete this readiness activity.

![Lab Book Sample Layout](../images/lab-book-sample.png)

---

## Required lab-book structure for this activity

Use one open notebook spread.

```text
LEFT PAGE  = 01 Tool Study Guide
RIGHT PAGE = 01 Tool Proof + Troubleshooting
```

### Left page must include

```text
Week:
Topic:
Lab name:
Environment:
```

Use:

```text
Week: W01
Topic: Alpine Linux tool readiness
Lab name: W01 — Be Ready
Environment: Windows host + Alpine Linux VM using VMware NAT
```

### Right page must capture evidence for

```text
1. Alpine IP address, static IP address, and default route
2. ping from Windows to Alpine and Alpine to Windows
3. traceroute/tracert from both directions
4. SSH from Windows to Alpine and Alpine to Windows
5. SCP from Windows to Alpine and Alpine to Windows
6. TFTP attempt, including success or failure evidence
```

---

## Environment

Use one of the following:

1. An Alpine Linux VM you install yourself.
2. An Alpine Linux OVA you find and import into VMware.

Recommended install source:

```text
Alpine Linux Virtual x86_64
```

Recommended VMware network mode:

```text
NAT
```

NAT is recommended because the Alpine VM should have internet access for package installation without requiring bridged-network troubleshooting.

Reference documentation:

- Alpine Linux networking documentation: <https://wiki.alpinelinux.org/wiki/Configure_Networking>
- Alpine Linux user handbook: <https://docs.alpinelinux.org/user-handbook/0.1a/index.html>

---

## Package readiness

Some tools may not be installed by default.

Students have used Linux since Level 1. Use the package manager and Alpine documentation to install missing tools as needed.

Commands may be similar to the following, depending on the Alpine image and installed packages:

```sh
apk update
apk add openssh openssh-client openssh-server traceroute tftp-hpa
rc-update add sshd default
rc-service sshd start
```

Record any package install commands you use in the lab book.

---

## Firewall and service note

Some tests require a service to be running on the target and firewall rules to allow inbound traffic.

| Tool | Direction | Target requirement | Common block |
|---|---|---|---|
| SSH | Windows to Alpine | Alpine SSH server running; TCP/22 listening | Alpine `sshd` not installed, stopped, or blocked |
| SSH | Alpine to Windows | Windows OpenSSH Server installed/running; inbound TCP/22 allowed | Windows OpenSSH Server disabled or Windows Defender Firewall blocks TCP/22 |
| SCP | Windows to Alpine | Same as SSH to Alpine | SSH failure, wrong username/path, permissions |
| SCP | Alpine to Windows | Same as SSH to Windows | SSH failure, Windows path syntax, permissions |
| TFTP | Client to server | TFTP server running; UDP/69 allowed; correct TFTP root | Firewall blocks UDP/69, wrong root folder, permission error |

If a firewall or service blocks the test, do not erase the result. Record the exact failure signal and the next command or check that would prove the cause.

---

# Part 1 — Alpine identity, network, and static IP

## Action

Identify the VMware network being used, determine the current Alpine IP information, choose an unused IP address on the same NAT network, and configure Alpine with a static IP address.

Do not guess an address. Confirm the network first. Use Alpine documentation as the reference for the exact persistent configuration method.

## Evidence

Record the commands you used to prove:

```text
1. Alpine version
2. active interface name
3. current IP address
4. current default route
5. VMware network type in use
6. unused static IP address selected
7. static IP address applied
8. network still works after the change
```

Commands may be similar to:

```sh
cat /etc/alpine-release
ip addr
ip route
ping -c 4 <gateway-or-known-target>
```

For persistent static IP configuration, use the Alpine networking documentation. Your command sequence may differ depending on whether your Alpine installation uses setup scripts, `/etc/network/interfaces`, NetworkManager, or another configuration method.

## Success indicator and failure signal

| Check | Evidence to record | Success indicator | Failure signal |
|---|---|---|---|
| Alpine version | Version command output | Alpine release number appears | No output or command error |
| Active interface | Interface listing | Interface has `UP` state or active carrier | Wrong interface selected |
| Current IP | IP address output | IPv4 address appears on expected interface | No IPv4 address |
| Default route | Route table output | `default via <gateway>` exists | No default route |
| Static IP selection | Notes showing selected address | Address is on NAT network and unused | Duplicate address or wrong network |
| Static IP applied | IP address output after change | Selected static IP appears on Alpine | Old IP remains or interface loses IP |
| Network still works | `ping -c 4 <gateway-or-known-target>` | Replies and packet-loss summary appear | 100% packet loss or unreachable |

## Lab-book entry

```text
Tool: Alpine network identity and static IP
Platform: Alpine Linux
Command:
Target:
Success output indicator:
Common failure output:
What this proves:
What this does not prove:
Next command/check if this fails:
```

---

# Part 2 — ping in both directions

## Action

Use ping to test reachability between Windows and Alpine in both directions.

Use the `-c` option on Alpine so the command stops after a fixed number of probes.

Refer to the Week 1 ping infographic posted in the course materials when recording common ping output characters and patterns.

## Evidence

From Windows to Alpine, use a command similar to:

```powershell
ping <alpine-static-ip>
```

From Alpine to Windows, use a command similar to:

```sh
ping -c 4 <windows-ip>
```

## Success indicator and failure signal

| Direction | Evidence to record | Success indicator | Failure signal |
|---|---|---|---|
| Windows to Alpine | Windows ping output | `Reply from <ip>` lines and packet summary | `Request timed out`, unreachable, 100% loss |
| Alpine to Windows | Alpine ping output | `64 bytes from <ip>` lines and packet-loss summary | `100% packet loss`, unreachable, temporary name-resolution failure |
| Output interpretation | Lab-book note | Student records what replies, loss, latency, and timeout mean | Student writes only “works” or “does not work” |
| Next troubleshooting step | Lab-book note | Student names the next command/check | No next step recorded |

## Lab-book entry

```text
Tool: ping
Platform:
Command:
Target:
Success output indicator:
Common failure output:
Common output characters or patterns:
What this proves:
What this does not prove:
Next command/check if this fails:
```

## Required interpretation notes

Record at least three common ping patterns from the posted ping infographic or from your own output.

Examples:

```text
Reply from <ip> = ICMP reply received
Request timed out = no reply received before timeout
Destination Host Unreachable = a device reports it cannot reach the destination
100% packet loss = no replies returned for the probes sent
```

---

# Part 3 — traceroute / tracert in both directions

## Action

Use path-testing commands from both platforms.

Windows uses:

```text
tracert
```

Alpine/Linux uses:

```text
traceroute
```

Refer to the Week 1 traceroute/tracert material posted in the course materials when recording common path output patterns.

## Evidence

From Windows to Alpine, use a command similar to:

```powershell
tracert <alpine-static-ip>
```

From Alpine to Windows, use a command similar to:

```sh
traceroute <windows-ip>
```

Optional internet-path check if NAT is working:

```sh
traceroute <known-internet-target>
```

## Success indicator and failure signal

| Direction | Evidence to record | Success indicator | Failure signal |
|---|---|---|---|
| Windows to Alpine | `tracert` output | Hop list or direct local result appears | Request timed out, stops early, no route |
| Alpine to Windows | `traceroute` output | Hop list or direct local result appears | `* * *`, stops early, command missing |
| Output interpretation | Lab-book note | Student identifies last responding hop or direct path | Student claims traceroute proves root cause |
| Next troubleshooting step | Lab-book note | Student names the next command/check | No next step recorded |

## Lab-book entry

```text
Tool: traceroute / tracert
Platform:
Command:
Target:
Success output indicator:
Common failure output:
Common output characters or patterns:
What this proves:
What this does not prove:
Next command/check if this fails:
```

## Required interpretation notes

Record at least three common traceroute/tracert patterns.

Examples:

```text
* * * = no reply for that probe
Path stops early = path appears to stop or stop responding
Different hop times = latency varied between probes
Same destination in one hop = local or directly reachable target
```

---

# Part 4 — SSH in both directions

## Action

Use SSH from Windows to Alpine and from Alpine to Windows.

This requires an SSH server on the target side. If the Windows SSH server is not enabled, record the failure signal and the Windows service/firewall check needed.

## Evidence

From Windows to Alpine, use a command similar to:

```powershell
ssh <alpine-user>@<alpine-static-ip>
```

From Alpine to Windows, use a command similar to:

```sh
ssh -l <windows-user> <windows-ip>
```

On Alpine, service checks may be similar to:

```sh
rc-service sshd status
ss -tlnp | grep :22
```

On Windows, students may need to verify that OpenSSH Server is installed/running and that inbound TCP/22 is allowed.

## Success indicator and failure signal

| Direction | Evidence to record | Success indicator | Failure signal |
|---|---|---|---|
| Windows to Alpine | SSH command output | Authenticated Alpine shell prompt appears | Connection refused, timeout, permission denied |
| Alpine to Windows | SSH command output | Authenticated Windows shell prompt appears | Connection refused, timeout, permission denied |
| Alpine SSH service | Service/listener output | `sshd` running and TCP/22 listening | Service stopped or TCP/22 absent |
| Windows SSH service/firewall | Service/firewall check notes | OpenSSH Server running and inbound TCP/22 allowed | Service missing/stopped or firewall block |
| Interpretation | Lab-book note | Student states what SSH success proves | Student claims SSH proves all network services work |

## Lab-book entry

```text
Tool: SSH
Platform:
Command:
Target:
Success output indicator:
Common failure output:
What this proves:
What this does not prove:
Next command/check if this fails:
```

---

# Part 5 — SCP in both directions

## Action

Use SCP to copy a small file in both directions.

SCP uses SSH. If SSH fails in a direction, SCP is expected to fail in that same direction. Record that dependency in the lab book.

## Evidence

Create a small test file on each side.

From Windows to Alpine, use a command similar to:

```powershell
scp <windows-test-file> <alpine-user>@<alpine-static-ip>:/home/<alpine-user>/
```

Then confirm on Alpine with commands similar to:

```sh
ls -l /home/<alpine-user>/<windows-test-file>
cat /home/<alpine-user>/<windows-test-file>
```

From Alpine to Windows, use a command similar to:

```sh
scp <alpine-test-file> <windows-user>@<windows-ip>:<windows-destination-path>
```

Then confirm on Windows that the file exists at the destination.

## Success indicator and failure signal

| Direction | Evidence to record | Success indicator | Failure signal |
|---|---|---|---|
| Windows to Alpine | SCP output plus Alpine file check | Transfer reaches 100%; file exists on Alpine with expected content | Permission denied, wrong path, SSH failure, file missing |
| Alpine to Windows | SCP output plus Windows file check | Transfer reaches 100%; file exists on Windows with expected content | Permission denied, Windows path issue, SSH failure, file missing |
| Destination proof | Directory listing or file content | Filename, location, and content/size match | Transfer message exists but destination not confirmed |
| Interpretation | Lab-book note | Student states SCP depends on SSH and confirms file arrival | Student records only the command, not destination proof |

## Lab-book entry

```text
Tool: SCP
Platform:
Command:
Target:
Success output indicator:
Common failure output:
What this proves:
What this does not prove:
Next command/check if this fails:
```

---

# Part 6 — TFTP attempt

## Action

Attempt a TFTP transfer.

TFTP may use one of these options:

```text
Windows TFTP server
Alpine TFTP server
container-based TFTP server
lab-provided TFTP server
```

This readiness activity requires a TFTP attempt and lab-book evidence. It does not require perfect TFTP success during Week 1.

## Evidence

Record which system is acting as the TFTP server and which system is acting as the TFTP client.

Client commands may be similar to:

```sh
tftp <server-ip>
tftp <server-ip> -c put <file>
tftp <server-ip> -c get <file>
```

If testing both directions, switch the server/client role and record the result.

## Success indicator and failure signal

| Direction or role | Evidence to record | Success indicator | Failure signal |
|---|---|---|---|
| Alpine client to TFTP server | TFTP command output and server folder check | File appears in TFTP root; filename and size/content match | Transfer timed out, access violation, file not found |
| Windows client to TFTP server | TFTP command output and server folder check | File appears in TFTP root; filename and size/content match | Client missing, timeout, access violation, file not found |
| Alpine as TFTP server | Service/process and folder check | TFTP service running; root folder identified | Service not running, wrong root, permission error |
| Windows/container as TFTP server | Server status and folder check | Server listening; root folder identified | Firewall blocks UDP/69, wrong root, permission error |
| Interpretation | Lab-book note | Student states TFTP is unencrypted and uses UDP/69 | Student treats TFTP as secure or omits failure evidence |

## Required failure documentation

If TFTP fails, record:

```text
Command used:
Exact error:
Client:
Server:
Likely failure point:
Next command or check:
```

## Lab-book entry

```text
Tool: TFTP
Platform:
Command:
Target:
Success output indicator:
Common failure output:
What this proves:
What this does not prove:
Next command/check if this fails:
```

---

# Final lab-book check

Before leaving this activity, confirm your lab book has one entry for each tool.

| Tool | Action | Evidence | Success Indicator | Failure Signal |
|---|---|---|---|---|
| Alpine IP / static IP / route | Find and set Alpine network state | Alpine network commands and Alpine docs reference | Static IP, interface, and default route visible; network still works | No IP, duplicate IP, wrong network, no default route |
| ping | Test both directions | Windows ping and Alpine `ping -c 4` | Replies and packet-loss summary visible both ways | Timeout, unreachable, 100% loss |
| traceroute / tracert | View path clues both directions | Windows `tracert`, Alpine `traceroute` | Hop list, direct path, or stop point visible | `* * *`, timeout, command missing |
| SSH | Connect both directions | Windows to Alpine; Alpine to Windows | Authenticated shell prompt appears | Refused, timeout, permission denied, firewall block |
| SCP | Copy and confirm files both directions | SCP output plus destination file check | File exists at destination with expected content | File missing, wrong path, SSH failure |
| TFTP | Attempt file transfer | TFTP put/get plus server folder check | File appears in TFTP root | Timeout, access violation, file not found, UDP/69 blocked |

---

# Reusable rule

Complete this sentence in your lab book:

```text
Next time I see __________________, I will run __________________ first.
```

Examples:

```text
Next time I see SSH fail, I will run ping -c 4 <target-ip> first.

Next time I see SCP fail, I will confirm SSH works in the same direction first.

Next time I see 100% packet loss, I will check ip addr and ip route first.

Next time I see traceroute stop early, I will compare it with ping before changing anything.

Next time I see TFTP timeout, I will check server process, TFTP root, and UDP/69 firewall rules first.
```

---

# Readiness outcome

This activity is complete when your lab book contains:

```text
1. command used
2. platform used
3. target used
4. direction tested
5. success output indicator
6. common failure output
7. what the command proves
8. what the command does not prove
9. next command/check if the test fails
```

for each Week 1 readiness tool.
