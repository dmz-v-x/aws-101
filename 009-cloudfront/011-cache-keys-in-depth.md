## Cache Keys in Depth in CloudFront

### 1. What is a Cache Key (Revisited Deeply)

#### 1.1 Definition

A **cache key** is the exact combination of request attributes CloudFront uses to determine:

> Whether a cached response can be reused or a new request must go to the origin

---

#### 1.2 Mental Model

Think of cache key as:

```
Cache Key = Unique fingerprint of a request
```

If two requests produce the same fingerprint → **Cache HIT**  
If different → **Cache MISS**

---

### 2. Default Cache Key Components

---

#### 2.1 What CloudFront Uses by Default

By default, CloudFront cache key includes:

- **Host (Domain name)**
- **Path (URL path)**

---

#### 2.2 Example

Request:

```
https://example.com/images/logo.png
```

Cache key:

```
example.com + /images/logo.png
```

---

#### 2.3 What is NOT Included by Default

- Query strings ❌  
- Headers ❌  
- Cookies ❌  

---

#### 2.4 Key Insight

> Default cache key is **simple and optimized for static content**

---

### 3. Why Default Cache Key is Not Enough

---

#### 3.1 Problem

Some applications depend on:

- Query parameters  
- Headers  
- Cookies  

---

#### 3.2 Example Problem

```
/product?id=1
/product?id=2
```

If query strings are ignored:

→ Same cached response returned ❌

---

#### 3.3 Solution

Customize the cache key using **Cache Policies**

---

### 4. Custom Cache Policies

---

#### 4.1 What is a Cache Policy?

A **cache policy** defines:

- What goes into the cache key  
- TTL settings  
- Compression behavior  

---

#### 4.2 Why Use Cache Policies

- Fine-grained control  
- Reusable configuration  
- Separation of concerns  

---

### 5. Components You Can Add to Cache Key

---

### 5.1 Query Strings

---

#### Options

- None  
- All  
- Selected  

---

#### Example

Include only:

```
?id
```

Ignore others

---

#### Use Case

- APIs  
- Filtering data  

---

### 5.2 Headers

---

#### Options

- None  
- All  
- Selected  

---

#### Example

Include:

```
Accept-Language
```

---

#### Use Case

- Localization  
- Device-specific content  

---

#### Warning

Including too many headers:
- Creates too many cache entries  
- Reduces cache efficiency  

---

### 5.3 Cookies

---

#### Options

- None  
- All  
- Selected  

---

#### Example

Include:

```
session-id
```

---

#### Use Case

- Personalized content  
- Authenticated users  

---

#### Warning

Too many cookies:
- Break caching  
- Increase origin load  

---

### 6. Cache Key Explosion (Critical Concept)

---

#### 6.1 What is It?

When too many variations exist:

```
Different headers + cookies + queries → many cache keys
```

---

#### 6.2 Result

- Low cache hit ratio  
- High origin load  
- Increased cost  

---

#### 6.3 Key Insight

> More parameters in cache key = less reuse

---

### 7. Cache Policy vs Origin Request Policy

---

### 7.1 Core Difference

| Feature | Cache Policy | Origin Request Policy |
|--------|-------------|----------------------|
| Purpose | Define cache key | Define what is sent to origin |
| Affects caching | Yes | No |
| Controls request forwarding | Partially | Yes |

---

### 7.2 Why Separation Exists

You may want:

- To **forward data to origin**
- But **not include it in cache key**

---

### 8. Example to Understand Separation

---

#### Scenario

```
/api/user?id=123
```

---

#### Requirement

- Send `Authorization` header to origin  
- Do NOT include it in cache key  

---

#### Solution

- Cache Policy → exclude header  
- Origin Request Policy → include header  

---

#### Result

- Same cached response reused  
- Origin still receives required header  

---

### 9. TTL in Cache Policy

---

#### 9.1 TTL Controls

Cache policy defines:

- Minimum TTL  
- Default TTL  
- Maximum TTL  

---

#### 9.2 Combined Effect

TTL works together with:
- Cache-Control headers  
- Expires headers  

---

#### 9.3 Key Insight

> Cache policy defines **upper and lower bounds**, origin defines actual value

---

### 10. Example Cache Policy Designs

---

### 10.1 Static Assets Policy

- No headers  
- No cookies  
- No query strings  
- Long TTL (1 day or more)

---

### 10.2 API Policy

- Include query strings  
- Include selected headers  
- Short TTL  

---

### 10.3 Personalized Content Policy

- Include cookies  
- Include headers  
- Very low or zero TTL  

---

### 11. Cache Key Design Strategy

---

#### 11.1 Goal

Maximize:

- Cache hit ratio  
- Performance  

Minimize:

- Origin requests  
- Cost  

---

#### 11.2 Best Practices

- Include only necessary parameters  
- Avoid `Include All` unless required  
- Separate static and dynamic paths  
- Use different cache behaviors  

---

### 12. Real-World Example

---

#### Scenario

```
/products?category=electronics&sort=price
```

---

#### Good Cache Key

Include:
- `category`  
- `sort`  

Ignore:
- unnecessary params  

---

#### Result

- Efficient caching  
- Correct data delivery  

---

### 13. Common Mistakes

---

#### Mistake 1: Including all headers

Leads to:
- Cache fragmentation  

---

#### Mistake 2: Including all cookies

Leads to:
- Almost zero caching  

---

#### Mistake 3: Ignoring important query strings

Leads to:
- Incorrect responses  

---

#### Mistake 4: Mixing cache and origin logic

Leads to:
- Debugging complexity  

---

### 14. Debugging Cache Key Issues

---

#### 14.1 Symptoms

- Frequent cache MISS  
- Unexpected responses  
- High origin traffic  

---

#### 14.2 Tools

- Check `X-Cache` header  
- Analyze request variations  
- Review cache policy settings  

---

### 15. Final Mental Model

> Cache key determines **what makes a request unique**

- More parameters → more uniqueness → less caching  
- Fewer parameters → more reuse → better performance  

CloudFront caching works best when:
- Cache key is minimal  
- Origin request policy is precise  
- TTL is optimized  
