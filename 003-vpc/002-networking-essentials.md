## Basic Networking Essentials You Must Know (IP, CIDR, Subnetting, NAT, TCP/UDP, Ports, DNS)

---

### 1. What Is an IP Address?

Every device connected to a network must have a **unique identifier** so that other devices can communicate with it.

That identifier is called an **IP address**.

An **IP address (Internet Protocol Address)** is:

> A unique numeric address assigned to a device on a network.

It works similar to a **home address**.

Example:

```
House address → identifies where you live
IP address → identifies where a computer exists on a network
```

---

### 2. IPv4 Address Structure

Most networks still use **IPv4 addresses**.

An IPv4 address looks like this:

```
192.168.1.10
```

It consists of **4 numbers separated by dots**.

```
A.B.C.D
```

Example:

```
10.0.0.5
172.16.10.8
192.168.1.25
8.8.8.8
```

Each section can have values from:

```
0 → 255
```

Why?

Because each part is **8 bits**.

```
2^8 = 256 values
0 to 255
```

So an IPv4 address has:

```
4 × 8 bits = 32 bits
```

Total possible IPv4 addresses:

```
2^32 = 4,294,967,296
```

Around **4.3 billion addresses**.

---

### 3. Public vs Private IP Addresses

Not every IP address is available on the internet.

There are two main types:

#### Public IP

A **public IP** is accessible from the internet.

Example:

```
8.8.8.8
1.1.1.1
```

These are globally reachable.

---

#### Private IP

Private IPs are used **inside private networks**.

They are **not reachable directly from the internet**.

Common private ranges:

```
10.0.0.0 – 10.255.255.255
172.16.0.0 – 172.31.255.255
192.168.0.0 – 192.168.255.255
```

Example home network:

```
Laptop → 192.168.1.10
Phone → 192.168.1.20
Printer → 192.168.1.30
```

These communicate internally.

---

### 4. What Is CIDR?

CIDR stands for:

```
Classless Inter-Domain Routing
```

CIDR is a **notation used to represent networks and their size**.

Example CIDR:

```
192.168.1.0/24
10.0.0.0/16
172.31.0.0/20
```

CIDR has two parts:

```
Network address + prefix length
```

Example:

```
192.168.1.0/24
```

Here:

```
192.168.1.0 → network
/24 → prefix length
```

The prefix tells us **how many bits belong to the network portion**.

---

### 5. Understanding Prefix Length

IPv4 addresses contain **32 bits**.

Example:

```
192.168.1.0/24
```

Means:

```
First 24 bits → network
Remaining 8 bits → host
```

So:

```
32 - 24 = 8 host bits
```

---

### 6. Calculating Number of Hosts

To calculate hosts in a subnet:

```
Hosts = 2^(host bits)
```

Example:

```
/24
```

Host bits:

```
32 - 24 = 8
```

Hosts:

```
2^8 = 256
```

But in most networks:

```
2 addresses are reserved
```

So usable hosts:

```
256 - 2 = 254
```

---

### 7. CIDR Examples

#### Example 1

```
10.0.0.0/24
```

Host bits:

```
32 - 24 = 8
```

Hosts:

```
2^8 = 256
Usable = 254
```

Range:

```
10.0.0.0 → network
10.0.0.1 → first host
10.0.0.254 → last host
10.0.0.255 → broadcast
```

---

#### Example 2

```
10.0.0.0/16
```

Host bits:

```
32 - 16 = 16
```

Hosts:

```
2^16 = 65536
Usable = 65534
```

---

#### Example 3

```
192.168.1.0/30
```

Host bits:

```
32 - 30 = 2
```

Hosts:

```
2^2 = 4
Usable = 2
```

Often used for **point-to-point links**.

---

### 8. Quick CIDR Reference Table

| CIDR | Total Hosts | Usable Hosts |
|-----|-----|-----|
| /30 | 4 | 2 |
| /29 | 8 | 6 |
| /28 | 16 | 14 |
| /27 | 32 | 30 |
| /26 | 64 | 62 |
| /25 | 128 | 126 |
| /24 | 256 | 254 |
| /23 | 512 | 510 |
| /22 | 1024 | 1022 |
| /21 | 2048 | 2046 |
| /20 | 4096 | 4094 |
| /19 | 8192 | 8190 |
| /18 | 16384 | 16382 |
| /17 | 32768 | 32766 |
| /16 | 65536 | 65534 |

---

### 9. What Is Subnetting?

Subnetting means **dividing a network into smaller networks**.

Example network:

```
10.0.0.0/16
```

This network is very large.

We can divide it into smaller networks:

```
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

Each subnet can be used for different systems.

Example architecture:

```
10.0.1.0/24 → Web servers
10.0.2.0/24 → Application servers
10.0.3.0/24 → Databases
```

This improves:

- security
- organization
- routing control

---

### 10. Subnetting Example

Start network:

```
10.0.0.0/16
```

Split into /24 networks.

Possible subnets:

```
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
10.0.4.0/24
...
```

Each subnet:

```
254 usable hosts
```

This is exactly what happens in **AWS VPC subnets**.

---

### 11. What Is NAT?

NAT stands for:

```
Network Address Translation
```

NAT allows **private IP addresses to access the internet**.

Example:

Your laptop has private IP:

```
192.168.1.10
```

Web server on internet:

```
142.250.183.14
```

Your laptop cannot reach internet directly because private IPs are not routable.

So a **router performs NAT**.

---

### 12. NAT Process Example

Step 1

Laptop sends request:

```
192.168.1.10 → google.com
```

Step 2

Router replaces source IP with public IP:

```
Public IP → 103.25.200.5
```

Step 3

Google responds.

Step 4

Router sends response back to laptop.

So:

```
Laptop → Router (NAT) → Internet
```

This is how home networks work.

---

### 13. What Is TCP?

TCP stands for:

```
Transmission Control Protocol
```

TCP is used for **reliable communication**.

Features:

- reliable delivery
- packet ordering
- error checking
- retransmission

Example uses:

```
HTTP
HTTPS
SSH
FTP
```

If packets are lost, TCP **re-sends them**.

---

### 14. What Is UDP?

UDP stands for:

```
User Datagram Protocol
```

UDP is **faster but unreliable**.

Features:

- no retransmission
- no ordering guarantee
- low latency

Used for:

```
video streaming
online gaming
DNS
VoIP
```

---

### 15. TCP vs UDP

| Feature | TCP | UDP |
|------|------|------|
| Reliability | Reliable | Unreliable |
| Speed | Slower | Faster |
| Ordering | Guaranteed | Not guaranteed |
| Use Cases | Web, SSH | Streaming, DNS |

---

### 16. What Are Ports?

An IP address identifies a **device**.

A **port identifies an application** on that device.

Example:

```
Server IP → 192.168.1.10
```

Multiple applications may run on this server.

Ports allow them to coexist.

Examples:

```
80 → HTTP
443 → HTTPS
22 → SSH
3306 → MySQL
5432 → PostgreSQL
```

Example connection:

```
192.168.1.5 → 192.168.1.10:80
```

Meaning:

Client connecting to **web server**.

---

### 17. What Is DNS?

DNS stands for:

```
Domain Name System
```

DNS converts **domain names into IP addresses**.

Example:

```
google.com
```

Your computer asks a DNS server:

```
What is the IP of google.com?
```

DNS replies:

```
142.250.183.14
```

Now the browser connects to that IP.

---

### 18. DNS Resolution Flow

Step 1

User types:

```
google.com
```

Step 2

Computer asks DNS resolver.

Step 3

Resolver queries DNS hierarchy.

Step 4

IP returned.

Step 5

Browser connects to server.

---

### 19. Putting Everything Together

When you open a website:

1. DNS resolves domain to IP  
2. Browser connects using **TCP port 443**  
3. Packets travel through routers  
4. NAT may translate IPs  
5. Server responds  

Networking concepts working together:

```
DNS + IP + Ports + TCP
```

---

### 20. Lab Exercise — CIDR Practice

Practice calculating hosts.

Example 1

```
/24
```

Host bits:

```
32 - 24 = 8
```

Hosts:

```
2^8 = 256
Usable = 254
```

---

Example 2

```
/16
```

Host bits:

```
32 - 16 = 16
```

Hosts:

```
2^16 = 65536
Usable = 65534
```

---

Example 3

```
/26
```

Host bits:

```
32 - 26 = 6
```

Hosts:

```
2^6 = 64
Usable = 62
```

---

### 21. Online Subnet Calculator

You can verify calculations using:

```
https://www.subnet-calculator.com/
```

Try entering:

```
10.0.0.0/24
10.0.0.0/16
```

Observe:

- host ranges
- broadcast address
- network address

---

### 22. Checkpoint Questions

Before moving to AWS VPC, you should answer these easily.

#### How many hosts are in /24?

```
256 total
254 usable
```

---

#### How many hosts are in /16?

```
65536 total
65534 usable
```

---

#### What does NAT do?

NAT converts **private IP addresses into public IP addresses** for internet communication.

---

#### What does DNS do?

DNS converts **domain names into IP addresses**.
