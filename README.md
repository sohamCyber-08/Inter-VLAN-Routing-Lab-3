

````markdown


🧪 Inter-VLAN Routing Lab
🎯 Objective

Configure communication between two different VLANs using a Cisco router
and understand how Layer 3 routing enables communication between
separate Layer 2 broadcast domains.

This lab demonstrates two approaches:

1. Inter-VLAN routing using separate physical router interfaces.
2. Router-on-a-Stick using one physical router interface with multiple subinterfaces.






## 🖥️ Topology 1 — Separate Router Interfaces

```text
                         Router
                    ┌──────┴──────┐
                    │             │
                 Gi0/0          Gi0/1
                10.1.1.1        10.2.2.1
                    │             │
                 Gi0/2          Gi0/3
                    │             │
                    └─── Switch ──┘
                       /       \
                    Gi0/0     Gi0/1
                      |          |
                   VLAN 10     VLAN 20
                      |          |
               VPC_VLAN10    VPC_VLAN20
               10.1.1.10     10.2.2.10
````

<img width="1835" height="902" alt="Screenshot 2026-09-21 173837" src="https://github.com/user-attachments/assets/e86122cf-aed3-49ec-8370-1493637286fa" />

<br/>

## 🌐 IP Addressing

| Device       | VLAN | IP Address | Subnet Mask   | Default Gateway |
| ------------ | ---: | ---------- | ------------- | --------------- |
| VPC_VLAN10   |   10 | 10.1.1.10  | 255.255.255.0 | 10.1.1.1        |
| Router Gi0/0 |   10 | 10.1.1.1   | 255.255.255.0 | -               |
| VPC_VLAN20   |   20 | 10.2.2.10  | 255.255.255.0 | 10.2.2.1        |
| Router Gi0/1 |   20 | 10.2.2.1   | 255.255.255.0 | -               |

<br/>

## 🔧 Technologies

* EVE-NG
* Cisco IOS
* VPCS
* IPv4
* VLAN
* Inter-VLAN Routing
* Router Interfaces
* Router-on-a-Stick
* IEEE 802.1Q
* ARP
* ICMP

<br/>

### Switch Configuration
<img width="1857" height="532" alt="Screenshot 2026-09-21 173958" src="https://github.com/user-attachments/assets/ea267cca-1319-44f4-90ff-13bee424ca4d" />
<br/>
<br/>

<img width="1896" height="418" alt="Screenshot 2026-09-21 174010" src="https://github.com/user-attachments/assets/d791f820-868d-44ff-bb7c-b14da5601a23" />




<br/>



<br/>
<br/>

# 1️⃣ Initial Design — Separate Physical Router Interfaces

In the initial design, each VLAN uses a separate physical router
interface as its Layer 3 gateway.

<br/>

```text
VLAN 10
10.1.1.0/24
     |
     | Gateway: 10.1.1.1
     |
Router Gi0/0

VLAN 20
10.2.2.0/24
     |
     | Gateway: 10.2.2.1
     |
Router Gi0/1
```

<br/>

### Router Configuration
<img width="1916" height="871" alt="Screenshot 2026-09-21 174047" src="https://github.com/user-attachments/assets/a8c6990d-cc0e-429b-883a-893980c79cd8" />



<br/>

---

<br/>

## 📊 Communication Process — Initial Design

When VPC_VLAN10 communicates with VPC_VLAN20, the destination
belongs to a different IP subnet.

<br/>

```mermaid
flowchart TD
    A[VPC_VLAN10<br/>10.1.1.10<br/>VLAN 10] --> B[Check Destination IP]
    B --> C{Same Subnet?}
    C -->|No| D[Use Default Gateway<br/>10.1.1.1]
    D --> E[Check ARP Table]
    E --> F[ARP Request]
    F --> G[ARP Reply]
    G --> H[Create Ethernet Frame]
    H --> I[Access Port]
    I --> J[Switch<br/>VLAN 10]
    J --> K[Router Gi0/0<br/>10.1.1.1]
    K --> L[Layer 3 Routing]
    L --> M[Router Gi0/1<br/>10.2.2.1]
    M --> N[Switch<br/>VLAN 20]
    N --> O[Destination Access Port]
    O --> P[VPC_VLAN20<br/>10.2.2.10<br/>VLAN 20]
    P --> Q[ICMP Echo Reply]
    Q --> R[Successful Inter-VLAN Communication]
```

<br/>

### How It Works

VPC_VLAN10 determines that `10.2.2.10` belongs to a different
subnet. Therefore, it sends the traffic to its default gateway
`10.1.1.1`.

The router receives the packet on `Gi0/0`, performs a Layer 3 routing
lookup, and forwards the packet through `Gi0/1` toward the
`10.2.2.0/24` network.

The destination host then returns the ICMP Echo Reply through its
default gateway `10.2.2.1`.

<br/>
<br/>
<img width="1906" height="971" alt="Screenshot 2026-09-21 174126" src="https://github.com/user-attachments/assets/3036cf34-31ba-4609-bdde-2fa956e79b19" />

---

<br/>

# ⚠️ Problem with the Initial Design

The initial design successfully provides inter-VLAN communication,
but it requires a separate physical router interface for each VLAN.

<br/>

### Example

```text
VLAN 10 → Router Gi0/0
VLAN 20 → Router Gi0/1
VLAN 30 → Router Gi0/2
VLAN 40 → Router Gi0/3
```

<br/>

As the number of VLANs increases, more physical router interfaces
are required.

<br/>

### Limitations

* Higher physical interface consumption
* More physical cabling
* Limited scalability
* Increased hardware requirements
* Less efficient use of router interfaces

<br/>

This creates a physical interface scalability problem.

<br/>
<br/>

---

<br/>

# 2️⃣ Solution — Router-on-a-Stick

Router-on-a-Stick solves the physical interface limitation by using
one physical router interface with multiple logical subinterfaces.

The router-to-switch connection operates as an 802.1Q trunk and
carries traffic for multiple VLANs over the same physical link.

<br/>

```text
                    Router
                     Gi0/0
                       |
                 802.1Q Trunk
                       |
                    Gi0/2
                    Switch
                   /      \
                Gi0/0     Gi0/1
                  |         |
              VLAN 10     VLAN 20
                  |         |
           10.1.1.10    10.2.2.10
```


<br/>

<img width="1881" height="907" alt="Screenshot 2026-09-21 175805" src="https://github.com/user-attachments/assets/c3e5f2d5-9674-4803-8b38-5b791b977041" />


<br/>

The trunk carries both VLAN 10 and VLAN 20 traffic between the
switch and router.

<br/>
<br/>

---

<br/>

## 🔗 Router-on-a-Stick Configuration



<br/>
<img width="1897" height="888" alt="Screenshot 2026-09-21 175859" src="https://github.com/user-attachments/assets/cfb94bf5-ce69-4466-9575-54108acb506d" />


<br/>
<br/>

## 🔗 Switch Trunk Configuration

The switch interface connected to the router is configured as an
802.1Q trunk.

<br/>
<img width="1887" height="765" alt="Screenshot 2026-09-21 175842" src="https://github.com/user-attachments/assets/da02b9ec-ecce-46b3-92aa-99929e0303d6" />


The trunk carries both VLAN 10 and VLAN 20 traffic between the
switch and router.

<br/>
<br/>

---

<br/>

## 📊 Communication Process — Router-on-a-Stick

```mermaid
flowchart TD
    A[VPC_VLAN10<br/>10.1.1.10<br/>VLAN 10] --> B[Check Destination IP]
    B --> C{Same Subnet?}
    C -->|No| D[Use Default Gateway<br/>10.1.1.1]
    D --> E[ARP for Gateway MAC]
    E --> F[Ethernet Frame]
    F --> G[Switch Access Port<br/>VLAN 10]
    G --> H[802.1Q Trunk]
    H --> I[Router Gi0/0]
    I --> J[Gi0/0.10<br/>VLAN 10]
    J --> K[Layer 3 Routing]
    K --> L[Gi0/0.20<br/>VLAN 20]
    L --> M[802.1Q Trunk]
    M --> N[Switch]
    N --> O[VLAN 20 Access Port]
    O --> P[VPC_VLAN20<br/>10.2.2.10]
    P --> Q[ICMP Echo Reply]
    Q --> R[Successful Inter-VLAN Communication]
```

<br/>

### How It Works

The source host identifies that the destination belongs to another
subnet and sends the packet to its default gateway.

The switch forwards the VLAN 10 traffic toward the router over the
802.1Q trunk.

The router receives the tagged traffic on the physical interface and
processes it through subinterface `Gi0/0.10`.

The router performs a Layer 3 routing lookup and forwards the packet
through subinterface `Gi0/0.20`, which represents VLAN 20.

The traffic then travels back through the trunk to the switch and is
forwarded through the VLAN 20 access port to the destination host.

<br/>
<br/>

---

<br/>

## 🔍 Verification
### Test VLAN 10 to VLAN 20

```bash
ping 10.2.2.10
```

<br/>

### Test VLAN 20 to VLAN 10

```bash
ping 10.1.1.10
```

<img width="1902" height="1007" alt="Screenshot 2026-09-21 175924" src="https://github.com/user-attachments/assets/56170997-a0c1-459e-8d41-f7b69ca17cff" />



<br/>

<br/>
<br/>

---

<br/>

## 📝 Note

The first topology demonstrates inter-VLAN routing using separate
physical router interfaces. Each VLAN has its own physical Layer 3
router interface acting as its default gateway.

Although this design works correctly, it requires a dedicated
physical router interface for each VLAN.

<br/>

The second topology solves this limitation using Router-on-a-Stick.
A single physical router interface is divided into multiple logical
subinterfaces. Each subinterface is associated with a VLAN using
802.1Q encapsulation.

<br/>

The switch-to-router connection operates as an 802.1Q trunk, allowing
multiple VLANs to share the same physical link while the router
performs Layer 3 routing between the VLANs.

<br/>
<br/>

---

<br/>

## 💡 What Was Solved

The initial design required:

```text
1 VLAN = 1 Physical Router Interface
```

<br/>

Router-on-a-Stick changes this to:

```text
Multiple VLANs
      ↓
One Physical Router Interface
      ↓
Multiple Subinterfaces
      ↓
802.1Q Trunk
      ↓
Inter-VLAN Routing
```

<br/>

This reduces physical interface requirements and provides a more
scalable approach for VLAN-based networks.

<br/>
<br/>

---

<br/>

## ✅ Result

The initial topology successfully provided communication between
VLAN 10 and VLAN 20 using separate physical router interfaces.

<br/>

The physical interface scalability limitation was then solved using
Router-on-a-Stick. A single physical router interface was configured
with separate subinterfaces for VLAN 10 and VLAN 20, while the
switch-to-router connection was configured as an 802.1Q trunk.

<br/>

Both VLANs successfully communicated through Layer 3 routing, while
the router used the appropriate subinterface as the gateway for each
VLAN.

<br/>
<br/>

---

<br/>

## 📚 Key Learning

This lab helped me understand inter-VLAN routing, default gateways,
Layer 3 packet forwarding, physical router interfaces, router
subinterfaces, 802.1Q trunking, ARP, ICMP, and Router-on-a-Stick.

<br/>

I also learned the scalability limitation of using separate physical
router interfaces for each VLAN and how Router-on-a-Stick solves this
problem by carrying multiple VLANs over a single physical router
interface.

<br/>

