## On-Demand Instances

When using EC2, the most straightforward pricing model is **On-Demand Instances**.

An **On-Demand Instance** is an EC2 instance where you **pay for compute capacity by the second (or hour depending on the instance)** with **no long-term commitment**.

This means you can:

- Launch an instance whenever you need it
- Use it for as long as you want
- Stop or terminate it whenever you want
- Pay only for the time the instance is running

Because of this flexibility, On-Demand instances are the **default and most commonly used EC2 pricing model**.

---

### 1. How On-Demand Instances Work

With On-Demand pricing, AWS charges you based on **how long the instance runs**.

The process typically works like this:

1. You launch an EC2 instance.
2. The instance starts running.
3. AWS begins charging for compute usage.
4. When you stop or terminate the instance, compute charges stop.

Billing is typically calculated **per second with a minimum of 60 seconds** for most Linux instances.

For example:

If you run an instance for **10 minutes**, you only pay for those 10 minutes of compute usage.

This pricing model makes it very easy to start using EC2 without worrying about long-term contracts.

---

### 2. Key Characteristics of On-Demand Instances

On-Demand instances offer several important benefits.

#### No Long-Term Commitment

You are not required to reserve instances for months or years.

You can start and stop instances whenever you want.

---

#### Full Flexibility

You can:

- Launch instances immediately
- Change instance types
- Terminate instances at any time

This makes On-Demand instances ideal for dynamic environments.

---

#### Predictable Availability

Unlike Spot Instances, On-Demand instances are **not interrupted by AWS reclaiming capacity**.

Once launched, the instance will continue running until you stop or terminate it.

---

#### Simple Pricing

Pricing is straightforward because you only pay for the time the instance is running.

There are no complex bidding systems or capacity risks.

---

### 3. When to Use On-Demand Instances

On-Demand instances are best suited for workloads that require **reliability, flexibility, and immediate availability**.

Common use cases include:

#### Development and Testing

Developers often launch instances temporarily for testing applications and then terminate them after use.

---

#### Short-Term Projects

Applications that run for limited durations benefit from the flexibility of On-Demand pricing.

---

#### Applications with Unpredictable Traffic

When traffic patterns are difficult to predict, On-Demand instances provide flexibility without long-term commitments.

---

#### Initial Deployment of Applications

When starting a new application, teams often use On-Demand instances before optimizing costs with other pricing models.

---

### 4. When On-Demand May Not Be Ideal

Although On-Demand instances provide flexibility, they are **the most expensive pricing model** compared to other options.

In long-running environments, other pricing models may be more cost-efficient.

Examples include:

#### Long-Term Stable Workloads

Applications that run continuously for months or years can benefit from **Reserved Instances or Savings Plans**, which offer lower pricing.

---

#### Large Compute Workloads

Workloads such as batch processing or big data analytics can often use **Spot Instances** to reduce costs significantly.

---

### 5. On-Demand vs Other Pricing Models

To better understand On-Demand instances, it helps to compare them with other EC2 pricing models.

| Pricing Model | Key Characteristic |
|------|------|
| On-Demand | Pay for compute as you use it with no commitment |
| Spot | Use spare AWS capacity at a lower price but with interruption risk |
| Reserved Instances | Commit to long-term usage for discounted pricing |
| Savings Plans | Flexible long-term pricing discount |

On-Demand instances provide the **highest flexibility but also the highest cost per hour**.

---

### 6. Summary

On-Demand instances are the simplest way to run EC2 workloads.

They allow users to launch virtual servers whenever needed and pay only for the compute time used.

Key characteristics include:

| Feature | Description |
|------|------|
| Pricing | Pay-as-you-go |
| Commitment | No long-term commitment |
| Availability | Instances run continuously unless stopped |
| Flexibility | Launch and terminate anytime |
| Best for | Development, testing, short-term workloads |

On-Demand instances are often the **starting point for most EC2 users** because they provide maximum flexibility and predictable behavior without requiring long-term planning.
