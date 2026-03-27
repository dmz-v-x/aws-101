## Caching Dynamic Content in CloudFront

### 1. What is Dynamic Content?

#### 1.1 Definition

Dynamic content is content that:

- Changes frequently  
- Depends on user input  
- Varies based on request context  

---

#### 1.2 Examples

- API responses  
- User dashboards  
- Search results  
- Personalized recommendations  

---

#### 1.3 Key Insight

> Dynamic content is **not always uncacheable** — it can be cached intelligently

---

### 2. Why Cache Dynamic Content?

---

#### 2.1 Benefits

- Reduced origin load  
- Faster response times  
- Lower infrastructure cost  

---

#### 2.2 Challenge

Dynamic content varies based on:

- Headers  
- Cookies  
- Query parameters  

---

#### 2.3 Solution

Control caching using:
- Cache key design  
- Cache policies  
- TTL strategies  

---

### 3. Caching Based on Query Strings

---

#### 3.1 Use Case

```
/products?category=electronics
/products?category=clothing
```

---

#### 3.2 Configuration

Include relevant query parameters in cache key.

---

#### 3.3 Strategy

- Include only required parameters  
- Ignore unnecessary ones  

---

#### 3.4 Key Insight

> Query strings allow caching of **filtered dynamic content**

---

### 4. Caching Based on Headers

---

#### 4.1 Use Case

```
Accept-Language: en
Accept-Language: fr
```

---

#### 4.2 Configuration

Include specific headers in cache key.

---

#### 4.3 Common Headers

- `Accept-Language`  
- `Authorization` (careful usage)  
- `User-Agent`  

---

#### 4.4 Key Insight

> Headers enable caching for **localized or device-specific content**

---

### 5. Caching Based on Cookies

---

#### 5.1 Use Case

```
user=123
user=456
```

---

#### 5.2 Configuration

Include selected cookies in cache key.

---

#### 5.3 Strategy

- Include only necessary cookies  
- Avoid session-wide cookies  

---

#### 5.4 Key Insight

> Cookies enable caching for **user-specific variations**

---

### 6. Combining Parameters

---

#### 6.1 Example

```
/products?category=electronics
Header: Accept-Language=en
Cookie: user-type=premium
```

---

#### 6.2 Cache Key

Includes:
- Query string (`category`)  
- Header (`Accept-Language`)  
- Cookie (`user-type`)  

---

#### 6.3 Result

- Different cache entries for each combination  

---

#### 6.4 Key Insight

> More parameters = more precise caching but lower reuse

---

### 7. Origin-Controlled Caching

---

#### 7.1 Cache-Control Header

Origin can define caching behavior:

```
Cache-Control: max-age=60
```

---

#### 7.2 Meaning

- Cache response for 60 seconds  

---

#### 7.3 Other Directives

- `no-cache`  
- `no-store`  
- `private`  
- `public`  

---

#### 7.4 Key Insight

> Origin defines **how long content should be cached**

---

### 8. CloudFront TTL Overrides

---

#### 8.1 What It Means

CloudFront can override origin headers using:

- Minimum TTL  
- Default TTL  
- Maximum TTL  

---

#### 8.2 Interaction Rules

---

##### Case 1: No Cache-Control Header

→ Use Default TTL  

---

##### Case 2: Header < Minimum TTL

→ Use Minimum TTL  

---

##### Case 3: Header > Maximum TTL

→ Use Maximum TTL  

---

#### 8.3 Example

```
Cache-Control: max-age=10
Minimum TTL: 60
```

Result:
→ Cached for 60 seconds  

---

#### 8.4 Key Insight

> CloudFront enforces boundaries on origin-defined caching

---

### 9. Dynamic Caching Strategies

---

### 9.1 Short TTL Caching

- Cache for few seconds (e.g., 5–60 sec)  
- Reduces origin load  
- Keeps data relatively fresh  

---

### 9.2 Stale-While-Revalidate (Conceptual)

- Serve cached response  
- Fetch new version in background  

---

### 9.3 Partial Personalization

- Cache shared parts  
- Fetch user-specific data separately  

---

### 10. API Caching Pattern

---

#### 10.1 Example

```
/api/products?category=electronics
```

---

#### 10.2 Strategy

- Include query string  
- Exclude user-specific headers  
- TTL = short  

---

#### 10.3 Result

- Same response reused for multiple users  
- Reduced backend load  

---

### 11. When NOT to Cache

---

#### 11.1 Cases

- Sensitive data (banking, auth tokens)  
- Highly personalized responses  
- Real-time data (stock prices)  

---

#### 11.2 Use Instead

- Disable caching (TTL = 0)  
- Forward all requests to origin  

---

### 12. Cache-Control Directives (Important)

---

#### 12.1 public

- Can be cached by CloudFront  

---

#### 12.2 private

- Should not be cached by shared caches  

---

#### 12.3 no-cache

- Must revalidate before serving  

---

#### 12.4 no-store

- Do not cache at all  

---

### 13. Real-World Example

---

#### Scenario: E-commerce API

```
/products?category=electronics
```

---

#### Configuration

- Include query string (`category`)  
- Exclude cookies  
- TTL = 30 seconds  

---

#### Result

- Fast responses  
- Reduced backend load  
- Acceptable freshness  

---

### 14. Performance Trade-offs

---

#### 14.1 High TTL

- Better performance  
- Risk of stale data  

---

#### 14.2 Low TTL

- Fresh data  
- Higher origin load  

---

### 15. Common Mistakes

---

#### Mistake 1: Including too many parameters

- Reduces cache efficiency  

---

#### Mistake 2: Not caching APIs at all

- Unnecessary load on backend  

---

#### Mistake 3: Ignoring TTL tuning

- Poor balance between freshness and performance  

---

#### Mistake 4: Caching sensitive data

- Security risk  

---

### 16. Best Practices

---

#### 16.1 Cache What Can Be Shared

- Public APIs  
- Product listings  

---

#### 16.2 Use Short TTLs for Dynamic Data

- 10–60 seconds  

---

#### 16.3 Minimize Cache Key Size

- Include only required parameters  

---

#### 16.4 Separate Static and Dynamic Paths

- `/static/*` vs `/api/*`  

---

### 17. Final Mental Model

> Dynamic content caching is about **controlled reuse**

You control it using:

- Cache key (what varies)  
- TTL (how long it lives)  
- Origin headers (intent)  

Correct balance ensures:
- Fast responses  
- Fresh data  
- Efficient backend usage  
