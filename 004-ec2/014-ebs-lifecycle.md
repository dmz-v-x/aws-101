## EBS Lifecycle

An **EBS volume** behaves like a virtual disk that can be attached to an EC2 instance. However, just like any storage device, it goes through different stages during its lifetime.

The sequence of actions that an EBS volume can go through is called the **EBS lifecycle**.

Understanding the EBS lifecycle is important because it helps you manage storage properly, avoid data loss, and control costs.

The main actions involved in the EBS lifecycle include:

- Attach
- Detach
- Delete
- Resize

Each of these actions controls how the storage volume interacts with EC2 instances.

---

### 1. Attach

Attaching a volume means **connecting the EBS volume to an EC2 instance** so that the instance can use it as a disk.

Once attached, the operating system inside the instance can detect the volume and use it like a normal disk.

After attaching a volume, you can:

- Format the disk
- Create a file system
- Mount it to a directory
- Store application data

For example, a common setup might be:

- One EBS volume for the operating system
- Another EBS volume for application data
- Another EBS volume for logs or databases

Important characteristics of attaching volumes:

- An EBS volume can only be attached to **instances in the same Availability Zone**
- Some volume types can only attach to **one instance at a time**
- The instance must be able to recognize the disk before it can be used

Attaching volumes allows you to expand storage for running servers.

---

### 2. Detach

Detaching a volume means **disconnecting the EBS volume from the EC2 instance**.

When a volume is detached:

- The instance no longer has access to the disk
- The data inside the volume remains intact
- The volume still exists in your AWS account

This is useful when you want to:

- move the volume to another instance
- perform maintenance
- create backups
- reuse storage

For example, if an EC2 instance fails, you can detach its EBS volume and attach it to a new instance to recover the data.

Before detaching a volume, it is usually recommended to:

- unmount the file system
- stop applications using the disk

This prevents data corruption.

---

### 3. Delete

Deleting a volume permanently removes the EBS volume from AWS.

Once deleted:

- all data stored in the volume is permanently lost
- the storage is removed from your account
- billing for that volume stops

Deletion usually happens when:

- the volume is no longer needed
- temporary storage was used
- infrastructure is being cleaned up

When an EC2 instance is terminated, the **root EBS volume may automatically be deleted**, depending on the configuration.

However, additional EBS volumes often remain unless manually deleted.

Because deletion is permanent, it is common to create a **snapshot backup before deleting a volume**.

---

### 4. Resize

One powerful feature of EBS is that volumes can be **resized without recreating them**.

Resizing allows you to increase:

- the storage size
- the IOPS performance (for some volume types)
- the throughput

For example, if a database server runs out of disk space, you can increase the EBS volume from **100 GB to 500 GB** without replacing the volume.

The process typically involves two steps:

1. Modify the EBS volume size in AWS.
2. Extend the file system inside the operating system.

After resizing, the instance can start using the additional storage.

This flexibility makes EBS much easier to manage compared to traditional physical disks.

---

### 5. Summary

The EBS lifecycle consists of several actions that manage how volumes are used with EC2 instances.

| Action | Description |
|------|------|
| Attach | Connects an EBS volume to an EC2 instance |
| Detach | Disconnects the volume while preserving data |
| Delete | Permanently removes the volume and its data |
| Resize | Increases storage capacity or performance |

By understanding these lifecycle actions, you can manage EC2 storage effectively while maintaining flexibility, reliability, and data safety.
