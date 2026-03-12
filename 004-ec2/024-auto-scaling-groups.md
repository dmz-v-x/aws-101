## Auto Scaling Groups

When running applications on EC2, traffic and workload can change over time. Sometimes traffic increases suddenly, and sometimes it decreases. If your infrastructure cannot adapt to these changes, it may either become overloaded or waste resources.

To solve this problem, AWS provides **Auto Scaling Groups (ASG)**.

An **Auto Scaling Group** is a service that automatically **launches or terminates EC2 instances** based on demand.

It helps maintain the right number of EC2 instances to handle the current workload.

Auto Scaling Groups provide three main benefits:

- automatic scaling
- high availability
- cost optimization

For example, if traffic suddenly increases, the Auto Scaling Group can launch additional EC2 instances. If traffic decreases, it can terminate unnecessary instances.

---

### 1. Scaling Policies

**Scaling policies** define the rules that determine **when Auto Scaling should add or remove instances**.

These policies use metrics such as CPU utilization or network traffic to decide when scaling should occur.

There are several types of scaling policies.

#### Target Tracking Scaling

This is the most commonly used scaling policy.

You define a target metric, and AWS automatically adjusts the number of instances to maintain that target.

Example:

- Target CPU utilization = 50%

If CPU usage goes above 50%, new instances are launched.  
If CPU usage drops below 50%, instances may be removed.

This approach allows the system to automatically maintain a stable workload.

---

#### Step Scaling

Step scaling adjusts the number of instances based on predefined thresholds.

Example:

| CPU Usage | Action |
|------|------|
| Above 60% | Add 1 instance |
| Above 80% | Add 2 instances |
| Below 30% | Remove 1 instance |

This method gives more control over scaling behavior.

---

#### Scheduled Scaling

Scheduled scaling is used when traffic patterns are predictable.

For example, if a website receives higher traffic every day at 9 AM, scheduled scaling can increase the number of instances before that time.

Example schedule:

- 9 AM → increase capacity
- 10 PM → reduce capacity

---

### 2. Desired / Min / Max Capacity

An Auto Scaling Group is configured with three important capacity settings.

#### Desired Capacity

The **desired capacity** defines how many instances the Auto Scaling Group should maintain under normal conditions.

For example:

Desired capacity = 3

The Auto Scaling Group will try to keep exactly **three instances running**.

If one instance fails, a new one will automatically be launched.

---

#### Minimum Capacity

The **minimum capacity** defines the **lowest number of instances that must always remain running**.

Example:

Minimum capacity = 2

Even if traffic is very low, the Auto Scaling Group will never reduce the number of instances below two.

This ensures the application always has some available capacity.

---

#### Maximum Capacity

The **maximum capacity** defines the **highest number of instances the Auto Scaling Group can launch**.

Example:

Maximum capacity = 10

Even if traffic increases dramatically, the system will never create more than ten instances.

This protects against uncontrolled scaling and unexpected costs.

---

### 3. Health Checks

Auto Scaling Groups continuously monitor the **health of EC2 instances**.

If an instance becomes unhealthy, the Auto Scaling Group automatically replaces it.

Health checks can come from two sources.

#### EC2 Health Checks

These checks verify whether the EC2 instance itself is functioning properly.

Examples of failures include:

- instance crash
- hardware failure
- operating system issues

If the instance fails the health check, Auto Scaling terminates it and launches a replacement.

---

#### Load Balancer Health Checks

If the Auto Scaling Group is connected to a load balancer, the load balancer can also perform health checks.

These checks determine whether the **application running on the instance is responding properly**.

Example:

- load balancer sends HTTP requests to `/health`
- if the instance does not respond correctly, it is marked unhealthy

The Auto Scaling Group then replaces the instance automatically.

---

### 4. Summary

Auto Scaling Groups allow EC2 infrastructure to automatically adjust based on workload demand.

They ensure that applications always have the right number of instances running.

Key components of Auto Scaling include:

| Component | Purpose |
|------|------|
| Scaling Policies | Rules that determine when instances are added or removed |
| Desired Capacity | Number of instances the system tries to maintain |
| Minimum Capacity | Lowest number of instances allowed |
| Maximum Capacity | Highest number of instances allowed |
| Health Checks | Automatically detect and replace unhealthy instances |

Auto Scaling Groups are a critical component of **highly available and scalable AWS architectures** because they allow systems to automatically adapt to changing workloads.
