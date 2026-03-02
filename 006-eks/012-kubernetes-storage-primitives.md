## Kubernetes Storage Primitives

### 1. Introduction

Applications running inside Kubernetes often need to store data. Some workloads such as APIs or stateless microservices can operate without persistent storage. However, many real-world systems require durable data storage.

Examples include:

Databases  
Message queues  
File storage systems  
Machine learning pipelines  
Log processing systems  

Containers themselves are **ephemeral**. This means that if a pod crashes or is rescheduled to another node, the container filesystem is lost. Any data stored inside the container disappears.

To solve this problem Kubernetes provides **storage primitives** that allow persistent storage independent of pod lifecycle.

The most important primitives are:

PersistentVolume (PV)  
PersistentVolumeClaim (PVC)  
StorageClass  
Reclaim Policies  

Understanding how these components work together is essential for building reliable stateful applications in Kubernetes.

---

### 2. Why Persistent Storage Is Required in Kubernetes

Containers are designed to be disposable.

Example scenario:

Pod running MySQL  
↓

Pod crashes  
↓

Kubernetes recreates the pod  
↓

Database files disappear

This would cause catastrophic data loss.

To prevent this, Kubernetes separates **compute from storage**.

Pods use external storage volumes that survive pod restarts.

This is achieved using the PersistentVolume system.

---

### 3. Kubernetes Storage Architecture Overview

Kubernetes storage follows a layered architecture.

Infrastructure storage (cloud disks, network storage)  
↓

PersistentVolume (cluster storage resource)  
↓

PersistentVolumeClaim (application request for storage)  
↓

Pod mounts the storage volume

This abstraction allows applications to request storage without knowing the underlying infrastructure details.

---

### 4. What is a PersistentVolume (PV)

A **PersistentVolume** represents actual storage available in the Kubernetes cluster.

It is a cluster-level resource that describes storage capacity and configuration.

Examples of underlying storage types include:

AWS EBS volumes  
NFS shares  
Local disks  
Azure disks  
Google persistent disks  

A PersistentVolume contains information such as:

Storage capacity  
Access modes  
Storage backend  
Reclaim policy  

---

### 5. Example PersistentVolume

Example PV configuration.

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: database-pv
    spec:
      capacity:
        storage: 20Gi
      accessModes:
        - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      awsElasticBlockStore:
        volumeID: vol-123456
        fsType: ext4

This PV represents a 20GB storage volume backed by an AWS EBS disk.

---

### 6. Understanding Access Modes

PersistentVolumes define how storage can be accessed.

Common access modes include:

ReadWriteOnce (RWO)

Volume can be mounted by one node for read/write.

ReadOnlyMany (ROX)

Multiple nodes can read but not write.

ReadWriteMany (RWX)

Multiple nodes can read and write simultaneously.

Most cloud block storage systems support **ReadWriteOnce**.

---

### 7. What is a PersistentVolumeClaim (PVC)

A PersistentVolumeClaim is a request for storage made by an application.

Instead of directly referencing storage infrastructure, applications request storage using PVCs.

PVC defines:

Storage size required  
Access mode  
Storage class

Kubernetes then finds a matching PersistentVolume.

---

### 8. Example PersistentVolumeClaim

Example PVC.

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: database-storage
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi

This PVC requests 20GB of storage from the cluster.

---

### 9. PV and PVC Binding Process

The binding process works as follows.

PVC created  
↓

Kubernetes searches for matching PV  
↓

If capacity and access mode match  
↓

PVC binds to the PV

After binding, the volume becomes available to pods.

---

### 10. Using PVC in a Pod

Once the PVC exists, it can be mounted into a pod.

Example pod configuration.

    apiVersion: v1
    kind: Pod
    metadata:
      name: database
    spec:
      containers:
        - name: mysql
          image: mysql
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-storage
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: database-storage

The MySQL container stores data on the persistent volume.

Even if the pod restarts, the data remains.

---

### 11. Static vs Dynamic Provisioning

There are two ways PVs can be created.

Static provisioning

Administrator manually creates PersistentVolumes.

Dynamic provisioning

Kubernetes automatically creates storage volumes when a PVC is requested.

Dynamic provisioning is the most common approach in cloud environments.

---

### 12. What is a StorageClass

A StorageClass defines how storage should be provisioned dynamically.

It specifies:

Storage provisioner  
Volume type  
Performance characteristics  
Reclaim policy  

When a PVC references a StorageClass, Kubernetes automatically creates the required storage.

---

### 13. Example StorageClass

Example StorageClass for AWS EBS.

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: ebs-sc
    provisioner: ebs.csi.aws.com
    parameters:
      type: gp3
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer

This StorageClass provisions **EBS gp3 volumes**.

---

### 14. Using StorageClass in PVC

Example PVC using StorageClass.

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: app-storage
    spec:
      storageClassName: ebs-sc
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi

Kubernetes automatically creates an EBS volume.

---

### 15. Volume Binding Modes

StorageClasses define when volume binding occurs.

Immediate

Volume created immediately when PVC is created.

WaitForFirstConsumer

Volume created only after pod scheduling.

WaitForFirstConsumer is commonly used in cloud environments to ensure volumes are created in the correct availability zone.

---

### 16. Reclaim Policies

Reclaim policy defines what happens to storage after the PVC is deleted.

Common reclaim policies include:

Retain

Volume remains after PVC deletion.

Delete

Volume is deleted automatically.

Recycle (deprecated)

Volume contents are wiped and reused.

---

### 17. Example Reclaim Policy

Example PV configuration.

    persistentVolumeReclaimPolicy: Retain

If the PVC is deleted, the storage remains available for manual recovery.

Example:

Database PVC deleted accidentally  
↓

PV remains  
↓

Administrator can reattach storage

---

### 18. Hands-On Example: Deploying Stateful Application

Step 1: Create StorageClass.

    kubectl apply -f storageclass.yaml

Step 2: Create PVC.

    kubectl apply -f pvc.yaml

Step 3: Deploy application.

    kubectl apply -f deployment.yaml

Verify PVC.

    kubectl get pvc

Verify PV.

    kubectl get pv

---

### 19. Differences from Local Kubernetes

In local clusters like kind:

Storage often uses local host directories.

Example:

hostPath volumes

    /var/lib/kubernetes

This is suitable for development but not production.

In cloud environments like EKS:

Storage integrates with cloud infrastructure such as:

EBS  
EFS  
FSx

These provide durable storage across nodes.

---

### 20. Production Storage Best Practices

Use dynamic provisioning via StorageClasses.

Use appropriate storage types for workloads.

Example:

Database workloads → high IOPS storage

File sharing workloads → network file systems

Use Retain reclaim policy for critical data.

Monitor volume usage and performance.

---

### 21. Common Storage Gotchas

Volume cannot mount across availability zones.

Pod scheduling failures due to storage constraints.

Incorrect access mode causing pod startup failures.

Deleting PVC without understanding reclaim policy.

Understanding these issues is important when operating stateful workloads.

