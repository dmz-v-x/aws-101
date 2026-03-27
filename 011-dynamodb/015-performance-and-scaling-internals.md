### 1. First Principle: DynamoDB Scales by PARTITIONS

DynamoDB does not scale by:
- Tables
- Indexes
- Regions

It scales by **partitions**.

A **partition** is:
- A chunk of storage
- A chunk of throughput
- Managed entirely by AWS

You never create partitions manually.  
But **your key design decides how many partitions are used**.

---

### 2. What Is a DynamoDB Partition? (Conceptual, Not Physical)

A partition represents:
- A fixed amount of storage
- A fixed amount of throughput

At a high level, a partition can support:
- A limited number of RCUs
- A limited number of WCUs
- A limited amount of data

When limits are reached:
- DynamoDB creates more partitions
- Data is redistributed internally

This is automatic — but not magic.

---

### 3. How DynamoDB Decides Partition Count

Partitions are created based on:
- Total table size
- Total provisioned throughput
- Index requirements

More data or more throughput:
- More partitions

But here is the key insight:

**More partitions only help if traffic is evenly distributed.**

---

### 4. Partition Key → Traffic Distribution (Critical)

DynamoDB uses:
- The partition key
- A hashing algorithm

To decide:
- Which partition handles a request

If many requests share the same partition key:
- They go to the same partition
- No matter how big your table is

This is the root of most performance problems.

---

### 5. What Is a Hot Key? (Root Cause Explained)

A **hot key** is a partition key value that:
- Receives disproportionate traffic
- Exceeds partition throughput limits

Examples of hot keys:
- status = "ACTIVE"
- date = "2024-01-01"
- region = "US"
- category = "SPORTS"

These look harmless.
They are deadly at scale.

---

### 6. Hot Partition vs Hot Key (Important Distinction)

Hot key:
- One key value gets too much traffic

Hot partition:
- One physical partition gets overloaded
- Often caused by hot keys

You fix:
- Hot keys with design
- Hot partitions with distribution + design

Monitoring alone does not fix this.

---

### 7. How to Design Partition Keys to Avoid Hot Keys

Good partition keys have:
- High cardinality
- Even access distribution
- Natural ownership boundaries

Good examples:
- userId
- accountId
- orderId
- deviceId

Bad examples:
- status
- country
- type
- boolean flags

Rule:
> If many users hit the same key value, it’s a bad partition key.

---

### 8. Time-Based Hot Keys (Very Common Trap)

This mistake happens a lot.

Bad:
	PK = "ORDERS#2024-01-10"

All writes today:
- Hit one partition
- Cause throttling

Better:
- Add bucketing
- Add randomness
- Add sharding

Example:
	PK = "ORDERS#2024-01#3"

Now traffic spreads across buckets.

---

### 9. Read Hotspots vs Write Hotspots

Read hotspots:
- Popular items
- Configuration data
- Feature flags

Write hotspots:
- Event ingestion
- Counters
- Logs

Solutions differ:
- Reads → caching (DAX, app cache)
- Writes → sharding, batching, bucketing

Never treat them the same.

---

### 10. Throughput Optimization: Think Per-Partition

Key insight:

**Provisioned capacity is divided across partitions.**

If you have:
- 1000 WCUs
- 10 partitions

Each partition roughly handles:
- 100 WCUs

If one partition needs 300 WCUs:
- Throttling occurs
- Even though table capacity looks sufficient

This surprises many teams.

---

### 11. Why Autoscaling Does NOT Fix Bad Key Design

Autoscaling:
- Increases total capacity
- Does NOT rebalance traffic

If traffic is uneven:
- One partition still overloads
- Others sit idle

Autoscaling helps **only after good key design**.

---

### 12. Best Practices to Avoid Throttling (Design-Time)

Design-time best practices:
- High-cardinality partition keys
- Avoid global buckets
- Use composite keys wisely
- Bucket time-series data
- Avoid unbounded one-to-many relationships

Most throttling issues are **design issues**, not config issues.

---

### 13. Best Practices to Avoid Throttling (Runtime)

Runtime best practices:
- Exponential backoff retries
- Respect throttling errors
- Avoid sudden traffic spikes
- Warm up tables before load
- Monitor per-index metrics

Retrying blindly makes things worse.

---

### 14. Monitoring the RIGHT Metrics

Key DynamoDB metrics to watch:
- ThrottledRequests
- ConsumedReadCapacityUnits
- ConsumedWriteCapacityUnits
- SuccessfulRequestLatency
- ReturnedItemCount vs ScannedItemCount

Always monitor:
- Table AND GSIs
- Separately

Many teams miss index throttling.

---

### 15. Benchmarking DynamoDB (Why It’s Mandatory)

Never assume:
- “This design will scale”
- “Autoscaling will save us”

Benchmarking answers:
- How many requests per second?
- Where does throttling start?
- How evenly traffic distributes?
- How does latency behave?

Benchmarking is not optional for serious systems.

---

### 16. What to Benchmark (Correct Scope)

Benchmark:
- Real access patterns
- Real item sizes
- Real concurrency
- Real GSIs
- Real conditions

Do NOT benchmark:
- One request at a time
- Fake uniform traffic
- Tiny items only

That gives false confidence.

---

### 17. Load Testing Strategy (Step-by-Step)

Correct approach:
1. Identify critical access patterns
2. Generate realistic traffic
3. Gradually increase load
4. Observe throttling and latency
5. Adjust design or capacity
6. Repeat

This is how you validate DynamoDB designs.

---

### 18. Common Benchmarking Mistakes

Testing only base table  
Ignoring GSIs  
Ignoring item size  
No concurrency  
No retries  

These invalidate results.

---

### 19. Advanced Insight: DynamoDB Rewards Predictability

DynamoDB excels when:
- Traffic patterns are known
- Keys distribute evenly
- Access paths are explicit

It struggles when:
- Traffic is random
- Keys concentrate traffic
- Scans are used

This is not a flaw — it’s the design philosophy.

---

### 20. Mental Model Lock-In (Critical)

At this point, you should think:

- DynamoDB scales by partitions
- Keys decide partition usage
- Hot keys kill scalability
- Capacity is per-partition
- Autoscaling helps but doesn’t fix design
- Benchmarking is mandatory

This is expert-level understanding.


