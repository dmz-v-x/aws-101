## Introduction to Cloudfront

### 1. What is a CloudFront Distribution?

#### 1.1 The Simplest Mental Model

Think of a **CloudFront Distribution** as:

> A global delivery system that sits between your users and your server (origin).

It does three important things:
- Receives user requests
- Caches content at edge locations
- Fetches from origin only when needed

#### Real-world analogy

- Your origin = kitchen (slow, far)
- CloudFront = local food delivery hubs (fast, near users)
- Users = customers

---

#### 1.2 What Exactly is a Distribution?

A **distribution** is a configuration object in CloudFront that defines:

- Where your content lives (origin)
- How it should be cached
- How it should be delivered
- How requests/responses should be handled

Without a distribution → CloudFront does nothing.

---

### 2. Types of CloudFront Distributions

CloudFront supports **two types of distributions**:

---

#### 2.1 Web Distribution (IMPORTANT — You will use this 99% of the time)

##### What it supports:
- HTTP / HTTPS traffic
- Static content (HTML, CSS, JS, images)
- Dynamic content (APIs, SSR apps)
- Streaming (modern protocols like HLS/DASH)

##### Used for:
- Websites
- APIs
- SPAs (React, Next.js)
- Media delivery

##### Key takeaway:
> Web Distribution = Default + Modern + Powerful

---

#### 2.2 RTMP Distribution (Legacy — Rarely Used)

##### What it was used for:
- Real-Time Messaging Protocol (RTMP)
- Flash-based video streaming

##### Why it's deprecated in practice:
- Flash is dead
- Modern streaming uses HTTP-based protocols

##### Key takeaway:
> Ignore RTMP for new systems — focus on Web Distribution

---

### 3. Core Components of a Distribution

Before lifecycle, you must understand what makes up a distribution.

---

#### 3.1 Origin

Your **source of truth** (where actual content lives)

Examples:
- S3 bucket
- EC2 server
- Load balancer
- API Gateway

---

#### 3.2 Cache Behavior

Defines:
- How requests are handled
- What gets cached
- Routing rules

Example:
- `/images/*` → cached aggressively
- `/api/*` → minimal caching

---

#### 3.3 Edge Locations

- Global AWS locations
- Serve cached content to users
- Reduce latency

---

#### 3.4 Viewer vs Origin Flow

Flow:
1. User → Edge location  
2. Edge checks cache  
3. If miss → fetch from origin  
4. Response cached + returned  

---

### 4. Distribution Lifecycle (VERY IMPORTANT)

This is the **heart of mastery**.

We will go step-by-step:

> Create → Deploy → Serve → Invalidate → Update → Delete

---

### 5. Step 1 — Create Distribution

#### 5.1 What happens during creation?

You define:
- Origin
- Cache rules
- Protocol (HTTP/HTTPS)
- Domain name
- Security settings

---

#### 5.2 Example (Conceptual)

You create a distribution like:

- Origin: S3 bucket  
- Default behavior: cache everything  
- Viewer protocol: HTTPS only  

---

#### 5.3 What AWS does internally

- Allocates distribution ID  
- Prepares edge network  
- Generates domain like:  
  `d123abcd.cloudfront.net`  

---

#### 5.4 Key Insight

At this point:
> Distribution exists but is NOT yet globally active

---

### 6. Step 2 — Deployment

#### 6.1 What is deployment?

Deployment = propagating configuration to all edge locations

---

#### 6.2 What happens internally?

- Config pushed globally  
- Edge nodes updated  
- DNS ready  

---

#### 6.3 Time required

Usually:
- 5 to 20 minutes  

---

#### 6.4 Status states

- In Progress  
- Deployed  

---

#### 6.5 Important concept

Until deployed:
> Requests may fail or behave inconsistently

---

### 7. Step 3 — Content Delivery (Serving Requests)

Now your distribution is LIVE.

---

#### 7.1 Request flow (critical understanding)

1. User hits:  
   `https://d123.cloudfront.net/image.png`  

2. CloudFront:
   - Finds nearest edge  
   - Checks cache  

3. If HIT:  
   → Returns instantly  

4. If MISS:  
   → Fetches from origin  
   → Stores in cache  
   → Returns response  

---

#### 7.2 Cache headers influence behavior

- `Cache-Control`  
- `Expires`  

These decide:
- How long content stays cached  

---

#### 7.3 Key learning

> Cache HIT = fast + cheap  
> Cache MISS = slower + costs origin resources  

---

### 8. Step 4 — Invalidation

#### 8.1 Problem

You updated a file:  
`app.js`  

But CloudFront still serves old version.

---

#### 8.2 Why?

Because:
> Cached version still valid  

---

#### 8.3 Solution: Invalidation

You tell CloudFront:
> "Remove this from cache immediately"

---

#### 8.4 Example

Invalidate:
- `/app.js`  
- `/images/*`  

---

#### 8.5 Important constraints

- Limited free invalidations  
- Costs beyond limit  

---

#### 8.6 Best practice

Instead of invalidation:
> Use versioned files  

Example:
- `app.v2.js`  

---

### 9. Step 5 — Update Distribution

#### 9.1 Why update?

To change:
- Origin  
- Cache rules  
- Headers  
- Security settings  

---

#### 9.2 Example updates

- Add new behavior `/api/*`  
- Change TTL  
- Enable compression  
- Attach SSL certificate  

---

#### 9.3 What happens after update?

Same as deployment:
- Global propagation again  

---

#### 9.4 Important insight

> Updates are NOT instant  

Always consider:
- Propagation delay  
- Cache consistency  

---

### 10. Step 6 — Delete Distribution

#### 10.1 When do you delete?

- System shutdown  
- Migration  
- Cleanup  

---

#### 10.2 Requirements before deletion

You MUST:
- Disable distribution first  

---

#### 10.3 Process

1. Disable distribution  
2. Wait for deployment  
3. Delete distribution  

---

#### 10.4 Why this is required?

To ensure:
- No active traffic  
- Safe removal across edge network  

---

### 11. Full Lifecycle Summary

```
CREATE → DEPLOY → SERVE → INVALIDATE → UPDATE → DELETE
```

---

### 12. Real-World Scenario (Putting It All Together)

Let’s say you deploy a website:

---

#### Step-by-step flow:

1. Create distribution with S3 origin  
2. Wait for deployment  
3. Users start accessing site  
4. Content gets cached globally  
5. You update frontend code  
6. Old cache still served  
7. You:
   - Either invalidate OR  
   - Use versioned files  
8. Later:
   - Update caching rules  
9. Eventually:
   - Disable and delete distribution  

---

### 13. Advanced Insights (Expert Level)

---

#### 13.1 Multiple Behaviors

You can define:

- `/images/*` → long cache  
- `/api/*` → no cache  

---

#### 13.2 Origin Groups

- Primary + failover origin  
- Improves reliability  

---

#### 13.3 Cache Key Optimization

Decide what affects cache:
- Headers  
- Cookies  
- Query params  

---

#### 13.4 Cost Optimization

- Increase cache hit ratio  
- Reduce invalidations  
- Compress content  

---

#### 13.5 Security Integration

- HTTPS enforcement  
- Signed URLs  
- WAF integration  

---

### 14. Common Mistakes (Very Important)

---

#### Mistake 1: Not understanding caching
Leads to:
- Stale content  
- Debugging nightmares  

---

#### Mistake 2: Overusing invalidations
Leads to:
- High costs  

---

#### Mistake 3: Wrong origin setup
Leads to:
- 403 errors  
- Latency issues  

---

#### Mistake 4: Ignoring propagation delays
Leads to:
- Confusion during updates  

---

### 15. Final Mastery Checklist

You have mastered distributions if you can:

- Explain how CloudFront serves content  
- Create a distribution from scratch  
- Configure cache behaviors correctly  
- Handle invalidations vs versioning  
- Update distributions safely  
- Debug cache HIT/MISS issues  
- Design multi-origin systems  
- Optimize for cost and performance  

---

### 16. Final Mental Model

> A CloudFront Distribution is a globally deployed caching and routing layer that intelligently sits between users and your origin, controlling how content is delivered, cached, secured, and optimized.
