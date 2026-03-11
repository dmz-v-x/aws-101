## Scaling NAT and Internet Gateway (IGW) — Best Practices and NAT Cost/Performance Tradeoffs

In AWS VPC networking, many workloads need **internet connectivity** while still keeping resources in **private subnets**.

Typical examples include:

```
installing OS packages
pulling container images
accessing external APIs
software updates
```

Private subnets cannot access the internet directly, so AWS uses **NAT (Network Address Translation)** to enable outbound connectivity.

At the same time, public subnets use **Internet Gateways (IGW)** to reach the internet.

---

### 1. Quick Recap — IGW vs NAT

Two components provide internet connectivity in AWS:

```
Internet Gateway (IGW)
NAT Gateway
```

#### Internet Gateway

Used for **public subnets**.

Example:

```
EC2 (Public Subnet)
    |
Route Table
    |
Internet Gateway
    |
Internet
```

Instances require a **public IP**.

---

#### NAT Gateway

Used for **private subnets**.

Example:

```
EC2 (Private Subnet)
    |
Route Table
    |
NAT Gateway
    |
Internet Gateway
    |
Internet
```

Instances remain private but can initiate outbound connections.

---

### 2. Typical Internet Access Architecture

Common production design:

```
Internet
   |
Internet Gateway
   |
Public Subnet
   |
NAT Gateway
   |
Private Subnet
   |
Application Servers
```

Traffic flow:

```
App Server → NAT → IGW → Internet
```

Inbound traffic is blocked.

---

### 3. NAT Gateway Performance Characteristics

NAT Gateway is a **managed AWS service**.

Key characteristics:

```
automatic scaling
high availability within AZ
up to ~100 Gbps throughput
millions of connections
```

Unlike NAT instances, NAT Gateway scales automatically.

However, it is **AZ-scoped**.

---

### 4. NAT Gateway Is AZ-Specific

A NAT Gateway belongs to a single AZ.

Example:

```
AZ-a
  NAT Gateway
```

If private subnets in **AZ-b** route to this NAT, traffic crosses AZ boundaries.

Example:

```
Private Subnet (AZ-b)
   |
Cross-AZ Traffic
   |
NAT (AZ-a)
```

This introduces:

```
higher latency
cross-AZ data charges
reduced resilience
```

---

### 5. Best Practice — One NAT Gateway Per AZ

Recommended architecture:

```
AZ-a
  Public Subnet
  NAT Gateway
  Private Subnet

AZ-b
  Public Subnet
  NAT Gateway
  Private Subnet
```

Routing:

```
Private Subnet AZ-a → NAT AZ-a
Private Subnet AZ-b → NAT AZ-b
```

Benefits:

```
high availability
no cross-AZ data transfer
better performance
```

---

### 6. Internet Gateway Scaling

Internet Gateways behave differently from NAT.

Key properties:

```
fully managed
automatically scalable
regional resource
no bandwidth limits exposed to users
```

Unlike NAT:

```
only one IGW per VPC
```

Example architecture:

```
VPC
 |
Internet Gateway
 |
Multiple Public Subnets
```

The IGW automatically scales to support traffic.

---

### 7. NAT Gateway Cost Model

NAT Gateway pricing has two components:

```
Hourly charge
Data processing charge
```

Typical pricing model:

```
Hourly NAT cost
+
Cost per GB processed
```

Example scenario:

```
Large microservice cluster
Thousands of API calls
Large container downloads
```

Costs can grow significantly.

---

### 8. Example NAT Cost Scenario

Example workload:

```
EKS cluster
Nodes in private subnet
Pulling images from Docker registry
```

Traffic path:

```
EKS Node
   |
NAT Gateway
   |
Internet
```

Every GB downloaded through NAT adds cost.

For high-throughput workloads, this becomes expensive.

---

### 9. Reducing NAT Costs With VPC Endpoints

A common optimization is using **VPC endpoints**.

Example services:

```
S3
DynamoDB
ECR
CloudWatch
SSM
```

Without endpoints:

```
EC2 → NAT → Internet → S3
```

With gateway endpoint:

```
EC2 → VPC Endpoint → S3
```

Benefits:

```
lower NAT traffic
reduced cost
private connectivity
```

---

### 10. NAT Traffic Optimization

Best practice architecture:

```
Private Subnet
     |
Application
     |
-----------------------------
|        |        |         |
S3 VPCE  ECR VPCE  SSM VPCE
     |
Remaining Traffic
     |
NAT Gateway
```

Most internal AWS traffic bypasses NAT.

---

### 11. High Throughput Workloads

Large-scale workloads may generate massive outbound traffic.

Examples:

```
data pipelines
container platforms
ML training clusters
large downloads
```

Scaling strategy:

```
multiple NAT gateways
load balancing traffic across AZs
use VPC endpoints wherever possible
```

---

### 12. NAT Instance Alternative

Before NAT Gateway existed, AWS used **NAT Instances**.

Example:

```
EC2 NAT Instance
```

Advantages:

```
lower cost
custom configuration
```

Disadvantages:

```
manual scaling
limited bandwidth
maintenance required
single point of failure
```

Today NAT Gateway is generally preferred.

---

### 13. When NAT Instances May Still Be Used

Rare scenarios include:

```
cost-sensitive environments
custom firewall rules
very small workloads
learning environments
```

But production systems typically avoid NAT instances.

---

### 14. Example Highly Available NAT Architecture

Example multi-AZ deployment:

```
                    Internet
                       |
                 Internet Gateway
                       |
-------------------------------------------
|                     |                   |
AZ-a                 AZ-b                AZ-c
Public Subnet       Public Subnet       Public Subnet
   |                    |                   |
NAT Gateway          NAT Gateway         NAT Gateway
   |                    |                   |
Private Subnet       Private Subnet      Private Subnet
```

Each AZ routes through its local NAT.

---

### 15. NAT Monitoring and Observability

To monitor NAT usage:

```
CloudWatch metrics
VPC Flow Logs
NAT Gateway metrics
```

Useful metrics:

```
BytesOutToDestination
BytesInFromDestination
ActiveConnectionCount
ErrorPortAllocation
```

These help detect scaling issues.

---

### 16. NAT Failure Scenarios

Possible issues include:

```
NAT gateway in wrong subnet
route tables misconfigured
single NAT used for multiple AZs
security group blocking traffic
```

Troubleshooting checklist:

```
Check route tables
Check NAT subnet
Check IGW attachment
Check VPC endpoints
Check DNS resolution
```

---

### 17. Example Production Networking Architecture

Example scalable VPC:

```
                 Internet
                    |
              Internet Gateway
                    |
-------------------------------------
|                |                  |
AZ-a            AZ-b               AZ-c
Public Subnet  Public Subnet      Public Subnet
   |               |                  |
 NAT Gateway     NAT Gateway        NAT Gateway
   |               |                  |
Private Subnet  Private Subnet     Private Subnet
   |               |                  |
 Application      Application       Application
```

This architecture supports high throughput workloads.

---

### 18. Mental Model

Think of NAT gateways like **secure exit gates from a private network**.

```
Private Servers → NAT Exit Gate → Internet
```

Multiple exits improve:

```
capacity
availability
performance
```

---

### 19. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Internet Gateway | Public internet access for VPC |
| NAT Gateway | Outbound internet access for private subnets |
| NAT AZ Scope | NAT belongs to a single AZ |
| Best Practice | One NAT per AZ |
| Optimization | Use VPC endpoints to reduce NAT traffic |

---

### 20. Checkpoint Questions

You should now be able to answer these.

#### What does a NAT Gateway do?

Allows instances in private subnets to access the internet.

---

#### Should one NAT Gateway serve multiple AZs?

```
No
Best practice is one NAT per AZ.
```

---

#### Why are VPC endpoints used?

```
To reduce NAT traffic and keep traffic inside AWS.
```

---

#### How many Internet Gateways can a VPC have?

```
One
```

---

#### What is the main NAT cost factor?

```
Data processing charges.
```
