## Autoscaling Amazon ECS Services — Fargate and ECS on EC2 with ALB, CloudWatch Metrics, and Custom Metrics

### 1. Introduction

Modern cloud applications often use **containers** instead of traditional virtual machines. Containers allow applications to run in isolated environments that are lightweight, portable, and easier to deploy.

AWS provides **Amazon Elastic Container Service (ECS)** to run and manage containerized applications.

However, container workloads experience changing demand just like traditional applications. A containerized service may need to scale when:

```
User traffic increases
API requests surge
Background job queues grow
Streaming workloads increase
```

To handle this dynamic demand, ECS services can automatically scale using **Application Auto Scaling**.

Autoscaling in ECS adjusts the number of running containers (tasks) based on metrics such as:

```
CPU utilization
Memory usage
Request count
Queue length
Custom application metrics
```

This enables containerized applications to automatically adapt to workload changes.

---

### 2. ECS Autoscaling Architecture Overview

Autoscaling in ECS involves several components working together.

Typical architecture:

```
Users
↓
Application Load Balancer (ALB)
↓
ECS Service
↓
ECS Tasks (containers)
↓
Compute infrastructure
    ├ ECS on EC2
    └ AWS Fargate
```

Autoscaling decisions are driven by monitoring metrics.

Monitoring pipeline:

```
Application metrics
↓
Amazon CloudWatch
↓
Scaling policy evaluation
↓
Application Auto Scaling
↓
ECS Service adjusts task count
```

---

### 3. ECS Tasks and Services

Before understanding autoscaling, it is important to understand ECS components.

---

#### ECS Task

A **task** is a running instance of a containerized application.

Example:

```
Docker container running a web server
Docker container running an API
Docker container running a worker process
```

A task is defined using a **task definition**, which specifies:

```
Container image
CPU and memory allocation
Networking configuration
Environment variables
Logging configuration
```

---

#### ECS Service

An **ECS service** ensures that a specified number of tasks are always running.

Example configuration:

```
Desired task count = 3
```

If a task crashes:

```
ECS automatically launches a replacement task
```

Autoscaling modifies the **desired task count** dynamically.

---

### 4. ECS Launch Types

ECS supports two launch types.

```
ECS on EC2
AWS Fargate
```

Both support autoscaling, but their infrastructure models differ.

---

### 5. ECS on EC2

In the ECS on EC2 model, containers run on EC2 instances managed by an ECS cluster.

Architecture:

```
ECS Cluster
↓
EC2 Instances
↓
ECS Tasks (containers)
```

Autoscaling occurs at two levels:

```
Task scaling → adjust number of containers
Instance scaling → adjust number of EC2 instances
```

Instance scaling is typically handled by **EC2 Auto Scaling Groups**.

---

### 6. AWS Fargate

AWS Fargate is a **serverless container platform**.

With Fargate:

```
No EC2 instances to manage
Containers run directly on AWS-managed infrastructure
```

Architecture:

```
ECS Service
↓
Fargate Tasks
↓
AWS-managed infrastructure
```

Autoscaling only adjusts the number of tasks.

Benefits include:

```
No server management
Simplified operations
Automatic infrastructure management
```

---

### 7. ECS Service Autoscaling

ECS service autoscaling automatically adjusts the number of running tasks in a service.

Example configuration:

```
Minimum tasks = 2
Desired tasks = 4
Maximum tasks = 20
```

If workload increases:

```
Auto Scaling increases task count
```

If workload decreases:

```
Auto Scaling reduces task count
```

Autoscaling decisions are based on metrics collected from CloudWatch.

---

### 8. Using Application Load Balancer with ECS

Application Load Balancers are commonly used with ECS services.

Architecture:

```
Users
↓
Application Load Balancer
↓
Target Group
↓
ECS Tasks
```

Each ECS task registers itself as a **target** in the target group.

Traffic is distributed across tasks.

This enables scaling based on request load.

---

### 9. Request-Based Scaling with ALB

One common autoscaling metric is:

```
RequestCountPerTarget
```

This metric measures how many requests each ECS task receives.

Example configuration:

```
Target requests per task = 100
```

Example scenario:

```
Total requests per second = 400
Target per task = 100
Required tasks = 4
```

If traffic increases:

```
Requests per task exceed target
Auto Scaling launches additional tasks
```

---

### 10. CPU-Based Autoscaling

CPU utilization is a commonly used autoscaling metric.

Example configuration:

```
Target CPU utilization = 60%
```

Scaling behavior:

```
CPU > 60% → scale out
CPU < 60% → scale in
```

Example:

```
CPU utilization rises to 80%
Auto Scaling launches additional tasks
```

More tasks distribute workload more evenly.

---

### 11. Memory-Based Autoscaling

Memory usage can also be used as a scaling metric.

Example configuration:

```
Target memory utilization = 70%
```

Scaling behavior:

```
Memory usage rises above 70%
Auto Scaling launches additional tasks
```

Memory-based scaling is useful for applications with heavy memory requirements.

---

### 12. CloudWatch Metrics for ECS Autoscaling

ECS autoscaling relies on metrics collected by **Amazon CloudWatch**.

Common ECS metrics include:

```
CPUUtilization
MemoryUtilization
RequestCountPerTarget
NetworkIn
NetworkOut
```

These metrics are monitored continuously.

If a metric crosses a threshold:

```
Scaling policy triggers
```

This automatically adjusts task capacity.

---

### 13. Custom Metrics for ECS Autoscaling

Sometimes built-in metrics do not accurately represent application workload.

Custom metrics allow scaling based on **application-specific signals**.

Examples include:

```
Number of messages in queue
Active user sessions
Orders per minute
API request latency
```

Applications can publish custom metrics to CloudWatch.

Autoscaling policies can then use these metrics.

---

### 14. Example Queue-Based Autoscaling

Consider a background worker system processing messages from an SQS queue.

Architecture:

```
Producers
↓
SQS Queue
↓
ECS Worker Tasks
```

Autoscaling metric:

```
ApproximateNumberOfMessagesVisible
```

Scaling rule:

```
50 messages per worker task
```

Example scenario:

```
Queue size = 500 messages
Required workers = 10 tasks
```

Autoscaling launches additional tasks automatically.

---

### 15. Target Tracking Autoscaling for ECS

The most common scaling policy for ECS services is **target tracking scaling**.

Example configuration:

```
Metric → CPUUtilization
Target value → 60%
Minimum tasks → 2
Maximum tasks → 20
```

Autoscaling behavior:

```
CPU rises above 60%
↓
New tasks launched
↓
CPU returns closer to target
```

This keeps container workloads balanced.

---

### 16. Scaling Workflow Example

Consider a microservice running in ECS.

Initial configuration:

```
Desired tasks = 3
```

Traffic spike occurs.

Scaling process:

```
User requests increase
↓
CPU utilization rises to 75%
↓
CloudWatch metric updated
↓
Target tracking policy triggers
↓
Auto Scaling launches additional tasks
↓
Load balancer distributes traffic
```

Eventually the system stabilizes.

---

### 17. Combining Task Scaling and Instance Scaling

For ECS on EC2 environments, two levels of scaling occur.

```
Task scaling → Application Auto Scaling
Instance scaling → EC2 Auto Scaling Group
```

Example process:

```
More ECS tasks needed
↓
Cluster capacity insufficient
↓
Auto Scaling launches additional EC2 instances
↓
Tasks scheduled on new instances
```

This ensures sufficient infrastructure capacity.

---

### 18. Benefits of ECS Autoscaling

Autoscaling ECS services provides several advantages.

---

#### Efficient Resource Usage

Containers scale based on real workload demand.

---

#### Improved Application Performance

New tasks are launched automatically during traffic spikes.

---

#### Cost Optimization

Unused capacity is reduced when demand decreases.

---

#### High Availability

Additional tasks improve system resilience.

---

### 19. Best Practices for ECS Autoscaling

To design reliable ECS autoscaling systems, several best practices should be followed.

---

#### Use Target Tracking Policies

Target tracking simplifies scaling configuration.

---

#### Choose Meaningful Metrics

Select metrics that represent actual workload demand.

---

#### Use ALB Request-Based Scaling

For web services, request count per target is often more accurate than CPU.

---

#### Monitor Scaling Activity

Use CloudWatch dashboards to observe scaling behavior.

---

### 20. Summary

ECS autoscaling automatically adjusts the number of running containers based on workload demand.

Autoscaling can operate with both ECS launch types:

```
ECS on EC2
AWS Fargate
```

Scaling decisions are driven by metrics collected in **Amazon CloudWatch**, including:

```
CPU utilization
Memory utilization
Request count per target
Custom application metrics
```

Application Load Balancers integrate with ECS services to distribute traffic across tasks.

By combining ECS autoscaling with monitoring and load balancing, organizations can build containerized applications that are:

```
Highly scalable
Cost efficient
Resilient
Automatically responsive to demand
```
