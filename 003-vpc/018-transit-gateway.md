## AWS Transit Gateway — Hub-and-Spoke Architecture, TGW Route Tables, and Scaling Advantages

As organizations grow in the cloud, they often create **many VPCs** instead of one large network.

Examples:

```
Production VPC
Development VPC
Analytics VPC
Security VPC
Shared Services VPC
```

If these VPCs need to communicate using **VPC Peering**, the number of connections quickly becomes difficult to manage.

Example:

```
10 VPCs → 45 peering connections
```

This creates complex routing and operational challenges.

To solve this problem, AWS introduced **Transit Gateway (TGW)**.

Transit Gateway allows organizations to build **large-scale network architectures** using a **hub-and-spoke model**.

---

### 1. What Is AWS Transit Gateway?

A **Transit Gateway** is a centralized router that connects multiple networks.

Definition:

> AWS Transit Gateway is a regional network transit hub that enables VPCs, VPNs, and on-premise networks to connect through a single gateway.

It acts like a **central router for cloud networks**.

Example:

```
Multiple VPCs → Transit Gateway → Central routing
```

Instead of creating many peer connections, all networks connect to one hub.

---

### 2. The Problem With VPC Peering

Suppose we have four VPCs:

```
VPC A
VPC B
VPC C
VPC D
```

With peering, connections look like this:

```
A ↔ B
A ↔ C
A ↔ D
B ↔ C
B ↔ D
C ↔ D
```

This is called **full mesh networking**.

Problems include:

```
many connections
complex routing
difficult management
scaling limitations
```

---

### 3. Transit Gateway Solution

Transit Gateway simplifies networking.

Architecture:

```
          VPC A
            |
            |
VPC B ---- TGW ---- VPC C
            |
            |
          VPC D
```

Each VPC connects to the **Transit Gateway hub**.

This architecture is called **hub-and-spoke networking**.

---

### 4. Hub-and-Spoke Architecture

In hub-and-spoke architecture:

```
Hub → Transit Gateway
Spokes → VPCs
```

Example:

```
           VPC A
             |
             |
VPC B --- Transit Gateway --- VPC C
             |
             |
           VPC D
```

Benefits:

```
simpler routing
centralized connectivity
better scalability
```

---

### 5. Transit Gateway Attachments

To connect a network to a TGW, an **attachment** is created.

Types of attachments:

```
VPC attachment
VPN attachment
Direct Connect attachment
Peering attachment (TGW to TGW)
```

Example:

```
VPC A → TGW attachment
```

Each attached VPC becomes part of the Transit Gateway network.

---

### 6. VPC Attachment Architecture

Example architecture:

```
VPC A
   |
Subnet (TGW attachment subnet)
   |
Transit Gateway
   |
Subnet
   |
VPC B
```

The TGW connects to a subnet inside each VPC.

Traffic flows through the attachment.

---

### 7. Transit Gateway Route Tables

Transit Gateway has its own **route tables**.

These determine where traffic should go.

Example TGW route table:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | VPC-A |
| 10.1.0.0/16 | VPC-B |
| 10.2.0.0/16 | VPC-C |

When traffic arrives, TGW checks the route table.

---

### 8. Route Propagation

TGW can automatically learn routes from attached VPCs.

This is called **route propagation**.

Example:

```
VPC A CIDR → 10.0.0.0/16
```

TGW automatically adds route:

```
10.0.0.0/16 → VPC A attachment
```

This reduces manual routing work.

---

### 9. Example Traffic Flow

Suppose an instance in VPC A wants to talk to an instance in VPC B.

Example:

```
EC2-A → EC2-B
```

Traffic flow:

```
EC2-A
   |
VPC Router
   |
Transit Gateway Attachment
   |
Transit Gateway
   |
Transit Gateway Route Table
   |
VPC-B Attachment
   |
VPC Router
   |
EC2-B
```

---

### 10. Transit Gateway vs VPC Peering

| Feature | VPC Peering | Transit Gateway |
|------|------|------|
| Architecture | Mesh | Hub-and-spoke |
| Routing | Manual | Centralized |
| Transitive routing | Not allowed | Supported |
| Scaling | Limited | Large scale |

Transit Gateway supports **transitive routing**, unlike peering.

---

### 11. Transitive Routing Example

With Transit Gateway:

```
VPC A → TGW → VPC B → TGW → VPC C
```

Communication works.

Example:

```
VPC A ↔ VPC C
```

No direct connection required.

---

### 12. Multi-Account Networking

Transit Gateway is often used in **multi-account AWS environments**.

Example:

```
Account A → VPC A
Account B → VPC B
Account C → VPC C
```

All connect to a shared TGW.

Architecture:

```
          Shared TGW
        /     |      \
     VPC A  VPC B  VPC C
```

This simplifies network architecture across accounts.

---

### 13. Hybrid Connectivity

Transit Gateway can also connect to on-premise networks.

Example architecture:

```
On-Prem Data Center
        |
     VPN
        |
Transit Gateway
   /    |    \
VPC A  VPC B  VPC C
```

This allows centralized connectivity.

---

### 14. TGW Route Table Segmentation

Transit Gateway can have **multiple route tables**.

This allows network segmentation.

Example:

```
Production Route Table
Development Route Table
Shared Services Route Table
```

Different VPCs can be associated with different route tables.

This controls traffic flow.

---

### 15. Example Segmented Architecture

Example:

```
Production VPC
Development VPC
Shared Services VPC
```

Routing policy:

```
Prod ↔ Shared allowed
Dev ↔ Shared allowed
Prod ↔ Dev blocked
```

Transit Gateway route tables enforce these rules.

---

### 16. Scaling Advantages

Transit Gateway is designed for **large networks**.

Capabilities include:

```
thousands of VPC attachments
high bandwidth
centralized routing
reduced operational complexity
```

Compared to peering:

```
far easier to manage large environments
```

---

### 17. Enterprise Network Architecture

Example enterprise architecture:

```
Internet
   |
Edge VPC
   |
Transit Gateway
   |
-----------------------------------
|            |            |       |
Prod VPC   Dev VPC   Security VPC Shared VPC
```

All communication flows through TGW.

---

### 18. Global Architecture Example

Transit Gateways can connect across regions.

Example:

```
Region A TGW ↔ Region B TGW
```

Architecture:

```
VPCs (Region A) → TGW A → TGW B → VPCs (Region B)
```

This enables global network connectivity.

---

### 19. Mental Model

Think of Transit Gateway as a **central airport hub**.

```
Cities → Flights → Hub Airport → Flights → Cities
```

Instead of direct flights between every city, everything goes through the hub.

Similarly:

```
VPCs → TGW → VPCs
```

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Transit Gateway | Central network router |
| Hub-and-Spoke | Central hub with connected networks |
| Attachment | Connection between TGW and VPC |
| TGW Route Table | Controls traffic routing |
| Route Propagation | Automatic route updates |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is AWS Transit Gateway?

A centralized router connecting multiple VPCs and networks.

---

#### What architecture does TGW use?

```
Hub-and-spoke
```

---

#### Does TGW support transitive routing?

```
Yes
```

---

#### What connects VPCs to a Transit Gateway?

```
TGW attachment
```

---

#### Why is TGW better for large networks?

Because it simplifies routing and scales better than VPC peering.
