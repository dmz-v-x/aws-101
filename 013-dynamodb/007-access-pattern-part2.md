## Access Pattern - Part 2

### 1. Access Patterns Are NOT Queries

A very common mistake is to think:
> “My access pattern is a Query operation”

That is wrong.

An **access pattern** is:
- A business question
- Independent of DynamoDB APIs

Examples:
- “Show user dashboard”
- “Get last 5 logins”
- “Check if user has an active subscription”
- “List unpaid invoices”

Only **after** defining access patterns do you decide:
- Query
- GetItem
- Index
- Or redesign the model

Never start with DynamoDB operations.

---

### 2. Turning One Access Pattern into Multiple Physical Reads

One business action often maps to **multiple reads**.

Example: “Show user dashboard”

This may require:
- User profile
- Last 5 orders
- Active subscription
- Notifications count

Correct DynamoDB modeling:
- Same partition key
- Different sort key prefixes

Example:

	PK = "USER#123" SK = "PROFILE"
	PK = "USER#123" SK = "ORDER#2024-01-10"
	PK = "USER#123" SK = "SUBSCRIPTION#ACTIVE"
	PK = "USER#123" SK = "NOTIFICATION#COUNT"

One query.
Many item types.
One partition.

This is intentional and powerful.

---

### 3. Designing Sort Keys for Range Queries

Sort keys are **ordered lexicographically**, not magically.

This means:
- Strings are sorted character by character
- Numbers are sorted numerically
- ISO-8601 timestamps sort correctly as strings

Correct time sort key:

	SK = "ORDER#2024-01-09T10:30:00Z"

Incorrect:

	SK = "ORDER#9-1-2024"

Bad formats break ordering and range queries.

Rule:
> Always design sort keys so that natural string order == logical order.

---

### 4. Multi-Dimensional Access Patterns

Real systems often need:
- Access by user
- Access by status
- Access by time

Bad instinct:
> “Let me put everything in the primary key”

This leads to:
- Impossible queries
- Hot partitions
- Redesigns

Correct approach:
- Primary key handles the **main access path**
- Secondary indexes handle **alternate paths** (later module)

For now, understand:
> One table does NOT mean one access pattern.

It means:
> One table, multiple carefully designed paths.

---

### 5. Hot Partitions

Earlier we said:
- Hot partitions happen when one partition gets too much traffic

Now let’s go deeper.

Hot partitions can be caused by:
- Read concentration
- Write concentration
- Both

Example:
- “Get today’s orders”
- Everyone hits the same partition
- Today’s date = hotspot

This happens even with “valid-looking” keys.

---

### 6. Write Hotspots vs Read Hotspots

**Write hotspot**
- Too many writes to same partition key
- Common in event ingestion systems

**Read hotspot**
- Popular item read by everyone
- Common in configuration or metadata tables

Both cause throttling.

Solution strategies differ.

---

### 7. Write Sharding Pattern

When you must write a lot of data quickly:

Instead of:

	PK = "EVENTS"

You shard:

	PK = "EVENTS#1"
	PK = "EVENTS#2"
	PK = "EVENTS#3"

Shard key chosen:
- Randomly
- Or via hashing

This spreads load across partitions.

Later, you aggregate results.

This is a **controlled tradeoff**.

---

### 8. Composite Key Pattern: Bucketing Time-Series Data

Pure time-series can still hotspot.

Instead of:

	PK = "LOGS"
	SK = "2024-01-10T10:30"

Use buckets:

	PK = "LOGS#2024-01"
	SK = "2024-01-10T10:30"

Now:
- Writes are spread monthly
- Reads are still predictable

This pattern is used heavily in production systems.

---

### 9. Composite Key Pattern: Hierarchical Relationships

You can model trees using sort keys.

Example:
- Organization → Teams → Users

	PK = "ORG#1"
	SK = "TEAM#A"
	SK = "TEAM#A#USER#1"
	SK = "TEAM#A#USER#2"

Benefits:
- Hierarchy encoded in key
- Prefix queries enable subtree fetches

This is impossible in SQL without recursive joins.

---

### 10. Modeling One-to-Many at Scale

One-to-many is easy…
until the “many” becomes very large.

Problem:
- Millions of items under one partition key
- Partition size limits
- Throughput contention

Solutions:
- Time bucketing
- Sharding
- Archival strategies
- Multiple access patterns

Rule:
> One-to-many is safe only when the “many” is bounded or bucketed.

---

### 11. Anti-Pattern: “God Partition”

This happens when developers do:

	PK = "ALL_USERS"
	PK = "ALL_ORDERS"

This guarantees:
- Hot partition
- Throttling
- Failure under load

Never use:
- Global buckets
- Singleton partition keys

DynamoDB is not built for this.

---

### 12. When You SHOULD Duplicate Data 

In DynamoDB:
- Duplication is normal
- Duplication is expected
- Duplication is a feature

You duplicate data to:
- Serve different access patterns
- Avoid scans
- Avoid joins

This feels wrong coming from SQL.
It is correct in DynamoDB.

---

### 13. Real Production Scenario: Notifications System

Requirements:
- Get unread notifications
- Mark notification as read
- Show recent notifications

Model:

	PK = "USER#123"
	SK = "NOTIF#2024-01-10T10:00"
	read = false

Later:
- Index by read status
- Or bucket unread separately

Design starts from access patterns, not storage.

---

### 14. Debugging a Bad Design

If you ever see:
- Frequent scans
- High throttling
- Unexpected cost spikes

Ask:
1. What is the partition key?
2. Is traffic evenly distributed?
3. Are access patterns known?
4. Are keys encoding meaning?

90% of DynamoDB issues come from wrong answers here.

---

### 15. Mental Model Upgrade

You should now think:

- Partition key = **who owns this data**
- Sort key = **what type and order is this data**
- Query = **fetch a slice of a bucket**
- Table = **a logical container, not a schema**

This mental model is DynamoDB mastery.
