### 1. DynamoDB Transactions: Big Picture

DynamoDB transactions give you **ACID guarantees across multiple items**.

ACID means:
- **Atomicity** – all succeed or all fail
- **Consistency** – invariants are preserved
- **Isolation** – no partial visibility
- **Durability** – committed data is permanent

Important clarification:

**DynamoDB is NOT eventually consistent for transactions.  
Transactions are strongly consistent by definition.**

---

### 2. Transaction APIs in DynamoDB

DynamoDB exposes **two transaction APIs**:

1. **TransactWriteItems**
2. **TransactGetItems**

They work together but serve different purposes.

---

### 3. TransactWriteItems (Core Concept)

**TransactWriteItems** means:

> “Perform multiple write operations  
> and commit them ONLY if all conditions pass.”

Supported operations inside a transaction:
- Put
- Update
- Delete
- ConditionCheck (check without modifying)

All operations:
- Succeed together
- Or fail together
- With automatic rollback

No partial success. Ever.

---

### 4. Atomic Rollback Behavior (Very Important)

If ANY operation in the transaction:
- Fails a condition
- Violates constraints
- Hits a conflict

Then:
- **NONE of the writes are applied**
- DynamoDB rolls everything back automatically

You never need to implement rollback logic yourself.

This is a huge safety guarantee.

---

### 5. TransactGetItems (Read Consistency)

**TransactGetItems** means:

> “Read multiple items  
> and guarantee they are mutually consistent.”

Key characteristics:
- Strongly consistent reads
- Snapshot-style view
- All reads see the same state

This prevents:
- Reading half-updated data
- Inconsistent cross-item reads

Used far less than TransactWriteItems, but critical in some flows.

---

### 6. Transaction Limits (Hard Limits You Must Know)

Transactions are powerful, but restricted.

Current limits:
- Maximum **25 items** per transaction
- Maximum **4 MB total data**
- Items can span multiple tables
- Items can span multiple partitions
- Items can include GSIs (with caveats)

These limits are intentional.
Transactions are for **coordination**, not bulk work.

---

### 7. Isolation Level (What Transactions Protect You From)

During a transaction:
- Other operations do NOT see partial results
- Conflicting writes are detected
- Optimistic locking is enforced automatically

DynamoDB internally uses:
- Conflict detection
- Item-level locking (abstracted away)

You don’t manage locks.
You don’t see locks.
You just get correctness.

---

### 8. When Are Transactions NECESSARY?

Use transactions when:
- Multiple items must stay in sync
- Partial success would corrupt data
- Business rules span items

Classic examples:
- Bank transfer (debit + credit)
- Inventory decrement + order creation
- Seat booking systems
- Subscription activation + billing record
- Unique constraint enforcement across items

Rule:
> If partial success is unacceptable, use a transaction.

---

### 9. When Transactions Are NOT Needed (Very Important)

Do NOT use transactions when:
- One item update is enough
- Eventual consistency is acceptable
- High throughput is required
- You can tolerate retries and compensation

Many systems overuse transactions and suffer for it.

Transactions are **precision tools**, not defaults.

---

### 10. Transaction Cost Model (Critical for Production)

Transactions are more expensive than normal writes.

Key cost rules:
- Each item in a transaction consumes **2× write capacity**
- Condition checks also consume capacity
- GSIs increase cost further
- Latency is higher than non-transactional writes

Why?
Because DynamoDB must:
- Coordinate across partitions
- Detect conflicts
- Guarantee atomicity

Correctness costs money.

---

### 11. Performance Tradeoffs of Transactions

Transactions:
- Are slower than normal writes
- Reduce maximum throughput
- Increase contention under load

They are best for:
- Low to moderate write frequency
- High correctness requirements

They are bad for:
- Hot paths
- High-volume ingestion
- Event streams

Design accordingly.

---

### 12. Multi-Item Consistency Pattern #1: Inventory Control

Problem:
- Prevent overselling

Pattern:
- Item A: Inventory (quantity)
- Item B: Order record

Transaction steps:
1. ConditionCheck: quantity > 0
2. Update: decrement quantity
3. Put: create order

Guarantee:
- No overselling
- No partial orders

Without transactions, this is unsafe.

---

### 13. Multi-Item Consistency Pattern #2: Unique Constraints

DynamoDB does NOT have unique indexes.

But you can enforce uniqueness using transactions.

Pattern:
- Item A: Main entity
- Item B: “Uniqueness lock” item

Transaction:
1. ConditionCheck: uniqueness item does not exist
2. Put: uniqueness item
3. Put: main entity

This forces global uniqueness safely.

---

### 14. Multi-Item Consistency Pattern #3: State Transitions

Problem:
- Enforce legal state changes

Example:
- PENDING → PAID → SHIPPED

Transaction:
1. ConditionCheck: current state = PENDING
2. Update: set state = PAID
3. Update: create payment record

This prevents invalid transitions.

---

### 15. Transactions vs Conditional Writes (Key Distinction)

Conditional writes:
- Protect ONE item
- Extremely fast
- Very scalable

Transactions:
- Protect MULTIPLE items
- Slower
- More expensive

Rule:
> Use conditional writes whenever possible  
> Use transactions only when necessary

Most systems need far fewer transactions than they think.

---

### 16. Failure Handling in Transactions

If a transaction fails:
- No data is modified
- You get a clear error reason
- You must decide next action

Typical responses:
- Retry (if conflict)
- Abort (business rule violation)
- Inform user

Never blindly retry transactions.
Understand WHY they failed.

---

### 17. Idempotency and Transactions (Advanced Note)

Transactions do NOT automatically make operations idempotent.

You should still:
- Use request tokens
- Track operation IDs
- Prevent duplicate execution

This becomes important in distributed systems and retries.

We’ll revisit this later in deeper modules.

---

### 18. Mental Model Lock-In (Critical)

At this stage, you should think:

- DynamoDB is atomic per item by default
- Transactions extend atomicity across items
- Transactions guarantee correctness, not speed
- They are expensive but powerful
- Most correctness bugs are multi-item bugs

This is **expert-level DynamoDB reasoning**.
