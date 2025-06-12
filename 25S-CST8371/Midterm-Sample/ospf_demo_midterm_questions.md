# OSPF Evaluation – Applied Questions

These questions are designed to assess your practical understanding of OSPF, as configured in the demo topology used in class.

---

## 🔸 Question 1 – OSPF Route Analysis

You are on **Router RB**. Use `show ip route` and `show ip ospf` to answer the following:

1. Which OSPF routes are installed? What is the **cost** to reach the 192.0.2.0/24 network?
2. What path is used to reach the TFTP server? Include next-hop and interface.
3. Suppose the reference bandwidth was left at default (100 Mbps), and all links are Gigabit. How would this affect cost accuracy?

---

## 🔸 Question 2 – Neighbourship Troubleshooting

RA and RB are directly connected over `E0/2`, but OSPF neighbourship is not forming. You run:

```
RB# show ip ospf interface e0/2
  Hello 10, Dead 40
RA# show ip ospf interface e0/2
  Hello 5, Dead 20
```

1. What is preventing the OSPF adjacency from forming?
2. Suggest and apply a configuration fix.
3. How would the issue differ if the subnet masks did not match?

---

## 🔸 Question 3 – Default Route Propagation

You configure this on RA:

```bash
ip route 0.0.0.0 0.0.0.0 203.0.113.2
router ospf 100
 default-information originate
```

On RB:
```
RB# show ip route ospf
```

...no default route is shown.

1. Identify two possible reasons why RB does **not** receive the default route.
2. What additional command could be added to RA to force the advertisement?
3. If RA loses its default route but is still originating it, what are the implications?

---

## 🔸 Question 4 – Router ID and Passive Interface

1. Router RB has loopback0 (`10.100.100.100/32`) and `E0/1` (`10.100.10.2/24`). Which will be selected as the Router ID and why?
2. You later configure `router-id 1.1.1.1` manually. What must you do for the change to take effect?
3. Why is it a good practice to make loopback interfaces **passive** in OSPF?

---

## 🔸 Question 5 – Cost Calculation and Path Preference

You are tasked with ensuring traffic from RB to REMOTE prefers the path through `RA → E0/2`.

1. Manually calculate the cost of each path if both links are 100 Mbps.
2. Use the `ip ospf cost` command to influence the route preference. Which interface will you change, and what value will you set?
3. How does adjusting the reference bandwidth globally differ from changing the cost locally?
