## EBS Snapshots

When working with storage systems, it is important to have reliable ways to **back up data and recover from failures**. In AWS, this is achieved using **EBS Snapshots**.

An **EBS snapshot** is a backup of an EBS volume stored in **Amazon S3**.

Snapshots allow you to:

- back up data
- restore volumes
- copy storage across regions
- create new volumes from backups

Snapshots are an essential part of managing EC2 storage because they provide a safe and efficient way to protect and replicate data.

---

### 1. What Snapshots Are

An **EBS snapshot** is a **point-in-time backup of an EBS volume**.

It captures the current state of the data stored on the volume.

For example, suppose you have an EBS volume attached to an EC2 instance containing:

- operating system files
- application code
- database data

When you create a snapshot, AWS records the state of that volume at that exact moment.

This snapshot can later be used to:

- restore the volume
- create a new EBS volume
- create an AMI
- recover from data loss

Snapshots are stored in **Amazon S3**, which provides high durability and reliability.

Because of this, snapshots are often used as part of **backup and disaster recovery strategies**.

---

### 2. Incremental Backups

One important feature of EBS snapshots is that they are **incremental**.

This means AWS does not copy the entire volume every time you create a snapshot.

Instead, AWS only saves the **blocks of data that have changed since the last snapshot**.

The process works like this:

1. The first snapshot of a volume copies **all data blocks**.
2. The next snapshot copies **only the blocks that changed** since the previous snapshot.
3. Each additional snapshot continues saving only new changes.

This approach has several advantages:

- reduces storage usage
- lowers backup costs
- speeds up snapshot creation
- improves efficiency

For example:

If a 100 GB volume changes only 5 GB of data between snapshots, the next snapshot stores only that **5 GB of new data**.

Even though snapshots are incremental, AWS ensures that **each snapshot can be used independently to restore the volume**.

---

### 3. Cross-Region Snapshots

Another powerful feature of EBS snapshots is the ability to **copy snapshots across AWS regions**.

Normally, an EBS volume exists in a specific **Availability Zone inside a region**.

However, snapshots can be copied to another region to create backups in a completely different geographic location.

This is called **cross-region snapshot replication**.

The process typically works like this:

1. Create a snapshot of an EBS volume.
2. Copy that snapshot to another AWS region.
3. Use the copied snapshot to create a new EBS volume in the destination region.

This feature is commonly used for:

- disaster recovery
- geographic redundancy
- migration of workloads
- compliance requirements

For example, an organization might store production data in **us-east-1** and maintain backup snapshots in **eu-west-1**.

If a regional failure occurs, the system can be restored from the snapshot in another region.

---

### 4. Summary

EBS snapshots provide a reliable method for backing up and restoring EC2 storage.

They capture the state of an EBS volume at a specific moment and store that backup safely in Amazon S3.

Key characteristics of EBS snapshots include:

| Feature | Description |
|------|------|
| Purpose | Backup of EBS volumes |
| Storage Location | Stored in Amazon S3 |
| Backup Type | Incremental backups |
| Recovery | Can restore new EBS volumes |
| Cross-Region Support | Snapshots can be copied to other AWS regions |

Snapshots are widely used for **backup, disaster recovery, infrastructure replication, and AMI creation**, making them a fundamental component of EC2 storage management.
