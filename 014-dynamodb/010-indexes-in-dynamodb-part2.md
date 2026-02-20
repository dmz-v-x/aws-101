## Indexes in DynamoDB - Part 2

### 1. Important Reality: An Index Is Its Own Table 

This is a crucial mental shift.

Although AWS calls them “indexes”, **GSIs behave like separate tables**:
- They have their own partitioning
- They have their own throughput consumption
- They have their own hot partition risks

Think like this:

- Base table → primary data store
- Each GSI → another DynamoDB table automatically kept in sync

This explains:
- Why GSIs cost money
- Why GSIs can throttle independently
- Why bad GSI keys cause outages

---

### 2. How Data Flows into a GSI

When you write an item to the base table:

1. Item is written to base table
2. DynamoDB checks each GSI
3. If the item contains the GSI key attributes:
   - It is written into that GSI
4. If not:
   - Item does NOT exist in that GSI

Important consequence:
- GSIs are **sparse by default**

This is not a feature — it’s a design lever.

---

### 3. Sparse Indexes 

A **sparse index** is a GSI where:
- Only some items appear
- Based on attribute presence

Example:

	Base table items:
	  userId = 123, role = "ADMIN"
	  userId = 456, role = "USER"

	GSI partition key = role

Result:
- Only items with `role` appear
- Querying role = "ADMIN" is fast
- No scan needed

Sparse indexes allow:
- Conditional access patterns
- Cleaner modeling
- Lower index size

This is an advanced but critical technique.

---

### 4. GSI Hot Partitions

A common misconception:
> “I fixed hot partitions in my table, so I’m safe”

Wrong.

You can have:
- Perfect base table design
- Completely broken GSI design

Example of a dangerous GSI partition key:
- status = "ACTIVE"
- region = "US"
- type = "ORDER"

All traffic hits:
- One GSI partition
- Even if base table is fine

Rule:
> Every GSI partition key must obey the same rules as base table keys.

---

### 5. Detecting GSI Hotspots

Before creating a GSI, ask:
- How many distinct values will this key have?
- Will one value dominate traffic?
- Will traffic spike unpredictably?

If answers look risky:
- Add bucketing
- Add time-based partitioning
- Or redesign the access pattern

Never rely on “we’ll monitor it later”.

---

### 6. Write Amplification in Real Numbers

Let’s make this concrete.

Assume:
- 1 table
- 3 GSIs
- 1 KB item
- 1 million writes/day

Actual writes:
- Base table: 1M
- GSI 1: 1M
- GSI 2: 1M
- GSI 3: 1M

Total:
- 4 million writes/day

This directly affects:
- Cost
- Throttling risk
- Latency under spikes

Indexes multiply write cost. Always.

---

### 7. Projection Strategy: The Real Cost Lever

Projection choice controls:
- Index storage size
- Write cost
- Read efficiency

Revisiting projection types with a deeper lens:

KEYS_ONLY:
- Best for “lookup then fetch”
- Lowest write cost
- Two-step read pattern

INCLUDE:
- Best balance
- Single-query reads
- Controlled index growth

ALL:
- Almost always overused
- Highest cost
- Large index replication

Rule used by experienced teams:
> Default to INCLUDE. Use ALL only when proven necessary.

---

### 8. The “Double Query” Pattern 

This is a healthy DynamoDB pattern:

1. Query GSI using KEYS_ONLY or INCLUDE
2. Use returned primary keys
3. BatchGet from base table

Benefits:
- Small index
- Cheap writes
- Flexible reads

This is normal and expected in DynamoDB.
Do not fight it.

---

### 9. LSI vs GSI: Deeper Tradeoff Analysis

LSI advantages:
- Strong consistency
- No write amplification
- Same partition = no extra hot key risk

LSI disadvantages:
- Must be known at table creation
- Limited flexibility
- Counts toward item size limit

GSI advantages:
- Add anytime
- New access patterns
- Independent scaling

GSI disadvantages:
- Eventual consistency
- Write amplification
- Independent hot partition risk

Rule of thumb:
- Use LSI only when you are 100% sure
- Use GSI when you need flexibility

---

### 10. Index Evolution: Adding and Removing GSIs Safely

Adding a GSI:
- Triggers backfilling
- Consumes table capacity
- Can slow writes temporarily

Best practice:
- Add GSIs during low-traffic windows
- Monitor throttling closely
- Prefer on-demand capacity during backfill

Removing a GSI:
- Immediately reduces write cost
- Deletes index data
- No impact on base table data

Indexes are not “free to try”.

---

### 11. Anti-Pattern: One GSI Per Screen

Common beginner thinking:
> “This API endpoint needs an index”

Result:
- 10 endpoints
- 10 GSIs
- Exploding cost
- Fragile system

Correct thinking:
- Consolidate access patterns
- Reuse indexes
- Design indexes at system level

Indexes are architecture, not convenience.

---

### 12. Naming Indexes for Longevity

Good index names:
- Describe access pattern
- Include entity + purpose

Examples:
- GSI_UserByEmail
- GSI_OrderByStatusDate
- LSI_UserOrdersByAmount

This matters because:
- Index names appear everywhere
- Bad names cause misuse years later

---

### 13. Advanced Mental Model 

You should now think:

- A GSI is a second DynamoDB table
- Every GSI key is a scalability decision
- Projections control cost, not convenience
- Sparse GSIs are a feature, not a trick
- Index count multiplies write cost


