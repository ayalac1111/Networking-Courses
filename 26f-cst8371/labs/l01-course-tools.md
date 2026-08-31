# Foundation Lab 01 — Introduction to Course Tools

---

## Section A — Lab Information

### A1 — Overview

This lab sets up some of the tools used in CST8371: `CML-Free`, Alpine as an operations host, SSH, and `x_remote` as an evidence-collection tool. You install the tools, complete a small provided CML topology, and collect a baseline with `x_remote`. It is not a configuration lab — do not change the supplied CML startup configuration or wipe nodes.

Environment: Cisco CML

### A2 — Why This Matters

This course does not use Packet Tracer. Most Foundation Labs collect evidence with `x_remote` or a similar Python script; CML is used for the labs built around a virtual topology, like this one, and for in-class demonstrations.

The evidence habits you practise here — run a command, check its output, know what it proves and what it does not — are the same habits every later lab expects.

### A3 — Learning Objectives

| Task | Objective |
|---|---|
| C01 | Confirm `CML-Free`, `Python 3`, and `x_remote` are installed and working on your own host machine. |
| C02 | Bring up the provided CML topology, rename both devices to include your own username, add a `Loopback0` to `W01-EDGE` using your own U value, and verify device identity, SSH access, and reachability from your own host. |
| C03 | Use an automated collection script to gather live evidence from network devices — including live device-identity proof, the `Loopback0` address, and reachability evidence — then verify that evidence before relying on it. |

This is not a configuration lab (C02's device rename and `Loopback0` addition are the narrow exceptions — see below). Do not change the supplied CML startup configuration or wipe nodes.

| Objective (from A3) | Checkpoint | Points |
|---|---|---|
| Tools installed and working | C01 | 0 (gate only) |
| CML topology up, devices renamed with your username, `Loopback0` added using your U value, SSH/reachability confirmed | C02 | 3 |
| `x_remote` evidence collected and verified (SSH connectivity + Alpine addressing) | C03 | 2 |
| **Base Total** | | **5** |

---

## Section C — Lab Tasks

### C01 — Install Your Tools

#### Goal

Confirm `CML-Free`, `Python 3`, and `x_remote` are installed and working on your own host machine.

#### Why This Matters

Every later lab assumes these three tools already work. A broken install discovered mid-lab costs far more time than confirming it now, before any lab depends on it.

#### Action

1. **`CML-Free`.** Follow [`../resources/cml-client-setup-guide.md`](../resources/cml-client-setup-guide.md) to install Cisco Modelling Labs — Personal Edition (`CML-Free`) on your own host.
2. **`Python 3`.** Install it if you do not already have it (any current `Python 3` release is sufficient).
3. **`x_remote`.** Clone or download the tool from <https://github.com/ayalac1111/networking-tools> and follow the setup instructions in that repository's `README`.

#### Verification

```bash
python3 --version
```

(On some Windows installs the command is `python --version` instead of `python3 --version`.)

```bash
python3 x_remote.py --help
```

Also confirm you can log in to your own local `CML-Free` web UI.

#### Success Indicator / Failure Signal

| Verification Item | Success Indicator | Failure Signal |
|---|---|---|
| `CML-Free` login | You can log in to your own local `CML-Free` web UI and see a lab list | Install fails, or the local web UI does not load |
| `Python 3` | `python3 --version` prints a version number | Command not found, or prints a Python 2.x version |
| `x_remote` | `python3 x_remote.py --help` prints usage text | Import error (e.g. `ModuleNotFoundError`) instead of usage text |

#### Troubleshooting

If `python3 x_remote.py --help` fails with a module error: re-run `pip install -r requirements.txt` from the folder containing `x_remote.py`, per the `x_remote` repository `README`. If `CML-Free` itself will not install or start: that is a Cisco-product issue — check the system requirements section of the official installation guide (CPU virtualization, RAM, disk space) before asking for help.

#### C01 — Collection of Information

No submission required for this task.

---

### C02 — Complete the CML Lab

#### Goal

Open the provided CML readiness topology, distinguish a node console from an SSH session, use Alpine to check its own addressing and connectivity, SSH from Alpine to the course router, verify DHCP addressing and reachability from your own host machine, and add a `Loopback0` to `W01-EDGE` using your own U value.

#### Why This Matters

Console access and SSH access are different things, and this course relies on both. You need to be able to tell them apart, and to prove reachability from your own host — not just from inside a CML console — before any later lab that depends on host-to-device connectivity.

This checkpoint also introduces the identity habit that carries through the rest of the course: renaming both devices to include your username, and adding a `Loopback0` address built from your own U value. You already know how to configure a loopback — that is not what this tests. From this lab forward, both your device identity and your addressing in submitted evidence must be individually yours, not just proof that the shared course topology exists. That's what prevents two students' files from being indistinguishable, or one student's evidence being reused by another.

#### Action

Credentials used throughout this lab (also shown inside the CML lab notes):

- **W01-EDGE (router):** username `admin`, secret `cisco`
- **Alpine (Linux host):** `USERNAME=admin`, `PASSWORD=cisco`

1. **Download** [`L01-Tool-Readiness.yaml`](../cml/L01-Tool-Readiness.yaml) from the course repository. In `CML-Free`, choose **Import** (or the equivalent "add lab from file" action) and select the file you just downloaded.
2. Open the imported lab. Do not start nodes yet.
3. **Before starting anything, note the IP address your own host machine is using to reach the CML-Free web UI.** That address's network is the same network `EXTERNAL-Bridge` will use to hand out addresses to `W01-EDGE` and `Alpine` — so you already know what network to expect their DHCP addresses on.
4. Click the **lab notes** panel for the exact commands; the steps below summarize the same work.
5. Start all four nodes: `W01-EDGE`, `Alpine`, `EXTERNAL-Bridge`, and `EXTERNAL-Sw`.
6. Open one console to `W01-EDGE` and one to `Alpine`. Do not change the configuration yet.
7. On the `W01-EDGE` console, verify its interfaces are configured properly before doing anything else:
   ```text
   W01-EDGE#show ip int brief | ex una
   Interface              IP-Address      OK? Method Status                Protocol
   Ethernet0/0            172.16.1.1      YES TFTP   up                    up
   Ethernet0/1            192.168.198.135 YES DHCP   up                    up
   ```
   `Ethernet0/0` is the static `172.16.1.1` internal link, preconfigured for this lab via TFTP. `Ethernet0/1` should show a DHCP address on the same network as the one you noted in step 3 for the CML-Free web UI — that's what gives your own host connectivity to the router.
8. On the `Alpine` console, do the same:
   ```text
   Alpine:~$ ip address
   ```
   `eth0` should show the static `172.16.1.10/24` internal link. `eth1` should show a DHCP address on the same bridge network as `W01-EDGE`'s `Ethernet0/1`.
9. **Test connectivity.**
   - From your own host machine (not a console), ping `W01-EDGE`'s `Ethernet0/1` DHCP address and `Alpine`'s `eth1` DHCP address — both should succeed.
   - From the `Alpine` console, ping `W01-EDGE`'s internal address:
     ```text
     Alpine:~$ ping -c 2 172.16.1.1
     PING 172.16.1.1 (172.16.1.1): 56 data bytes
     64 bytes from 172.16.1.1: seq=0 ttl=42 time=1.413 ms
     64 bytes from 172.16.1.1: seq=1 ttl=42 time=1.487 ms
     --- 172.16.1.1 ping statistics ---
     2 packets transmitted, 2 packets received, 0% packet loss
     round-trip min/avg/max = 1.413/1.450/1.487 ms
     ```
     **Evidence limit:** this proves Alpine can exchange ICMP traffic with its local gateway. It does not prove that an application, a remote network, or every route works. This exact output shape (packets transmitted/received, 0% loss) is what the automated collection checks for later.
10. From `Alpine`, connect to `W01-EDGE` over SSH to confirm SSH access specifically — not just console access — using the training credentials from the CML startup configuration (username `admin`, secret `cisco`):
    ```text
    ssh admin@172.16.1.1
    ```
11. From your own host machine, confirm SSH also succeeds to both devices using their DHCP addresses (password `cisco` for both):
    ```bash
    ssh admin@{W01-EDGE-DHCP-address}
    ssh admin@{Alpine-DHCP-address}
    ```

**Now you're ready to make changes.** Start with `W01-EDGE`, then `Alpine`, then move to C03 to collect everything.

12. **Rename `W01-EDGE`** to include your own Algonquin username, using the SSH session from step 11. From this lab forward, device identity in course evidence must be traceable to you individually, not just to the shared course topology.
    ```text
    W01-EDGE# configure terminal
    W01-EDGE(config)# hostname W01-EDGE-{username}
    W01-EDGE-{username}(config)# end
    ```
13. Confirm the rename and the session state, including live identity proof:
    ```text
    show ip ssh
    show version | include uptime
    show users
    ```
    **Evidence limit:** this proves an authenticated SSH session reached the renamed `W01-EDGE`. It does not prove that any later course service or remote destination is reachable.
14. **Find your U value.** In Brightspace, under Grades, locate the grade item literally labeled **U** — a small integer (e.g. `10`, `47`, `133`). Use it as-is: it is not a raw student number and requires no calculation.
15. **Compute your `Loopback0` address.** Your address is `203.0.113.{U}`, where `{U}` is exactly the number shown as your U grade — no derivation or formula. Example: a U value of `10` gives `203.0.113.10/32`. `203.0.113.0/24` is RFC 5737 TEST-NET-3 space — reserved for documentation, not routable on the Internet, and not part of any private range (`10/8`, `172.16/12`, `192.168/16`), so it cannot collide with your home network.
16. **Configure `Loopback0`** on `W01-EDGE`, using the same session.
    ```text
    W01-EDGE-{username}# configure terminal
    W01-EDGE-{username}(config)# interface Loopback0
    W01-EDGE-{username}(config-if)# ip address 203.0.113.{U} 255.255.255.255
    W01-EDGE-{username}(config-if)# end
    ```
17. **Confirm the interface is up** before relying on it as evidence:
    ```text
    show ip interface brief | include Loopback0
    ```
18. **Rename `Alpine`** the same way, using its live `hostname` command. This is a session-local `CML-Free` instance you control — a transient rename is sufficient; you do not need to edit `/etc/hostname` for this to survive the C03 collection within the same session.
    ```bash
    hostname Alpine-{username}
    ```

#### Verification

Steps 7–9 confirm the topology is up and reachable before you touch anything. Steps 13, 17, and 18 are the verification that your changes took: an authenticated SSH session on the renamed `W01-EDGE` showing SSH enabled and the active session; `Loopback0` showing your U-based address, up/up; and `Alpine` returning its renamed `hostname`.

#### Success Indicator / Failure Signal

| Verification Item | Success Indicator | Failure Signal |
|---|---|---|
| Node consoles | All node consoles are available after start | Console will not open, or node stuck starting |
| W01-EDGE interfaces (pre-rename) | `show ip int brief \| ex una` shows Ethernet0/0 up/up at 172.16.1.1 and Ethernet0/1 up/up with a DHCP address matching your host's web-UI network | An interface is missing, down, or Ethernet0/1 has no address |
| Alpine addressing | `ip address` shows eth0 at 172.16.1.10/24 and eth1 with a DHCP address on the same bridge network as W01-EDGE's Ethernet0/1 | No address on either interface |
| Internal reachability | `ping -c 2 172.16.1.1` from Alpine shows 2 packets transmitted, 2 received, 0% packet loss | Any packet loss, or ping fails entirely |
| Host reachability | Ping and SSH succeed from your own host to both devices' DHCP addresses | Address not visible, ping fails, or SSH fails from host |
| W01-EDGE renamed | `show version \| include uptime` shows `W01-EDGE-{username} uptime is ...`; `show ip ssh` reports SSH enabled; `show users` shows the session | Hostname unchanged, SSH disabled, or connection refused |
| `Loopback0` address | `show ip interface brief \| include Loopback0` shows your `203.0.113.{U}` address, status `up`, protocol `up` | `Loopback0` missing, address does not match your U value, or status is not up/up |
| Alpine renamed | `hostname` returns `Alpine-{username}` | `hostname` still returns the original name |

#### Troubleshooting

If a node console will not open: refresh the CML page once, then report the lab name, node name, and visible error to your instructor — do not use **Wipe** as a first response. If host-to-device ping/SSH fails: confirm `EXTERNAL-Bridge` is actually bridged to your host's network (the address you noted in step 3) and that your host is on the same network segment before repeatedly retrying. If the rename does not stick: on W01-EDGE, confirm you were in global config mode (`hostname` is rejected outside config mode); on Alpine, re-run `hostname Alpine-{username}` and confirm with a plain `hostname` call — a transient rename only needs to hold for the current session's collections. If `Loopback0` does not appear: confirm you typed `interface Loopback0` (capital L, no space) and `end` to leave config mode. If the address is wrong: recheck the U value shown in Brightspace Grades — it is used exactly as shown, never calculated, and never copied from another student.

#### C02 — Collection of Information

C02 has no separately submitted evidence block. All of C02's evidence — the device rename, the `Loopback0` address, the SSH session, and host reachability — is captured automatically inside the `x_remote` output file produced in C03 (`l01-{username}.txt`). See C03's Collection of Information for what that single file must contain.

---

### C03 — Use `x_remote` to Collect Evidence

#### Goal

Validate and run the supplied `x_remote` command file, then confirm the resulting output file exists, is non-empty, has the expected name, and contains real evidence from both devices.

#### Why This Matters

`x_remote` is the evidence-collection tool most Foundation Labs use instead of Packet Tracer. A collection that "ran" without errors is not the same as a collection that proves anything — the resulting file must still be checked before you trust or submit it. Complete C02 first: you need the confirmed DHCP addresses before this step.

#### Action

**Download** [`l01-xremote.yaml`](yaml/l01-xremote.yaml) from the course repository.

Edit the file before running it:

- Replace `{W01-EDGE-DHCP-address}` and `{Alpine-DHCP-address}` with the DHCP addresses you verified in C02.
- Replace `{username}` with your Algonquin username.

Everything else in the manifest — including `show tcp brief`, `show version | include uptime`, and `ping -c 2 172.16.1.1` — runs automatically with the rest of the collection; you do not need to run any of it separately.

Then run the collection from the host:

```bash
python3 x_remote.py l01-xremote.yaml
```

#### Verification

Open `l01-{username}.txt` and confirm it contains W01-EDGE's renamed identity (via `show version | include uptime`) and all router command output, plus Alpine's renamed identity (via `hostname`) and all Linux command output, including a successful `ping -c 2 172.16.1.1` — not only an end-of-collection message.

#### Success Indicator / Failure Signal

| Verification Item | Success Indicator                                          | Failure Signal                                                      |
| ----------------- | ---------------------------------------------------------- | ------------------------------------------------------------------- |
| Live collection   | Command completes without connection errors                | `Authentication failed` or `Connection timed out` for either device |
| Output file       | `l01-{username}.txt` exists and is non-empty               | File missing, still default-named, or size is zero                  |
| Content           | Both W01-EDGE and Alpine sections show real command output, including the renamed identity in each and a 100% successful ping | Missing section, error text, unrenamed identity, or a partial/failed ping |

#### C03 — Collection of Information

Your submission **is** the `x_remote` output file — there is no separate manually-assembled evidence block. This single collection run captures **all** evidence from both C02 and C03: the C02 device rename (`W01-EDGE` and Alpine), the C02 `Loopback0` address, the C02 SSH/reachability proof, and C03's own Alpine addressing and reachability evidence — one file, one run.

**What to Include**

| Requirement | Details |
|---|---|
| Filename | `l01-{username}.txt` |
| Non-empty | File size greater than zero |
| W01-EDGE section | Renamed identity (`show version \| include uptime`), all router command output, and `Loopback0` up/up at your U-based address (`show ip interface brief \| include Loopback0`) |
| Alpine section | Renamed identity (`hostname`) and all Linux command output present, including a 100% successful `ping -c 2 172.16.1.1` |
| Content check | No error text or empty output in place of a command's result |

This single file is what you submit to Brightspace — see Section D.

---

## Section D — Submission

### D1 — Submission Requirements

Submit a single file:

```text
l01-{username}.txt
```

This is the completed, non-empty `x_remote` collection file from C03, renamed and unedited. Do not add screenshots, a separate readiness record, or a manually assembled output file.

### D2 — Submit to Brightspace

Your submission is your responsibility. Before leaving the lab, prove:

1. The Brightspace assignment for Foundation Lab 01 shows the file as successfully uploaded.
2. The uploaded file name is exactly `l01-{username}.txt`.
3. The uploaded file has non-zero size — open it from Brightspace (or your local copy) and confirm real command output is visible, not just an end-of-collection banner.

If your collection fails, retain the exact error and contact your instructor before the deadline. Do not fabricate or manually assemble a collection file.
