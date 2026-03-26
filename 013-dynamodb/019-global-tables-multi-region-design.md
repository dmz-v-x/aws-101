### 1. What Are DynamoDB Global Tables (v2)?

**Global Tables v2** provide:

> A fully managed, multi-region,  
> active–active DynamoDB replication system.

Key idea:
- Same table exists in multiple regions
- All replicas are writable
- Data replicates automatically

You do NOT manage:
- Replication pipelines
- Conflict propagation
- Synchronization

AWS handles it.

---

### 2. Global Tables v1 vs v2 (Quick Context)

Global Tables v2:
- Easier setup
- Better monitoring
- Improved consistency handling
- Supports on-demand and provisioned

Interview note:
> Always mention v2 — v1 is legacy.

---

### 3. Active–Active Model (Critical Concept)

Active–active means:
- Clients can read/write in any region
- No primary region
- No manual failover needed

This enables:
- Low-latency local writes
- Regional fault tolerance

But it introduces complexity.

---

### 4. How Replication Works (High-Level)

Replication is:
- Asynchronous
- Based on DynamoDB Streams internally
- Region-to-region

Flow:
- Write in Region A
- Change captured
- Replicated to Regions B, C, etc.

Important:
- Replication is NOT instantaneous
- Eventual consistency across regions

---

### 5. Conflict Resolution (Very Important for Interviews)

Conflicts occur when:
- Same item is updated in multiple regions
- Before replication converges

Global Tables use:
> **Last writer wins (LWW)**

Based on:
- Timestamps
- Region-level versioning

This is simple, fast, and dangerous if misunderstood.

---

### 6. Why Conflict Resolution Matters

LWW means:
- Later write overwrites earlier one
- No merge logic
- No field-level reconciliation

This is fine for:
- Idempotent updates
- Stateless writes
- Overwrite-safe data

Dangerous for:
- Counters
- Financial balances
- Incremental updates

Interview insight:
> “Global Tables are great, but only if writes are designed to be conflict-safe.”

---

### 7. Designing Conflict-Safe Data Models

Good patterns:
- Immutable events
- Append-only records
- Versioned items
- Idempotent updates

Bad patterns:
- Increment counters
- Read-modify-write
- Mutable shared state

Correct design reduces conflicts naturally.

---

### 8. Latency Considerations (Why Use Global Tables)

Benefits:
- Users read/write to nearest region
- Lower p99 latency
- Better user experience

Tradeoff:
- Cross-region replication delay
- Eventual global consistency

Interview framing:
> “We trade strict consistency for global availability and low latency.”

---

### 9. Cross-Region Cost Implications

Global Tables are more expensive.

Cost factors:
- Write replicated to every region
- Each region consumes WCUs
- Data transfer costs
- Storage multiplied by regions

Reads:
- Local region only

Writes:
- Paid in every replica

This is a common interview trap.

---

### 10. When Global Tables Are a GOOD Idea

Use Global Tables when:
- Global user base
- Read/write latency matters
- High availability is required
- Eventual consistency is acceptable

Examples:
- User profiles
- Session metadata
- Feature flags
- IoT metadata

---

### 11. When Global Tables Are a BAD Idea

Avoid Global Tables when:
- Strong consistency is required
- Write conflicts are common
- Costs must be minimal
- Single-region suffices

Alternatives:
- Single region + failover
- Read replicas via Streams
- Application-level replication

---

### 12. Failover Strategies (Important Distinction)

With Global Tables:
- Failover is automatic
- No DNS switch needed
- Clients switch regions

Without Global Tables:
- You must manage:
  - DNS
  - Backups
  - Restore
  - Application logic

Interview tip:
> “Global Tables simplify failover but increase complexity elsewhere.”

---

### 13. Designing Global Applications (Practical Guidance)

Key design rules:
- Avoid mutable shared state
- Design idempotent writes
- Prefer append-only models
- Minimize cross-region coordination
- Assume replication delay

These rules prevent subtle bugs.

---

### 14. Global Tables + Transactions (Important Limitation)

Transactions:
- Work within a single region
- Do NOT span regions

Implication:
- You cannot have multi-region atomicity
- Design must tolerate temporary divergence

This is a critical interview detail.

---

### 15. Monitoring & Observability

Monitor:
- Replication latency
- Throttled writes per region
- Conflicts (via app logic)
- Region health

Failures are rare but impactful.

---

### 16. Common Global Tables Anti-Patterns

Using for counters  
Assuming strong consistency  
Ignoring write amplification costs  
Forgetting region-specific IAM  
Overusing Global Tables unnecessarily  

These lead to:
- Data corruption
- Cost explosions
- Hard-to-debug issues

---

### 17. Interview-Ready Summary (What to Say)

If asked:
> “Would you use Global Tables?”

Strong answer:
> “Only if we need multi-region low-latency writes and can design conflict-safe, eventually consistent data models. Otherwise, single-region with failover is simpler and cheaper.”

This sounds senior.

---

### 18. Mental Model Lock-In

You should now think:

- Global Tables = active–active replication
- Eventual consistency across regions
- Last-writer-wins conflict resolution
- Write cost multiplies by regions
- Design must be conflict-safe


