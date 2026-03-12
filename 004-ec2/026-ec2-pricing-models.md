## EC2 Pricing Models

When running EC2 instances, AWS provides several pricing options depending on how long and how predictably you plan to use the instances.

Different workloads have different usage patterns. Some applications run continuously for months, while others run only occasionally or for short periods. To support these different needs, AWS offers multiple **EC2 pricing models**.

The main EC2 pricing models include:

- On-Demand Instances
- Reserved Instances
- Savings Plans
- Spot Instances

Each pricing model offers a different balance between **flexibility, cost savings, and commitment**.

---

### 1. On-Demand Instances

**On-Demand instances** are the most flexible EC2 pricing option.

With this model, you pay for compute capacity **only for the time the instance is running**, without any long-term commitment.

Key characteristics:

- no upfront commitment
- pay per second or per hour
- launch or terminate instances anytime
- highest flexibility

Example:

If you launch an EC2 instance and run it for 10 minutes, you only pay for those 10 minutes.

On-Demand instances are commonly used for:

- development environments
- testing environments
- short-term workloads
- applications with unpredictable traffic

However, because of the flexibility they offer, On-Demand instances usually have **higher cost per hour** compared to other pricing models.

---

### 2. Reserved Instances

**Reserved Instances (RI)** provide a pricing discount in exchange for a **long-term commitment**.

With Reserved Instances, you commit to using a specific instance configuration for a period of:

- 1 year
- 3 years

In return, AWS provides significant cost savings compared to On-Demand pricing.

Reserved Instances can reduce costs by **up to about 70%** depending on the payment option.

There are different payment options available:

- No upfront payment
- Partial upfront payment
- All upfront payment

Reserved Instances are ideal for workloads that run **continuously and predictably**, such as:

- production web servers
- backend application servers
- databases
- long-running services

Because they require long-term commitment, they are best suited for stable workloads.

---

### 3. Savings Plans

**Savings Plans** are a newer pricing model introduced by AWS to provide cost savings with more flexibility than Reserved Instances.

Instead of committing to a specific instance type, you commit to spending a certain **amount per hour on compute usage**.

For example:

You may commit to spending **$10 per hour on compute services**.

As long as your usage stays within that commitment, AWS applies discounted pricing.

Savings Plans can provide savings similar to Reserved Instances, often **up to around 72%** compared to On-Demand pricing.

There are two main types of Savings Plans:

- **Compute Savings Plans** (most flexible)
- **EC2 Instance Savings Plans** (specific to instance family and region)

Savings Plans are commonly used for:

- long-running applications
- predictable workloads
- organizations with consistent cloud usage

They provide a good balance between **cost savings and flexibility**.

---

### 4. Spot Instances

**Spot Instances** allow you to use **unused AWS compute capacity at a significantly reduced price**.

These instances are often **70–90% cheaper** than On-Demand instances.

However, there is an important limitation:

AWS can **interrupt or terminate Spot Instances** when it needs the capacity back.

Before interruption, AWS usually provides a **2-minute warning**.

Because of this behavior, Spot Instances are suitable only for workloads that can tolerate interruptions.

Common use cases include:

- batch processing
- big data analytics
- machine learning training
- CI/CD pipelines
- distributed computing workloads

Spot Instances are extremely useful for reducing costs in large-scale compute workloads.

---

### 5. Summary

AWS offers multiple EC2 pricing models to support different workload patterns.

| Pricing Model | Key Feature | Best Use Case |
|------|------|------|
| On-Demand | Pay-as-you-go with no commitment | Short-term or unpredictable workloads |
| Reserved Instances | Long-term commitment with discounts | Stable production workloads |
| Savings Plans | Flexible commitment with discounted pricing | Consistent compute usage |
| Spot Instances | Extremely low cost but interruptible | Fault-tolerant batch workloads |

Choosing the correct pricing model helps organizations **optimize cloud costs while maintaining performance and flexibility**.
