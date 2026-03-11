## NAT in AWS — NAT Gateway vs NAT Instance

When designing secure cloud networks, many systems **should not be exposed directly to the internet**. Examples include:

- application servers
- internal services
- background workers
- databases

However, these systems sometimes still need **outbound internet access** for things like:

- downloading software updates
- accessing external APIs
- pulling container images
- installing packages
- contacting license servers

AWS solves this problem using **NAT (Network Address Translation)**.

---

### 1. The Problem NAT Solves

Consider a typical secure architecture:

```
Internet
   |
Public Subnet
   |
Private Subnet
   |
Application Server
```

The application server is in a **private subnet**.

Private subnets:

```
no direct route to Internet Gateway
```

This means the instance **cannot reach the internet**.

Example problem:

```
yum update
apt update
docker pull
npm install
```

All of these require internet access.

But we **do not want the server to be reachable from the internet**.

This is exactly what NAT solves.

---

### 2. What Is NAT?

NAT stands for:

```
Network Address Translation
```

Definition:

> NAT allows instances with private IP addresses to access the internet by translating their private IP into a public IP.

The key rule:

```
Outbound internet access allowed
Inbound internet access blocked
```

This protects internal systems.

---

### 3. NAT Traffic Flow Example

Example private instance:

```
Private IP → 10.0.2.15
```

Step-by-step flow:

```
Private EC2 → NAT → Internet Gateway → Internet
```

When the server sends a request:

```
10.0.2.15 → google.com
```

NAT replaces the private IP with a **public IP**.

Example:

```
54.12.45.19 → google.com
```

The response returns to the NAT device, which forwards it back to the instance.

This process is **Network Address Translation**.

---

### 4. NAT in AWS

AWS provides two NAT options:

```
NAT Gateway
NAT Instance
```

Both provide the same core function:

```
Allow private subnets to access the internet
```

But they differ in **architecture, management, scalability, and cost**.

---

### 5. NAT Gateway

A **NAT Gateway** is a fully managed NAT service provided by AWS.

Definition:

> NAT Gateway allows instances in private subnets to connect to the internet or other AWS services while preventing the internet from initiating connections.

Characteristics:

```
managed by AWS
automatically scalable
high availability within AZ
no OS maintenance
high bandwidth
```

Architecture example:

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
Application Server
```

---

### 6. NAT Gateway Traffic Flow

Example request:

```
Private EC2 (10.0.2.15)
     |
     v
NAT Gateway
     |
     v
Internet Gateway
     |
     v
Internet
```

Response:

```
Internet
     |
     v
IGW
     |
     v
NAT Gateway
     |
     v
Private EC2
```

The instance remains **invisible to the internet**.

---

### 7. NAT Gateway Requirements

To use NAT Gateway you must:

1. Place NAT Gateway in a **public subnet**

2. Attach **Elastic IP**

3. Configure private subnet route table:

```
0.0.0.0/0 → NAT Gateway
```

Example route table:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | NAT Gateway |

---

### 8. NAT Instance

A **NAT Instance** is simply an **EC2 instance configured to perform NAT**.

Before NAT Gateway existed, NAT instances were the standard solution.

Definition:

> A NAT Instance is an EC2 instance configured to forward traffic from private subnets to the internet.

Architecture example:

```
Internet
   |
Internet Gateway
   |
Public Subnet
   |
NAT Instance (EC2)
   |
Private Subnet
   |
Application Server
```

---

### 9. NAT Instance Traffic Flow

Flow is identical to NAT Gateway.

```
Private EC2
   |
NAT Instance
   |
Internet Gateway
   |
Internet
```

However, the NAT device is **a normal EC2 server**.

---

### 10. NAT Instance Setup Requirements

To create NAT Instance you must:

1. Launch EC2 in public subnet
2. Disable source/destination check
3. Configure routing
4. Attach Elastic IP
5. Configure OS forwarding rules

Example Linux configuration:

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

iptables rule example:

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

---

### 11. NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|------|------|------|
| Managed by AWS | Yes | No |
| Scaling | Automatic | Manual |
| Availability | High | Depends on EC2 |
| Maintenance | None | You maintain OS |
| Bandwidth | Very high | Limited by instance |
| Security patches | AWS handles | You handle |
| Cost | Higher | Potentially cheaper |

---

### 12. Performance Differences

NAT Gateway:

```
scales automatically
up to 100 Gbps
highly reliable
```

NAT Instance:

```
limited by EC2 size
manual scaling
single point of failure
```

Example:

```
t3.micro NAT instance → limited throughput
```

---

### 13. Availability Differences

NAT Gateway:

```
built-in high availability
managed service
```

NAT Instance:

```
single EC2
can fail
requires failover design
```

Production environments prefer **NAT Gateway**.

---

### 14. Cost Considerations

NAT Gateway pricing:

```
hourly charge
+ data processing charge
```

NAT Instance pricing:

```
only EC2 cost
```

For **very small workloads**, NAT Instance may be cheaper.

For production systems, NAT Gateway is usually preferred.

---

### 15. Typical Production Architecture

Large systems deploy NAT Gateway per AZ.

Example:

```
VPC (10.0.0.0/16)

AZ-a
 ├ Public Subnet
 │   └ NAT Gateway
 └ Private Subnet
     └ Application Servers

AZ-b
 ├ Public Subnet
 │   └ NAT Gateway
 └ Private Subnet
     └ Application Servers
```

This ensures **high availability**.

---

### 16. Example Route Tables

Public subnet route table:

```
10.0.0.0/16 → local
0.0.0.0/0 → IGW
```

Private subnet route table:

```
10.0.0.0/16 → local
0.0.0.0/0 → NAT Gateway
```

---

### 17. Example CIDR Layout

```
VPC → 10.0.0.0/16

Public Subnet
10.0.1.0/24
(NAT Gateway here)

Private Subnet
10.0.2.0/24
(Application servers)

Isolated Subnet
10.0.3.0/24
(Database)
```

---

### 18. When Should You Use NAT Gateway?

Use NAT Gateway when:

```
production environments
high traffic workloads
high availability required
minimal operational overhead desired
```

Most modern AWS architectures use **NAT Gateway**.

---

### 19. When Should You Use NAT Instance?

Use NAT Instance when:

```
cost-sensitive environments
learning labs
very small traffic
custom NAT configuration required
```

However it requires **manual maintenance**.

---

### 20. Mental Model

Think of NAT as a **secure internet proxy for private servers**.

```
Private Server → NAT → Internet
```

The outside world never sees the private servers.

---

### 21. Complete Architecture Example

```
                Internet
                   |
             Internet Gateway
                   |
            ------------------
            |                |
        Public Subnet     Public Subnet
         NAT Gateway       NAT Gateway
            |                |
        Private Subnet   Private Subnet
       Application EC2   Application EC2
            |                |
         Database        Database
```

This architecture is used in **many production systems**.

---

### 22. Checkpoint Questions

You should now be able to answer these.

#### What is NAT?

Network Address Translation allows private IPs to access the internet using a public IP.

---

#### Why do private subnets need NAT?

Because they do not have direct internet access.

---

#### What does NAT Gateway provide?

Managed NAT service for private subnet internet access.

---

#### What is a NAT Instance?

An EC2 instance configured to perform NAT.

---

#### Which one is recommended for production?

```
NAT Gateway
```
