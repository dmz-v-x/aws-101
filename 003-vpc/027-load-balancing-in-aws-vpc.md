## Load Balancing in AWS VPC — ALB, NLB, and Private/Internal Load Balancers

In distributed systems, multiple servers often handle requests for the same application. To distribute traffic efficiently and avoid overloading a single server, **load balancers** are used.

In AWS, load balancers operate inside a **VPC** and route traffic to targets such as:

```
EC2 instances
containers (ECS/EKS)
Lambda functions
IP addresses
```

AWS provides multiple load balancer types, the most commonly used being:

```
Application Load Balancer (ALB)
Network Load Balancer (NLB)
```

These can be deployed as **public-facing load balancers** or **internal/private load balancers** depending on the architecture.

---

### 1. What Is Load Balancing?

Load balancing distributes incoming requests across multiple backend targets.

Example:

```
Client Request
      |
Load Balancer
  /        \
Server A  Server B
```

Benefits:

```
improved scalability
better availability
fault tolerance
traffic distribution
```

---

### 2. AWS Elastic Load Balancing (ELB)

AWS provides a managed service called **Elastic Load Balancing (ELB)**.

Main types:

```
Application Load Balancer (ALB)
Network Load Balancer (NLB)
Gateway Load Balancer (GWLB)
```

Most application architectures use **ALB or NLB**.

---

### 3. Application Load Balancer (ALB)

ALB operates at **Layer 7 (Application Layer)**.

Supported protocols:

```
HTTP
HTTPS
WebSockets
gRPC
```

ALB can inspect request content such as:

```
URL paths
host headers
query parameters
cookies
```

This enables advanced routing.

Example:

```
example.com/api → API service
example.com/images → Image service
```

Architecture:

```
Internet
   |
Application Load Balancer
   |
-------------------------
|                       |
App Server A        App Server B
```

---

### 4. ALB Routing Features

ALB supports advanced routing rules.

Examples:

#### Path-based routing

```
/api → API service
/images → image service
```

#### Host-based routing

```
api.example.com → API service
app.example.com → Web app
```

#### Weighted routing

Traffic split:

```
90% → version A
10% → version B
```

Useful for **canary deployments**.

---

### 5. Network Load Balancer (NLB)

NLB operates at **Layer 4 (Transport Layer)**.

Supported protocols:

```
TCP
UDP
TLS
```

NLB focuses on **high performance and low latency**.

Benefits:

```
millions of requests per second
extremely low latency
static IP addresses
high throughput
```

Architecture:

```
Client
   |
Network Load Balancer
   |
------------------------
|                      |
Backend A           Backend B
```

---

### 6. ALB vs NLB Comparison

| Feature | ALB | NLB |
|------|------|------|
| OSI Layer | Layer 7 | Layer 4 |
| Protocols | HTTP/HTTPS | TCP/UDP/TLS |
| Routing | Content-based | Connection-based |
| Latency | Moderate | Very low |
| Static IP | No | Yes |

Choose based on workload requirements.

---

### 7. Load Balancers Inside a VPC

Load balancers are deployed inside **multiple subnets in a VPC**.

Example architecture:

```
VPC
 |
--------------------------------
|                              |
Subnet AZ-a                Subnet AZ-b
     |                          |
 Load Balancer Nodes       Load Balancer Nodes
```

AWS automatically scales load balancer nodes.

This provides **multi-AZ fault tolerance**.

---

### 8. Public Load Balancers

Public load balancers are **internet-facing**.

Requirements:

```
public subnets
internet gateway
public DNS name
```

Architecture:

```
Internet
   |
Internet-Facing ALB
   |
Private Subnets
   |
Application Servers
```

The load balancer receives internet traffic.

---

### 9. Internal Load Balancers

Internal load balancers are **private**.

They are used for **internal service communication**.

Example architecture:

```
Internal Service
      |
Internal Load Balancer
      |
--------------------------
|                        |
Service A              Service B
```

Traffic stays inside the VPC.

No internet exposure.

---

### 10. Example Microservices Architecture

Example microservice platform:

```
Public ALB
   |
Frontend Service
   |
Internal ALB
   |
--------------------------------
|              |               |
Auth Service   Order Service   Payment Service
```

The internal load balancer routes traffic between services.

---

### 11. Security Groups and Load Balancers

Load balancers use **security groups**.

Example rules:

```
Inbound
Allow HTTP from internet

Outbound
Allow traffic to application servers
```

Application servers typically allow traffic **only from the load balancer security group**.

Example:

```
App SG
Allow port 80 from ALB SG
```

---

### 12. Target Groups

Load balancers route traffic to **target groups**.

Targets can include:

```
EC2 instances
IP addresses
Lambda functions
containers
```

Example:

```
ALB
 |
Target Group
 |
--------------------
|                  |
Instance A      Instance B
```

Health checks determine which targets receive traffic.

---

### 13. Health Checks

Load balancers continuously check backend health.

Example configuration:

```
Path: /health
Port: 80
Interval: 30 seconds
```

Healthy targets receive traffic.

Unhealthy targets are removed automatically.

---

### 14. Private Load Balancer Example

Example architecture:

```
VPC
 |
Private Subnet
 |
Internal ALB
 |
-----------------------------
|                           |
Service Instance A     Service Instance B
```

Used for:

```
microservices
internal APIs
service meshes
```

---

### 15. Multi-AZ Load Balancer Architecture

Example highly available design:

```
Internet
   |
Application Load Balancer
  /                      \
AZ-a                   AZ-b
  |                       |
EC2 Instance         EC2 Instance
```

Benefits:

```
fault tolerance
traffic distribution
automatic failover
```

---

### 16. NLB for High Throughput Systems

Example high-performance architecture:

```
Clients
   |
Network Load Balancer
   |
--------------------------
|                        |
Game Server A         Game Server B
```

NLB is commonly used for:

```
gaming servers
real-time streaming
financial trading platforms
high-performance APIs
```

---

### 17. PrivateLink Integration

NLB is also used with **AWS PrivateLink**.

Example:

```
Consumer VPC
   |
Interface Endpoint
   |
PrivateLink
   |
NLB
   |
Service
```

This enables private service access across VPCs.

---

### 18. Mental Model

Think of load balancers like **traffic controllers at a busy intersection**.

```
Incoming cars → intersection → multiple roads
```

The controller directs traffic to avoid congestion.

Similarly:

```
Client requests → load balancer → backend servers
```

---

### 19. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Load Balancer | Distributes incoming traffic |
| ALB | Layer 7 HTTP/HTTPS load balancer |
| NLB | Layer 4 high-performance load balancer |
| Internal LB | Private load balancer inside VPC |
| Target Group | Backend resource group |

---

### 20. Checkpoint Questions

You should now be able to answer these.

#### What is the main function of a load balancer?

Distribute traffic across multiple backend servers.

---

#### Which load balancer supports HTTP routing?

```
Application Load Balancer
```

---

#### Which load balancer provides the lowest latency?

```
Network Load Balancer
```

---

#### What is an internal load balancer?

A load balancer that is accessible only inside the VPC.

---

#### What component stores backend targets?

```
Target Groups
```
