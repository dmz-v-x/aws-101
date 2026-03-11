## Transit VPC Patterns and Multi-Account Networking — Using AWS RAM for Network Sharing

As cloud environments grow, organizations rarely operate in a **single AWS account** or a **single VPC**. Large systems often use **multiple AWS accounts**, each containing its own VPCs and workloads.

Reasons include:

```
security isolation
team ownership
environment separation (prod/dev/test)
billing separation
compliance requirements
```

However, all these VPCs often need **shared network services** such as:

```
internet access
inspection firewalls
NAT gateways
centralized logging
shared APIs
```

To solve this, AWS architectures often implement **Transit VPC patterns** and **multi-account networking using AWS Resource Access Manager (RAM)**.

---

### 1. The Multi-Account AWS Environment

Large companies rarely place all workloads in one account.

Example organization structure:

```
AWS Organization
   |
-----------------------------------------
|           |            |               |
Prod       Dev        Analytics      Security
Account    Account      Account       Account
```

Each account can have one or more VPCs.

Example:

```
Prod Account
   └ Prod VPC

Dev Account
   └ Dev VPC

Analytics Account
   └ Data VPC
```

The challenge becomes:

```
How do these VPCs communicate securely?
```

---

### 2. Traditional Approach: VPC Peering

One option is **VPC Peering**.

Example:

```
Prod VPC ↔ Dev VPC
Prod VPC ↔ Analytics VPC
Dev VPC ↔ Analytics VPC
```

This creates a **full mesh network**.

Problem:

```
scales poorly
complex routing
difficult security management
```

For many VPCs, this becomes unmanageable.

---

### 3. What Is a Transit VPC?

A **Transit VPC** is a dedicated VPC used as a **central network hub**.

Definition:

> A Transit VPC is a central VPC that routes traffic between multiple VPCs and networks.

Architecture:

```
              Transit VPC
             (Network Hub)
            /       |       \
          VPC A    VPC B    VPC C
```

This architecture follows the **hub-and-spoke model**.

---

### 4. Hub-and-Spoke Architecture

In this model:

```
Hub → Central networking VPC
Spokes → Application VPCs
```

Example:

```
                Transit VPC
                 (Hub)
              /     |     \
         Prod VPC Dev VPC Data VPC
```

Traffic between spokes goes through the hub.

Example flow:

```
Dev VPC → Transit VPC → Prod VPC
```

---

### 5. Why Transit VPCs Exist

Transit VPC patterns allow organizations to centralize networking services.

Examples of shared services:

```
NAT gateways
firewalls
IDS/IPS inspection
VPN connections
Direct Connect connectivity
logging systems
```

Instead of duplicating these in every VPC, they are centralized.

Example:

```
Transit VPC
   |
Firewall
   |
Internet Gateway
```

All VPC traffic flows through the inspection layer.

---

### 6. Transit VPC vs Transit Gateway

Originally, Transit VPC architectures used:

```
VPN tunnels
routing appliances
network virtual appliances
```

This was complex.

AWS introduced **Transit Gateway** to simplify this architecture.

Modern architectures use:

```
Transit Gateway instead of Transit VPC routers
```

But the **Transit VPC pattern concept** remains important.

---

### 7. Centralized Internet Egress

Example centralized architecture:

```
Application VPCs
      |
Transit Gateway
      |
Egress VPC
      |
NAT Gateway
      |
Internet
```

Benefits:

```
centralized security
simplified monitoring
reduced NAT duplication
```

---

### 8. Centralized Security Inspection

Transit VPC patterns often include **network inspection layers**.

Example:

```
App VPC
   |
Transit Gateway
   |
Inspection VPC
   |
Firewall
   |
Internet
```

All outbound traffic passes through security inspection.

---

### 9. Multi-Account Networking Challenge

When using multiple AWS accounts, resources normally cannot be accessed across accounts.

Example:

```
Transit Gateway in Network Account
Application VPC in Application Account
```

To connect them, AWS provides **Resource Access Manager (RAM)**.

---

### 10. What Is AWS Resource Access Manager (RAM)?

AWS RAM allows sharing AWS resources across accounts.

Definition:

> AWS RAM allows resources to be securely shared with other AWS accounts or organizations.

Resources that can be shared include:

```
Transit Gateway
Subnets
Route 53 Resolver rules
License configurations
```

---

### 11. Example RAM Sharing Architecture

Example architecture:

```
Network Account
   |
Transit Gateway
   |
Shared via AWS RAM
   |
Application Accounts
```

Other accounts attach their VPCs to the shared TGW.

Example:

```
Prod Account VPC → TGW
Dev Account VPC → TGW
Analytics Account VPC → TGW
```

---

### 12. Example Multi-Account Network

Example organization network:

```
               Transit Gateway
                    |
------------------------------------------------
|              |             |                  |
Prod VPC      Dev VPC     Data VPC       Security VPC
(Account A)  (Account B) (Account C)    (Account D)
```

All VPCs communicate through the TGW.

---

### 13. Resource Sharing Workflow

Typical workflow using RAM:

Step 1:

```
Create Transit Gateway in Network Account
```

Step 2:

```
Share TGW using AWS RAM
```

Step 3:

```
Accept share in other accounts
```

Step 4:

```
Attach VPCs to shared TGW
```

Now all VPCs can connect.

---

### 14. Route Control with TGW

Transit Gateway route tables can control connectivity.

Example policy:

```
Prod ↔ Shared Services allowed
Dev ↔ Shared Services allowed
Prod ↔ Dev blocked
```

Architecture:

```
Prod VPC
   |
TGW Route Table
   |
Shared Services VPC
```

Segmentation improves security.

---

### 15. Shared Services VPC Pattern

Many organizations use a **Shared Services VPC**.

Example services hosted:

```
Active Directory
logging infrastructure
internal APIs
authentication systems
monitoring tools
```

Architecture:

```
Shared Services VPC
       |
Transit Gateway
       |
--------------------------------
|              |                |
Prod VPC     Dev VPC        Data VPC
```

All VPCs access shared infrastructure.

---

### 16. Enterprise AWS Networking Architecture

Typical enterprise network structure:

```
Network Account
   |
Transit Gateway
   |
-----------------------------------------
|             |             |            |
Prod Account  Dev Account  Data Account Security Account
```

Each account maintains its own resources.

Networking is centralized.

---

### 17. Security Advantages

Transit VPC and multi-account networking provide:

```
centralized network control
improved segmentation
better compliance
reduced attack surface
simplified routing
```

This is the recommended architecture for large environments.

---

### 18. Example Traffic Flow

Example:

```
Dev Application → Shared Authentication Service
```

Flow:

```
Dev VPC
   |
Transit Gateway
   |
Shared Services VPC
   |
Auth Service
```

Traffic stays private.

---

### 19. Mental Model

Think of a **Transit VPC architecture like a central highway interchange**.

```
Cities → Highway Hub → Other Cities
```

Instead of building roads between every city, they all connect to the central interchange.

In AWS:

```
VPCs → Transit Gateway → VPCs
```

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Transit VPC | Central VPC routing traffic |
| Hub-and-Spoke | Central hub with connected VPCs |
| AWS RAM | Resource sharing across accounts |
| Multi-Account Architecture | Separate accounts for workloads |
| Shared Services VPC | Central services used by multiple VPCs |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is a Transit VPC?

A central VPC used to route traffic between multiple networks.

---

#### What architecture pattern does it follow?

```
Hub-and-spoke
```

---

#### What does AWS RAM do?

Allows AWS resources to be shared across accounts.

---

#### Why use multi-account networking?

For isolation, security, and organizational structure.

---

#### What commonly sits in a shared services VPC?

Authentication systems, monitoring tools, logging, and internal APIs.
