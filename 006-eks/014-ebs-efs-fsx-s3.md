## EBS, EFS, FSx for Lustre, and S3 – when to use which.

### 1. Introduction

Applications running in Kubernetes often need to store and retrieve data. Some applications require high-performance storage for databases, while others require shared storage for multiple pods, and some need object storage for storing large files or datasets.

In Amazon EKS, Kubernetes integrates with several AWS storage services. The most commonly used ones are:

EBS (Elastic Block Store)  
EFS (Elastic File System)  
FSx for Lustre  
S3 (Simple Storage Service)

Each of these services provides a different type of storage with different characteristics. Choosing the correct storage system is critical for performance, scalability, and cost efficiency.

---

### 2. Why Storage Matters in Kubernetes

Containers are ephemeral by design. When a pod stops or is rescheduled, the container filesystem is lost.

Example scenario:

Pod running a database  
↓

Pod crashes  
↓

New pod starts on another node  
↓

Database data disappears

To prevent this problem, Kubernetes allows pods to use external persistent storage.

In Amazon EKS, this persistent storage is typically provided by AWS services such as EBS, EFS, FSx, or S3.

---

### 3. Why Local Kubernetes (kind or minikube) Feels Simpler

When running Kubernetes locally using kind or minikube, storage often uses **hostPath volumes**.

Example:

    /var/lib/kubernetes/data

The container mounts a directory from the host machine.

This works because:

The cluster runs on a single machine  
Storage is local to the node  
No cloud infrastructure is involved

However, in EKS:

Pods run on distributed EC2 nodes  
Nodes can terminate or scale dynamically  
Applications require durable network storage

Therefore cloud storage systems are required.

---

### 4. Categories of Storage in AWS

AWS storage services generally fall into three categories.

Block Storage

Used like a disk attached to a machine.

Example:

EBS

File Storage

Shared filesystem accessible by multiple systems.

Example:

EFS  
FSx

Object Storage

Stores objects instead of files or blocks.

Example:

S3

Each category has different use cases.

---

### 5. Amazon EBS (Elastic Block Store)

Amazon EBS provides block-level storage volumes that attach to EC2 instances.

It behaves like a traditional hard drive.

Example:

An EC2 instance mounts an EBS volume at:

    /mnt/data

Applications write files to this disk as if it were a local drive.

In Kubernetes, EBS volumes are typically used with PersistentVolumes.

---

### 6. EBS Characteristics

Key properties of EBS include:

Block storage

High performance

Low latency

Single node attachment (in most cases)

Availability zone specific

Because EBS behaves like a disk attached to a single machine, it is ideal for workloads that require dedicated storage.

---

### 7. When to Use EBS

EBS is best suited for:

Databases

MySQL  
PostgreSQL  
MongoDB

Stateful applications

Kafka brokers  
Elasticsearch nodes

Single-writer workloads

Applications where one node writes data

Example scenario:

A MySQL database running in Kubernetes uses an EBS volume to store data files.

---

### 8. Example: Using EBS in Kubernetes

Step 1: Install EBS CSI driver.

    aws eks create-addon \
    --cluster-name my-cluster \
    --addon-name aws-ebs-csi-driver

Step 2: Create StorageClass.

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: ebs-sc
    provisioner: ebs.csi.aws.com
    parameters:
      type: gp3

Step 3: Create PVC.

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-storage
    spec:
      storageClassName: ebs-sc
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi

Kubernetes dynamically provisions an EBS volume.

---

### 9. Limitations of EBS

EBS volumes are limited to one availability zone.

Only one node can write to the volume at a time.

Because of these limitations, EBS is not suitable for shared storage across multiple nodes.

---

### 10. Amazon EFS (Elastic File System)

Amazon EFS provides a **managed network file system**.

It behaves like a shared filesystem that multiple machines can mount simultaneously.

Example mount point:

    /mnt/shared

Multiple pods across multiple nodes can access the same EFS filesystem.

---

### 11. EFS Characteristics

Key properties include:

Shared filesystem

Multi-node access

Automatic scaling

Regional availability

Unlike EBS, EFS is not limited to a single node.

This makes it suitable for distributed applications.

---

### 12. When to Use EFS

EFS is commonly used for:

Shared file storage

Media storage

Content management systems

Machine learning pipelines

Web applications storing uploaded files

Example scenario:

Multiple web servers writing uploaded files to a shared directory.

---

### 13. Example: Using EFS in Kubernetes

Install EFS CSI driver.

    helm repo add aws-efs-csi-driver \
    https://kubernetes-sigs.github.io/aws-efs-csi-driver/

Create PVC referencing EFS StorageClass.

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: efs-storage
    spec:
      accessModes:
        - ReadWriteMany
      storageClassName: efs-sc
      resources:
        requests:
          storage: 5Gi

Multiple pods can now mount the same volume.

---

### 14. Amazon FSx for Lustre

FSx for Lustre provides high-performance parallel file systems.

It is designed for:

High-performance computing  
Large-scale analytics  
Machine learning workloads

Lustre is commonly used in research and scientific computing.

---

### 15. FSx Characteristics

Extremely high throughput

Parallel file system architecture

Integration with S3

Optimized for large-scale compute workloads

FSx can process massive datasets at high speed.

---

### 16. When to Use FSx for Lustre

FSx is ideal for:

Machine learning training pipelines

Genomics workloads

Scientific simulations

High-performance analytics

Example scenario:

A machine learning training cluster processing terabytes of data.

---

### 17. Example Architecture

Data stored in S3  
↓

FSx for Lustre imports data

↓

Kubernetes compute cluster processes dataset

↓

Results exported back to S3

This architecture is commonly used for large-scale ML pipelines.

---

### 18. Amazon S3 (Simple Storage Service)

S3 is object storage rather than block or file storage.

Objects are stored inside buckets.

Example object path:

    s3://my-bucket/images/file.jpg

S3 is designed for massive scalability and durability.

---

### 19. S3 Characteristics

Virtually unlimited storage capacity

Highly durable

Accessible through APIs

Cost-effective for large datasets

Unlike EBS and EFS, S3 is not mounted like a filesystem by default.

Applications interact with S3 using APIs.

---

### 20. When to Use S3

S3 is ideal for:

Data lakes

Application assets

Backups

Logs

Machine learning datasets

Example scenario:

Application storing uploaded images in an S3 bucket.

---

### 21. Accessing S3 from Kubernetes

Pods typically access S3 using SDKs.

Example using AWS CLI inside container.

    aws s3 cp file.txt s3://my-bucket/

Applications can also use IAM roles for service accounts to access S3 securely.

---

### 22. Comparison of Storage Types

EBS

Block storage  
High performance  
Single node access

EFS

Shared file system  
Multiple nodes access

FSx

High-performance parallel filesystem  
Large compute workloads

S3

Object storage  
Massive scalability

Each storage system is designed for different workloads.

---

### 23. Real Production Architecture Example

Example EKS architecture.

EBS

Used for databases

EFS

Used for shared application files

S3

Used for backups and object storage

FSx

Used for machine learning data processing

Combining these services allows scalable architectures.

---

### 24. Common Storage Design Mistakes

Using EBS for shared storage

This fails because EBS supports only single-node writes.

Using S3 as a filesystem

S3 is object storage and behaves differently from filesystems.

Using EFS for high-performance databases

EFS latency is higher than EBS.

Understanding storage characteristics prevents performance issues.
