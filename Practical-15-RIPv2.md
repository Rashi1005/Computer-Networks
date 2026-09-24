# Practical 15 – Configuring RIPv2 (Classless) Routing in Cisco Packet Tracer

**Name:** Rashi
**Institution:** SRM Institute of Science and Technology (Delhi NCR Campus)
**Date:** 24 September 2026
**Tool:** Cisco Packet Tracer

---

## Aim

To configure **RIPv2 (classless)** routing on a three-router topology using **VLSM** on a single Class C network (`192.168.20.0/24`), and to verify end-to-end connectivity.

## Theory

RIPv1 is **classful**: it does not send the subnet mask in its updates, so it cannot handle mixed mask lengths (VLSM) inside one major network. RIPv2 is **classless**: every route advertisement carries its subnet mask.

| | RIPv1 | RIPv2 |
|---|---|---|
| Classful / Classless | Classful | Classless |
| Subnet mask in updates | No | Yes |
| Supports VLSM | No | Yes |
| Update delivery | Broadcast (255.255.255.255) | Multicast (224.0.0.9) |
| Authentication | Not supported | Supported (plain text / MD5) |
| Auto-summarization | Always on | Configurable (`no auto-summary`) |

Here the LANs use `/26` and the WAN links use `/30`, both inside `192.168.20.0`. RIPv1 cannot distinguish them; RIPv2 with `no auto-summary` advertises each subnet with its real mask.

---

## Topology

- 3 routers (**R0, R1, R2**, Router-PT-Empty) connected in a line via serial links
- 3 switches (**S0, S1, S2**, 2960) – one per router
- 6 PCs (**PC0–PC5**) – two per switch

### Step 1 – Device placement

![Devices placed](images/01-devices-placed.png)

### Step 2 – Devices renamed

![Devices renamed](images/02-devices-renamed.png)

### Step 3 – Cabling

| From | To | Interface |
|------|----|-----------|
| PC0 / PC1 | S0 | fa0/1, fa0/2 |
| S0 | R0 | fa0/24 → fa2/0 |
| PC2 / PC3 | S1 | fa0/1, fa0/2 |
| S1 | R1 | fa0/24 → fa2/0 |
| PC4 / PC5 | S2 | fa0/1, fa0/2 |
| S2 | R2 | fa0/24 → fa2/0 |
| R0 ↔ R1 | Serial | se0/0 ↔ se1/0 |
| R1 ↔ R2 | Serial | se0/0 ↔ se1/0 |

![Cabling complete](images/03-cabling.png)

---

## IP Addressing Scheme (VLSM)

| Subnet | Devices | Mask |
|--------|---------|------|
| 192.168.20.0/26 | PC0, PC1, R0 (fa2/0) | 255.255.255.192 |
| 192.168.20.64/26 | PC2, PC3, R1 (fa2/0) | 255.255.255.192 |
| 192.168.20.128/26 | PC4, PC5, R2 (fa2/0) | 255.255.255.192 |
| 192.168.20.192/30 | R0 (se0/0) ↔ R1 (se1/0) | 255.255.255.252 |
| 192.168.20.196/30 | R1 (se0/0) ↔ R2 (se1/0) | 255.255.255.252 |

### PC configuration

| PC | IP Address | Subnet Mask | Default Gateway |
|----|-----------|-------------|-----------------|
| PC0 | 192.168.20.10 | 255.255.255.192 | 192.168.20.1 |
| PC1 | 192.168.20.11 | 255.255.255.192 | 192.168.20.1 |
| PC2 | 192.168.20.70 | 255.255.255.192 | 192.168.20.65 |
| PC3 | 192.168.20.71 | 255.255.255.192 | 192.168.20.65 |
| PC4 | 192.168.20.140 | 255.255.255.192 | 192.168.20.129 |
| PC5 | 192.168.20.141 | 255.255.255.192 | 192.168.20.129 |

**PC0**
![PC0 IP](images/09-pc0-ip.png)

**PC1**
![PC1 IP](images/08-pc1-ip.png)

**PC2**
![PC2 IP](images/07-pc2-ip.png)

**PC3**
![PC3 IP](images/06-pc3-ip.png)

**PC4**
![PC4 IP](images/05-pc4-ip.png)

**PC5**
![PC5 IP](images/04-pc5-ip.png)

---

## Router Configuration

### R0

```
enable
configure terminal
hostname R0

interface fa2/0
ip address 192.168.20.1 255.255.255.192
no shutdown
exit

interface se0/0
ip address 192.168.20.193 255.255.255.252
clock rate 64000
no shutdown
exit

router rip
version 2
no auto-summary
network 192.168.20.0
exit

end
write memory
```

![R0 configuration](images/10-r0-config.png)

### R1

```
enable
configure terminal
hostname R1

interface fa2/0
ip address 192.168.20.65 255.255.255.192
no shutdown
exit

interface se1/0
ip address 192.168.20.194 255.255.255.252
no shutdown
exit

interface se0/0
ip address 192.168.20.197 255.255.255.252
clock rate 64000
no shutdown
exit

router rip
version 2
no auto-summary
network 192.168.20.0
exit

end
write memory
```

![R1 configuration](images/11-r1-config.png)

### R2

```
enable
configure terminal
hostname R2

interface fa2/0
ip address 192.168.20.129 255.255.255.192
no shutdown
exit

interface se1/0
ip address 192.168.20.198 255.255.255.252
no shutdown
exit

router rip
version 2
no auto-summary
network 192.168.20.0
exit

end
write memory
```

![R2 configuration](images/12-r2-config.png)

> **Note:** `write memory` is a privileged-EXEC command. Entering it in global config mode returns `% Invalid input detected`; it was re-run after `end` and returned `[OK]`.

---

## Verification

### 1. `show ip protocols` – confirm RIP version 2

On R0: RIP sends and receives **version 2** on `FastEthernet2/0` and `Serial0/0`, and **automatic network summarization is not in effect**. Routing is enabled for `192.168.20.0`, with neighbour `192.168.20.194` (R1) as a routing information source.

![R0 show ip protocols](images/13-r0-show-ip-protocols.png)

### 2. `show ip route` – routing table with correct masks

On R0, the table shows `192.168.20.0/24 is variably subnetted, 5 subnets, 2 masks`:

| Code | Network | Via |
|------|---------|-----|
| C | 192.168.20.0/26 | FastEthernet2/0 (connected) |
| R | 192.168.20.64/26 [120/1] | 192.168.20.194, Serial0/0 |
| R | 192.168.20.128/26 [120/2] | 192.168.20.194, Serial0/0 |
| C | 192.168.20.192/30 | Serial0/0 (connected) |
| R | 192.168.20.196/30 [120/1] | 192.168.20.194, Serial0/0 |

Each route is shown with its real `/26` or `/30` mask instead of being collapsed into a `/24` – this confirms classless (VLSM) operation.

![R0 show ip route](images/14-r0-show-ip-route.png)

### 3. `show ip route rip` – RIP-learned routes only

![R0 show ip route rip](images/15-r0-show-ip-route-rip.png)

### 4. R1 verification

R1 learns `192.168.20.0/26` from R0 (via 192.168.20.193, Serial1/0) and `192.168.20.128/26` from R2 (via 192.168.20.198, Serial0/0); it uses two RIP neighbours.

![R1 verification](images/16-r1-verification.png)

### 5. R2 verification

R2 learns `192.168.20.0/26` [120/2], `192.168.20.64/26` [120/1] and `192.168.20.192/30` [120/1] via 192.168.20.197 (R1).

![R2 verification](images/17-r2-verification.png)

---

## Connectivity Test

### PC3 → PC4 (`ping 192.168.20.140`)

4 packets sent, 4 received, 0% loss, TTL = 126.

![PC3 ping PC4](images/18-pc3-ping-pc4.png)

### PC0 → PC2 and PC0 → PC4

- `ping 192.168.20.70` – 3/4 received (first request timed out due to ARP resolution), TTL = 126
- `ping 192.168.20.140` – 3/4 received (first request timed out due to ARP resolution), TTL = 125

The TTL values (126 across two routers, 125 across three) match the expected hop counts.

![PC0 ping and ipconfig](images/19-pc0-ping-ipconfig.png)

---

## Result

RIPv2 was configured with `no auto-summary` on R0, R1 and R2. All subnets (`/26` LANs and `/30` WAN links) were advertised with their correct masks, routing tables converged, and all PCs across different networks could ping each other successfully. This confirms that RIPv2 supports VLSM, which RIPv1 cannot.

## Conclusion

Using RIPv2 with `no auto-summary` allows a single classful network to be divided with variable-length masks and still be routed correctly. The first ping timeout in each test is normal ARP behaviour; subsequent packets succeed.
