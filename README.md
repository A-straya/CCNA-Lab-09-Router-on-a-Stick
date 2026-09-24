
# Cisco Router-on-a-Stick — Inter-VLAN Routing

![Cisco](https://img.shields.io/badge/Cisco-IOS-blue)
![Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-orange)
![Networking](https://img.shields.io/badge/Topic-Router-green)
![Level](https://img.shields.io/badge/Level-Intermediate-red)


A Cisco Packet Tracer lab demonstrating **Inter-VLAN Routing using the Router-on-a-Stick method**.

The lab covers VLAN creation, access-port configuration, 802.1Q trunking, router subinterfaces, default gateways, and connectivity verification.

---

## 1. Objectives

* Create VLAN 10 and VLAN 30 on a Cisco switch.
* Assign switch interfaces to the appropriate VLANs.
* Configure access ports.
* Configure an 802.1Q trunk between the switch and router.
* Configure router subinterfaces.
* Configure inter-VLAN routing using Router-on-a-Stick.
* Configure IPv4 addressing and default gateways.
* Verify connectivity using ICMP `ping`.
* Verify VLAN, trunk, and interface configurations using Cisco IOS commands.

---

## 2. Network Topology

```text
                              802.1Q TRUNK
                         Gi0/1             Gi0/0
                    +-------------+   +-------------+
                    |             |   |             |
                    |     S1      +===+     R1      |
                    |  Catalyst   |   |   Router    |
                    |    2960     |   |             |
                    +--+-------+--+   +------+------+
                       |       |              |
                    Fa0/11   Fa0/6            |
                       |       |              |
                       |       |              |
                     [PC1]   [PC3]            |
                       |       |              |
                    VLAN 10  VLAN 30           |
                       |       |              |
               172.17.10.10  172.17.30.10     |
```

### Logical topology

```text
[PC1]
  |
  | Fa0/11
  | VLAN 10
  |
 [S1]
  |
  | Gi0/1
  | 802.1Q TRUNK
  |
 [R1]
  |
  +-- Gi0/0.10 --> VLAN 10
  |
  +-- Gi0/0.30 --> VLAN 30
  |
 [Inter-VLAN Routing]
  |
 [S1]
  |
  | Fa0/6
  | VLAN 30
  |
[PC3]
```

---

## 3. Addressing Table

| Device | Interface | IPv4 Address | Subnet Mask   | Default Gateway | VLAN |
| ------ | --------- | ------------ | ------------- | --------------- | ---- |
| R1     | Gi0/0.10  | 172.17.10.1  | 255.255.255.0 | N/A             | 10   |
| R1     | Gi0/0.30  | 172.17.30.1  | 255.255.255.0 | N/A             | 30   |
| PC1    | NIC       | 172.17.10.10 | 255.255.255.0 | 172.17.10.1     | 10   |
| PC3    | NIC       | 172.17.30.10 | 255.255.255.0 | 172.17.30.1     | 30   |

The addressing information follows the Cisco Packet Tracer lab provided for this exercise.

---

# 4. Physical Connections

| Device | Interface | Connection | Device | Interface |
| ------ | --------- | ---------- | ------ | --------- |
| PC1    | NIC       | Access     | S1     | Fa0/11    |
| PC3    | NIC       | Access     | S1     | Fa0/6     |
| S1     | Gi0/1     | Trunk      | R1     | Gi0/0     |

The lab assigns the PC-connected interfaces as access ports and uses the switch interface connected to the router for trunking.

---

# 5. VLAN Configuration — S1

Enter privileged EXEC mode:

```cisco
S1> enable
S1# configure terminal
```

Create VLAN 10:

```cisco
S1(config)# vlan 10
S1(config-vlan)# name VLAN10
S1(config-vlan)# exit
```

Create VLAN 30:

```cisco
S1(config)# vlan 30
S1(config-vlan)# name VLAN30
S1(config-vlan)# exit
```

Verify:

```cisco
S1# show vlan brief
```

---

# 6. Access Port Configuration

## PC1 — VLAN 10

PC1 is connected to `Fa0/11`.

```cisco
S1(config)# interface fa0/11
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 10
S1(config-if)# exit
```

## PC3 — VLAN 30

PC3 is connected to `Fa0/6`.

```cisco
S1(config)# interface fa0/6
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 30
S1(config-if)# exit
```

Verify:

```cisco
S1# show vlan brief
```

Expected VLAN assignments:

```text
VLAN  Name       Status    Ports
----  ---------  --------  ----------------
10    VLAN10     active    Fa0/11
30    VLAN30     active    Fa0/6
```

The Cisco lab identifies `Fa0/11` and `Fa0/6` as the two access interfaces and assigns them to VLAN 10 and VLAN 30 respectively.

---

# 7. Configure the 802.1Q Trunk

The link between S1 and R1 must carry traffic from both VLAN 10 and VLAN 30.

Configure `S1 Gi0/1`:

```cisco
S1(config)# interface gi0/1
S1(config-if)# switchport mode trunk
S1(config-if)# exit
```

Verify:

```cisco
S1# show interfaces trunk
```

The Cisco lab requires the switch port connected to the router to be configured as a trunk because the router uses multiple VLAN subinterfaces.

---

# 8. Router-on-a-Stick Configuration

Router-on-a-Stick uses a single physical router interface with multiple logical subinterfaces.

```text
                         R1
                          |
                        Gi0/0
                          |
             +------------+------------+
             |                         |
         Gi0/0.10                   Gi0/0.30
          VLAN 10                    VLAN 30
             |                         |
      172.17.10.1                172.17.30.1
```

---

## 8.1 Enable the Physical Interface

```cisco
R1> enable
R1# configure terminal

R1(config)# interface gi0/0
R1(config-if)# no shutdown
R1(config-if)# exit
```

---

## 8.2 Configure VLAN 10 Subinterface

```cisco
R1(config)# interface gi0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 172.17.10.1 255.255.255.0
R1(config-subif)# exit
```

The Cisco lab specifies `G0/0.10`, 802.1Q VLAN 10 encapsulation, and the `172.17.10.1/24` address.

---

## 8.3 Configure VLAN 30 Subinterface

```cisco
R1(config)# interface gi0/0.30
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 172.17.30.1 255.255.255.0
R1(config-subif)# exit
```

---

## 8.4 Verify Router Interfaces

```cisco
R1# show ip interface brief
```

Expected:

```text
Interface              IP-Address      Status    Protocol
Gi0/0                  unassigned      up        up
Gi0/0.10               172.17.10.1     up        up
Gi0/0.30               172.17.30.1     up        up
```

The physical `Gi0/0` must be enabled because the subinterfaces depend on the physical interface.

---

# 9. PC Configuration

## PC1

Configure the PC1 NIC:

```text
IP Address:      172.17.10.10
Subnet Mask:     255.255.255.0
Default Gateway: 172.17.10.1
```

## PC3

Configure the PC3 NIC:

```text
IP Address:      172.17.30.10
Subnet Mask:     255.255.255.0
Default Gateway: 172.17.30.1
```

The default gateways correspond to the router subinterfaces for each VLAN.

---

# 10. Connectivity Verification

## PC1 → VLAN 10 Gateway

From PC1:

```text
C:\> ping 172.17.10.1
```

Expected:

```text
Reply from 172.17.10.1
```

---

## PC3 → VLAN 30 Gateway

From PC3:

```text
C:\> ping 172.17.30.1
```

Expected:

```text
Reply from 172.17.30.1
```

---

## PC1 → PC3

From PC1:

```text
C:\> ping 172.17.30.10
```

Expected:

```text
Reply from 172.17.30.10
```

This verifies communication between VLAN 10 and VLAN 30 through R1.

---

## PC3 → PC1

From PC3:

```text
C:\> ping 172.17.10.10
```

Expected:

```text
Reply from 172.17.10.10
```

The lab states that, when the configuration is correct, PC1 and PC3 should be able to ping their default gateways and each other.

---

# 11. Verification Commands

## Display VLANs

```cisco
S1# show vlan brief
```

## Display trunk status

```cisco
S1# show interfaces trunk
```

## Display interface status

```cisco
S1# show ip interface brief
```

On the router:

```cisco
R1# show ip interface brief
```

## Display running configuration

```cisco
R1# show running-config
```

## Display routing table

```cisco
R1# show ip route
```

---

# 12. Troubleshooting

If inter-VLAN communication fails, verify the following.

### VLAN assignment

```cisco
S1# show vlan brief
```

Expected:

```text
Fa0/11 → VLAN 10
Fa0/6  → VLAN 30
```

### Trunk

```cisco
S1# show interfaces trunk
```

Expected:

```text
Gi0/1 → trunk
```

### Router subinterfaces

```cisco
R1# show ip interface brief
```

Expected:

```text
Gi0/0.10 → 172.17.10.1
Gi0/0.30 → 172.17.30.1
```

### Physical router interface

```cisco
R1(config)# interface gi0/0
R1(config-if)# no shutdown
```

### PC addressing

PC1:

```text
172.17.10.10/24
Gateway: 172.17.10.1
```

PC3:

```text
172.17.30.10/24
Gateway: 172.17.30.1
```

---

# 13. How Router-on-a-Stick Works

The packet flow between the two VLANs is:

```text
              VLAN 10
                 |
              [PC1]
                 |
              Fa0/11
                 |
                 v
                [S1]
                 |
              Gi0/1
                 ||
                 || 802.1Q TRUNK
                 ||
              Gi0/0
                [R1]
                 |
        +--------+--------+
        |                 |
     Gi0/0.10          Gi0/0.30
     VLAN 10           VLAN 30
        |                 |
  172.17.10.1       172.17.30.1
        |                 |
        +---- ROUTING ----+
                 |
                [S1]
                 |
              Fa0/6
                 |
              [PC3]
                 |
              VLAN 30
```

The switch uses the trunk to transport traffic from multiple VLANs to the router.

The router uses the subinterfaces to identify the VLANs and route traffic between their IP networks.

---

# 14. Key Cisco Concepts

## VLAN

A VLAN creates a separate Layer 2 broadcast domain.

```text
VLAN 10 → 172.17.10.0/24
VLAN 30 → 172.17.30.0/24
```

## Access Port

An access port carries traffic for a single VLAN.

```text
Fa0/11 → VLAN 10
Fa0/6  → VLAN 30
```

## Trunk Port

A trunk carries traffic for multiple VLANs.

```text
S1 Gi0/1
     ||
     || 802.1Q
     ||
R1 Gi0/0
```

## Router Subinterface

A router subinterface is a logical interface associated with a VLAN.

```text
Gi0/0.10 → VLAN 10 → 172.17.10.1
Gi0/0.30 → VLAN 30 → 172.17.30.1
```

## Inter-VLAN Routing

The router provides Layer 3 communication between different VLANs.

```text
VLAN 10
172.17.10.0/24
      |
      | R1
      |
VLAN 30
172.17.30.0/24
```

---

# 15. Configuration Summary

### S1

```cisco
vlan 10
vlan 30

interface fa0/11
 switchport mode access
 switchport access vlan 10

interface fa0/6
 switchport mode access
 switchport access vlan 30

interface gi0/1
 switchport mode trunk
```

### R1

```cisco
interface gi0/0
 no shutdown

interface gi0/0.10
 encapsulation dot1Q 10
 ip address 172.17.10.1 255.255.255.0

interface gi0/0.30
 encapsulation dot1Q 30
 ip address 172.17.30.1 255.255.255.0
```

---

# 16. Lab Files

```text
.
├── README.md
└── router-on-a-stick.pkt
```

Open the `.pkt` file with **Cisco Packet Tracer** to reproduce the topology and configuration.

---

# 17. Technologies

```text
Cisco Packet Tracer
Cisco IOS
VLAN
802.1Q
Trunking
Router-on-a-Stick
Inter-VLAN Routing
IPv4
ICMP
```

---

# 18. Conclusion

This lab demonstrates how a Cisco router can provide communication between separate VLANs using the **Router-on-a-Stick** architecture.

The essential architecture is:

```text
VLANs
  |
  v
[SWITCH]
  |
  | 802.1Q TRUNK
  v
[ROUTER]
  |
  +-- Gi0/0.10 → VLAN 10
  |
  +-- Gi0/0.30 → VLAN 30
  |
  v
INTER-VLAN ROUTING
```

The main configuration relationship to remember is:

```text
Access Port
    ↓
VLAN
    ↓
Trunk
    ↓
Router Subinterface
    ↓
Default Gateway
    ↓
Inter-VLAN Routing
```

---

## Cisco IOS Verification

```cisco
S1# show vlan brief
S1# show interfaces trunk

R1# show ip interface brief
R1# show ip route
R1# show running-config
```

Connectivity:

```text
PC1> ping 172.17.10.1
PC1> ping 172.17.30.10

PC3> ping 172.17.30.1
PC3> ping 172.17.10.10
```
