## Query vs Scan

### 1. First Principle: DynamoDB Never “Searches” Data

This is critical.

DynamoDB does NOT:
- Search rows
- Look through the whole table
- Guess where data might be

It either:
- Goes **directly to a partition** (Query / GetItem)
- Or **reads everything** (Scan)

There is no middle ground.

---

### 2. What Is a Query?

A **Query** means:

> “I know the partition key.  
> Give me items from THAT partition only.”

Key facts about Query:
- Partition key is **mandatory**
- Sort key is **optional but powerful**
- Reads only ONE partition (or logical partition)
- Extremely fast
- Predictable cost

Mental model:
- Go to one bucket
- Pick specific files from inside

---

### 3. Query in Plain English

Imagine a filing cabinet:
- Each drawer = partition key
- Files inside = sort keys

A Query is:
> “Open drawer USER#123 and give me the files I want”

DynamoDB does not open other drawers.
That’s why it’s fast.

---

### 4. Query Examples

Assume data like this:

	PK = "USER#123"  SK = "PROFILE"
	PK = "USER#123"  SK = "ORDER#2024-01-01"
	PK = "USER#123"  SK = "ORDER#2024-01-10"
	PK = "USER#456"  SK = "PROFILE"

Valid Queries:
- Get everything for USER#123
- Get only orders for USER#123
- Get latest order for USER#123

Invalid Query:
- Get all users
- Get all orders
- Get users by email (without index)

Why?
Because Query requires a **known partition key**.

---

### 5. Sort Key Power: Range & Prefix Queries

When a table has a sort key, Query becomes powerful.

You can:
- Fetch a range
- Fetch latest items
- Fetch by prefix

Examples:
- Orders between two dates
- Last 5 events
- All items starting with ORDER#

This works because:
- Sort keys are ordered
- DynamoDB walks an ordered list

---

### 6. Query Is Cheap and Predictable

Query:
- Reads only what it needs
- Cost is proportional to returned data
- Scales linearly
- Never surprises you

This is why:
> **All good DynamoDB designs revolve around Query**

Indexes exist to enable more Queries — not Scans.

---

### 7. What Is a Scan?

A **Scan** means:

> “I don’t know where the data is.  
> Read EVERYTHING and I’ll filter later.”

Scan behavior:
- Reads every item in the table (or index)
- Applies filters AFTER reading
- Ignores keys for efficiency
- Gets slower as table grows

Mental model:
- Open every drawer
- Read every file
- Throw most of them away

---

### 8. The Most Dangerous Misunderstanding About Scan

This is critical:

**FilterExpression does NOT reduce read cost.**

Scan does:
1. Read item
2. Consume capacity
3. THEN apply filter

So even if:
- You return 1 item
- DynamoDB still read everything

This is why Scan is dangerous.

---

### 9. Why Scan Feels “Okay” in Development

In dev:
- Small table
- Few items
- Scan feels fast

In production:
- Millions of items
- Scan becomes slow
- Costs explode
- Throttling happens

This is a classic DynamoDB failure story.

---

### 10. When Is Scan Actually Acceptable?

Scan is NOT evil — it’s **situational**.

Valid Scan use cases:
- One-time migrations
- Backfills
- Data exports
- Admin dashboards
- Small reference tables
- Scheduled offline jobs

Rule:
> Scan is okay when humans wait, not users.

---

### 11. Query vs Scan: Side-by-Side Truth Table

Query:
- Requires partition key
- Reads one partition
- Predictable performance
- Predictable cost
- Scales infinitely

Scan:
- Requires no key
- Reads entire table
- Performance degrades
- Cost grows with table
- Does not scale

One is for production paths.
One is for maintenance paths.

---

### 12. Why Access Patterns Force Query Usage

Recall Module 3:

Access pattern example:
- “Get all orders for a user”

Correct DynamoDB answer:
- Query PK = USER#123

Wrong answer:
- Scan orders and filter by userId

If you need Scan to satisfy an access pattern:
👉 **Your data model is wrong**

This is the single most important diagnostic rule.

---

### 13. Query Prepares You for Indexes 

Primary key:
- Enables Query on base table

Secondary index:
- Exists ONLY to enable another Query
- With a different partition key

Indexes do not:
- Replace Scan
- Fix bad models
- Magically improve performance

Indexes exist to:
> Create new Query paths

---

### 14. Common Anti-Patterns

Using Scan in API handlers  
Using Scan + Filter as “query”  
Adding GSIs to “avoid thinking”  
Designing tables first, queries later  

All lead to:
- Cost overruns
- Latency spikes
- Emergency redesigns

---

### 15. Mental Model Lock-In 

You should now think:

- Query = targeted, safe, scalable
- Scan = broad, expensive, dangerous
- Access patterns must be Query-able
- If it’s not Query-able, redesign
