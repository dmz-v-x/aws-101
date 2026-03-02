## EBS CSI Driver

### 1. Introduction

Modern applications running in Kubernetes often need persistent storage. Stateless applications can run without storing data, but many real-world systems such as databases, analytics pipelines, and message brokers require durable storage.

In Amazon EKS, persistent storage is typically provided through **Amazon Elastic Block Store (EBS)**. However, Kubernetes itself does not know how to communicate directly with AWS storage systems. A special integration layer is required to allow Kubernetes to provision and manage EBS volumes.

This integration layer is called the **EBS CSI driver**.

The EBS CSI driver allows Kubernetes to dynamically create, attach, mount, and delete Amazon EBS volumes when applications request storage using PersistentVolumeClaims.

Understanding how the EBS CSI driver works is essential when running stateful workloads on EKS.

---

### 2. Recap of Kubernetes Storage Workflow

Before discussing the EBS CSI driver, it is useful to review the Kubernetes storage workflow.

Application Pod  
↓

PersistentVolumeClaim (PVC)  
↓

StorageClass  
↓

PersistentVolume (PV)  
↓

Actual storage backend (EBS, NFS, etc.)

The application only interacts with the PVC. Kubernetes and the storage plugin handle the rest.

In EKS, the component that communicates with AWS to create EBS volumes is the **EBS CSI driver**.

---

### 3. What is CSI

CSI stands for **Container Storage Interface**.

CSI is a standardized interface that allows Kubernetes to communicate with storage providers.

Before CSI existed, Kubernetes used **in-tree storage plugins** built directly into the Kubernetes source code.

This approach caused several problems:

Storage plugins had to be maintained inside Kubernetes  
New storage systems required Kubernetes code changes  
Updates were difficult to manage

CSI solved this problem by moving storage integrations outside Kubernetes.

Now storage vendors provide **CSI drivers** that run as Kubernetes components.

---

### 4. What the EBS CSI Driver Does

The EBS CSI driver acts as a bridge between Kubernetes and AWS.

When Kubernetes needs storage, the CSI driver communicates with the AWS API to perform operations.

Examples of operations include:

Creating EBS volumes  
Attaching volumes to EC2 instances  
Mounting volumes inside containers  
Resizing volumes  
Deleting unused volumes

Without the CSI driver, Kubernetes would have no way to interact with AWS EBS storage.

---

### 5. Why This Is Not Visible in Local Clusters

In local clusters such as kind or minikube, storage is usually implemented using **hostPath volumes**.

Example:

A directory on your laptop:

```
/var/lib/data
```

The container mounts this directory directly.

Because the cluster runs on a single machine, Kubernetes does not need to communicate with external storage systems.

In EKS, however:

Pods run on EC2 instances  
Nodes can be replaced or rescheduled  
Applications require durable cloud storage

Therefore Kubernetes must integrate with cloud storage services such as EBS.

This integration is provided by the EBS CSI driver.

---

### 6. Architecture of the EBS CSI Driver

The EBS CSI driver runs inside the Kubernetes cluster as several pods.

Major components include:

Controller components

Responsible for provisioning and deleting EBS volumes.

Node components

Responsible for attaching and mounting volumes on worker nodes.

The driver runs inside the kube-system namespace.

You can inspect it using:

```
kubectl get pods -n kube-system
```

---

### 7. Installing the EBS CSI Driver on EKS

EKS provides the EBS CSI driver as a **managed add-on**.

First verify the cluster name.

```
aws eks list-clusters
```

Install the add-on:

```
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name aws-ebs-csi-driver
```

Verify installation:

```
kubectl get pods -n kube-system
```

You should see pods similar to:

```
ebs-csi-controller
ebs-csi-node
```

These components manage storage operations.

---

### 8. IAM Permissions for the CSI Driver

The EBS CSI driver needs permission to create and manage EBS volumes.

This is done using **IAM roles for service accounts (IRSA)**.

The driver must have permissions such as:

CreateVolume  
AttachVolume  
DeleteVolume  
DescribeVolumes

Example AWS managed policy:

```
AmazonEBSCSIDriverPolicy
```

This policy allows the driver to interact with the EBS API.

---

### 9. Creating a StorageClass for EBS

A StorageClass defines how storage should be created dynamically.

Example StorageClass for EBS:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Important fields include:

provisioner

Specifies which CSI driver to use.

parameters

Defines storage characteristics such as volume type.

reclaimPolicy

Determines what happens to the volume after deletion.

---

### 10. Creating a PersistentVolumeClaim

Now create a PVC that uses the StorageClass.

Example:

```
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
```

Apply the configuration:

```
kubectl apply -f pvc.yaml
```

Kubernetes automatically creates an EBS volume.

---

### 11. Verifying Dynamic Volume Provisioning

Check the PVC status.

```
kubectl get pvc
```

Example output:

```
NAME          STATUS   VOLUME
app-storage   Bound    pvc-abc123
```

Check the corresponding PersistentVolume.

```
kubectl get pv
```

In AWS console, you will also see a new EBS volume created.

---

### 12. Mounting the Volume in a Pod

Now the PVC can be used in a pod.

Example pod configuration:

```
apiVersion: v1
kind: Pod
metadata:
  name: storage-test
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: app-storage
```

The container now stores files in the EBS volume.

---

### 13. Testing Persistent Storage

Exec into the container.

```
kubectl exec -it storage-test -- bash
```

Create a file:

```
echo hello > /data/test.txt
```

Delete the pod:

```
kubectl delete pod storage-test
```

Create the pod again.

The file will still exist because it is stored in the persistent EBS volume.

---

### 14. Resizing EBS Volumes

The EBS CSI driver supports volume expansion.

Example:

Update PVC size:

```
resources:
  requests:
    storage: 20Gi
```

Apply the update.

```
kubectl apply -f pvc.yaml
```

Kubernetes automatically resizes the EBS volume.

---

### 15. Volume Binding Modes

Two binding modes exist.

Immediate

Volume is created immediately when the PVC is created.

WaitForFirstConsumer

Volume is created when a pod is scheduled.

WaitForFirstConsumer ensures the volume is created in the correct availability zone.

This is important because EBS volumes are **zone specific**.

---

### 16. Common EBS CSI Limitations

EBS volumes can only be attached to one EC2 instance at a time.

This means access mode is typically:

ReadWriteOnce

Applications requiring shared file access must use network storage such as EFS.

Another limitation is availability zone constraints.

Pods using EBS must run in the same zone as the volume.

---

### 17. Monitoring Storage

Useful commands include:

Check PVC status

```
kubectl get pvc
```

Check PV details

```
kubectl describe pv
```

Check CSI driver pods

```
kubectl get pods -n kube-system
```

These commands help diagnose storage problems.

---

### 18. Production Best Practices

Use gp3 EBS volumes for most workloads.

Use WaitForFirstConsumer binding mode.

Separate storage classes for different performance needs.

Monitor volume usage and IOPS.

Use Retain reclaim policy for critical data such as databases.

---

### 19. Common Problems

Pod stuck in Pending

This often indicates volume provisioning issues.

Volume cannot attach

Occurs if the pod is scheduled to a different availability zone.

Permission errors

Usually caused by missing IAM permissions for the CSI driver.

Understanding these issues is essential when operating stateful workloads.

