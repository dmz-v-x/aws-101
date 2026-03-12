## EBS Volume Types

Not all applications require the same type of storage performance. Some workloads need **very fast disk performance**, while others only need **cheap storage for large amounts of data**.

To support different workloads, AWS provides multiple **EBS volume types**, each optimized for specific use cases such as:

- general workloads
- high-performance databases
- large data storage

The main EBS volume categories include:

- gp2 / gp3 (General Purpose SSD)
- io1 / io2 (Provisioned IOPS SSD)
- st1 (Throughput Optimized HDD)
- sc1 (Cold HDD)

Each type offers a different balance between **performance and cost**.

---

### 1. General Purpose SSD (gp2 / gp3)

**General Purpose SSD volumes** are the most commonly used EBS volumes.

They provide a **balanced combination of price and performance**, making them suitable for most workloads.

These volumes use **Solid State Drives (SSD)**, which provide fast read and write speeds.

There are two main versions:

- gp2 (older generation)
- gp3 (newer and recommended)

#### gp2

gp2 volumes automatically scale their performance based on the size of the volume.

The larger the volume size, the more performance it can deliver.

These volumes also support burst performance using an internal credit system.

---

#### gp3

gp3 is the **latest generation of general purpose EBS volumes**.

Compared to gp2, gp3 offers several improvements:

- better performance
- lower cost
- ability to configure IOPS independently from storage size

This means you can increase disk performance **without increasing the storage capacity**.

gp3 volumes are widely used for:

- application servers
- development environments
- small to medium databases
- web servers

Because of their flexibility and cost efficiency, gp3 volumes are usually the **default choice for most EC2 workloads**.

---

### 2. Provisioned IOPS SSD (io1 / io2)

Provisioned IOPS volumes are designed for workloads that require **extremely high and consistent disk performance**.

These volumes allow you to **explicitly specify the number of I/O operations per second (IOPS)** that the storage should support.

This makes them ideal for performance-sensitive applications.

There are two main types:

- io1
- io2

---

#### io1

io1 volumes were designed for applications that require **high I/O throughput and low latency**.

They allow you to provision thousands of IOPS to support demanding workloads.

---

#### io2

io2 volumes are an improved version of io1.

They provide:

- higher durability
- better reliability
- more consistent performance

These volumes are typically used for **mission-critical applications**.

Common use cases include:

- enterprise databases
- high transaction systems
- large relational databases
- financial systems

Because of their high performance guarantees, io1 and io2 volumes are **more expensive** than general purpose volumes.

---

### 3. Throughput Optimized HDD (st1)

st1 volumes are **hard disk drive (HDD) based storage** designed for workloads that require **high throughput rather than low latency**.

These volumes are ideal for workloads that process **large amounts of sequential data**.

Examples include:

- big data processing
- log processing
- data warehousing
- streaming workloads

Unlike SSD volumes, HDD volumes are slower for random reads and writes but perform well for **large sequential operations**.

Because they use traditional hard disk technology, they are also **cheaper than SSD-based volumes**.

---

### 4. Cold HDD (sc1)

sc1 volumes are designed for **low-cost storage of infrequently accessed data**.

They are the cheapest EBS storage option but also provide the lowest performance.

These volumes are typically used for:

- archival storage
- infrequently accessed logs
- backup data
- long-term storage

Applications that need fast disk performance should avoid using sc1 volumes.

However, they are useful when the main goal is **reducing storage cost**.

---

### 5. Summary

AWS offers several EBS volume types to support different storage performance requirements.

| Volume Type | Storage Technology | Best Use Case |
|------|------|------|
| gp2 / gp3 | SSD | General purpose workloads |
| io1 / io2 | SSD | High-performance databases |
| st1 | HDD | Large sequential workloads |
| sc1 | HDD | Cold and infrequently accessed data |

Choosing the correct EBS volume type is important because it directly affects:

- storage performance
- application responsiveness
- infrastructure cost

In most modern workloads, **gp3 volumes are the default choice**, while specialized workloads may require provisioned IOPS or throughput optimized storage.
