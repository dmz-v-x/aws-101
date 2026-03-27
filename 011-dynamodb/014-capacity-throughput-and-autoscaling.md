## Capacity, Throughput and AutoScaling

### 1. First Principle: DynamoDB Is Throughput-Based

DynamoDB does NOT think in terms of:
- Queries per second
- Rows per second
- Connections

DynamoDB thinks in:
- **Read Capacity Units (RCUs)**
- **Write Capacity Units (WCUs)**

Everything you do:
- Reads
- Writes
- Queries
- Index operations

Consumes capacity units.

If you understand RCUs and WCUs, you control DynamoDB.

---

### 2. Capacity Modes: Big Picture

DynamoDB offers **two capacity modes**:

1. **Provisioned capacity**
2. **On-Demand capacity**

This is a **business and engineering tradeoff**, not just a technical one.

---

### 3. Provisioned Capacity Mode 

Provisioned mode means:

> “I tell DynamoDB how much capacity I expect,  
> and DynamoDB reserves it for me.”

You explicitly set:
- Read capacity units (RCUs)
- Write capacity units (WCUs)

Characteristics:
- Predictable cost
- Cheaper at steady workloads
- Requires planning
- Can throttle if under-provisioned

Provisioned capacity is best for:
- Predictable traffic
- Stable workloads
- Cost-sensitive systems at scale

---

### 4. On-Demand Capacity Mode

On-Demand mode means:

> “I don’t want to think about capacity.  
> Handle whatever traffic comes.”

Characteristics:
- No capacity planning
- No throttling (within service limits)
- Higher cost per request
- Extremely simple

On-Demand is best for:
- New applications
- Spiky or unpredictable workloads
- Low operational overhead

Tradeoff:
> You pay for convenience.

---

### 5. Provisioned vs On-Demand 

Provisioned:
- Lower cost at scale
- Requires monitoring
- Can throttle
- Supports autoscaling

On-Demand:
- Higher cost
- No throttling under normal limits
- No autoscaling needed
- Minimal ops work

Rule of thumb:
- Start with On-Demand
- Move to Provisioned + Autoscaling once traffic stabilizes

---

### 6. What Is a Read Capacity Unit (RCU)?

An **RCU** represents:
- One strongly consistent read of up to **4 KB**
- OR two eventually consistent reads of up to **4 KB**

Important:
- Item size matters
- Consistency mode matters

This is where math starts.

---

### 7. RCU Calculation 

Example 1:  
Read 1 item, size = 3 KB, strong consistency

- Size ≤ 4 KB → 1 RCU
- Strong consistency → 1 RCU

Total = **1 RCU**

---

Example 2:  
Read 1 item, size = 6 KB, strong consistency

- 6 KB rounds up to 8 KB
- 8 KB / 4 KB = 2
- Strong consistency → 2 RCUs

Total = **2 RCUs**

---

Example 3:  
Read 1 item, size = 6 KB, eventual consistency

- 8 KB rounded
- Normally 2 RCUs
- Eventual consistency → divide by 2

Total = **1 RCU**

---

### 8. What Is a Write Capacity Unit (WCU)?

A **WCU** represents:
- One write of up to **1 KB**

Writes are always:
- Strongly consistent
- Fully replicated internally

Item size directly impacts WCUs.

---

### 9. WCU Calculation 

Example 1:  
Write item of size 700 bytes

- ≤ 1 KB → 1 WCU

---

Example 2:  
Write item of size 2.3 KB

- Rounded up to 3 KB
- 3 KB / 1 KB = 3 WCUs

---

Example 3:  
UpdateItem that changes a small attribute

Important rule:
- WCU is based on **full item size**, not changed fields

Even a small update to a large item can be expensive.

---

### 10. Capacity Is Per-Second, Not Per-Request

This is critical.

If you provision:
- 100 RCUs

That means:
- 100 RCUs per second

If you suddenly need:
- 300 RCUs in one second

Result:
- Throttling

Capacity planning is about **peaks**, not averages.

---

### 11. Burst Capacity 

DynamoDB allows **temporary bursts** above provisioned capacity.

How it works:
- DynamoDB uses unused capacity from the past
- Allows short spikes
- Burst is finite

Important:
- Burst is NOT guaranteed
- Burst is NOT a scaling strategy

Burst is a cushion, not a plan.

---

### 12. Adaptive Capacity

Adaptive capacity is DynamoDB’s internal mechanism to:
- Redistribute capacity across partitions
- Handle uneven access patterns

It helps when:
- Some partitions get more traffic
- Others are idle

But adaptive capacity:
- Cannot fix bad partition keys
- Cannot fix hot partitions completely

It reduces pain, not bad design.

---

### 13. Throttling: What It Really Means

Throttling happens when:
- Requested capacity > allowed capacity

Symptoms:
- ProvisionedThroughputExceeded errors
- Increased latency
- Failed requests

Throttling is NOT random.
It’s a signal.

---

### 14. Why Throttling Happens 

Common causes:
- Under-provisioned capacity
- Hot partitions
- Sudden traffic spikes
- Large item sizes
- Too many GSIs

Fixing throttling requires:
- Capacity tuning
- Key redesign
- Or autoscaling

---

### 15. Autoscaling 

Autoscaling allows DynamoDB to:
- Adjust RCUs and WCUs automatically
- Based on utilization

You configure:
- Minimum capacity
- Maximum capacity
- Target utilization (e.g., 70%)

DynamoDB then:
- Scales up when needed
- Scales down when traffic drops

---

### 16. Autoscaling Is Reactive 

Autoscaling:
- Reacts to traffic
- Is NOT instantaneous
- Has a delay

This means:
- Sudden spikes can still throttle
- Burst capacity helps temporarily

Autoscaling is not magic.
It’s a safety net.

---

### 17. Autoscaling Targets and Alarms

You typically configure:
- Target utilization (e.g., 70%)
- CloudWatch alarms behind the scenes

Best practice:
- Leave headroom
- Don’t target 100%
- 60–80% is common

This allows:
- Faster reaction
- Less throttling

---

### 18. Capacity and GSIs 

Every GSI:
- Has its own RCUs and WCUs
- Consumes capacity independently
- Multiplies write cost

When planning capacity:
- Always include GSIs
- Most surprises come from index writes

---

### 19. Cost Control Mental Model

You should now think:

- RCUs/WCUs are the currency
- Item size matters
- Indexes multiply cost
- Strong consistency costs more
- Provisioned = cheaper but planned
- On-Demand = expensive but easy
