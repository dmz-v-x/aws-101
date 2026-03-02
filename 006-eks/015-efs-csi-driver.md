## EFS CSI Driver

### 1. Introduction

Many Kubernetes workloads require **shared storage** that can be accessed by multiple pods simultaneously.

Examples include:

Web applications storing uploaded files  
Content management systems  
Machine learning pipelines sharing datasets  
Multiple application replicas reading the same files  

In such cases, block storage systems like **Amazon EBS** are not suitable because they can usually be attached to only one node at a time.

To solve this problem, AWS provides **Amazon EFS (Elastic File System)**, which is a shared network filesystem. Kubernetes integrates with EFS through a component called the **EFS CSI driver**.

The EFS CSI driver allows Kubernetes pods to dynamically mount and use EFS storage as persistent volumes.

---

### 2. Recap: Why Kubernetes Needs Persistent Storage

Pods are ephemeral. When a pod restarts or is rescheduled, any data stored inside the container filesystem disappears.

Example:

Pod running a web application  
↓

User uploads files  
↓

Pod crashes  
↓

New pod starts

Uploaded files disappear.

Persistent storage solves this problem by storing data outside the pod lifecycle.

---

### 3. Why EBS Is Not Enough for Many Workloads

Amazon EBS is block storage attached to a single EC2 instance.

Example:

EC2 instance mounts EBS disk

    /mnt/data

EBS works well for:

Databases  
Single-node workloads

However, EBS has an important limitation.

A volume can usually only be mounted by **one node at a time**.

If multiple pods running on different nodes need access to the same data, EBS cannot provide that functionality.

This is where **shared file systems** become necessary.

---

### 4. What is Amazon EFS

Amazon EFS is a **fully managed network file system**.

It allows multiple EC2 instances or Kubernetes pods to mount the same filesystem simultaneously.

Example mount point:

    /mnt/shared

Multiple pods across multiple nodes can read and write files to this directory.

EFS behaves like a standard POSIX filesystem.

Applications interact with it using normal file operations such as:

read()  
write()  
open()  
delete()

---

### 5. EFS Architecture

EFS is designed as a **regional distributed filesystem**.

Architecture:

EKS Worker Nodes  
↓

EFS mount targets in each availability zone  
↓

Distributed storage backend

Mount targets allow EC2 instances in each availability zone to connect to the filesystem.

Because EFS is regional, it can be accessed by nodes across multiple zones.

---

### 6. Why We Need the EFS CSI Driver

Kubernetes does not know how to communicate with AWS EFS directly.

The **EFS CSI driver** acts as an integration layer between Kubernetes and EFS.

It performs tasks such as:

Mounting the EFS filesystem into pods  
Creating persistent volumes referencing EFS  
Managing access points for isolation  
Handling lifecycle operations

Without the CSI driver, Kubernetes cannot use EFS storage.

---

### 7. Why This Is Not Required in Local Kubernetes

In local Kubernetes clusters such as kind or minikube, storage usually comes from the host machine.

Example:

hostPath volume

    /Users/dev/data

Because everything runs on a single machine, Kubernetes does not need to communicate with external storage services.

In EKS:

Pods run on multiple EC2 instances  
Nodes may scale dynamically  
Applications need shared storage

Therefore integration with network file systems such as EFS becomes necessary.

---

### 8. Installing the EFS CSI Driver

The EFS CSI driver can be installed using Helm.

Add the Helm repository:

    helm repo add aws-efs-csi-driver \
    https://kubernetes-sigs.github.io/aws-efs-csi-driver/

Install the driver:

    helm install aws-efs-csi-driver \
    aws-efs-csi-driver/aws-efs-csi-driver \
    -n kube-system

Verify installation:

    kubectl get pods -n kube-system

You should see pods similar to:

    efs-csi-controller
    efs-csi-node

These components handle mounting EFS volumes to pods.

---

### 9. Creating an EFS File System

Before Kubernetes can use EFS, the filesystem must be created in AWS.

Example using AWS CLI:

    aws efs create-file-system \
    --performance-mode generalPurpose

This creates a filesystem ID such as:

    fs-12345678

Next create mount targets in each subnet.

    aws efs create-mount-target \
    --file-system-id fs-12345678 \
    --subnet-id subnet-abc123

Mount targets allow nodes to access the filesystem.

---

### 10. Creating a StorageClass for EFS

Next create a Kubernetes StorageClass that uses the EFS CSI driver.

Example:

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: efs-sc
    provisioner: efs.csi.aws.com
    parameters:
      provisioningMode: efs-ap
      fileSystemId: fs-12345678
      directoryPerms: "700"

This StorageClass allows dynamic provisioning using EFS access points.

---

### 11. Creating a PersistentVolumeClaim

Now applications can request storage using a PVC.

Example:

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: shared-storage
    spec:
      storageClassName: efs-sc
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 5Gi

Unlike EBS, EFS supports **ReadWriteMany**, meaning multiple pods can write simultaneously.

---

### 12. Using EFS in a Pod

Example pod configuration.

    apiVersion: v1
    kind: Pod
    metadata:
      name: web-app
    spec:
      containers:
      - name: app
        image: nginx
        volumeMounts:
        - mountPath: /data
          name: shared-storage
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: shared-storage

All pods mounting this PVC will share the same filesystem.

---

### 13. Testing Shared Storage

Create two pods using the same PVC.

Pod A writes a file.

    echo "hello from pod A" > /data/test.txt

Pod B reads the file.

    cat /data/test.txt

The file is accessible to both pods.

This demonstrates shared storage capability.

---

### 14. EFS Access Points

Access points provide **directory-level isolation** within an EFS filesystem.

Each application can have its own isolated directory.

Example:

    /app1
    /app2
    /analytics

Access points allow better security and multi-tenant usage.

The EFS CSI driver automatically creates access points when dynamic provisioning is enabled.

---

### 15. EFS Performance Modes

EFS offers two performance modes.

General Purpose

Lower latency  
Best for most workloads

Max I/O

Higher throughput  
Higher latency  
Used for large-scale parallel workloads

Most applications should use General Purpose mode.

---

### 16. EFS Throughput Modes

EFS also provides different throughput models.

Bursting throughput

Performance scales with storage size.

Provisioned throughput

Fixed throughput regardless of storage size.

Elastic throughput

Automatically adjusts based on workload demand.

Selecting the correct throughput mode affects both performance and cost.

---

### 17. Cost Considerations

EFS pricing depends on:

Storage capacity used  
Throughput mode  
Access frequency

Compared to EBS, EFS is generally more expensive per GB.

However it provides shared access across nodes.

Choosing between EBS and EFS depends on workload requirements.

---

### 18. When to Use EFS

EFS is ideal for:

Shared application data

Content management systems

Media processing pipelines

Multiple pods accessing the same files

Example scenario:

A web application with multiple replicas storing user uploads in a shared directory.

---

### 19. When Not to Use EFS

EFS is not ideal for:

High-performance databases

Low-latency workloads

Single-node applications

For these workloads, EBS provides better performance.

---

### 20. Real Production Example

Example architecture.

EKS cluster running multiple web application replicas.

Each pod mounts an EFS volume at:

    /uploads

Users upload files.

All replicas can access uploaded files because they share the same EFS filesystem.

---

### 21. Common Problems

Mount failures

Often caused by security group rules blocking NFS traffic.

Performance issues

Incorrect throughput configuration.

Access permission problems

Improper directory permissions.

Monitoring logs from the CSI driver can help diagnose issues.

---

### 22. Monitoring EFS

Useful commands include:

Check PVC status

    kubectl get pvc

Check PV

    kubectl get pv

Check CSI driver pods

    kubectl get pods -n kube-system

AWS CloudWatch also provides metrics for EFS usage.

---

### 23. EFS vs EBS Summary

EBS

Block storage  
Single node attachment  
High performance

EFS

Shared filesystem  
Multiple node access  
Higher latency

Choosing between them depends on workload characteristics.

---

### 24. Production Best Practices

Use EFS only when shared access is required.

Use access points for multi-tenant workloads.

Use General Purpose mode for most workloads.

Monitor throughput usage.

Avoid storing extremely high IOPS databases on EFS.

