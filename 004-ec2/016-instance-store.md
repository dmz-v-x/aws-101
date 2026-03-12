## Instance Store

When running EC2 instances, storage is usually provided through **Amazon EBS**, which is persistent and managed by AWS. However, some EC2 instance types also provide another type of storage called **Instance Store**.

Instance store is a **temporary storage system that is physically attached to the host machine running the EC2 instance**.

Because it is located on the underlying hardware, instance store provides **very fast disk performance**. However, it comes with an important limitation: **the data stored in instance store is temporary and can be lost under certain conditions**.

Understanding instance store is important because it is useful for specific workloads that require high performance but do not require long-term storage.

---

### 1. What Instance Store Is

An **instance store** is temporary block storage that is **directly attached to the physical server hosting the EC2 instance**.

Unlike EBS volumes, which are network-attached storage, instance store uses disks that are physically connected to the machine running the instance.

Because of this design, instance store provides:

- very low latency
- extremely fast read and write performance
- high throughput

However, instance store has a major difference from EBS: **it is not persistent storage**.

This means the storage only exists **for the lifetime of the EC2 instance**.

If the instance stops, terminates, or fails, the data stored in the instance store may be lost.

To the operating system inside the instance, instance store appears just like a **local disk drive**.

You can:

- create file systems
- write data
- store temporary files
- run applications using the disk

But it should only be used for **temporary data**.

---

### 2. When to Use Instance Store

Instance store is useful for workloads that require **high-speed temporary storage** but do not rely on the data being preserved permanently.

Some common use cases include:

#### Temporary Caches

Applications that store cached data that can be rebuilt if lost can use instance store for fast access.

---

#### Buffering and Intermediate Data

Some applications generate temporary intermediate data during processing.

For example:

- big data processing
- data transformation pipelines

Instance store can store intermediate results before final output is written to persistent storage.

---

#### High Performance Scratch Space

Certain workloads require extremely fast disk access for temporary computation.

Examples include:

- scientific simulations
- machine learning training
- large data processing jobs

Instance store can act as a **scratch disk** for these workloads.

---

#### Temporary Logs or Session Data

Some systems generate temporary logs or session data that do not need long-term persistence.

Instance store can be used to store this data temporarily.

---

### 3. Why Data Loss Happens

Data loss in instance store occurs because the storage is tied directly to the **physical host machine** running the EC2 instance.

If the instance moves away from that machine or the machine fails, the storage is no longer accessible.

Data loss can occur in several situations.

#### Instance Stop

When an EC2 instance with instance store is stopped, AWS may move the instance to a different host when it starts again.

Because the instance store was physically attached to the previous host, the data is lost.

---

#### Instance Termination

When an instance is terminated, the physical resources allocated to that instance are released.

This includes the instance store disks, which are wiped and reused.

---

#### Hardware Failure

If the physical host machine fails, the instance store data stored on that machine may be lost.

---

#### Instance Migration

In some situations, AWS may move the instance to another physical host for maintenance or infrastructure changes.

When this happens, instance store data cannot move with the instance.

---

### 4. Summary

Instance store provides **high-performance temporary storage directly attached to the EC2 host machine**.

It offers extremely fast disk access but does not provide persistence.

Key characteristics include:

| Feature | Description |
|------|------|
| Storage Type | Temporary block storage |
| Location | Physically attached to the EC2 host machine |
| Performance | Very high read/write speed |
| Persistence | Data is lost if the instance stops or terminates |
| Best Use Cases | Caching, temporary files, scratch space, intermediate processing |

Because of the risk of data loss, instance store should only be used for **temporary data that can be recreated if necessary**.
