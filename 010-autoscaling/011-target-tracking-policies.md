## Target Tracking Scaling Policies — How to Choose the Right Metric and Target Value

### 1. Introduction

Auto Scaling Groups adjust infrastructure capacity based on system demand. One of the most commonly used and recommended scaling mechanisms in AWS is **Target Tracking Scaling**.

Target tracking scaling automatically adjusts the number of instances in an Auto Scaling Group to maintain a **specific metric at a defined target value**.

This mechanism works similarly to a thermostat in a heating system.

Example analogy:

```
Thermostat target temperature → 22°C
Room temperature rises → cooling system activates
Room temperature drops → heating system activates
```

Similarly, Auto Scaling works to maintain a target value for a metric.

Example:

```
Target CPU utilization → 50%
```

If CPU rises above the target:

```
Auto Scaling launches more instances
```

If CPU drops below the target:

```
Auto Scaling removes instances
```

This allows infrastructure to automatically balance performance and cost.

---

### 2. What is a Target Tracking Scaling Policy?

A **Target Tracking Scaling Policy** automatically adjusts the capacity of an Auto Scaling Group to keep a chosen metric close to a specified target value.

Instead of defining specific scaling thresholds, target tracking continuously adjusts capacity to maintain equilibrium.

Example configuration:

```
Metric → CPUUtilization
Target → 50%
```

Auto Scaling automatically determines:

```
When to add instances
When to remove instances
How many instances to add or remove
```

This removes the need for complex manual scaling rules.

---

### 3. How Target Tracking Works

The system continuously monitors the chosen metric and compares it to the defined target value.

Workflow:

```
Metric collected from CloudWatch
↓
Target value comparison
↓
Deviation detected
↓
Scaling action triggered
↓
Auto Scaling adjusts instance count
```

Example process:

```
CPU utilization rises to 70%
Target value = 50%
↓
Auto Scaling launches more instances
↓
Traffic distributed across more servers
↓
CPU utilization drops closer to 50%
```

The system continually tries to keep the metric near the target.

---

### 4. Components of a Target Tracking Policy

A target tracking policy consists of several key components.

---

#### Metric

The metric represents the workload indicator used to trigger scaling decisions.

Examples include:

```
CPU utilization
Request count per target
Network throughput
Queue length
Custom application metrics
```

---

#### Target Value

The target value represents the desired level of the metric.

Example:

```
CPU utilization target = 50%
```

The system will scale instances to maintain this level.

---

#### Cooldown and Stabilization

Auto Scaling includes stabilization mechanisms to prevent rapid scaling oscillations.

These include:

```
Instance warm-up time
Scale-in stabilization windows
```

These mechanisms ensure scaling decisions remain stable.

---

### 5. Choosing the Right Metric

Choosing the correct metric is one of the most important decisions when designing a target tracking policy.

The chosen metric must accurately represent **real workload demand**.

A good scaling metric should have the following characteristics:

```
Directly related to system load
Responsive to traffic changes
Proportional to capacity usage
Stable and measurable
```

---

### 6. Common Metrics Used for Target Tracking

Several metrics are commonly used for scaling.

---

#### CPU Utilization

CPU utilization is one of the most frequently used metrics.

Example target:

```
CPU utilization = 50%
```

This works well for workloads where CPU usage directly reflects application demand.

Example workloads:

```
Compute-intensive applications
Data processing services
Machine learning inference
Video encoding workloads
```

---

#### Request Count per Target

Request count per target measures how many requests each instance receives.

Example metric:

```
ALB RequestCountPerTarget
```

Example target:

```
100 requests per instance
```

If request load increases:

```
Auto Scaling launches additional instances
```

This metric works well for:

```
Web applications
REST APIs
Microservices
```

---

#### Network Throughput

Network metrics can be used when traffic volume is the primary workload indicator.

Example metrics:

```
NetworkIn
NetworkOut
```

Example use cases:

```
File transfer services
Streaming platforms
Network proxies
```

---

#### Queue Length

Queue length is useful for background processing systems.

Example metric:

```
NumberOfMessagesVisible (SQS)
```

Example target:

```
50 messages per worker instance
```

If queue length grows:

```
Auto Scaling launches more workers
```

This approach works well for:

```
Message processing systems
Batch processing pipelines
Event-driven workloads
```

---

#### Custom Application Metrics

Sometimes built-in metrics do not reflect application demand accurately.

In these cases, custom metrics can be used.

Examples:

```
Active user sessions
Database query rate
Orders processed per minute
Cache miss rate
```

Custom metrics allow scaling based on **business-level signals**.

---

### 7. Choosing the Right Target Value

Choosing the right target value is just as important as choosing the correct metric.

If the target is too high:

```
Instances become overloaded
Performance degrades
User experience suffers
```

If the target is too low:

```
Too many instances run unnecessarily
Infrastructure cost increases
```

The target value must balance:

```
Performance
Cost efficiency
System stability
```

---

### 8. Typical Target Value Guidelines

Some commonly used target values include:

```
CPU utilization → 40–60%
Request count per target → depends on application capacity
Queue length → depends on worker processing speed
Network throughput → depends on instance network limits
```

These values vary depending on application characteristics.

---

### 9. Example Target Tracking Configuration

Consider a web application running in an Auto Scaling Group.

Configuration:

```
Metric → CPUUtilization
Target → 50%
Minimum capacity → 2
Maximum capacity → 10
```

Initial state:

```
4 instances
CPU utilization = 45%
```

System behavior:

```
No scaling needed
```

Traffic spike occurs.

CPU rises to:

```
75%
```

Scaling action:

```
Auto Scaling launches additional instances
```

Traffic spreads across instances and CPU drops toward the target.

---

### 10. Example Using Request-Based Scaling

For web applications behind an Application Load Balancer:

Configuration:

```
Metric → RequestCountPerTarget
Target → 200 requests per instance
```

Example flow:

```
Traffic increases
↓
Each instance receives more requests
↓
Metric exceeds target
↓
Auto Scaling launches additional instances
↓
Request load per instance decreases
```

This keeps request load balanced across servers.

---

### 11. Scale-Out vs Scale-In Behavior

Target tracking scaling treats scale-out and scale-in differently.

---

#### Scale-Out Behavior

Scale-out happens quickly to prevent performance degradation.

Example:

```
CPU rises above target
Auto Scaling launches instances rapidly
```

This ensures the system responds quickly to increased demand.

---

#### Scale-In Behavior

Scale-in occurs more cautiously.

Example:

```
CPU falls below target
Auto Scaling waits before removing instances
```

This prevents frequent scaling oscillations.

---

### 12. Instance Warm-Up Considerations

New instances require time to become fully operational.

Example tasks:

```
Application startup
Configuration loading
Dependency initialization
```

Instance warm-up time ensures that new instances are not immediately included in scaling calculations.

Example:

```
Instance warm-up = 120 seconds
```

During this period:

```
Auto Scaling ignores metrics from the new instance
```

This prevents premature scaling decisions.

---

### 13. Combining Target Tracking with Other Policies

Target tracking can be combined with other scaling strategies.

Example:

```
Target tracking → maintain CPU at 50%
Scheduled scaling → increase capacity during peak hours
```

This combination ensures the system is prepared for predictable traffic while still responding dynamically to unexpected demand.

---

### 14. Best Practices for Target Tracking Policies

To design effective target tracking policies, several best practices should be followed.

---

#### Use Metrics That Represent Real Load

Choose metrics that reflect true application demand.

---

#### Avoid Extremely High Target Values

High targets can overload instances before scaling occurs.

---

#### Monitor Scaling Behavior

Observe scaling activity and adjust targets as necessary.

---

#### Use Detailed Monitoring

1-minute metrics provide faster scaling reactions.

---

### 15. Real-World Example

Consider an e-commerce platform experiencing variable traffic.

Configuration:

```
Metric → RequestCountPerTarget
Target → 150 requests per instance
```

During a sale event:

```
Traffic increases dramatically
↓
Requests per instance increase
↓
Target tracking triggers scaling
↓
Auto Scaling launches additional instances
↓
Load distributed across servers
```

After the sale:

```
Traffic decreases
↓
Instances gradually terminate
↓
Infrastructure returns to baseline capacity
```

---

### 16. Summary

Target tracking scaling policies automatically adjust infrastructure capacity to maintain a chosen metric at a defined target value.

They simplify scaling configuration by automatically determining when and how to scale.

Key components include:

```
Metric → workload indicator
Target value → desired metric level
Scaling behavior → automatic capacity adjustments
```

Common metrics used in target tracking include:

```
CPU utilization
Request count per target
Network throughput
Queue length
Custom application metrics
```
