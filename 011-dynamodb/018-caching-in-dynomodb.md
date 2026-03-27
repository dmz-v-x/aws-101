### 1. Why Caching Matters in DynamoDB (Interview + Production Reality)

So far, you’ve learned how to:
- Design DynamoDB tables correctly
- Scale reads and writes
- Avoid throttling with good keys

Caching answers a different question:

> “What if my data access pattern is correct  
> but my latency is still too high or reads are too expensive?”

This is very common in:
- User-facing APIs
- Read-heavy workloads
- Hot items (profiles, configs, metadata)

This module is **very interview-relevant**.

---

### 2. Two Caching Approaches with DynamoDB

There are **two major caching strategies** used with DynamoDB:

1. **DAX (DynamoDB Accelerator)** — AWS-native
2. **External caches** (Redis / ElastiCache / Memcached)

They solve **similar problems**, but in **very different ways**.

Understanding the difference is critical.

---

### 3. What Is DAX? (From Absolute Zero)

**DAX (DynamoDB Accelerator)** is:

> A fully managed, in-memory cache  
> designed specifically for DynamoDB

Key idea:
- Sits **in front of DynamoDB**
- Applications talk to DAX, not DynamoDB
- DAX talks to DynamoDB behind the scenes

Think of DAX as:
> “A DynamoDB-aware read cache”

---

### 4. What DAX Is Optimized For

DAX is optimized for:
- Read-heavy workloads
- Microsecond latency
- Hot keys
- Repeated reads

Typical improvement:
- Milliseconds → microseconds
- Reduced RCUs
- Reduced DynamoDB load

This is extremely attractive in user-facing APIs.

---

### 5. How DAX Works (Read-Through Cache Model)

DAX uses a **read-through cache** model.

Flow:
1. Application requests data from DAX
2. If data is in cache → return immediately
3. If not in cache → fetch from DynamoDB
4. Cache the result
5. Return to application

Important:
- Application code does NOT manage cache logic
- DAX handles it automatically

This simplicity is a big selling point.

---

### 6. What DAX Caches (Important Detail)

DAX caches:
- GetItem
- Query
- Scan results (yes, but dangerous)

DAX does NOT cache:
- Writes directly
- Transactions
- Streams

Writes:
- Always go to DynamoDB
- Cache is updated/invalidated automatically

---

### 7. DAX and TTL (Time-To-Live)

DAX uses **TTL-based expiration** internally.

Key points:
- Cache entries expire after a configured TTL
- TTL is per DAX cluster
- You cannot set TTL per item

TTL tradeoff:
- Short TTL → fresher data, less cache hit
- Long TTL → better performance, more staleness

DAX prioritizes **performance over absolute freshness**.

---

### 8. Consistency Model of DAX (Very Important)

DAX provides:
- **Eventually consistent reads only**

Even if:
- Base table supports strong consistency
- Your code requests strong reads

DAX will still return:
- Eventually consistent data

Interview gold line:
> “DAX trades strong consistency for performance.”

This is a key tradeoff interviewers look for.

---

### 9. When You SHOULD Use DAX

Use DAX when:
- Read-heavy workloads
- Hot keys accessed frequently
- Slight staleness is acceptable
- Low latency is critical
- You want minimal cache logic

Examples:
- User profiles
- Product catalogs
- Feature flags
- Configuration data
- Session lookups

---

### 10. When You SHOULD NOT Use DAX

Do NOT use DAX when:
- Strong consistency is required
- Writes dominate reads
- Data changes constantly
- You need custom cache logic
- You already use Redis effectively

DAX is not a universal cache.

---

### 11. External Caches (Redis / ElastiCache)

External caches are:
- General-purpose
- Not DynamoDB-aware
- Application-managed

Examples:
- Redis (ElastiCache)
- Memcached

These caches:
- Sit alongside DynamoDB
- Require explicit cache logic in your code

---

### 12. External Cache Access Pattern

Typical pattern:
1. Application checks cache
2. If hit → return
3. If miss → read from DynamoDB
4. Store result in cache
5. Return response

This is a **cache-aside** pattern.

Unlike DAX:
- You manage TTL
- You manage invalidation
- You manage consistency

---

### 13. DAX vs Redis (Interview Comparison)

| Aspect | DAX | Redis / ElastiCache |
|----|----|----|
| Managed by AWS | Yes | Yes |
| DynamoDB-aware | Yes | No |
| Cache logic | Automatic | Manual |
| Consistency | Eventual only | You decide |
| Latency | Microseconds | Microseconds |
| Flexibility | Low | High |
| Use outside DynamoDB | No | Yes |

Interview insight:
> DAX is specialized, Redis is flexible.

---

### 14. Cache Invalidation (The Hardest Problem)

You may hear the quote:
> “There are only two hard things in computer science: cache invalidation and naming things.”

This is very true.

Cache invalidation answers:
> “When data changes, how do we avoid serving stale data?”

---

### 15. Cache Invalidation with DAX

With DAX:
- Writes go directly to DynamoDB
- DAX automatically invalidates affected items
- TTL ensures eventual refresh

You do NOT manually invalidate items.

This makes DAX:
- Safer
- Simpler
- Less error-prone

---

### 16. Cache Invalidation with External Caches

With Redis:
- You must invalidate cache manually
- On every write
- Across all code paths

Common strategies:
- Delete cache on write
- Update cache on write
- TTL-only expiration

Each has tradeoffs.

---

### 17. Consistency Tradeoffs in Caching

All caches introduce:
- Potential staleness
- Eventual consistency

Key question:
> “How stale is acceptable?”

Patterns:
- Read-your-own-writes → hard with caches
- Eventual consistency → common
- Strong consistency → avoid caching

Correct answer in interviews:
> “Caching is a business decision, not just technical.”

---

### 18. Common Caching Anti-Patterns

Caching everything blindly  
Caching highly volatile data  
Ignoring consistency requirements  
Mixing DAX and Redis unnecessarily  
Assuming cache = correctness  

Caching is for **performance**, not correctness.

---

### 19. Interview-Level Decision Framework (Very Important)

When asked:
> “Would you use DAX or Redis?”

Answer structure:
1. Access pattern
2. Consistency requirement
3. Read/write ratio
4. Operational complexity
5. Cost

This shows senior-level thinking.

---

### 20. Mental Model Lock-In

At this stage, you should think:

- DynamoDB is fast but not free
- Caching reduces latency and cost
- DAX = simple, DynamoDB-native cache
- Redis = flexible, app-managed cache
- Caching always trades freshness for speed

This is **interview-ready understanding**.
