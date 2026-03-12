## Load Balancers

In real-world applications, a single server is usually not enough to handle all incoming traffic. As user traffic grows, relying on one EC2 instance can cause several problems such as performance bottlenecks, downtime, or server crashes.

To solve this problem, AWS provides **Elastic Load Balancing (ELB)**.

A **load balancer** is a service that automatically distributes incoming traffic across multiple servers.

Instead of sending all requests to a single EC2 instance, the load balancer spreads the requests across several instances.

This improves:

- reliability
- scalability
- availability
- fault tolerance

Load balancers are commonly placed in front of multiple EC2 instances to ensure that applications remain responsive and available even under heavy traffic.

---

### 1. Why Load Balancers Exist

Load balancers exist to solve several important problems in distributed systems.

#### Traffic Distribution

When many users access an application simultaneously, sending all traffic to one server can overwhelm it.

A load balancer distributes requests across multiple servers so that no single server becomes overloaded.

Example:

Without load balancer:

```
Users → EC2 Instance
```

With load balancer:

```
Users → Load Balancer → Multiple EC2 Instances
```

This ensures that each server handles only a portion of the workload.

---

#### High Availability

If one EC2 instance fails, the load balancer can automatically redirect traffic to healthy instances.

This prevents the application from going offline.

For example:

- Instance A fails
- Load balancer stops sending traffic to Instance A
- Traffic continues flowing to Instance B and C

This improves system reliability.

---

#### Scalability

Load balancers work together with **Auto Scaling Groups**.

When traffic increases:

- new EC2 instances are launched
- the load balancer automatically starts sending traffic to them

When traffic decreases:

- instances can be removed

This allows applications to scale dynamically.

---

#### Health Checks

Load balancers continuously check whether instances are healthy.

If an instance fails health checks, the load balancer removes it from traffic routing.

This prevents users from being routed to broken servers.

---

### 2. Types of AWS Load Balancers

AWS provides three main types of load balancers:

- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- Classic Load Balancer (CLB)

Each is designed for different use cases.

---

### 3. Application Load Balancer (ALB)

The **Application Load Balancer** operates at **Layer 7 (Application Layer)** of the OSI model.

This means it understands **HTTP and HTTPS requests**.

ALB can make routing decisions based on:

- URL paths
- hostnames
- HTTP headers
- query parameters

Example routing:

```
example.com/api → API servers
example.com/images → Image servers
example.com/shop → Ecommerce servers
```

Common use cases include:

- web applications
- microservices architectures
- REST APIs
- containerized applications

Key features:

- path-based routing
- host-based routing
- WebSocket support
- integration with containers and Kubernetes

ALB is the most commonly used load balancer for modern web applications.

---

### 4. Network Load Balancer (NLB)

The **Network Load Balancer** operates at **Layer 4 (Transport Layer)**.

Instead of understanding HTTP requests, it routes traffic based on:

- TCP
- UDP
- IP protocols

NLB is designed for **extremely high performance and low latency**.

It can handle **millions of requests per second**.

Key characteristics:

- ultra-low latency
- static IP support
- high throughput
- supports TCP and UDP traffic

Common use cases include:

- high-performance applications
- real-time systems
- gaming servers
- financial trading systems
- IoT systems

---

### 5. Classic Load Balancer (CLB)

The **Classic Load Balancer** is the older generation of AWS load balancers.

It supports both:

- Layer 4 (TCP)
- Layer 7 (HTTP)

However, it lacks many advanced features available in newer load balancers.

Because of this, AWS generally recommends using **ALB or NLB instead of CLB** for new applications.

Classic Load Balancers are mostly found in **legacy systems** that were created before newer load balancers were introduced.

---

### 6. ALB vs NLB vs CLB

The main differences between these load balancers can be summarized as follows:

| Feature | ALB | NLB | CLB |
|------|------|------|------|
| OSI Layer | Layer 7 | Layer 4 | Layer 4 & 7 |
| Protocols | HTTP, HTTPS | TCP, UDP | HTTP, HTTPS, TCP |
| Routing Logic | Advanced request routing | Simple packet routing | Basic routing |
| Performance | High | Extremely high | Moderate |
| Typical Use | Web apps, APIs | High performance networking | Legacy systems |

---

### 7. Summary

Load balancers are critical components in scalable and highly available cloud architectures.

They distribute incoming traffic across multiple EC2 instances to improve performance, reliability, and scalability.

AWS provides three main types of load balancers:

| Load Balancer | Best For |
|------|------|
| Application Load Balancer | Web applications and APIs |
| Network Load Balancer | High-performance network traffic |
| Classic Load Balancer | Older legacy applications |

In modern architectures, **Application Load Balancers and Network Load Balancers are the preferred choices**, while Classic Load Balancers are mostly used for legacy systems.
