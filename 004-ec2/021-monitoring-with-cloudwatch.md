## Monitoring with CloudWatch

When running EC2 instances in production, it is important to **monitor the health and performance of the servers**. Monitoring helps detect issues early, analyze system behavior, and ensure applications are running smoothly.

AWS provides a built-in monitoring service called **Amazon CloudWatch** for this purpose.

**Amazon CloudWatch** is a monitoring and observability service that collects **metrics, logs, and events** from AWS resources.

For EC2 instances, CloudWatch allows you to monitor important system metrics such as:

- CPU utilization
- disk activity
- network usage
- application-specific metrics

These metrics help administrators understand how the instance is performing and whether resources need to be adjusted.

---

### 1. CPU Metrics

CPU metrics show how much of the instance's **processing power is being used**.

The most common CPU metric provided by CloudWatch is **CPUUtilization**.

CPU utilization measures the percentage of CPU resources currently in use.

For example:

| CPU Utilization | Meaning |
|------|------|
| 10% | Instance is mostly idle |
| 50% | Moderate workload |
| 90%+ | Instance is under heavy load |

Monitoring CPU utilization helps determine whether an instance has sufficient processing capacity.

High CPU usage over long periods may indicate that the instance type is too small and needs to be upgraded.

Low CPU usage may suggest that the instance is oversized and resources are being wasted.

Common scenarios where CPU monitoring is important include:

- identifying performance bottlenecks
- scaling applications
- detecting abnormal workload spikes

---

### 2. Disk Metrics

Disk metrics track how the instance is interacting with its storage volumes.

For EC2 instances using EBS, CloudWatch provides metrics such as:

- **Read operations**
- **Write operations**
- **Disk throughput**
- **Disk latency**

These metrics help monitor how frequently the application reads or writes data to storage.

For example:

| Metric | Description |
|------|------|
| DiskReadOps | Number of disk read operations |
| DiskWriteOps | Number of disk write operations |
| DiskReadBytes | Amount of data read from disk |
| DiskWriteBytes | Amount of data written to disk |

Monitoring disk activity helps detect problems such as:

- slow database performance
- storage bottlenecks
- excessive disk usage

It can also help determine whether a different EBS volume type is needed for better performance.

---

### 3. Custom Metrics

CloudWatch also allows users to create **custom metrics**.

Custom metrics are metrics that are **defined and sent by the application or server itself** rather than automatically provided by AWS.

This allows organizations to monitor application-specific behavior.

Examples of custom metrics include:

- number of active users
- request latency
- queue length
- number of processed jobs
- application error rates

For example, a backend API might send a custom metric representing the **number of requests processed per minute**.

This data can then be visualized in CloudWatch dashboards and used to trigger alerts.

Custom metrics are especially useful for monitoring **application-level performance**, not just infrastructure-level metrics.

---

### 4. Summary

Amazon CloudWatch is the primary monitoring service used to observe the performance and health of EC2 instances.

It collects metrics that help administrators understand how resources are being used.

Key monitoring categories include:

| Metric Type | Purpose |
|------|------|
| CPU Metrics | Measure processor usage |
| Disk Metrics | Monitor storage activity and performance |
| Custom Metrics | Track application-specific behavior |

By analyzing these metrics, teams can detect issues early, optimize resource usage, and maintain reliable cloud infrastructure.
