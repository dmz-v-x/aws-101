## Route 53 Interview Questions

## WHY ALIAS OVER CNAME?

---

### 1. Absolute Zero — What Problem Are We Comparing?

We are comparing:

**Alias Record vs CNAME Record**

Both appear to “point one name to another”.

But behave very differently.

---

### 2. Absolute Zero — What Is a CNAME?

CNAME means:

**Canonical Name**

Maps:

    domain → another domain

Example:

    api.example.com → myservice.amazonaws.com

---

### 3. Critical DNS Limitation of CNAME

CNAME **cannot be used at root domain**.

This is not AWS-specific.

This is DNS protocol rule.

---

### 4. Why CNAME Cannot Exist at Root

Root domain must contain:

• SOA record  
• NS records

CNAME conflicts with required records.

DNS standard restriction.

---

### 5. What Alias Record Solves

Alias records allow:

**Root domain → AWS resource mapping**

Without violating DNS rules.

---

### 6. Key Technical Advantage

Alias behaves like:

A record internally.

Resolvers receive:

Final IP resolution.

No extra DNS hop.

---

### 7. Performance Advantage

CNAME:

Requires additional lookup.

Alias:

Resolved directly by Route 53.

Reduced latency.

---

### 8. AWS Infrastructure Advantage

Alias integrates natively with:

• ALB / NLB  
• CloudFront  
• S3  
• API Gateway  
• Elastic Beanstalk  
• Others

Dynamic AWS endpoints supported.

---

### 9. Cost Advantage (Subtle but Important)

Alias queries to AWS resources:

Often free in Route 53.

CNAME queries:

Standard DNS pricing.

---

### 10. Interview-Grade Answer

Alias preferred over CNAME because:

• Works at root domain  
• No additional DNS lookup  
• AWS-native integration  
• Better performance  
• Cost optimization  
• Compatible with dynamic AWS infrastructure

---

## HOW FAILOVER WORKS INTERNALLY

---

### 11. Absolute Zero — What Is DNS Failover?

Failover routing means:

Return backup endpoint when primary unhealthy.

---

### 12. Critical Insight — DNS Does NOT “Move Traffic”

DNS only changes:

Returned answers.

No traffic rerouting occurs.

---

### 13. Health Check as Decision Engine

Failover depends on:

Health checks.

Primary Healthy → Return Primary  
Primary Unhealthy → Return Secondary

---

### 14. Internal Mechanism Simplified

Route 53 continuously:

Evaluates health state → Filters eligible records → Responds to queries accordingly.

---

### 15. Why This Is NOT Instantaneous

Two delays exist:

• Health check detection time  
• DNS cache (TTL) expiration

---

### 16. Resolver Caching Behavior

Resolvers may continue using:

Cached primary IP.

Until TTL expires.

---

### 17. Critical Interview Insight

Failover is:

**Resolution-layer protection**

Not connection-layer protection.

---

### 18. Comparison with Load Balancer Failover

DNS Failover:

Slow reaction (TTL governed).

Load Balancer:

Instant reaction (live traffic awareness).

---

### 19. Interview-Grade Answer

Failover works by:

Health-check-driven record eligibility filtering + DNS response selection + TTL/caching governed propagation.

---

## LATENCY VS GEOPROXIMITY ROUTING

---

### 20. Absolute Zero — Why This Question Matters

Many candidates confuse:

Latency routing ↔ Geoproximity routing.

They solve different problems.

---

### 21. Latency-Based Routing — Core Principle

Decision based on:

**Measured network latency**

Not physical distance.

---

### 22. How AWS Determines Latency

AWS uses:

Real traffic measurement data between regions & users.

Continuously updated models.

---

### 23. Geoproximity Routing — Core Principle

Decision based on:

**Geographic distance**

Plus optional bias.

---

### 24. Fundamental Difference

Latency Routing:

Performance-aware.

Geoproximity Routing:

Distance-aware.

---

### 25. Why Distance ≠ Latency

Latency depends on:

• Network topology  
• ISP routing  
• Peering agreements  
• Congestion  
• Backbone efficiency

---

### 26. Bias — Unique Geoproximity Feature

Geoproximity allows:

Artificial influence shifting.

Latency routing cannot bias geography.

---

### 27. When Latency Routing Preferred

Use when:

Performance optimization critical.

---

### 28. When Geoproximity Preferred

Use when:

Traffic shaping / gradual influence control required.

---

### 29. Interview-Grade Answer

Latency routing optimizes for:

Network performance.

Geoproximity routing optimizes for:

Geographic closeness + controllable bias.

---

## ROUTE 53 VS CLOUDFRONT ROUTING

---

### 30. Absolute Zero — Why This Confuses Candidates

Both appear to “route traffic”.

But operate at completely different layers.

---

### 31. Route 53 Routing — DNS Layer

Route 53 decides:

**Which endpoint clients discover**

Before connection begins.

---

### 32. CloudFront Routing — CDN / Edge Layer

CloudFront decides:

**Which edge location serves content**

After connection begins.

---

### 33. Critical Timing Difference

Route 53:

Pre-traffic decision.

CloudFront:

Active traffic optimization.

---

### 34. Nature of Decision

Route 53:

Returns DNS answers.

CloudFront:

Optimizes delivery path & caching.

---

### 35. Scope of Control

Route 53:

Endpoint selection.

CloudFront:

Content acceleration + caching + backbone routing.

---

### 36. Example Clarification

Route 53 may direct user to:

CloudFront distribution.

CloudFront then directs request to:

Nearest edge location.

Layered system.

---

### 37. Critical Interview Insight

Route 53 = Discovery Control Plane  
CloudFront = Delivery Optimization Plane

---

### 38. Interview-Grade Answer

Route 53 routing determines:

Where clients connect.

CloudFront routing determines:

How content is delivered optimally after connection.

Complementary mechanisms.

---

## COMMON INTERVIEW TRAPS

---

### 39. Trap #1 — “DNS Sends Traffic”

Incorrect.

DNS only influences connection targets.

---

### 40. Trap #2 — “Alias = CNAME”

Incorrect.

Alias = AWS-native resolution optimization.

---

### 41. Trap #3 — “Failover = Instant”

Incorrect.

TTL & detection delays apply.

---

### 42. Trap #4 — “Latency = Distance”

Incorrect.

Network behavior dominates.

---

### 43. Trap #5 — “Route 53 vs CloudFront = Competitors”

Incorrect.

Different layers of architecture.

---

## FINAL INTERVIEW MENTAL MODEL

---

### 44. Route 53 Core Role

DNS-based:

Discovery & traffic steering.

---

### 45. Alias Core Value

DNS-compatible AWS infrastructure abstraction.

---

### 46. Failover Core Mechanism

Health-driven eligibility filtering + TTL governed visibility.

---

### 47. Latency vs Geoproximity Core Distinction

Performance intelligence vs Distance intelligence.

---

### 48. Route 53 vs CloudFront Core Distinction

Resolution-layer routing vs Delivery-layer optimization.

---

### 49. Elegant Big Picture

Modern AWS traffic flow often involves:

DNS Decision → CDN Decision → Load Balancer Decision → Backend Decision

Each layer solves different class of problems.

