## AWS VPC Resource Limits and Regional Quotas — ENIs, IP Addresses, and Transit Gateway Attachments

AWS networking resources operate within **service quotas (limits)**. These limits exist to ensure stability of the AWS platform and to prevent accidental overconsumption of infrastructure resources.

Understanding these limits is important when designing **large-scale architectures**, because systems may fail or scale incorrectly if quotas are reached.

---

### 1. What Are AWS Service Quotas?

Service quotas are the **maximum number of resources allowed per account per region**.

Examples include limits for:

```
VPCs
subnets
Elastic Network Interfaces (ENIs)
Elastic IP addresses
NAT gateways
Transit Gateway attachments
```

These limits help AWS maintain reliable infrastructure.

Some quotas are:

```
soft limits → can be increased
hard limits → fixed by AWS
```

---

### 2. Why Quotas Matter in Architecture Design

In small environments, quotas are rarely a concern.

However, in large systems such as:

```
large Kubernetes clusters
high-scale microservices
enterprise multi-VPC environments
```

quotas can become bottlenecks.

Example scenario:

```
EKS cluster launches hundreds of pods
Each pod requires IP addresses
Subnet runs out of IPs
```

The cluster fails to scale.

Understanding quotas prevents these issues.

---

### 3. Elastic Network Interface (ENI) Limits

An **Elastic Network Interface (ENI)** is the virtual network adapter attached to an EC2 instance.

Each instance type supports a limited number of ENIs.

Example limits:

| Instance Type | Max ENIs |
|------|------|
| t3.micro | 2 |
| m5.large | 3 |
| c5.9xlarge | 8 |
| c6gn.16xlarge | 15 |

Each ENI supports **multiple IP addresses**.

---

### 4. IP Address Limits Per ENI

Each ENI can hold multiple private IP addresses.

Example limits:

| Instance | IPs per ENI |
|------|------|
| t3.micro | 2 |
| m5.large | 10 |
| c5.9xlarge | 30 |

Total available IPs for an instance:

```
ENIs × IPs per ENI
```

Example:

```
m5.large
3 ENIs × 10 IPs = 30 IP addresses
```

These limits are important for container platforms.

---

### 5. Subnet IP Address Limits

Subnet capacity depends on its **CIDR block size**.

Example:

```
/24 subnet → 256 IP addresses
```

However AWS reserves **5 IPs per subnet**:

```
network address
VPC router
DNS server
future use
broadcast equivalent
```

Usable IPs:

```
256 - 5 = 251
```

Example usable addresses:

| Subnet | Usable IPs |
|------|------|
| /28 | 11 |
| /24 | 251 |
| /16 | 65531 |

Poor subnet planning can cause scaling issues.

---

### 6. VPC Limits Per Region

Default limit:

```
5 VPCs per region
```

This limit can usually be increased.

Large organizations often use:

```
dozens or hundreds of VPCs
```

This requires quota increases.

---

### 7. Subnet Limits

Default subnet limit:

```
200 subnets per VPC
```

This can usually be increased.

Large environments may require many subnets for:

```
multi-AZ architectures
microservice isolation
Kubernetes networking
```

---

### 8. Elastic IP Limits

Elastic IP addresses are limited per region.

Default:

```
5 Elastic IPs per region
```

Elastic IPs are typically used for:

```
NAT gateways
load balancers
bastion hosts
static public endpoints
```

High-scale environments must request quota increases.

---

### 9. NAT Gateway Limits

Default limit:

```
5 NAT gateways per AZ
```

These limits affect large architectures with many private subnets.

Example architecture:

```
AZ-a
  NAT Gateway
AZ-b
  NAT Gateway
AZ-c
  NAT Gateway
```

Large-scale workloads may require multiple NAT gateways.

---

### 10. Transit Gateway Limits

Transit Gateway supports large-scale networking.

However it also has limits.

Example limits:

| Resource | Default Limit |
|------|------|
| VPC attachments | 5000 |
| VPN attachments | 50 |
| Route tables | 20 |
| Routes per table | 10,000 |

These limits support large enterprise architectures.

---

### 11. TGW Attachment Example

Example architecture:

```
Transit Gateway
      |
-------------------------------------
|        |        |        |         |
VPC-A  VPC-B  VPC-C  VPC-D  VPC-E
```

Each connection is a **TGW attachment**.

Large organizations may have hundreds of attachments.

---

### 12. Route Table Limits

VPC route tables have limits.

Example:

```
50 routes per route table (default)
```

This can be increased.

Large architectures using:

```
peering
Transit Gateway
VPN
Direct Connect
```

may require more routes.

---

### 13. Security Group Limits

Security group limits include:

```
60 inbound rules
60 outbound rules
```

Per security group.

Security group per instance limit:

```
5 security groups per ENI
```

These limits affect complex microsegmentation designs.

---

### 14. Viewing Service Quotas

Quotas can be viewed using:

```
AWS Service Quotas console
```

Console path:

```
AWS Console → Service Quotas
```

You can also use CLI:

```
aws service-quotas list-service-quotas \
  --service-code ec2
```

This shows EC2 networking limits.

---

### 15. Requesting Quota Increases

Many limits can be increased.

Example process:

```
Service Quotas Console
→ Select quota
→ Request increase
```

AWS typically approves requests quickly.

Some limits may take longer depending on scale.

---

### 16. Example Scaling Problem

Example scenario:

```
EKS cluster scaling
```

Nodes attempt to launch new pods.

Failure occurs because:

```
Subnet IP range exhausted
```

Solution:

```
expand CIDR
create additional subnets
```

Planning subnet sizes early prevents this issue.

---

### 17. Best Practices for Large Networks

Recommended practices include:

```
plan CIDR ranges carefully
use large VPC CIDR blocks
monitor IP utilization
track service quotas
design scalable subnet layouts
```

These practices prevent networking bottlenecks.

---

### 18. Monitoring Resource Usage

Monitoring tools include:

```
AWS Service Quotas dashboard
CloudWatch metrics
VPC IP address usage metrics
```

These help detect approaching limits.

---

### 19. Mental Model

Think of AWS quotas like **capacity limits in a building**.

```
Max number of rooms → subnets
Max number of doors → ENIs
Max number of hallways → routes
```

If the building exceeds capacity, expansion is required.

---

### 20. Key Concepts Summary

| Resource | Example Limit |
|------|------|
| VPCs per region | 5 |
| Subnets per VPC | 200 |
| Elastic IPs | 5 |
| Routes per table | 50 |
| TGW attachments | 5000 |

Always check quotas before designing large architectures.

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is an AWS service quota?

A limit on the number of resources allowed per account per region.

---

#### How many IPs are reserved in every subnet?

```
5
```

---

#### What resource connects VPCs to Transit Gateway?

```
TGW attachments
```

---

#### Why are ENI limits important for Kubernetes?

Because each pod or node may require IP addresses.

---

#### How can you increase quotas?

```
AWS Service Quotas console
```
