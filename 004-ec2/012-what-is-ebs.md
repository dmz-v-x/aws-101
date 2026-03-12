## What is EBS?

When an EC2 instance runs, it needs storage to hold the **operating system, application files, logs, and data**. In AWS, the most commonly used storage system for EC2 instances is **Amazon EBS**.

**EBS stands for Elastic Block Store.**

It is a storage service that provides **persistent block-level storage volumes** for EC2 instances.

These storage volumes act like **virtual hard drives** that can be attached to EC2 instances.

In simple terms:

EBS is the **disk that your EC2 instance uses to store its data**.

For example, when you launch an EC2 instance with Linux, the operating system files are stored on an **EBS volume** attached to that instance.

---

### 1. Block Storage Concept

To understand EBS properly, it is important to understand the idea of **block storage**.

Storage systems are typically categorized into three types:

- Block storage
- File storage
- Object storage

EBS uses **block storage**.

Block storage divides storage into **fixed-sized blocks**, and each block is stored separately. The operating system manages how these blocks are organized into files.

From the perspective of the EC2 instance, the EBS volume behaves exactly like a **local disk drive**.

This means the instance can:

- Read and write files
- Create partitions
- Format the disk
- Install operating systems
- Store application data

For example, when you install software on your EC2 server, the files are stored in blocks on the EBS volume.

Block storage is particularly useful for:

- operating systems
- databases
- applications that require low latency storage
- transactional systems

Because the operating system manages the file system on top of the block storage, it offers high flexibility.

---

### 2. EBS vs Hard Disk

Although EBS behaves like a hard disk, it is important to understand that it is **not physically attached to the EC2 instance**.

Instead, EBS is a **network-attached storage system** provided by AWS.

However, from the perspective of the EC2 instance, it appears just like a normal disk.

The comparison below helps clarify the difference.

| Feature | EBS | Traditional Hard Disk |
|------|------|------|
| Location | Network-attached storage in AWS | Physically inside the computer |
| Persistence | Data persists even if instance stops | Data persists while disk exists |
| Scalability | Size can be increased easily | Limited by physical hardware |
| Snapshots | Can create snapshots for backups | Requires manual backup systems |
| Availability | Managed and replicated by AWS | Depends on local hardware reliability |

Because EBS is managed by AWS, it offers advantages such as:

- high durability
- easy backups through snapshots
- flexible resizing
- improved reliability

To the operating system, however, it still behaves like a normal disk.

---

### 3. AZ-Specific Nature of EBS

One important characteristic of EBS is that it is **Availability Zone (AZ) specific**.

This means an EBS volume exists within **one specific Availability Zone inside a region**.

For example:

If an EBS volume is created in **us-east-1a**, it can only be attached to EC2 instances that are also running in **us-east-1a**.

It cannot be directly attached to instances in another AZ such as **us-east-1b**.

This design helps ensure:

- low latency between the instance and the storage
- high performance
- reliable operation

However, it also means that moving data between Availability Zones requires additional steps.

A common approach is:

1. Create a **snapshot** of the EBS volume.
2. Use that snapshot to create a new volume in another Availability Zone.

Snapshots allow data to be safely copied and reused across zones or even across regions.

---

### 4. Summary

Amazon EBS is the primary storage system used by EC2 instances.

It provides **persistent block-level storage volumes** that function like virtual hard drives attached to EC2 servers.

Key characteristics of EBS include:

| Feature | Description |
|------|------|
| Storage type | Block storage |
| Purpose | Store operating system, applications, and data |
| Persistence | Data remains even if the instance stops |
| Attachment | Appears like a normal disk to the operating system |
| Availability | Volumes exist within a single Availability Zone |

Because of its reliability, flexibility, and performance, EBS is one of the most important components used when running applications on EC2.
