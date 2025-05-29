# Static Routing – Route Not Present in the Routing Table

### Question: Static Route Not Installed

Router **R3** has the following network interfaces:
```
R3# show ip interface brief
Interface              IP-Address      OK? Method Status      Protocol
GigabitEthernet0/0     10.1.1.1        YES manual up          up
GigabitEthernet0/1     192.168.10.1    YES manual up          up
Loopback0              172.16.1.1      YES manual up          up
```

No dynamic routing protocol is running in R3.

You are given the following static route configuration on Router R3:

```
R3(config)# ip route 192.168.50.0 255.255.255.0 10.10.10.2
```

And the following command output:

```
R3# show ip route static
% No static routes in routing table
```

**Question:**

1. Why is the static route not present in the routing table?  
2. What two conditions must be met for a static route to appear in the routing table?  
3. What could you do to ensure the missing route gets installed?
