## Access Pattern

### 1. What Is an Access Pattern? (From Absolute Zero)

An **access pattern** is simply:
> How your application reads or writes data.

Examples of access patterns:
- Get user by userId
- Get all orders for a user
- Get latest 10 orders for a user
- Update order status
- Delete a session
- Fetch comments for a post

These are NOT queries yet.  
These are **questions your app asks the database**.

In DynamoDB:
- If you can’t answer an access pattern efficiently
- Your data model is wrong

---

### 2. The Cardinal Rule of DynamoDB (Non-Negotiable)

**You must design your primary keys from access patterns.**

Not from:
- Entities
- ER diagrams
- Tables you “feel” are right
- SQL habits

This is the exact opposite of relational databases.

Relational mindset:
- Model entities
- Normalize
- Query later

DynamoDB mindset:
- List access patterns
- Design keys to answer them
- Store data accordingly

---

### 3. Step-by-Step: Designing Keys From Access Patterns

Let’s do this properly.

#### Step 1: Write access patterns in plain English

Example application: User Orders

Access patterns:
1. Get user profile by userId
2. Get all orders for a user
3. Get latest order for a user

No database words yet. Just business questions.

---

#### Step 2: Identify the grouping key

Ask:
> What data must live together?

Answer:
- All data related to ONE user

So:
- Partition Key = userId

This ensures:
- One query
- One partition
- No scans
- Predictable performance

---

#### Step 3: Encode meaning in the sort key

Now ask:
> How do we distinguish different item types?

We encode **type + order** into the sort key.

Example:

	PK = "USER#123"   SK = "PROFILE"
	PK = "USER#123"   SK = "ORDER#2024-01-01"
	PK = "USER#123"   SK = "ORDER#2024-01-10"

Now:
- One query fetches profile + orders
- Orders are sorted automatically
- Latest order is easy to fetch

This is DynamoDB done correctly.

---

### 4. Mental Model: DynamoDB as Ordered Buckets

Visualize DynamoDB like this:

- Partition Key → Bucket
- Sort Key → Ordered files inside bucket

Example:

	Bucket: USER#123
	  ├── PROFILE
	  ├── ORDER#2024-01-01
	  ├── ORDER#2024-01-10

DynamoDB is extremely fast because:
- It jumps directly to the bucket
- Then walks an ordered list

No searching. No scanning. No guessing.

---

### 5. What Are Hot Partitions? (Critical Topic)

A **hot partition** happens when:
- Too many requests hit the same partition
- That partition exceeds its throughput limits

Result:
- Throttling
- Increased latency
- Failed requests

This is one of the most common DynamoDB production issues.

---

### 6. What Causes Hot Partitions?

Common causes:

Low-cardinality partition keys  
Examples:
- status = "ACTIVE"
- type = "ORDER"
- date = "2024-01-01"

All traffic goes to:
- One partition
- One physical resource

Sequential or time-based keys  
Examples:
- auto-increment IDs
- timestamps as partition keys

Recent data gets all traffic → hotspot.

---

### 7. How to Identify Hot Partitions

You detect hot partitions by:
- Sudden throttling
- Uneven traffic patterns
- One access pattern dominating load

From a design perspective, ask:
> Will many users hit the same partition key value?

If yes → danger.

---

### 8. How to Avoid Hot Partitions (Design Rules)

Good partition keys must be:
- High cardinality (many distinct values)
- Evenly distributed
- Aligned with access patterns

Good examples:
- userId
- orderId
- sessionId
- deviceId

Bad examples:
- status
- region
- date
- boolean flags

**Partition keys are scalability levers.**

---

### 9. Composite Key Patterns (Advanced but Essential)

Now let’s go deeper into **sort key patterns**.

Sort keys are not random strings.
They encode:
- Type
- Order
- Time
- Version
- Relationship

---

### 10. Time-Series Pattern

Used for:
- Logs
- Orders
- Events
- Metrics

Pattern:

	PK = "USER#123"
	SK = "EVENT#2024-01-10T10:30:00Z"

Why it works:
- Items automatically sorted by time
- Range queries are easy
- Fetch latest N items efficiently

This pattern is extremely common.

---

### 11. Versioning Pattern

Used for:
- Audit history
- Document versions
- Configuration changes

Pattern:

	PK = "DOC#A1"
	SK = "VERSION#1"
	PK = "DOC#A1"
	SK = "VERSION#2"

Benefits:
- All versions grouped
- Easy rollback
- Full history preserved

---

### 12. Prefixing Pattern (Type Encoding)

Prefixing is when you embed **meaning** in keys.

Example:

	SK = "PROFILE"
	SK = "ORDER#2024-01-01"
	SK = "SESSION#abc123"

Why prefixing matters:
- Enables begins_with queries
- Separates item types cleanly
- Avoids attribute overloading

This is foundational to single-table design.

---

### 13. Modeling One-to-Many Relationships (Core DynamoDB Skill)

Relational thinking:
- One user → many orders → separate table + join

DynamoDB thinking:
- One partition → many items via sort key

Pattern:

	PK = "USER#123"
	SK = "ORDER#1"
	SK = "ORDER#2"
	SK = "ORDER#3"

Now:
- Query PK = USER#123
- You get all orders
- Sorted
- In one request

This is **one-to-many done correctly**.

---

### 14. Advanced One-to-Many: Filtering by Type

Because of prefixing, you can do:

- Get only orders
- Ignore profile
- Ignore sessions

By querying:
- PK = USER#123
- SK begins_with "ORDER#"

This is intentional design, not coincidence.

---

### 15. Common Anti-Patterns to Avoid

Using Scan instead of Query  
Designing keys after building the app  
Random sort keys with no meaning  
Putting unrelated entities under same partition  
Overloading partition keys with business states  

All of these cause:
- Cost explosion
- Performance collapse
- Painful refactors

---

### 16. Why DynamoDB Feels Hard at First (Truth)

DynamoDB:
- Forces you to think early
- Punishes lazy modeling
- Rewards correct mental models

Once you “get it”:
- It becomes simpler than SQL
- Performance becomes predictable
- Scaling becomes effortless

This module is where that shift happens.

This is **core DynamoDB engineering knowledge**.
