## How DNS Works

### 1. Starting From Absolute Zero — What Problem Are We Solving?

When you type:

    google.com

Your computer faces a problem.

Computers do NOT understand domain names.

They understand:

    IP addresses

Example:

    142.250.xxx.xxx

So before any website loads, something must translate:

Domain Name → IP Address

This translation process is called:

**DNS Resolution**

---

### 2. What “DNS Resolution” Really Means

DNS resolution means:

Finding the IP address associated with a domain name.

Simple definition:

Name → Number conversion

But behind this simplicity lies a multi-step distributed process.

---

### 3. High-Level View of DNS Query Flow

The lookup journey typically follows:

Browser → Resolver → Root → TLD → Authoritative → Resolver → Browser

Each component plays a specific role.

---

### 4. Step 1 — Browser Cache Check

First stop:

Your **browser cache**

Browser asks:

“Have I recently looked up google.com?”

If YES:

Use cached IP → Done (fastest path)

If NO:

Ask operating system.

---

### 5. Step 2 — OS Cache Check

Operating system maintains its own DNS cache.

If found:

Return IP → Done

If not:

Query DNS resolver.

---

### 6. Step 3 — Recursive Resolver Contacted

Your system queries:

**Recursive DNS Resolver**

Usually provided by:

• ISP  
• Public DNS (8.8.8.8, 1.1.1.1)

Resolver’s job:

Obtain the final answer on your behalf.

Key idea:

Resolver does the heavy lifting.

---

### 7. What Makes It “Recursive”?

Recursive means:

Client asks resolver:

“Give me the FINAL answer. I don’t care how.”

Resolver assumes full responsibility.

---

### 8. If Resolver Has Cached Answer

Resolver also maintains cache.

If known:

Return IP immediately → Very fast

If unknown:

Begin hierarchical lookup.

---

### 9. Step 4 — Query Root DNS Server

Resolver asks:

Root Server:

“What is the IP for google.com?”

Root server replies:

“I don’t know google.com, but ask .com TLD servers”

Important:

Root servers do NOT store domain mappings.

They store directions.

---

### 10. Root Servers — The Internet’s Directory of Directories

Root servers know:

Where to find TLD servers.

They are the top of DNS hierarchy.

---

### 11. Step 5 — Query TLD Server

Resolver asks:

TLD Server (.com):

“What is the IP for google.com?”

TLD replies:

“I don’t know the IP, but ask Google’s authoritative name servers”

Again:

TLD servers provide referrals, not final answers.

---

### 12. Step 6 — Query Authoritative Name Server

Resolver asks:

Authoritative Server:

“What is google.com’s IP address?”

Authoritative server replies:

“Here is the IP address”

This is the definitive answer.

---

### 13. Why “Authoritative”?

Because this server owns the truth for that domain.

No guessing.

No referral.

Final answer.

---

### 14. Step 7 — Response Travels Back

Authoritative → Resolver → OS → Browser

Each layer may cache the response.

Improving future performance.

---

### 15. Visualizing the Full Query Flow

Client Perspective:

Browser  
→ Resolver  
→ Root  
→ TLD  
→ Authoritative  
→ Back to Resolver  
→ Back to Client

All within milliseconds.

---

### 16. Recursive vs Iterative Queries — Core Distinction

This is where deeper understanding begins.

---

### 17. Recursive Query — Delegated Responsibility

In recursive queries:

Client says:

“Resolver, give me final answer”

Resolver performs ALL steps.

Client waits.

Client does NOT talk to root/TLD servers.

---

### 18. Iterative Query — Step-by-Step Guidance

In iterative queries:

Requester asks server:

“What is google.com’s IP?”

Server replies:

“I don’t know, but ask this other server”

Requester then asks next server.

This continues until answer is found.

---

### 19. Who Uses Recursive Queries?

End-user devices:

• Browsers  
• Operating systems  
• Applications

They prefer simplicity.

---

### 20. Who Uses Iterative Queries?

DNS resolvers.

Resolvers interact with:

• Root servers  
• TLD servers  
• Authoritative servers

Resolvers follow referrals iteratively.

---

### 21. Gotcha — Users Rarely See Iterative Queries

From user perspective:

Everything appears recursive.

But internally:

Resolvers perform iterative lookups.

---

### 22. Why This Design Exists

Because:

• End devices stay lightweight  
• Resolvers centralize complexity  
• Infrastructure scales globally  
• Reduces redundant traffic

---

### 23. DNS Resolution Is Not a Single Request

Common misconception:

“One lookup → One server”

Reality:

Often multiple network hops.

---

### 24. Latency in DNS Resolution

Resolution time affected by:

• Cache hits vs misses  
• Network distance  
• Resolver performance  
• Server load

Cache hit → microseconds  
Full lookup → milliseconds

---

### 25. DNS Caching Layers — Performance Secret

Caching occurs at:

• Browser  
• OS  
• Router  
• Resolver

Why critical?

Because DNS queries are extremely frequent.

Without caching:

Internet would slow dramatically.

---

### 26. TTL — Governing Cache Behavior

Each DNS record includes:

TTL (Time To Live)

Meaning:

“How long can this answer be reused?”

Lower TTL:

• Faster updates  
• More queries

Higher TTL:

• Better speed  
• Slower updates

---

### 27. Gotcha — DNS Propagation Confusion

When records change:

Caches may retain old data.

Leads to:

“Inconsistent behavior across networks”

---

### 28. Negative Caching (Advanced Concept)

Even failures are cached.

Example:

“Domain not found”

Cached temporarily to reduce repeated load.

---

### 29. DNS Resolution Failures — Why They Happen

Possible causes:

• Network issues  
• Resolver failure  
• Authoritative server downtime  
• Misconfigured records  
• Expired domain

---

### 30. DNS Timeouts — Invisible Delays

If DNS stalls:

Everything stalls.

Because HTTP cannot begin without IP address.

DNS is prerequisite infrastructure.

---

### 31. Parallel Queries (Advanced Optimization)

Resolvers often send:

Multiple queries simultaneously.

Improves speed and reliability.

---

### 32. Load Balancing via DNS

Authoritative servers may return:

Different IPs per query.

Enables:

• Traffic distribution  
• Geographic optimization  
• Failover strategies

---

### 33. DNS and Anycast Routing

Multiple servers share same IP.

User automatically connects to nearest instance.

Improves latency and resilience.

---

### 34. DNS Resolution — Final Mental Model

DNS resolution is:

A **multi-step discovery process**

Where:

• Client delegates to resolver  
• Resolver queries hierarchy  
• Root provides direction  
• TLD provides direction  
• Authoritative provides truth  
• Result is cached  
• Connection proceeds

---

### 35. Elegant Big Picture

Typing:

    google.com

Triggers:

Cache checks → Resolver query → Hierarchical lookup → IP retrieval

All before website loads.

Usually unnoticed.

Always essential.

