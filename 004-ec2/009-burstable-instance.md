## Burstable Instances (t2, t3, t4)

Not every application needs high CPU performance all the time. Many workloads spend most of their time doing very little processing and only occasionally require more CPU power.

To handle this kind of workload efficiently, AWS provides **burstable performance instances**, mainly in the **t family** such as:

- t2
- t3
- t4g

These instances are designed to provide **low baseline CPU performance most of the time**, while allowing the instance to **temporarily burst to higher CPU performance when needed**.

This design makes burstable instances **very cost-efficient** for workloads that do not constantly use the CPU.

Examples of burstable instance types include:

- t2.micro
- t3.small
- t3.medium
- t4g.micro
- t4g.small

These instances are commonly used for small workloads, development environments, and applications with **sporadic CPU usage**.

---

### 1. CPU Credits

The burst capability of these instances is controlled through a system called **CPU credits**.

A **CPU credit** represents the ability to use **one full vCPU for one minute**.

Burstable instances earn CPU credits over time when they are running below their baseline CPU usage. These saved credits can later be used when the instance needs more processing power.

The system works roughly like this:

1. When the instance is mostly idle, it **earns CPU credits**.
2. When the application needs more CPU power, the instance **spends those credits**.
3. Once credits are used up, the instance **returns to its baseline CPU performance**.

This allows the instance to temporarily handle spikes in workload without requiring a permanently powerful (and expensive) instance.

For example:

A small web server might usually run at **5% CPU usage**, slowly accumulating CPU credits.  
Later, if traffic spikes and CPU usage jumps to **80%**, the instance can temporarily use its saved credits to handle the load.

Once the burst period ends, it goes back to its normal baseline performance.

---

### 2. When to Use Burstable Instances

Burstable instances are ideal for workloads where CPU usage is **low most of the time but occasionally spikes**.

Common use cases include:

#### Development and Testing Environments

Development servers are often idle for long periods and only used occasionally.

Burstable instances are a cost-effective option for these environments.

---

#### Small Web Servers

Many small websites do not constantly use high CPU.

Traffic may spike occasionally, but the server is usually under light load.

Burstable instances can handle these patterns efficiently.

---

#### Microservices

Some microservices receive requests only occasionally.

These services benefit from burstable instances because they can scale CPU temporarily when requests arrive.

---

#### Low Traffic APIs

APIs that receive infrequent requests often have long idle periods, making burstable instances a good choice.

---

### 3. When to Avoid Burstable Instances

Burstable instances are not suitable for workloads that require **consistent high CPU performance**.

In such cases, CPU credits may run out, causing performance degradation.

Examples where burstable instances should be avoided include:

---

#### High Traffic Applications

Applications that consistently run at high CPU usage should use **compute optimized instances** instead.

---

#### Data Processing Jobs

Tasks such as batch processing, video encoding, or large computations require stable CPU performance.

Burstable instances may not provide consistent performance for these workloads.

---

#### High Performance Backend Systems

Systems that must maintain predictable performance under constant load should use instance types with dedicated CPU resources.

---

### 4. Summary

Burstable instances in the **t family (t2, t3, t4)** are designed for workloads with **variable CPU usage**.

They provide a baseline level of CPU performance while allowing temporary bursts using CPU credits.

Key characteristics include:

| Concept | Explanation |
|------|------|
| Burstable instances | Instances that allow temporary CPU spikes |
| CPU credits | Units that allow the instance to burst above baseline CPU performance |
| Best use cases | Development servers, small websites, microservices |
| Avoid when | Workloads require sustained high CPU usage |

Burstable instances are often the **most cost-efficient EC2 option** for lightweight workloads that do not constantly need high CPU power.
