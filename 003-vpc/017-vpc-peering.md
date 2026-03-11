## VPC Peering — Single-Region and Cross-Region Connectivity (and the No Transitive Routing Limitation)

As cloud architectures grow, organizations often deploy **multiple VPCs** instead of placing everything inside one large VPC.

Reasons include:

```
environment separation
security isolation
multi-team architectures
microservice architectures
multi-region deployments
```

However, once multiple VPCs exist, systems inside them must sometimes **communicate with each other**.

AWS provides a feature called **VPC Peering** to enable this communication.

---

### 1. Why Multiple VPCs Exist

In simple systems, one VPC might be enough.

Example:

```
VPC
 ├ Web Servers
 ├ Application Servers
 └ Databases
```

But larger systems often separate infrastructure.

Example architecture:

```
VPC-A → Production
VPC-B → Development
VPC-C → Shared Services
```

Benefits include:

```
better security isolation
team autonomy
network segmentation
reduced blast radius
```

However, sometimes these VPCs must communicate.

Example:

```
Production App → Shared Authentication Service
```

This is where **VPC Peering** is used.

---

### 2. What Is VPC Peering?

Definition:

> VPC Peering is a networking connection that enables two VPCs to communicate privately using their private IP addresses.

Key characteristics:

```
private communication
no internet required
low latency
high bandwidth
```

Traffic flows directly across the AWS network.

---

### 3. Basic VPC Peering Architecture

Example architecture:

```
VPC A
10.0.0.0/16
   |
VPC Peering Connection
   |
VPC B
10.1.0.0/16
```

Instances inside VPC A can reach instances inside VPC B.

Example communication:

```
10.0.1.10 → 10.1.2.20
```

---

### 4. How VPC Peering Is Created

Creating a VPC peering connection involves three steps.

Step 1:

```
Create peering request
```

Step 2:

```
Accept peering request
```

Step 3:

```
Update route tables
```

Both VPCs must add routes pointing to the peering connection.

---

### 5. Route Table Configuration

Example:

```
VPC A CIDR → 10.0.0.0/16
VPC B CIDR → 10.1.0.0/16
```

Routes required:

VPC A route table:

```
Destination → 10.1.0.0/16
Target → Peering Connection
```

VPC B route table:

```
Destination → 10.0.0.0/16
Target → Peering Connection
```

This enables bidirectional communication.

---

### 6. Security Groups and NACLs

Even after peering is established, security rules still apply.

Communication must pass through:

```
Security Groups
Network ACLs
```

Example:

```
Allow TCP 443 from 10.0.0.0/16
```

Without proper rules, traffic will still be blocked.

---

### 7. Single-Region VPC Peering

Single-region peering connects VPCs within the **same AWS region**.

Example:

```
Region: ap-south-1
```

Architecture:

```
VPC A (10.0.0.0/16)
   |
Peering Connection
   |
VPC B (10.1.0.0/16)
```

Advantages:

```
low latency
high bandwidth
simple routing
```

---

### 8. Cross-Region VPC Peering

AWS also allows peering across regions.

Example:

```
Region 1 → ap-south-1
Region 2 → eu-west-1
```

Architecture:

```
VPC A (Mumbai)
   |
Inter-Region Peering
   |
VPC B (Ireland)
```

Traffic flows across the AWS global network backbone.

Benefits:

```
secure cross-region communication
no internet exposure
low latency global connectivity
```

---

### 9. Cross-Region Routing Example

Example CIDRs:

```
VPC A → 10.0.0.0/16
VPC B → 172.16.0.0/16
```

Routes:

```
10.0.0.0/16 → peering connection
172.16.0.0/16 → peering connection
```

Once configured, instances communicate using private IPs.

---

### 10. DNS Resolution Across Peering

AWS supports **private DNS resolution across peered VPCs**.

Example:

```
service.internal
```

Instance in VPC A can resolve a private hostname inside VPC B.

This simplifies service communication.

---

### 11. Important Limitation — No Transitive Routing

The most important limitation of VPC peering is:

```
No transitive routing
```

Definition:

> A VPC cannot route traffic to another VPC through an intermediate peered VPC.

Example architecture:

```
VPC A
   |
Peering
   |
VPC B
   |
Peering
   |
VPC C
```

Traffic **cannot flow**:

```
VPC A → VPC B → VPC C
```

A cannot reach C.

---

### 12. Why Transitive Routing Is Not Allowed

AWS prevents transitive routing to maintain:

```
network isolation
security boundaries
simpler routing
```

Allowing transitive routing could create complex routing loops.

---

### 13. Correct Way to Connect Three VPCs

To connect three VPCs, each pair needs its own peering connection.

Example:

```
VPC A ↔ VPC B
VPC B ↔ VPC C
VPC A ↔ VPC C
```

This creates a **full mesh network**.

---

### 14. Full Mesh Architecture

Example:

```
         VPC A
        /     \
       /       \
     VPC B --- VPC C
```

Each connection is a separate peering link.

---

### 15. Scaling Problem with Peering

For many VPCs, full mesh peering becomes complex.

Example:

```
10 VPCs
```

Required connections:

```
45 peering connections
```

Routing becomes difficult.

This is why AWS introduced **Transit Gateway**.

---

### 16. Example Production Architecture

Example multi-VPC architecture:

```
Shared Services VPC
      |
Peering
      |
Application VPC
      |
Peering
      |
Analytics VPC
```

Each connection allows private communication.

---

### 17. Peering Restrictions

Important limitations include:

```
no transitive routing
no overlapping CIDR ranges
no edge-to-edge routing
```

Overlapping CIDRs example:

```
VPC A → 10.0.0.0/16
VPC B → 10.0.0.0/16
```

These cannot be peered.

---

### 18. Traffic Flow Example

Example communication:

```
App Server → Database
```

Flow:

```
EC2 (VPC A)
   |
Route Table
   |
Peering Connection
   |
VPC Router
   |
Database (VPC B)
```

Traffic stays inside AWS infrastructure.

---

### 19. Mental Model

Think of VPC peering like **private tunnels between networks**.

```
City A → Tunnel → City B
```

But tunnels do not connect automatically to other tunnels.

Each connection must be built individually.

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| VPC Peering | Private connection between VPCs |
| Single-Region Peering | Peering inside same region |
| Cross-Region Peering | Peering across regions |
| Private Communication | Uses private IP addresses |
| No Transitive Routing | Traffic cannot pass through intermediate VPC |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is VPC Peering?

A private network connection between two VPCs.

---

#### Does VPC peering use the public internet?

```
No
Traffic stays within AWS infrastructure.
```

---

#### Can VPC A communicate with VPC C through VPC B?

```
No
Transitive routing is not allowed.
```

---

#### What is the requirement for VPC peering?

```
CIDR blocks must not overlap.
```

---

#### When does cross-region peering occur?

When VPCs exist in different AWS regions.

