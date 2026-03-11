## AWS PrivateLink — Hosting Services Privately and Accessing Them via Interface Endpoints

Modern cloud architectures often require **secure service-to-service communication across VPCs, accounts, or organizations** without exposing services to the public internet.

AWS provides a feature called **PrivateLink** to enable this type of connectivity.

PrivateLink allows a service hosted in one VPC to be accessed privately from another VPC using **interface endpoints**, while keeping traffic entirely within the AWS network.

---

### 1. The Problem PrivateLink Solves

Suppose you host a service in one VPC.

Example:

```
VPC A
 └ Application Service
```

Another VPC needs access.

Example:

```
VPC B
 └ Application Client
```

Traditional options include:

```
VPC Peering
Transit Gateway
Public load balancer
```

Problems with these approaches:

```
exposes network ranges
requires routing configuration
can expose services to internet
not granular enough
```

PrivateLink solves this by allowing **private service exposure without network-level connectivity**.

---

### 2. What Is AWS PrivateLink?

Definition:

> AWS PrivateLink enables private connectivity between VPCs, AWS services, and third-party services using interface endpoints.

Key idea:

```
The service remains private
Consumers connect using private endpoints
Traffic stays inside AWS backbone
```

Example architecture:

```
Consumer VPC
     |
Interface Endpoint
     |
AWS PrivateLink
     |
Network Load Balancer
     |
Service VPC
```

---

### 3. PrivateLink Roles

PrivateLink involves two parties.

```
Service Provider
Service Consumer
```

---

#### Service Provider

The service provider hosts the application.

Example:

```
VPC A
 └ Internal API Service
```

The provider exposes the service through **PrivateLink**.

---

#### Service Consumer

The consumer accesses the service privately.

Example:

```
VPC B
 └ Application needing API
```

The consumer creates an **interface endpoint**.

---

### 4. Core Components of PrivateLink

PrivateLink architectures include several components.

```
Network Load Balancer (NLB)
Endpoint Service
Interface Endpoint
Elastic Network Interfaces
```

Each plays a role in enabling connectivity.

---

### 5. Network Load Balancer

The service provider places their service behind a **Network Load Balancer**.

Example architecture:

```
Service VPC
     |
Network Load Balancer
     |
Application Service
```

The NLB becomes the entry point for PrivateLink.

---

### 6. Endpoint Service

The service provider creates an **endpoint service**.

Definition:

> An endpoint service exposes the NLB to other VPCs using PrivateLink.

Example:

```
NLB → Endpoint Service
```

Permissions determine which accounts or VPCs can connect.

---

### 7. Interface Endpoint

The consumer creates an **interface endpoint**.

Definition:

> An interface endpoint is an ENI created inside the consumer VPC that connects to the PrivateLink service.

Architecture:

```
Consumer VPC
     |
Interface Endpoint (ENI)
     |
PrivateLink
     |
Service VPC
```

This ENI receives a private IP address.

---

### 8. Example Traffic Flow

Example request:

```
Application → Private Service
```

Traffic flow:

```
App Server
   |
Interface Endpoint
   |
AWS PrivateLink
   |
Network Load Balancer
   |
Service Instance
```

All traffic stays inside AWS.

---

### 9. DNS Resolution

PrivateLink supports **private DNS names**.

Example request:

```
api.internal.company
```

DNS resolves to:

```
Private IP of interface endpoint
```

This allows applications to connect transparently.

---

### 10. Cross-Account PrivateLink

PrivateLink is commonly used across **AWS accounts**.

Example architecture:

```
Account A
 └ Service VPC

Account B
 └ Consumer VPC
```

Connectivity:

```
Consumer VPC → Interface Endpoint → PrivateLink → Service
```

This enables **secure service sharing across organizations**.

---

### 11. SaaS Provider Architecture

PrivateLink is widely used by SaaS providers.

Example:

```
SaaS Provider VPC
     |
Network Load Balancer
     |
Endpoint Service
```

Customer architecture:

```
Customer VPC
     |
Interface Endpoint
     |
PrivateLink
     |
SaaS Service
```

The SaaS provider does not need to expose public endpoints.

---

### 12. Example Enterprise Architecture

Example enterprise setup:

```
Shared Services VPC
     |
PrivateLink Endpoint Service
     |
-------------------------------
|             |               |
App VPC     Dev VPC      Analytics VPC
```

Multiple VPCs consume shared services.

---

### 13. Security Advantages

PrivateLink improves security because:

```
no internet exposure
no VPC peering required
no CIDR sharing
fine-grained service access
```

Only the service is exposed, not the entire network.

---

### 14. PrivateLink vs VPC Peering

| Feature | PrivateLink | VPC Peering |
|------|------|------|
| Network connectivity | Service-level | Full network |
| CIDR exposure | No | Yes |
| Transitive routing | Not applicable | Not supported |
| Security | Higher isolation | Less granular |

PrivateLink provides **better service isolation**.

---

### 15. PrivateLink vs Transit Gateway

| Feature | PrivateLink | Transit Gateway |
|------|------|------|
| Connectivity | Service access | Network routing |
| Use case | APIs/services | Multi-VPC networks |
| Isolation | High | Medium |

Transit Gateway connects networks.

PrivateLink connects **services**.

---

### 16. Example Internal Service Architecture

Example microservice platform:

```
Auth Service VPC
     |
PrivateLink Endpoint Service
     |
------------------------------
|            |               |
Payments VPC Orders VPC  Users VPC
```

Each service accesses the authentication service privately.

---

### 17. Performance Characteristics

PrivateLink traffic uses the **AWS internal backbone network**.

Benefits:

```
low latency
high bandwidth
high reliability
```

This is ideal for internal APIs.

---

### 18. Cost Model

PrivateLink pricing includes:

```
hourly cost per endpoint
data processing charges
```

Providers pay for NLB.

Consumers pay for interface endpoints.

---

### 19. Mental Model

Think of PrivateLink like **a private service door between buildings**.

Without PrivateLink:

```
Building A → Public Street → Building B
```

With PrivateLink:

```
Building A → Private hallway → Building B
```

Only the specific service door is exposed.

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| AWS PrivateLink | Private service connectivity |
| Service Provider | Hosts the service |
| Service Consumer | Accesses the service |
| Network Load Balancer | Entry point for service |
| Interface Endpoint | Private endpoint inside consumer VPC |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is AWS PrivateLink?

A service that enables private connectivity to services across VPCs.

---

#### What load balancer is required for PrivateLink?

```
Network Load Balancer (NLB)
```

---

#### What does the consumer create?

```
Interface Endpoint
```

---

#### Does PrivateLink require VPC peering?

```
No
```

---

#### What is the biggest advantage of PrivateLink?

Secure service exposure without exposing entire networks.
