## Consistency & Conditional Logic

### 1. First Principle: DynamoDB Is a Distributed System

This must be internalized.

DynamoDB:
- Runs across many machines
- Replicates data internally
- Handles failures automatically

Because of this:
- Reads and writes don’t always see the same instant view
- Concurrency must be handled explicitly

This is why **consistency modes exist**.

---

### 2. Strong vs Eventual Consistency

Consistency answers this question:

> “When I read data, do I see the LATEST write or maybe a slightly older one?”

DynamoDB offers **two read consistency modes**:
1. Strongly consistent reads
2. Eventually consistent reads

Writes are always **strongly consistent** internally.  
Reads are where the choice matters.

---

### 3. Strongly Consistent Reads

A **strongly consistent read** guarantees:

> “When this read returns, it reflects all writes that completed before it.”

In simple terms:
- You always see the latest data
- No stale reads

Characteristics:
- More coordination internally
- Slightly higher cost
- Slightly higher latency

Strong consistency is only available:
- On the **base table**
- NOT on GSIs

---

### 4. Eventually Consistent Reads

An **eventually consistent read** means:

> “You might see slightly stale data for a short time.”

Key facts:
- Usually milliseconds of delay
- Sometimes seconds under heavy load
- No correctness guarantee at a specific instant

But:
- Much cheaper
- Much more scalable

This is the default read mode.

---

### 5. Cost Difference

Read capacity cost:

- Strongly consistent read = **2 read units**
- Eventually consistent read = **1 read unit**

So:
- Strong consistency costs **2×**
- This matters at scale

Rule:
> Use strong consistency only when correctness demands it.

---

### 6. When Should You Use Strong Consistency?

Use strong consistency when:
- Checking account balances
- Enforcing limits
- Reading immediately after a write
- Preventing double-spend or double-booking

Examples:
- “Has this coupon already been used?”
- “What is the current subscription state?”

Correctness > cost in these cases.

---

### 7. When Eventual Consistency Is Perfectly Fine

Use eventual consistency when:
- Reading user profiles
- Showing feeds
- Analytics and dashboards
- Logs and events
- Caches and metadata

Most reads in most systems:
👉 Do NOT need strong consistency.

This is how DynamoDB scales so well.

---

### 8. Why GSIs Are Always Eventually Consistent

GSIs are updated **asynchronously**.

This means:
- Writes go to base table first
- Index updates follow later

Therefore:
- Strong consistency is impossible on GSIs
- You must design around this

This is a core DynamoDB constraint, not a limitation.

---

### 9. The Real Problem: Race Conditions

A **race condition** happens when:
- Two or more writers act on the same data
- Based on outdated assumptions

Classic example:
- Two users buy the last item
- Both see “1 left”
- Both succeed
- Inventory goes negative

This happens if you rely on:
- Read → modify → write (without protection)

---

### 10. DynamoDB’s Solution: Conditional Writes

DynamoDB provides **conditional writes** to prevent race conditions.

A conditional write means:

> “Perform this write ONLY IF a condition is true.”

This check + write is:
- Atomic
- Safe
- Performed server-side

No other system can sneak in between.

---

### 11. ConditionExpression

A **ConditionExpression** lets you define rules like:
- Attribute must exist
- Attribute must NOT exist
- Attribute must equal a value
- Attribute must be greater / less than a value

If the condition fails:
- The write is rejected
- No data is modified

This is your main concurrency tool.

---

### 12. Common Conditional Write Patterns

Examples (conceptual):

- Create item only if it doesn’t exist
- Update status only if current status is “PENDING”
- Decrement inventory only if quantity > 0
- Update profile only if version matches

These patterns are everywhere in real systems.

---

### 13. Avoiding Race Conditions the RIGHT Way

Wrong pattern:
1. Read item
2. Check value in code
3. Write item

Correct pattern:
- Use ConditionExpression
- Let DynamoDB enforce correctness

Rule:
> If correctness matters, NEVER rely on application-side checks alone.

---

### 14. Optimistic Concurrency Control

Optimistic concurrency assumes:
- Conflicts are rare
- But must be detected

This is implemented using a **version attribute**.

---

### 15. Version Attribute Pattern

Each item has:
- version = 1

Update flow:
1. Read item
2. Remember version
3. Update item ONLY IF version is unchanged
4. Increment version

Conceptually:

	version = version + 1
	ONLY IF version == old_version

If another writer updated first:
- Version mismatch
- Update fails safely

This prevents lost updates.

---

### 16. Why This Pattern Is Called “Optimistic”

It’s called optimistic because:
- You assume no conflict
- You detect conflict only if it happens
- You retry if needed

This scales much better than locking.

---

### 17. Return Values: Knowing What Happened

DynamoDB can return values from writes.

Return options:
- NONE
- ALL_OLD
- ALL_NEW
- UPDATED_OLD
- UPDATED_NEW

Why this matters:
- You can detect what changed
- You can react without extra reads
- You can debug safely

Example uses:
- Log previous state
- Show updated data
- Retry intelligently

---

### 18. Handling Conditional Failures Correctly

When a condition fails:
- DynamoDB returns a specific error
- This is NOT a system failure
- This is a **business-level conflict**

Correct handling:
- Catch the failure
- Decide to retry, abort, or inform user

Never treat conditional failures as crashes.

---

### 19. Combining Conditions with Updates

You can combine:
- Update expressions
- Condition expressions
- Return values

This allows:
- Atomic state machines
- Safe counters
- Inventory systems
- Payment flows

This is where DynamoDB becomes extremely powerful.

---

### 20. Common Anti-Patterns

Read → modify → write without condition  
Ignoring conditional failures  
Using GSIs for strong consistency logic  
Using locks instead of conditions  

These cause:
- Data corruption
- Double writes
- Financial bugs

---

### 21. Mental Model Lock-In

At this stage, you should think:

- Reads may be stale
- Writes are atomic
- Conditions protect correctness
- Versioning prevents lost updates
- GSIs are eventually consistent
- Strong consistency is expensive but precise
