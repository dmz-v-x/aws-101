## Database Patterns on Kubernetes

### 1. Introduction

Almost every real-world application requires a database. When running applications on Kubernetes (including Amazon EKS), engineers must decide **where the database should run**.

There are two common approaches:

Database running **inside Kubernetes** (StatefulSet + persistent storage)

Database running **outside Kubernetes using a managed service**, such as Amazon RDS

This architectural decision has major implications for:

Reliability  
Operational complexity  
Performance  
Scaling  
Cost  
Security  

Understanding the tradeoffs between these two approaches is critical when designing production systems.

---

### 2. Why Databases Are Different From Normal Applications

Most Kubernetes workloads are **stateless**.

Examples include:

Web APIs  
Frontend services  
Microservices  

If a pod crashes, Kubernetes simply starts another pod.

Databases behave differently because they store **critical persistent data**.

Example scenario:

Pod running PostgreSQL crashes  
↓

New pod starts  
↓

Database files are missing

This would cause data loss.

Because of this, databases require:

Persistent storage  
Backup systems  
Replication  
Consistency guarantees  

These requirements make databases harder to operate than stateless applications.

---

### 3. Two Database Deployment Models

There are two primary ways to run databases when using Kubernetes.

Model 1 — Database inside Kubernetes

Example:

PostgreSQL running in a StatefulSet with EBS storage.

Model 2 — Managed database outside Kubernetes

Example:

Amazon RDS PostgreSQL instance.

Applications running in Kubernetes connect to the database using a network endpoint.

Example connection string:

```
postgresql://db.example.amazonaws.com:5432
```

Each model has advantages and disadvantages.

---

### 4. Running Databases Inside Kubernetes

In this approach, the database runs as a **Kubernetes workload**.

Typical architecture:

StatefulSet  
↓

PersistentVolumeClaim  
↓

EBS / EFS storage

Example PostgreSQL deployment:

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
```

The StatefulSet ensures stable identity and persistent storage.

---

### 5. Storage for Kubernetes Databases

Databases running inside Kubernetes usually rely on block storage.

Example:

EBS volumes in EKS.

Storage architecture:

PostgreSQL Pod  
↓

PersistentVolumeClaim  
↓

EBS Volume

The EBS volume stores the database files.

If the pod restarts, it reconnects to the same volume.

---

### 6. Advantages of Running Databases in Kubernetes

One major advantage is **environment consistency**.

The database runs using the same infrastructure model as other applications.

Benefits include:

Unified deployment pipelines

Infrastructure defined in Kubernetes manifests

Portability across environments

This approach also allows full control over database configuration.

For example:

Custom tuning parameters  
Custom extensions  
Custom replication strategies

---

### 7. Kubernetes-Native Database Operators

Modern Kubernetes environments often use **database operators**.

Operators automate database management tasks.

Examples include:

Crunchy PostgreSQL Operator  
MongoDB Kubernetes Operator  
Percona MySQL Operator

Operators can handle tasks such as:

Replication  
Backups  
Failover  
Scaling

Example architecture:

Operator  
↓

StatefulSet database nodes  
↓

Persistent storage

---

### 8. Challenges of Running Databases in Kubernetes

Running databases in Kubernetes introduces operational complexity.

Teams must manage:

Backups  
Replication  
Failover  
Storage management  
Monitoring  
Security updates

Example scenario:

A node running the database fails.

Kubernetes restarts the pod on another node.

However the storage must also reattach correctly.

These operations require careful configuration.

---

### 9. Understanding Managed Databases (Amazon RDS)

Amazon RDS is a **managed relational database service**.

AWS handles infrastructure and operational tasks.

Supported database engines include:

PostgreSQL  
MySQL  
MariaDB  
Oracle  
SQL Server  
Aurora

In this model, the database runs outside Kubernetes but applications connect to it over the network.

---

### 10. Example RDS Architecture

Example architecture:

EKS cluster running applications  
↓

Applications connect to RDS endpoint

Example connection string:

```
postgres://app:password@rds-instance.amazonaws.com:5432/db
```

The database infrastructure is fully managed by AWS.

---

### 11. What AWS Manages in RDS

RDS automates many database operations.

These include:

Automatic backups

Point-in-time recovery

High availability

Patch management

Automatic failover

Monitoring integration

These features significantly reduce operational burden.

---

### 12. High Availability in RDS

RDS supports multi-availability zone deployments.

Architecture:

Primary database instance  
↓

Synchronous replica in another availability zone

If the primary instance fails, AWS automatically promotes the replica.

Failover typically happens within seconds.

Implementing this manually in Kubernetes is significantly more complex.

---

### 13. Scaling Databases in RDS

RDS provides two scaling mechanisms.

Vertical scaling

Increase instance size.

Example:

```
db.t3.medium → db.m6g.large
```

Read replicas

Create additional replicas for read-heavy workloads.

Applications send read queries to replicas.

This improves scalability.

---

### 14. Advantages of Managed Databases

Managed databases provide several benefits.

Reduced operational complexity

Infrastructure is managed by AWS.

Built-in backups

Automated snapshot creation.

High availability

Automatic failover across availability zones.

Security integration

IAM authentication and encryption.

Because of these features, managed databases are often preferred in production.

---

### 15. Limitations of Managed Databases

Managed databases also have limitations.

Less configuration control

Some database parameters cannot be modified.

Vendor lock-in

Applications become tightly coupled with AWS.

Higher cost for large workloads

Managed services include operational overhead.

These tradeoffs must be considered.

---

### 16. Performance Considerations

Database performance depends on storage and compute.

Kubernetes databases using EBS can provide excellent performance for certain workloads.

However RDS includes optimized infrastructure such as:

Dedicated storage subsystems

Automated tuning

Performance monitoring tools

For most workloads, RDS provides strong performance out of the box.

---

### 17. Security Differences

Kubernetes database security requires managing:

Secrets

TLS certificates

Network policies

Backup encryption

RDS integrates security features automatically.

Examples include:

Encryption at rest

IAM authentication

Automatic patching

This reduces security management overhead.

---

### 18. Disaster Recovery

Disaster recovery is complex for self-managed databases.

Tasks include:

Cross-region replication

Backup restoration

Failover orchestration

RDS provides built-in disaster recovery features such as:

Automated snapshots

Cross-region read replicas

Point-in-time recovery

These capabilities simplify recovery operations.

---

### 19. When to Run Databases Inside Kubernetes

Running databases inside Kubernetes may make sense when:

Full control over database configuration is required.

The environment must be portable across cloud providers.

Workloads require specialized database configurations.

Large-scale internal platforms operate their own database infrastructure.

---

### 20. When to Use Managed Databases

Managed databases are usually the better option when:

The team wants minimal operational overhead.

High availability must be guaranteed.

Security and compliance requirements are strict.

The team prefers focusing on application development instead of database operations.

For most production workloads, managed databases such as RDS are recommended.

---

### 21. Hybrid Architectures

Many organizations use a hybrid architecture.

Example:

EKS cluster runs application services.

RDS runs the primary production database.

Kubernetes hosts supporting services such as caching layers or analytics pipelines.

This approach balances operational simplicity with flexibility.

