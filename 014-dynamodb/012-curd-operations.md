## CRUD Operations in DynamoDB

### 1. DynamoDB CRUD APIs: Big Picture

DynamoDB exposes a small set of **primitive operations**.

Core CRUD APIs:
- GetItem
- PutItem
- UpdateItem
- DeleteItem

Supporting APIs:
- Query
- Scan
- BatchGetItem
- BatchWriteItem

Everything you ever build is composed of these primitives.

---

### 2. GetItem — Read ONE Item by Primary Key

**GetItem** means:

> “I know the FULL primary key.  
> Give me exactly ONE item.”

Requirements:
- Partition key (mandatory)
- Sort key (mandatory if table has one)

Example (conceptual):

	PK = "USER#123"
	SK = "PROFILE"

What GetItem does:
- Direct lookup
- No searching
- No scanning
- Extremely fast
- Strongly consistent (optional)

Use GetItem when:
- You want exactly one known item
- You know both keys
- Latency must be minimal

GetItem is the fastest read in DynamoDB.

---

### 3. PutItem — Create or Replace an Item

**PutItem** means:

> “Store this item at this primary key.”

Important behavior:
- If item does NOT exist → created
- If item EXISTS → completely overwritten

This is critical:
- PutItem is NOT a partial update
- Missing attributes are deleted

Example danger:

Old item:
	name = "Alice"
	age = 30

PutItem with:
	name = "Alice Updated"

Result:
	age is gone

Use PutItem when:
- Creating new items
- You control the full item shape
- Overwrites are acceptable

Avoid PutItem for frequent updates.

---

### 4. UpdateItem — Modify Parts of an Item 

**UpdateItem** means:

> “Change specific attributes of an existing item.”

Key advantages:
- Partial updates
- No overwrite
- Atomic operations

You can:
- Set attributes
- Remove attributes
- Increment numbers
- Append to lists

Example concepts:
- Increment login count
- Update order status
- Add lastLogin timestamp

UpdateItem is:
- Safer than PutItem
- More expressive
- Core to real applications

Most production systems rely heavily on UpdateItem.

---

### 5. DeleteItem — Remove One Item

**DeleteItem** means:

> “Remove this item identified by its primary key.”

Behavior:
- Deletes exactly one item
- Requires full primary key
- Atomic

Common uses:
- Delete sessions
- Remove expired data
- Hard deletes

Advanced pattern:
- Soft deletes (mark as deleted instead of removing)

DeleteItem is simple but dangerous if misused.
Always know what key you’re deleting.

---

### 6. Query vs Scan 

You already learned the concept.
Now let’s focus on **API behavior and cost**.

---

### 7. Query — Targeted, Safe, Scalable

Query:
- Requires partition key
- Reads one item collection
- Optional sort key conditions
- Uses KeyConditionExpression

Key facts:
- Reads minimal data
- Cost proportional to returned data
- Predictable performance

Query is the **default production read**.

---

### 8. Scan — Broad, Expensive, Dangerous

Scan:
- Reads every item
- Ignores keys
- Applies filters after reading

Key facts:
- Cost grows with table size
- Slow at scale
- Unpredictable

Scan should NOT be used in:
- User-facing APIs
- Hot paths
- Real-time systems

---

### 9. Cost Difference 

This is non-negotiable:

- Query reads only matching items
- Scan reads everything first

Filter expressions do NOT reduce read cost.

If your app uses Scan frequently:
👉 Your data model is wrong.

---

### 10. FilterExpression vs KeyConditionExpression

This distinction is critical and often misunderstood.

---

### 11. KeyConditionExpression 

KeyConditionExpression:
- Applied BEFORE reading data
- Uses partition key
- Uses sort key (optional)

Examples:
- PK = USER#123
- SK begins_with ORDER#
- SK between two timestamps

This determines:
- What DynamoDB reads
- How much it costs

Key conditions are efficient.

---

### 12. FilterExpression 

FilterExpression:
- Applied AFTER data is read
- Filters results in memory
- Does NOT reduce read cost

Use filters for:
- Minor refinements
- Low-cardinality filtering
- After key-based narrowing

Never rely on filters alone.

---

### 13. Correct Pattern: Keys First, Filters Second

Correct flow:
1. Narrow data using partition + sort key
2. Then apply FilterExpression if needed

Wrong flow:
- Scan + FilterExpression

This distinction separates good and bad DynamoDB usage.

---

### 14. BatchGetItem — Read Multiple Known Items

BatchGetItem means:

> “Get many items by their primary keys.”

Rules:
- Max 100 items per request
- Max 16 MB total response
- Items must be identified by full keys

Use cases:
- Fetch multiple profiles
- Resolve references
- Fan-out reads

BatchGetItem is:
- Efficient
- Parallelized internally
- Better than many GetItem calls

---

### 15. BatchWriteItem — Write/Delete Many Items

BatchWriteItem supports:
- PutItem
- DeleteItem

Rules:
- Max 25 items per batch
- No UpdateItem
- No transactions
- Partial success possible

Important:
- Unprocessed items must be retried
- Order is not guaranteed

BatchWriteItem is for:
- Bulk imports
- Migrations
- Backfills

Not for transactional workflows.

---

### 16. Batch APIs: Hidden Gotchas

Batch APIs:
- Are NOT atomic
- Can partially fail
- Require retry logic
- Consume throughput quickly

They are powerful but require discipline.

---

### 17. CRUD Anti-Patterns 

Using PutItem for updates  
Using Scan in APIs  
Using FilterExpression instead of keys  
Ignoring unprocessed batch items  
Fetching entire items unnecessarily  

These lead to:
- High cost
- Throttling
- Data loss bugs

---

### 18. Mental Model Lock-In 

You should now think:

- GetItem = exact lookup
- PutItem = overwrite
- UpdateItem = safe mutation
- DeleteItem = remove
- Query = correct read path
- Scan = last resort
- Key conditions reduce cost
- Filters refine results
