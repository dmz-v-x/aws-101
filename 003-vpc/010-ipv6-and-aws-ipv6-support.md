## IPv6 Basics and AWS IPv6 Support — Dual Stack, EUI-64, and Egress-Only Internet Gateway

As cloud infrastructure grows, the number of internet-connected devices has increased dramatically. IPv4, the traditional internet addressing system, has a limited address space.

To solve this limitation, a newer protocol called **IPv6 (Internet Protocol version 6)** was introduced.

AWS fully supports IPv6 in VPC networking. Understanding IPv6 is important because modern cloud architectures often use **dual-stack networking**, where both IPv4 and IPv6 operate together.

---

### 1. Why IPv6 Exists

IPv4 addresses use **32 bits**.

Total possible IPv4 addresses:

```
2^32 = 4,294,967,296
```

That is about **4.3 billion addresses**.

With billions of devices connected to the internet:

- smartphones
- computers
- servers
- IoT devices
- cloud infrastructure

IPv4 address space became exhausted.

IPv6 was designed to solve this problem.

---

### 2. IPv6 Address Space

IPv6 uses **128-bit addresses**.

Total addresses:

```
2^128
```

This equals approximately:

```
340 undecillion addresses
```

This number is extremely large.

Example IPv6 address:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

IPv6 addresses use:

```
hexadecimal numbers
```

Each section is called a **hextet**.

---

### 3. IPv6 Address Structure

IPv6 addresses contain **8 blocks** separated by colons.

Example:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

Each block represents **16 bits**.

Total:

```
8 × 16 = 128 bits
```

---

### 4. IPv6 Address Shortening

IPv6 allows shortening rules.

Example original address:

```
2001:0db8:0000:0000:0000:0000:1428:57ab
```

Remove leading zeros:

```
2001:db8:0:0:0:0:1428:57ab
```

Compress continuous zeros:

```
2001:db8::1428:57ab
```

This shortened format is commonly used.

---

### 5. IPv6 vs IPv4

| Feature | IPv4 | IPv6 |
|------|------|------|
| Address size | 32 bits | 128 bits |
| Format | Decimal | Hexadecimal |
| Example | 192.168.1.1 | 2001:db8::1 |
| NAT required | Often | Usually unnecessary |
| Address space | Limited | Extremely large |

---

### 6. IPv6 in AWS VPC

AWS allows adding **IPv6 CIDR blocks** to a VPC.

Example:

```
IPv4 CIDR
10.0.0.0/16
```

Add IPv6 CIDR:

```
2600:1f18:abcd::/56
```

This provides a large IPv6 address range.

Each subnet receives a portion of the IPv6 range.

---

### 7. IPv6 Subnet Size in AWS

AWS assigns **/64 subnet blocks**.

Example VPC IPv6 CIDR:

```
2600:1f18:abcd::/56
```

Possible subnets:

```
2600:1f18:abcd:0001::/64
2600:1f18:abcd:0002::/64
2600:1f18:abcd:0003::/64
```

Each subnet contains:

```
2^64 addresses
```

Which is an extremely large number.

---

### 8. Dual-Stack Networking

Many systems still rely on IPv4.

Therefore AWS supports **dual-stack networking**.

Definition:

> Dual-stack means running IPv4 and IPv6 simultaneously.

Example instance:

```
Private IPv4 → 10.0.1.10
IPv6 → 2600:1f18:abcd:1::10
```

This allows communication with both:

```
IPv4 systems
IPv6 systems
```

Architecture example:

```
Internet
   |
IGW
   |
Dual-Stack Subnet
   |
EC2 Instance
   |-- IPv4
   |-- IPv6
```

---

### 9. EUI-64 Address Generation

When assigning IPv6 addresses automatically, AWS uses **EUI-64**.

EUI-64 stands for:

```
Extended Unique Identifier (64-bit)
```

It generates the **host portion of the IPv6 address** from the network interface.

Example process:

```
Subnet prefix → 2600:1f18:abcd:1::
```

Host portion generated automatically.

Example instance IPv6:

```
2600:1f18:abcd:1:4c5d:6eff:fe78:1234
```

This allows automatic address assignment.

---

### 10. Internet Connectivity for IPv6

Unlike IPv4, IPv6 does **not typically use NAT**.

Every instance can have a **globally routable IPv6 address**.

Traffic path:

```
Internet
   |
Internet Gateway
   |
IPv6 Subnet
   |
EC2 Instance
```

---

### 11. Security Implications

Because IPv6 addresses are globally reachable, security groups become extremely important.

Security groups control access.

Example rules:

```
Allow HTTP
Allow HTTPS
Allow SSH from specific IP
```

Without these rules, instances could be exposed.

---

### 12. Egress-Only Internet Gateway

IPv6 introduces a special gateway called an **Egress-Only Internet Gateway**.

Definition:

> An Egress-Only Internet Gateway allows outbound IPv6 traffic but blocks inbound connections.

It is similar to **NAT behavior for IPv6**.

Traffic allowed:

```
Instance → Internet
```

Traffic blocked:

```
Internet → Instance
```

Architecture:

```
Private IPv6 Subnet
        |
Egress-Only Internet Gateway
        |
Internet
```

This protects internal infrastructure.

---

### 13. IPv6 Routing Example

Route table example:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |
| 2600:1f18:abcd::/56 | local |
| ::/0 | igw |

Explanation:

```
::/0 → all IPv6 internet traffic
```

---

### 14. IPv6 Private Subnet Example

Private IPv6 route table:

```
::/0 → Egress-Only IGW
```

Traffic flow:

```
EC2 → Egress-Only IGW → Internet
```

Inbound connections are blocked.

---

### 15. Full Dual-Stack Architecture Example

```
                Internet
                   |
            Internet Gateway
                   |
            -----------------
            |               |
     Public Dual-Stack  Public Dual-Stack
        Subnet AZ-a       Subnet AZ-b
            |               |
        Web Servers      Web Servers
            |
      Private IPv4 + IPv6 Subnets
            |
        Application Servers
            |
      Database Subnets
```

Both IPv4 and IPv6 operate simultaneously.

---

### 16. Advantages of IPv6

Benefits include:

```
massive address space
no need for NAT
simpler routing
better end-to-end connectivity
future-proof networking
```

---

### 17. Key Concepts Summary

| Concept | Meaning |
|------|------|
| IPv6 | 128-bit internet addressing system |
| Dual Stack | IPv4 + IPv6 running together |
| EUI-64 | Automatic IPv6 host address generation |
| Egress-Only IGW | Outbound-only IPv6 gateway |
| IPv6 Subnet | Always /64 in AWS |

---

### 18. Checkpoint Questions

You should now be able to answer these.

#### Why was IPv6 created?

Because IPv4 address space became exhausted.

---

#### What is dual-stack networking?

Running IPv4 and IPv6 simultaneously.

---

#### What is EUI-64?

A method to generate IPv6 host addresses automatically.

---

#### What does an Egress-Only Internet Gateway do?

Allows outbound IPv6 internet access while blocking inbound connections.

---

#### What subnet size does AWS use for IPv6?

```
/64
```

