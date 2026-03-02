## Deployment vs StatefulSets vs DaemonSets

### 1. Introduction

Kubernetes provides multiple workload controllers that manage how pods are created, updated, and maintained in a cluster. The three most important workload controllers used in real systems are:

Deployments  
StatefulSets  
DaemonSets  

At first glance, they all appear similar because they all create and manage pods. However, they are designed for **very different types of workloads**.

Understanding when to use each controller is critical when building reliable Kubernetes systems, especially when running production workloads on platforms like Amazon EKS.

This guide explains these controllers from absolute basics to advanced production usage, including when and why StatefulSets are required.

---

### 2. The Core Problem Kubernetes Solves

Kubernetes is designed to manage **containers at scale**.

Instead of manually starting containers, Kubernetes ensures that the desired number of pods are always running.

Example requirement:

Run 3 instances of a web application.

Kubernetes ensures that:

If a pod crashes → a new pod is created  
If a node fails → pods are rescheduled  
If scaling is required → more pods are created

Workload controllers automate this process.

---

### 3. What is a Deployment

A Deployment is the most commonly used Kubernetes workload controller.

Deployments manage **stateless applications**.

Stateless applications are applications where:

Pods are interchangeable  
No unique identity is required  
No persistent storage is tied to a specific pod

Examples of stateless workloads include:

Web servers  
REST APIs  
Microservices  
Frontend applications

---

### 4. How Deployments Work

A Deployment manages pods using an underlying resource called a **ReplicaSet**.

Architecture:

Deployment  
↓

ReplicaSet  
↓

Pods

The Deployment ensures that the desired number of replicas are running.

Example Deployment configuration:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
```

This Deployment ensures that **3 identical pods** are always running.

---

### 5. Characteristics of Deployments

Deployments provide the following capabilities:

Stateless pods

Pods do not have unique identities.

Horizontal scaling

Replica count can be increased or decreased.

Rolling updates

Pods are updated gradually during application upgrades.

Self-healing

Failed pods are automatically recreated.

Because of these properties, Deployments are the default choice for most Kubernetes applications.

---

### 6. Limitations of Deployments

Deployments work well for stateless workloads but fail in certain scenarios.

Consider a distributed database cluster.

Example:

MySQL cluster  
Kafka cluster  
MongoDB cluster

Each node in the cluster must have:

Stable identity  
Persistent storage  
Ordered startup

Deployments cannot guarantee these requirements.

For such workloads, Kubernetes provides StatefulSets.

---

### 7. What is a StatefulSet

A StatefulSet is a Kubernetes workload controller designed for **stateful applications**.

Stateful applications require:

Stable network identity  
Stable storage  
Ordered deployment and scaling

StatefulSets provide these guarantees.

Example workloads:

Databases  
Kafka brokers  
Zookeeper clusters  
Elasticsearch nodes

---

### 8. How StatefulSets Work

Unlike Deployments, StatefulSets assign **unique identities to pods**.

Example StatefulSet with 3 replicas:

```
mysql-0
mysql-1
mysql-2
```

Each pod keeps the same name even after restarts.

This identity allows distributed systems to maintain cluster membership.

---

### 9. StatefulSet Storage Behavior

StatefulSets also integrate with persistent storage.

Each pod receives its own persistent volume.

Example:

mysql-0 → volume-mysql-0  
mysql-1 → volume-mysql-1  
mysql-2 → volume-mysql-2

Even if pods restart, they reconnect to the same storage volume.

This ensures that application data remains intact.

---

### 10. Example StatefulSet Configuration

Example StatefulSet.

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
```

Each replica automatically receives a unique storage volume.

---

### 11. Ordered Pod Creation

StatefulSets create pods sequentially.

Example:

First pod created:

```
mysql-0
```

Then:

```
mysql-1
```

Then:

```
mysql-2
```

This ordering is important for distributed systems where nodes must start in sequence.

---

### 12. Ordered Pod Deletion

StatefulSets also delete pods in reverse order.

Example:

First:

```
mysql-2
```

Then:

```
mysql-1
```

Then:

```
mysql-0
```

This prevents cluster disruption.

---

### 13. Stable Network Identity

StatefulSets provide predictable DNS names.

Example DNS:

```
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
mysql-2.mysql.default.svc.cluster.local
```

Applications can use these names for cluster communication.

---

### 14. What is a DaemonSet

DaemonSets are designed for a completely different purpose.

A DaemonSet ensures that **one pod runs on every node**.

Example use cases:

Log collectors  
Monitoring agents  
Networking plugins

Examples include:

Fluentd  
Prometheus node exporters  
AWS VPC CNI plugin

---

### 15. Example DaemonSet

Example configuration.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
      - name: fluentd
        image: fluentd
```

When a new node joins the cluster, a new pod is automatically created on that node.

---

### 16. Comparison of Controllers

Deployment

Stateless applications  
Pods interchangeable  
Rolling updates supported

StatefulSet

Stateful applications  
Pods have stable identities  
Ordered startup and shutdown

DaemonSet

Runs one pod per node  
Used for cluster-level services

Each controller solves a different operational problem.

---

### 17. When to Use Deployments

Use Deployments when:

Applications are stateless

Pods do not require persistent identity

Horizontal scaling is required

Example workloads:

Web servers  
REST APIs  
Frontend applications

---

### 18. When to Use StatefulSets

Use StatefulSets when:

Applications require persistent storage

Pods must maintain identity

Ordered startup and shutdown are required

Example workloads:

Databases

MySQL  
PostgreSQL

Streaming platforms

Kafka  
Pulsar

Search engines

Elasticsearch

---

### 19. When to Use DaemonSets

Use DaemonSets when:

A pod must run on every node.

Examples include:

Monitoring agents

Node-level logging

Networking plugins

Security scanners

---

### 20. Example Production Architecture

Example Kubernetes architecture.

Deployment

Frontend API service

StatefulSet

PostgreSQL database cluster

DaemonSet

Node-level monitoring agents

Each controller handles a different operational responsibility.

---

### 21. Common Mistakes

Running databases using Deployments

This can lead to data loss.

Using StatefulSets without persistent volumes

This defeats the purpose of stateful workloads.

Using DaemonSets for application workloads

DaemonSets are intended for node-level services.
