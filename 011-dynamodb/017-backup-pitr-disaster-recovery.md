### 1. First Principle: DynamoDB Does NOT Protect You From Yourself

Important truth:

- DynamoDB is highly durable
- AWS hardware failures are rare
- BUT DynamoDB will happily delete your data if you tell it to

There is no “undo” button unless:
- Backups are enabled
- PITR is enabled

This module is about building that safety net.

---

### 2. Types of Data Protection in DynamoDB

DynamoDB offers **three main recovery mechanisms**:

1. On-demand backups
2. Continuous backups (PITR)
3. Export to S3 (for long-term recovery & analytics)

Each serves a different purpose.

---

### 3. On-Demand Backups (Manual or Scheduled Snapshots)

An **on-demand backup** is:
- A full snapshot of a table
- Taken at a specific point in time
- Stored securely by AWS

Key properties:
- No performance impact
- No capacity impact
- Does not consume RCUs/WCUs
- Can be taken anytime

Backups are **independent** of the table lifecycle.

---

### 4. When to Use On-Demand Backups

Use on-demand backups:
- Before major deployments
- Before data migrations
- Before schema changes
- For compliance snapshots
- For long-term retention

Think of these as:
> “Safe checkpoints”

---

### 5. Restoring From an On-Demand Backup

When restoring:
- You **cannot overwrite** an existing table
- DynamoDB creates a **new table**
- With the same schema and data

After restore:
- You can switch traffic
- Or migrate data manually

This avoids accidental overwrites during recovery.

---

### 6. Continuous Backups (Point-in-Time Recovery — PITR)

**PITR** allows you to:

> Restore a table to **any second** within the last **35 days**

Key properties:
- Always-on rolling window
- Per-second granularity
- Covers INSERT, UPDATE, DELETE
- No performance impact

This is your **time machine**.

---

### 7. How PITR Works (Conceptual)

Internally:
- DynamoDB keeps change history
- Tracks item-level mutations
- Allows rewind to any timestamp

You don’t see this log.
You just choose:
- Date
- Time

And DynamoDB rebuilds the table.

---

### 8. PITR Restore Behavior (Very Important)

PITR restore:
- Creates a **new table**
- With data as of chosen time
- Does NOT overwrite original table

This allows:
- Safe investigation
- Comparison
- Controlled recovery

---

### 9. When PITR Is Absolutely Mandatory

Enable PITR if:
- Data is business-critical
- You have production users
- You have background jobs modifying data
- You run scripts or migrations
- You care about sleep 😄

In practice:
> PITR should be ON for almost all production tables.

---

### 10. Cost of PITR (Honest View)

PITR:
- Costs extra
- Priced per GB-month of data
- Very cheap compared to data loss

This is a **no-brainer cost** in most systems.

---

### 11. Disaster Recovery vs Backups (Important Distinction)

Backups and PITR handle:
- Logical data loss
- Human errors

They do NOT handle:
- Region-wide outages

That’s where **DR strategies** come in.

---

### 12. Disaster Recovery Strategies (High-Level)

Common DynamoDB DR strategies:

1. Backup + restore to another region
2. Export to S3 + rehydrate
3. Global Tables (active-active)
4. Application-level replication via Streams

Each has tradeoffs.

---

### 13. Simple DR: Backup Restore to Another Region

Basic approach:
- Take on-demand backups
- Restore in another region when needed

Pros:
- Simple
- Cheap

Cons:
- Slow RTO (hours)
- Manual steps

Suitable for:
- Non-critical systems

---

### 14. Faster DR: PITR + Restore

Better approach:
- Use PITR
- Restore to another region at failure time

Pros:
- Fine-grained recovery
- Better RPO

Cons:
- Still manual
- Still slower than global tables

---

### 15. Export to S3 (Long-Term Safety Net)

DynamoDB supports:
- Exporting table data to S3

Use cases:
- Long-term archival
- Compliance
- Offline analysis
- Cold recovery

Exports:
- Do not impact table performance
- Are consistent
- Stored in open formats

This is your “deep freezer”.

---

### 16. Global Tables vs Backups (Quick Preview)

Global Tables:
- Active-active replication
- Near real-time
- Expensive
- Complex

Backups/PITR:
- Passive
- Cheap
- Slower recovery

We’ll cover Global Tables later as a separate module.

---

### 17. Common Recovery Mistakes (Seen in Real Incidents)

PITR disabled in prod  
No recent on-demand backup  
No tested restore process  
Assuming “AWS won’t fail”  
No runbook for recovery  

Backups that aren’t tested are fake backups.

---

### 18. Best Practices (Checklist)

Production DynamoDB checklist:
- Enable PITR on all prod tables
- Take on-demand backups before major changes
- Periodically test restore
- Document recovery steps
- Restrict delete permissions

This turns disasters into incidents.

---

### 19. Mental Model Lock-In (Critical)

You should now think:

- DynamoDB durability ≠ data safety
- Backups protect from humans
- PITR is a time machine
- Restore always creates new tables
- DR is about recovery time, not storage

This is **production reliability thinking**.
