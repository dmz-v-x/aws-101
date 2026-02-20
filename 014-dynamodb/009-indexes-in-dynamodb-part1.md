## Indexes in DynamoDB - Part 1

### 1. What Is a Secondary Index? 

A **secondary index** is:
> An alternate view of your table with a different key definition

Think of it as:
- Same data
- Different keys
- Different query possibilities

Important clarifications:
- Indexes are NOT copies of the entire table (unless you choose so)
- Indexes only contain projected attributes
- Indexes exist purely to enable Query

There are **two types**:
1. LSI — Local Secondary Index
2. GSI — Global Secondary Index

They solve different problems.

---

### 2. Local Secondary Index (LSI): Core Idea

An **LSI** allows you to:
- Keep the SAME partition key as the table
- Use a DIFFERENT sort key

Mental model:
- Same bucket
- Different ordering inside the bucket

Example:
- Table PK: USER#123
- Table SK: createdAt
- LSI SK: orderStatus

This lets you:
- Query the same user
- But sort/filter items differently

---

### 3. LSI Rules and Hard Limitations

LSIs are powerful but very restricted.

LSI rules:
- Same partition key as base table
- Different sort key
- Must be defined **at table creation**
- Cannot be added later
- Strongly consistent reads supported
- Maximum **5 LSIs per table**

Why these limitations exist:
- LSI data is stored with the base table partition
- AWS must allocate space at creation time

If you forget an LSI:
👉 You cannot add it later

---

### 4. When Should You Use an LSI?

LSIs are good when:
- You KNOW all access patterns upfront
- You need different sorting for the same entity
- You need **strongly consistent reads**
- You want zero additional write throughput cost

Common use cases:
- Orders sorted by date vs status
- Messages sorted by timestamp vs priority
- Logs sorted by severity

LSIs are rare in practice because:
- Real systems evolve
- Access patterns change

---

### 5. Why LSIs Are Less Popular in Real Systems

LSIs sound nice, but:
- They lock your design early
- They reduce flexibility
- They require perfect foresight

Most production systems:
- Prefer GSIs
- Accept eventual consistency
- Pay extra cost for flexibility

This is a real-world tradeoff.

---

### 6. Global Secondary Index (GSI): Core Idea

A **GSI** allows you to:
- Use a DIFFERENT partition key
- Use a DIFFERENT sort key
- Create NEW access patterns

Mental model:
- A completely new query entry point
- A new way to “enter” your data

Example:
- Table PK: USER#123
- GSI PK: email
- GSI SK: createdAt

Now you can:
- Query users by email
- Without scanning the table

This is extremely powerful.

---

### 7. GSI Rules and Characteristics

Key GSI characteristics:
- Partition key can be any attribute
- Sort key is optional
- Can be added AFTER table creation
- Supports ONLY eventual consistency
- Up to **20 GSIs per table** (current AWS limit)

GSIs are:
- Flexible
- Powerful
- Dangerous if misused

---

### 8. Eventual Consistency in GSIs

GSIs are **eventually consistent**.

This means:
- Writes go to the base table first
- GSI updates happen asynchronously
- Reads from GSI may lag slightly

In practice:
- Usually milliseconds
- Sometimes seconds under heavy load

You must NOT use GSIs for:
- Strong consistency requirements
- Financial balance checks
- Critical sequencing logic

Use base table or transactions instead.

---

### 9. Write Amplification: The Hidden Cost of GSIs

Every write to the base table:
- Also writes to each GSI that projects that item

This is called **write amplification**.

Example:
- 1 table write
- 3 GSIs
→ 4 total writes

Consequences:
- Higher cost
- More throughput consumption
- Slower writes under load

This is why:
> GSIs must be deliberate, not casual.

---

### 10. Index Projections: What Gets Copied into the Index

Indexes do NOT automatically store all attributes.

You choose what gets projected.

There are **three projection types**.

---

### 11. Projection Type: KEYS_ONLY

KEYS_ONLY means:
- Partition key (index)
- Sort key (index)
- Table primary key attributes

Nothing else.

Pros:
- Cheapest
- Smallest index
- Fast writes

Cons:
- Requires extra fetch from base table if you need more data

Use when:
- Index is only for lookup
- Full item fetched later

---

### 12. Projection Type: INCLUDE

INCLUDE means:
- Keys
- Plus specific non-key attributes you choose

Pros:
- Balanced approach
- Avoids extra fetches
- Controls index size

Cons:
- Still increases write cost

This is the **most recommended projection type** in practice.

---

### 13. Projection Type: ALL

ALL means:
- Entire item copied into the index

Pros:
- One query gets everything
- Simple reads

Cons:
- Expensive
- Large indexes
- High write amplification

Use sparingly.
ALL is often a beginner mistake.

---

### 14. Choosing the Right Projection

Ask:
- What attributes does this access pattern need?

Then:
- Project ONLY those attributes
- Nothing more

Rule:
> Over-projection is the #1 DynamoDB cost mistake.

---

### 15. Index Naming Best Practices

Good index names:
- Describe access pattern
- Not implementation detail

Examples:
- GSI_EmailLookup
- GSI_StatusByDate
- LSI_OrdersByStatus

Bad names:
- index1
- gsi_new
- temp_index

Good naming:
- Improves team understanding
- Reduces accidental misuse

---

### 16. Cost Implications

Indexes cost money in:
- Storage
- Write throughput
- Read throughput

Cost grows with:
- Number of GSIs
- Size of projected attributes
- Write frequency

Key takeaway:
> GSIs optimize READS by increasing WRITE cost.

This is a deliberate tradeoff.

---

### 17. Common Index Anti-Patterns

Creating GSIs “just in case”  
Using ALL projection everywhere  
Using GSIs to replace bad partition keys  
Querying GSIs like SQL indexes  
Forgetting write amplification  

These patterns cause:
- Exploding bills
- Throttling
- Redesign pain

---

### 18. Advanced Mental Model 

You should now think:

- Table PK = primary access path
- LSI = alternate sort within same group
- GSI = brand new access path
- Projection = cost control lever
- Index count = write cost multiplier
