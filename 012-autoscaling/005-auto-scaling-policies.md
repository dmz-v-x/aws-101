## AWS Auto Scaling Policies — Target Tracking, Step Scaling, Simple Scaling, and Scheduled Scaling

### 1. Introduction

Auto Scaling Groups can automatically adjust the number of EC2 instances running in response to changing application demand.

However, Auto Scaling does not randomly add or remove instances. Instead, it follows **rules called scaling policies**.

A **scaling policy** defines:

```
When scaling should happen
How many instances should be added or removed
What metric should trigger scaling
```

Without scaling policies, an Auto Scaling Group would simply maintain a fixed number of instances.

Scaling policies enable the system to react dynamically to real-world conditions such as:

- Increasing CPU usage
- Growing request traffic
- Rising queue length
- Predictable traffic patterns during specific times

AWS provides several types of scaling policies, each designed for different scenarios.

The four primary scaling policy types are:

```
Target Tracking Scaling
Step Scaling
Simple Scaling
Scheduled Scaling
```

Each of these approaches provides different levels of automation and control.

---

### 2. Why Scaling Policies Are Necessary

An Auto Scaling Group maintains **minimum, desired, and maximum capacity**, but it does not inherently know when to scale.

Scaling policies act as the **decision engine** that determines when scaling actions should occur.

Without scaling policies:

```
Auto Scaling Group = fixed number of instances
```

With scaling policies:

```
Auto Scaling Group = dynamic infrastructure that adapts to workload changes
```

Scaling policies rely on **metrics collected from monitoring systems**, typically from Amazon CloudWatch.

Example scaling workflow:

```
Application load increases
        ↓
CPU utilization rises
        ↓
CloudWatch detects metric threshold
        ↓
Scaling policy triggers
        ↓
Auto Scaling Group launches new instances
```

---

### 3. Types of Scaling Policies

AWS Auto Scaling supports four main types of scaling policies:

```
Target Tracking Scaling
Step Scaling
Simple Scaling
Scheduled Scaling
```

Each type solves a different scaling problem.

---

### 4. Target Tracking Scaling Policy

Target tracking is the **most commonly used and recommended scaling method**.

It works similarly to a **thermostat system**.

The system attempts to maintain a **target value for a specific metric**.

Example target:

```
Maintain CPU utilization at 50%
```

---

#### 4.1 How Target Tracking Works

Target tracking continuously monitors a metric.

If the metric deviates from the target value, Auto Scaling automatically adjusts capacity.

Example scenario:

```
Target CPU utilization = 50%
```

If CPU increases to:

```
70%
```

Auto Scaling launches additional instances.

If CPU decreases to:

```
30%
```

Auto Scaling removes unnecessary instances.

---

#### 4.2 Example Target Tracking Workflow

Example configuration:

```
Target metric = CPU utilization
Target value = 50%
Minimum capacity = 2
Maximum capacity = 10
```

Initial state:

```
4 instances running
CPU utilization = 45%
```

System behavior:

```
No scaling needed
```

Traffic increases.

CPU rises to:

```
65%
```

Scaling action:

```
Auto Scaling launches additional instances
```

Result:

```
CPU utilization drops closer to 50%
```

---

#### 4.3 Advantages of Target Tracking

Target tracking is popular because it provides:

```
Automatic scaling adjustments
Minimal configuration complexity
Continuous optimization
```

It automatically determines:

```
How many instances to add
When to remove instances
```

This makes it ideal for most workloads.

---

### 5. Step Scaling Policy

Step scaling provides **fine-grained control over scaling behavior**.

Instead of a single threshold, step scaling defines **multiple scaling steps based on metric ranges**.

This allows different scaling actions depending on how severe the load increase is.

---

#### 5.1 How Step Scaling Works

Step scaling policies define a series of thresholds.

Example configuration:

```
CPU > 60% → add 1 instance
CPU > 75% → add 2 instances
CPU > 90% → add 4 instances
```

As load increases, scaling becomes more aggressive.

---

#### 5.2 Example Step Scaling Scenario

Configuration:

```
Minimum = 2
Desired = 4
Maximum = 10
```

CPU increases to:

```
65%
```

Scaling action:

```
Add 1 instance
```

Later CPU increases to:

```
80%
```

Scaling action:

```
Add 2 instances
```

If CPU spikes to:

```
95%
```

Scaling action:

```
Add 4 instances
```

This allows scaling to **react proportionally to demand**.

---

#### 5.3 Advantages of Step Scaling

Step scaling offers:

```
More precise scaling control
Better response to sudden load spikes
Custom scaling strategies
```

However, it requires **careful tuning of thresholds**.

---

### 6. Simple Scaling Policy

Simple scaling is an older scaling approach.

It is based on **CloudWatch alarms triggering scaling actions**.

A simple scaling policy performs **one scaling action per alarm trigger**.

---

#### 6.1 How Simple Scaling Works

Example configuration:

```
If CPU > 70% → add 1 instance
If CPU < 30% → remove 1 instance
```

When the alarm triggers:

```
Auto Scaling performs the scaling action
```

After the scaling action completes, the system waits for a **cooldown period** before executing another scaling action.

---

#### 6.2 Cooldown Period

A cooldown period is a waiting time after a scaling action.

Example:

```
Cooldown period = 5 minutes
```

During this time:

```
No additional scaling actions occur
```

This prevents:

```
Rapid scaling loops
Frequent instance launches and terminations
```

---

#### 6.3 Limitations of Simple Scaling

Simple scaling has several limitations:

```
Slow response to load changes
Less flexible than modern scaling methods
Requires manual tuning
```

Because of these limitations, AWS recommends **target tracking or step scaling instead**.

---

### 7. Scheduled Scaling

Scheduled scaling adjusts infrastructure capacity based on **time-based schedules**.

Instead of reacting to metrics, scaling occurs at predefined times.

This is useful for workloads with **predictable traffic patterns**.

---

#### 7.1 How Scheduled Scaling Works

Example scenario:

A company runs a web application used primarily during business hours.

Traffic pattern:

```
High traffic → 9 AM to 6 PM
Low traffic → night hours
```

Scheduled scaling configuration:

```
9 AM → scale to 10 instances
6 PM → scale down to 3 instances
```

This ensures the infrastructure is prepared for daily traffic spikes.

---

#### 7.2 Scheduled Scaling Example

Example schedule:

```
Monday–Friday
09:00 → desired capacity = 10
18:00 → desired capacity = 3
```

During weekends:

```
Desired capacity = 2
```

This reduces costs during periods of low usage.

---

#### 7.3 Use Cases for Scheduled Scaling

Scheduled scaling is ideal for workloads with predictable patterns such as:

```
Business applications
Educational platforms
Daily batch processing systems
Corporate internal systems
```

---

### 8. Combining Multiple Scaling Policies

Auto Scaling Groups can use **multiple scaling policies simultaneously**.

Example configuration:

```
Target tracking → maintain CPU around 50%
Scheduled scaling → increase capacity during business hours
```

This combination provides both:

```
Predictive preparation
Dynamic response to unexpected load
```

---

### 9. Metrics Used in Scaling Policies

Scaling policies rely on metrics collected from monitoring systems.

Common metrics include:

```
CPU utilization
Network traffic
Request count
Application latency
Queue length
Memory usage
```

In AWS, these metrics are typically collected using:

```
Amazon CloudWatch
```

Custom application metrics can also be used.

---

### 10. Best Practices for Designing Scaling Policies

When designing scaling policies, consider several best practices.

---

#### Use Target Tracking When Possible

Target tracking automatically adjusts scaling and requires minimal tuning.

---

#### Choose Metrics Carefully

The chosen metric should represent **actual application load**.

Examples:

```
CPU for compute-heavy workloads
Queue length for worker systems
Request count for web applications
```

---

#### Avoid Overly Aggressive Scaling

Scaling too aggressively can lead to:

```
Cost spikes
Frequent instance creation
System instability
```

---

#### Use Cooldown or Stabilization Periods

Cooldown periods prevent scaling actions from triggering too frequently.

---

### 11. Example End-to-End Scaling Flow

A typical Auto Scaling process using scaling policies looks like this:

```
Application receives increased traffic
        ↓
CPU utilization rises
        ↓
CloudWatch records metric
        ↓
Scaling policy evaluates threshold
        ↓
Auto Scaling Group launches new instances
        ↓
Load balancer distributes traffic
```

When traffic drops:

```
Utilization decreases
        ↓
Scaling policy triggers scale-in
        ↓
Instances terminate
```

---

### 12. Summary

Scaling policies are the **decision-making mechanism of Auto Scaling**.

They determine when and how infrastructure should scale.

AWS provides four major scaling policy types:

```
Target Tracking Scaling
Step Scaling
Simple Scaling
Scheduled Scaling
```

Each policy type serves different use cases:

```
Target Tracking → automatic optimization
Step Scaling → granular control
Simple Scaling → basic alarm-driven scaling
Scheduled Scaling → time-based scaling
```
