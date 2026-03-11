## Site-to-Site VPN and AWS Managed VPN — IPsec Fundamentals and Hybrid Connectivity

Many organizations already have **on-premise data centers** before moving to the cloud. When they start using AWS, they often need their cloud infrastructure to communicate securely with their existing networks.

This architecture is called **hybrid cloud networking**.

AWS provides **Site-to-Site VPN** to securely connect an on-premise network with an AWS VPC over the internet using encrypted tunnels.

---

### 1. What Is Hybrid Cloud Networking?

Hybrid cloud means combining **on-premise infrastructure** with **cloud infrastructure**.

Example architecture:

```
On-Premise Data Center
        |
        |
      VPN
        |
        |
AWS VPC
```

This allows systems in both environments to communicate privately.

Example communication:

```
On-Prem Database → AWS Application
AWS Analytics → On-Prem Data Warehouse
```

---

### 2. What Is a Site-to-Site VPN?

A **Site-to-Site VPN** connects two networks securely across the internet.

Definition:

> A Site-to-Site VPN establishes an encrypted tunnel between an on-premise network and an AWS VPC.

This tunnel uses encryption so that data traveling over the public internet cannot be read by attackers.

Example architecture:

```
On-Prem Router
      |
      |  Encrypted Tunnel
      |
AWS VPN Gateway
      |
      |
VPC Network
```

Traffic between both networks flows through the encrypted tunnel.

---

### 3. AWS Managed VPN Components

AWS Site-to-Site VPN involves two main components.

```
Customer Gateway (CGW)
Virtual Private Gateway (VGW) or Transit Gateway
```

#### Customer Gateway

Represents the **on-premise VPN device**.

Examples:

```
Corporate firewall
VPN router
Network appliance
```

Example device vendors:

```
Cisco
Juniper
Palo Alto
Fortinet
```

---

#### Virtual Private Gateway (VGW)

The **AWS-side VPN endpoint**.

It attaches directly to a VPC.

Architecture:

```
VPC
 |
Virtual Private Gateway
 |
VPN Tunnel
 |
Customer Gateway
 |
On-Prem Network
```

---

### 4. Alternative Architecture Using Transit Gateway

Large architectures often use **Transit Gateway instead of VGW**.

Example:

```
On-Prem Network
      |
VPN
      |
Transit Gateway
      |
Multiple VPCs
```

This centralizes connectivity.

---

### 5. What Is IPsec?

AWS Site-to-Site VPN uses **IPsec (Internet Protocol Security)**.

Definition:

> IPsec is a suite of protocols used to encrypt and authenticate IP network traffic.

IPsec protects:

```
confidentiality
integrity
authentication
```

This ensures traffic cannot be read or modified.

---

### 6. IPsec Tunnel Concept

A VPN tunnel wraps original packets inside encrypted packets.

Normal packet:

```
Source IP → Destination IP
```

IPsec packet:

```
Encrypted Payload
```

Example:

```
Original Packet
10.0.1.10 → 192.168.1.20
```

Inside tunnel:

```
Encrypted Packet
```

Only the VPN endpoints can decrypt it.

---

### 7. Two VPN Tunnels for High Availability

AWS creates **two VPN tunnels** automatically.

Example:

```
On-Prem Router
   |      |
Tunnel A  Tunnel B
   |      |
AWS VPN Gateway
```

If one tunnel fails:

```
Traffic automatically switches to the second tunnel
```

This improves reliability.

---

### 8. VPN Traffic Flow Example

Example communication:

```
On-Prem Server → AWS EC2
```

Traffic flow:

```
On-Prem Server
      |
Customer Gateway
      |
Encrypted Tunnel
      |
Virtual Private Gateway
      |
VPC Router
      |
EC2 Instance
```

Data remains encrypted during transit.

---

### 9. Static vs Dynamic Routing

AWS VPN supports two routing methods.

#### Static Routing

Routes are manually configured.

Example:

```
192.168.0.0/16 → VPN
```

Advantages:

```
simple setup
```

Disadvantages:

```
not scalable
manual updates
```

---

#### Dynamic Routing (BGP)

Uses **Border Gateway Protocol (BGP)**.

BGP automatically exchanges routes.

Example:

```
On-Prem advertises routes
AWS advertises VPC routes
```

Advantages:

```
automatic failover
scalable
preferred for production
```

---

### 10. Example Routing Architecture

Example network ranges:

```
On-Prem → 192.168.0.0/16
AWS VPC → 10.0.0.0/16
```

Routing tables:

```
On-Prem Router
10.0.0.0/16 → VPN Tunnel

AWS Route Table
192.168.0.0/16 → Virtual Private Gateway
```

Now both networks can communicate.

---

### 11. VPN Encryption Algorithms

IPsec uses cryptographic algorithms such as:

```
AES-256
SHA-256
Diffie-Hellman key exchange
```

These ensure secure encryption and authentication.

---

### 12. Example Hybrid Architecture

Example production setup:

```
On-Prem Data Center
      |
Firewall / VPN Router
      |
Site-to-Site VPN
      |
AWS Virtual Private Gateway
      |
VPC
   |
Application Servers
```

Applications in AWS can access on-prem resources securely.

---

### 13. Multi-VPC Hybrid Architecture

Using Transit Gateway:

```
On-Prem Data Center
      |
VPN
      |
Transit Gateway
   /      |      \
VPC-A   VPC-B   VPC-C
```

All VPCs gain connectivity to on-prem systems.

---

### 14. VPN vs Direct Connect

| Feature | VPN | Direct Connect |
|------|------|------|
| Connectivity | Internet | Dedicated line |
| Cost | Lower | Higher |
| Latency | Variable | Consistent |
| Setup time | Minutes | Weeks |

VPN is often used first.

Direct Connect is used for **high-performance workloads**.

---

### 15. Example Use Cases

Common use cases include:

```
hybrid cloud architecture
cloud disaster recovery
data center extension
secure corporate connectivity
```

Example:

```
On-Prem ERP → AWS Analytics
```

---

### 16. Security Advantages

Benefits of Site-to-Site VPN include:

```
encrypted communication
secure hybrid networking
redundant tunnels
automatic failover
```

This protects data over the public internet.

---

### 17. Mental Model

Think of a Site-to-Site VPN like a **secure underground tunnel between two buildings**.

```
Building A → Secure Tunnel → Building B
```

Even though the tunnel runs under a public road (internet), nobody outside can see inside.

---

### 18. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Hybrid Cloud | On-prem + cloud infrastructure |
| Site-to-Site VPN | Encrypted connection between networks |
| Customer Gateway | On-prem VPN device |
| Virtual Private Gateway | AWS VPN endpoint |
| IPsec | Encryption protocol for VPN tunnels |

---

### 19. Checkpoint Questions

You should now be able to answer these.

#### What is Site-to-Site VPN?

A secure encrypted tunnel connecting an on-prem network to an AWS VPC.

---

#### What encryption protocol does AWS VPN use?

```
IPsec
```

---

#### What is a Customer Gateway?

The on-premise VPN device.

---

#### What is a Virtual Private Gateway?

The AWS-side VPN endpoint attached to a VPC.

---

#### Why does AWS create two VPN tunnels?

For high availability and failover.
