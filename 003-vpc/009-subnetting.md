## Subnetting in AWS

Before designing real cloud networks in AWS, it is important to **revisit subnetting**, but this time with **AWS-specific rules and constraints**.

Traditional networking and AWS networking are similar, but AWS introduces **additional restrictions and behaviors** that you must understand when designing VPC networks.

---

### 1. Quick Recap — What Is Subnetting?

Subnetting means:

> Dividing a large network into smaller networks.

Example network:

```
10.0.0.0/16
```

Total IP addresses:

```
2^(32 - 16) = 65536
```

Instead of using one huge network, we divide it into smaller subnets.

Example:

```
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

Each subnet can host different infrastructure components.

Example:

```
10.0.1.0/24 → Public subnet
10.0.2.0/24 → Application subnet
10.0.3.0/24 → Database subnet
```

---

### 2. Why Subnetting Matters in AWS

Subnetting is important because it helps you control:

- security boundaries
- routing rules
- availability zone placement
- service architecture
- scaling limits

Good subnet design prevents:

```
IP exhaustion
routing conflicts
network redesign later
```

This is why **CIDR planning is critical before creating a VPC**.

---

### 3. VPC CIDR Block

When creating a VPC you must specify an **IPv4 CIDR block**.

Example:

```
10.0.0.0/16
```

This defines the **total IP address space** available inside the VPC.

Example sizes:

| CIDR | Total IPs |
|-----|-----|
| /16 | 65536 |
| /17 | 32768 |
| /18 | 16384 |
| /19 | 8192 |
| /20 | 4096 |
| /21 | 2048 |
| /22 | 1024 |
| /23 | 512 |
| /24 | 256 |

---

### 4. AWS VPC Size Limits

AWS places limits on VPC size.

Allowed VPC CIDR range:

```
/16 (largest)
to
/28 (smallest)
```

Examples of valid VPC CIDRs:

```
10.0.0.0/16
10.1.0.0/20
192.168.0.0/24
```

Invalid example:

```
/15
/29
```

These are not allowed for VPC CIDR blocks.

---

### 5. Multiple CIDR Blocks in a VPC

AWS allows a VPC to have **multiple CIDR blocks**.

Example:

```
Primary CIDR
10.0.0.0/16
```

Additional CIDR:

```
10.1.0.0/16
```

This allows network expansion without recreating the VPC.

---

### 6. Subnet CIDR Limits in AWS

Subnets also have size restrictions.

Allowed subnet size:

```
/16 (largest)
to
/28 (smallest)
```

Example subnets:

```
10.0.1.0/24
10.0.2.0/26
10.0.3.0/28
```

Minimum subnet size:

```
/28
```

A /28 subnet contains:

```
16 IP addresses
```

---

### 7. AWS Reserved IP Addresses

AWS reserves **five IP addresses in every subnet**.

These addresses cannot be used.

Example subnet:

```
10.0.1.0/24
```

Reserved IPs:

| Address | Purpose |
|------|------|
| .0 | Network address |
| .1 | VPC router |
| .2 | DNS server |
| .3 | Reserved for future |
| .255 | Broadcast address |

So AWS reserves **5 addresses per subnet**.

---

### 8. Example Reserved IPs

Example subnet:

```
10.0.1.0/24
```

Reserved addresses:

```
10.0.1.0
10.0.1.1
10.0.1.2
10.0.1.3
10.0.1.255
```

Usable range:

```
10.0.1.4 → 10.0.1.254
```

---

### 9. Calculating Usable IPs in AWS

Standard networking rule:

```
usable hosts = total hosts - 2
```

But in AWS:

```
usable hosts = total hosts - 5
```

Because AWS reserves five IPs.

---

### 10. Example Calculations

#### Example 1 — /24 subnet

Total IPs:

```
256
```

Reserved by AWS:

```
5
```

Usable IPs:

```
251
```

---

#### Example 2 — /26 subnet

Total IPs:

```
64
```

Usable:

```
64 - 5 = 59
```

---

#### Example 3 — /28 subnet

Total IPs:

```
16
```

Usable:

```
16 - 5 = 11
```

---

### 11. Why AWS Reserves These IPs

AWS uses reserved IPs for internal infrastructure.

Examples:

```
VPC router
DNS service
future services
```

These services operate **inside the subnet automatically**.

---

### 12. Subnet Planning Strategy

Example VPC:

```
10.0.0.0/16
```

We divide it like this:

```
Public Subnet AZ-a → 10.0.1.0/24
Public Subnet AZ-b → 10.0.2.0/24

Private Subnet AZ-a → 10.0.10.0/24
Private Subnet AZ-b → 10.0.11.0/24

Database Subnet AZ-a → 10.0.20.0/24
Database Subnet AZ-b → 10.0.21.0/24
```

This gives **six subnets across two availability zones**.

---

### 13. Example Enterprise Design

Large infrastructure example:

```
VPC → 10.0.0.0/16
```

Subnet layout:

```
Public Subnets
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24

Private App Subnets
10.0.10.0/24
10.0.11.0/24
10.0.12.0/24

Database Subnets
10.0.20.0/24
10.0.21.0/24
10.0.22.0/24
```

This architecture supports **multi-AZ deployments**.

---

### 14. Example Scaling Problem

Poor subnet planning example:

```
VPC → 10.0.0.0/24
```

This provides:

```
256 IPs total
251 usable
```

If your infrastructure grows:

```
EC2
EKS nodes
Load balancers
NAT
Databases
```

You may run out of IPs.

This is why production environments often use:

```
/16 VPC networks
```

---

### 15. Recommended CIDR Sizes

Typical recommendations:

| Environment | VPC CIDR |
|------|------|
| Small labs | /24 |
| Small startup | /20 |
| Medium systems | /18 |
| Large production | /16 |

---

### 16. Visual Example

```
VPC 10.0.0.0/16

AZ-a
 ├ Public → 10.0.1.0/24
 ├ Private → 10.0.10.0/24
 └ Database → 10.0.20.0/24

AZ-b
 ├ Public → 10.0.2.0/24
 ├ Private → 10.0.11.0/24
 └ Database → 10.0.21.0/24
```

This supports **high availability architecture**.

---

### 17. Key Concepts Summary

| Concept | Meaning |
|------|------|
| VPC CIDR | Total network range |
| Subnet | Smaller network inside VPC |
| VPC Size Limits | /16 to /28 |
| Subnet Size Limits | /16 to /28 |
| AWS Reserved IPs | 5 per subnet |
| Usable IP Formula | Total IPs - 5 |

---

### 18. Checkpoint Questions

You should now be able to answer these.

#### What is the largest VPC size in AWS?

```
/16
```

---

#### What is the smallest subnet allowed?

```
/28
```

---

#### How many IPs are reserved by AWS in a subnet?

```
5
```

---

#### How many usable IPs exist in a /24 subnet?

```
256 - 5 = 251
```

---

#### Why must subnet planning be done carefully?

Because poor planning can lead to:

```
IP exhaustion
infrastructure redesign
network conflicts
```
