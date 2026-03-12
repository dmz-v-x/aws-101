## Mixed Instance Policies in AWS Auto Scaling — Using On-Demand, Spot Instances, and Instance Weighting

### 1. Introduction

Auto Scaling Groups traditionally launch **a single EC2 instance type** using a Launch Template or Launch Configuration.

Example configuration:

```
Instance type → m5.large
Auto Scaling Group launches only m5.large instances
```

Although this approach works, it has several limitations.

Possible problems include:

```
Instance type unavailable in a particular availability zone
Limited EC2 capacity during peak demand
Higher infrastructure cost
Reduced flexibility
```

To solve these problems, AWS introduced **Mixed Instance Policies**.

Mixed instance policies allow an Auto Scaling Group to launch **multiple instance types and purchase options simultaneously**.

This enables a combination of:

```
On-Demand instances
Spot instances
Multiple EC2 instance types
Instance weighting
```

This feature improves:

```
Cost efficiency
Capacity availability
Infrastructure resilience
```

---

### 2. What is a Mixed Instance Policy?

A **Mixed Instance Policy** allows an Auto Scaling Group to use **multiple EC2 instance types and purchase options** when launching instances.

Instead of relying on a single instance type, the Auto Scaling Group can select from a pool of instance types.

Example configuration:

```
Instance types:
m5.large
m5a.large
m5n.large
c5.large
```

The Auto Scaling Group chooses available instances dynamically.

Additionally, the policy allows mixing different purchasing models:

```
On-Demand instances
Spot instances
```

This provides flexibility and cost optimization.

---

### 3. Understanding EC2 Purchase Options

To understand mixed instance policies, it is important to understand the EC2 purchasing models.

AWS provides several purchasing options.

The two most relevant for mixed instance policies are:

```
On-Demand Instances
Spot Instances
```

---

### 4. On-Demand Instances

On-Demand instances are the **standard EC2 pricing model**.

Characteristics include:

```
Pay per second or per hour
No long-term commitment
Always available (subject to capacity)
Stable and reliable
```

Advantages:

```
Predictable performance
No interruptions
Guaranteed availability
```

Disadvantages:

```
Higher cost compared to other purchasing models
```

On-Demand instances are typically used for **baseline capacity**.

---

### 5. Spot Instances

Spot instances allow users to utilize **unused EC2 capacity at significantly reduced prices**.

Spot instance pricing can be:

```
Up to 90% cheaper than On-Demand instances
```

However, Spot instances come with an important limitation:

```
AWS can interrupt Spot instances when capacity is needed elsewhere
```

When this happens:

```
Instance receives a 2-minute interruption notice
Instance is terminated
```

Because of this behavior, Spot instances are best suited for workloads that can tolerate interruptions.

---

### 6. Combining On-Demand and Spot Instances

Mixed instance policies allow Auto Scaling Groups to use **both On-Demand and Spot instances simultaneously**.

Typical configuration example:

```
Baseline capacity → On-Demand instances
Additional capacity → Spot instances
```

Example:

```
Desired capacity = 10 instances

On-Demand instances = 4
Spot instances = 6
```

Benefits include:

```
Guaranteed baseline capacity
Cost savings through Spot instances
Improved scalability
```

If Spot instances become unavailable, the Auto Scaling Group can fall back to On-Demand instances.

---

### 7. Instance Type Diversification

Mixed instance policies also allow **multiple instance types** to be used.

Example configuration:

```
m5.large
m5a.large
m5n.large
c5.large
```

Benefits of instance diversification include:

```
Higher probability of finding available capacity
Improved resilience during EC2 capacity shortages
Better Spot instance availability
```

Instead of relying on a single instance type, Auto Scaling can choose from several options.

---

### 8. Instance Weighting

Instance weighting allows different instance types to represent different **capacity units**.

Not all EC2 instance types provide the same amount of compute power.

Example:

```
m5.large → 2 vCPUs
m5.xlarge → 4 vCPUs
m5.2xlarge → 8 vCPUs
```

If these instances were treated equally, capacity calculations would be inaccurate.

Instance weighting solves this problem.

---

### 9. How Instance Weighting Works

Each instance type is assigned a **weight value** representing its relative capacity.

Example configuration:

```
m5.large → weight = 1
m5.xlarge → weight = 2
m5.2xlarge → weight = 4
```

Auto Scaling calculates capacity based on the total weight instead of instance count.

Example target capacity:

```
Total capacity units = 10
```

Possible combinations:

```
10 m5.large instances
5 m5.xlarge instances
2 m5.2xlarge + 2 m5.large
```

This allows flexible scaling across different instance sizes.

---

### 10. Capacity Units vs Instance Count

Without weighting, scaling decisions are based on **number of instances**.

Example:

```
Desired capacity = 5 instances
```

With instance weighting, scaling decisions are based on **capacity units**.

Example:

```
Desired capacity = 10 capacity units
```

Auto Scaling can satisfy this capacity using different combinations of instances.

This provides more flexibility when launching instances.

---

### 11. Spot Allocation Strategies

When launching Spot instances, Auto Scaling must choose which instance types to use.

AWS provides several allocation strategies.

---

#### Lowest Price Strategy

Selects the Spot instance pool with the lowest price.

Advantages:

```
Maximum cost savings
```

Disadvantages:

```
Higher risk of interruption
```

---

#### Capacity Optimized Strategy

Selects instance pools with the highest available capacity.

Advantages:

```
Reduced risk of Spot interruptions
More stable infrastructure
```

This strategy is commonly recommended for production workloads.

---

#### Price-Capacity Optimized Strategy

Balances cost and capacity availability.

Advantages:

```
Cost efficiency
Improved reliability
```

---

### 12. Example Mixed Instance Configuration

Example Auto Scaling configuration:

```
Desired capacity = 10 capacity units

Instance types:
m5.large (weight 1)
m5.xlarge (weight 2)
m5.2xlarge (weight 4)

On-Demand base capacity = 4 units
Spot capacity = remaining units
Spot allocation strategy = capacity optimized
```

Possible outcome:

```
2 m5.large On-Demand instances
2 m5.xlarge On-Demand instances
2 m5.2xlarge Spot instances
```

Total capacity:

```
10 units
```

---

### 13. Real-World Example

Consider a large web application serving millions of users.

Baseline requirement:

```
4 instances always available
```

Traffic spikes require additional capacity.

Configuration:

```
Baseline capacity → On-Demand instances
Extra capacity → Spot instances
Instance types → multiple options
```

Example Auto Scaling behavior:

```
Normal traffic → 4 On-Demand instances
Traffic spike → 6 additional Spot instances launched
```

When traffic decreases:

```
Spot instances terminated
Baseline On-Demand instances remain
```

This approach significantly reduces infrastructure cost.

---

### 14. Benefits of Mixed Instance Policies

Mixed instance policies provide several advantages.

---

#### Cost Optimization

Spot instances dramatically reduce infrastructure cost.

---

#### Improved Capacity Availability

Using multiple instance types increases the chance of obtaining EC2 capacity.

---

#### Greater Flexibility

Auto Scaling can choose from many instance types.

---

#### Better Fault Tolerance

If one instance type becomes unavailable, other types can be used.

---

### 15. Limitations of Mixed Instance Policies

Despite the advantages, mixed instance policies also have some limitations.

---

#### Application Compatibility

Some applications require specific instance types.

Example:

```
GPU workloads
Memory-intensive workloads
```

These applications may not support multiple instance types.

---

#### Spot Instance Interruptions

Spot instances can terminate unexpectedly.

Applications must be designed to tolerate interruptions.

---

#### Increased Configuration Complexity

Mixed instance policies require careful configuration of:

```
Instance types
Weighting
Capacity distribution
Spot allocation strategies
```

---

### 16. Best Practices for Mixed Instance Policies

To use mixed instance policies effectively, several best practices should be followed.

---

#### Diversify Instance Types

Use multiple instance families to increase capacity availability.

Example:

```
m5
m5a
m5n
c5
```

---

#### Use Capacity-Optimized Spot Strategy

This reduces the risk of Spot interruptions.

---

#### Maintain Baseline Capacity with On-Demand Instances

Ensure critical capacity is always available.

---

#### Use Instance Weighting for Larger Instances

Weighting ensures accurate capacity calculations when using different instance sizes.

---

### 17. Summary

Mixed instance policies allow Auto Scaling Groups to use multiple EC2 instance types and purchasing models.

This approach improves cost efficiency and infrastructure resilience.

Key components of mixed instance policies include:

```
On-Demand instances → reliable baseline capacity
Spot instances → cost-efficient additional capacity
Instance diversification → multiple EC2 types
Instance weighting → flexible capacity units
```

By combining these techniques, organizations can build Auto Scaling systems that are:

```
Cost efficient
Highly available
Flexible
Scalable
```

Mixed instance policies are widely used in modern cloud architectures to optimize both performance and infrastructure cost.
