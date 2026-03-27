## Cache Key & Cache Behavior in CloudFront

### 1. What is a Cache Key?

#### 1.1 Definition

A **cache key** is the unique identifier CloudFront uses to decide:

> “Do I already have this exact response cached, or do I need to fetch it from the origin?”

---

#### 1.2 Simple Mental Model

Think of a cache key like a **fingerprint of a request**.

If two requests have the same fingerprint → same cached response is used  
If different → CloudFront treats them as different

---

#### 1.3 Default Components of Cache Key

By default, CloudFront uses:

- Host (domain)
- Path (URL path)

Example:

```
https://example.com/images/logo.png
```

Cache key (simplified):

```
example.com + /images/logo.png
```

---

### 2. Cache Key Components (Deep Dive)

You can control what goes into the cache key.

---

#### 2.1 Host

Represents the domain name.

Example:
```
app.example.com
api.example.com
```

Different hosts → different cache entries

---

#### 2.2 Path

Represents the URL path.

Example:
```
/images/logo.png
/videos/movie.mp4
```

Each unique path → separate cache entry

---

#### 2.3 Query Strings

Used for dynamic variations.

Example:
```
/product?id=1
/product?id=2
```

---

##### Behavior Options

- Ignore all query strings  
- Include all  
- Include selected  

---

##### Impact

If included:
- Each query variation → different cache entry

If ignored:
- Same response reused for all queries

---

#### 2.4 Headers

Used when response depends on request headers.

Examples:
- `Authorization`
- `User-Agent`
- `Accept-Language`

---

##### Use Case

Different language versions:

```
Accept-Language: en → English content  
Accept-Language: fr → French content
```

---

##### Important

Including too many headers:
- Reduces cache hit ratio
- Increases origin load

---

#### 2.5 Cookies

Used for personalization.

Example:
```
user=123
user=456
```

---

##### Behavior Options

- Ignore cookies  
- Include all  
- Include selected  

---

##### Use Case

- Logged-in vs guest users  
- A/B testing  

---

### 3. What is Cache Behavior?

#### 3.1 Definition

A **cache behavior** defines:

> How CloudFront handles requests for a specific path pattern

---

#### 3.2 Example

```
/images/* → long cache TTL  
/api/* → no cache  
```

---

#### 3.3 What Cache Behavior Controls

- Cache policy  
- Origin selection  
- Viewer protocol (HTTP/HTTPS)  
- Allowed HTTP methods  
- TTL settings  
- Headers, cookies, query string handling  

---

### 4. Cache Policy (Modern Approach)

#### 4.1 What is a Cache Policy?

A **cache policy** defines:

- What goes into cache key  
- TTL settings  
- Compression settings  

---

#### 4.2 Components of Cache Policy

- Headers to include  
- Cookies to include  
- Query strings to include  
- TTL values  

---

#### 4.3 Why Cache Policies Exist

To:
- Reuse configurations
- Standardize behavior
- Improve clarity

---

#### 4.4 Example

Policy for static assets:

- No cookies  
- No headers  
- Ignore query strings  
- TTL = long  

---

### 5. Origin Request Policy (Important Distinction)

#### 5.1 What it Does

Defines:
> What gets forwarded to the origin

---

#### 5.2 Difference from Cache Policy

| Aspect | Cache Policy | Origin Request Policy |
|------|-------------|----------------------|
| Purpose | Cache key | Forwarding |
| Affects caching | Yes | No |
| Sent to origin | Optional | Yes |

---

#### 5.3 Key Insight

> Not everything sent to origin should be part of cache key

---

### 6. Legacy Whitelist Behavior (Old System)

#### 6.1 What It Was

Earlier, CloudFront used:
- Header whitelist  
- Cookie whitelist  
- Query string forwarding  

---

#### 6.2 Problems

- Mixed caching + forwarding logic  
- Hard to manage  
- Less flexible  

---

#### 6.3 Modern Replacement

Now replaced by:
- Cache Policy  
- Origin Request Policy  

---

#### 6.4 Key Insight

> Always use modern policies — avoid legacy settings

---

### 7. TTL (Time To Live) in CloudFront

#### 7.1 Definition

TTL defines:
> How long a cached object remains valid

---

#### 7.2 Types of TTL

- Minimum TTL  
- Default TTL  
- Maximum TTL  

---

### 8. TTL vs Cache-Control Headers (CRITICAL)

This is one of the most important concepts.

---

#### 8.1 Origin-Controlled TTL

Origin can send:

```
Cache-Control: max-age=3600
```

Meaning:
- Cache for 1 hour

---

#### 8.2 CloudFront Behavior

CloudFront:

1. Reads origin headers  
2. Applies cache policy rules  
3. Enforces TTL within allowed range  

---

#### 8.3 Interaction Rules

##### Case 1: Header within limits

- Used directly

---

##### Case 2: Header < Minimum TTL

- CloudFront uses **Minimum TTL**

---

##### Case 3: Header > Maximum TTL

- CloudFront caps it at **Maximum TTL**

---

#### 8.4 Example

```
Cache-Control: max-age=10
Minimum TTL: 60
```

Result:
→ Cached for 60 seconds

---

#### 8.5 Another Example

```
Cache-Control: max-age=10000
Maximum TTL: 3600
```

Result:
→ Cached for 3600 seconds

---

### 9. Default TTL Behavior

#### 9.1 When No Headers Exist

If origin does NOT send caching headers:

CloudFront uses:
- Default TTL from cache policy

---

### 10. Cache-Control vs Expires

---

#### 10.1 Cache-Control (Modern)

```
Cache-Control: max-age=3600
```

Preferred method

---

#### 10.2 Expires (Legacy)

```
Expires: Wed, 21 Oct 2026 07:28:00 GMT
```

Older method, still supported

---

#### 10.3 Priority

```
Cache-Control > Expires
```

---

### 11. Cache Key Design Strategy

---

#### 11.1 Goal

Maximize:
- Cache hit ratio  
- Performance  

Minimize:
- Origin load  
- Cost  

---

#### 11.2 Best Practices

- Avoid unnecessary headers  
- Ignore cookies unless needed  
- Limit query parameters  
- Separate static vs dynamic paths  

---

### 12. Real-World Examples

---

#### 12.1 Static Assets

```
/images/logo.png
```

Cache key:
- Host + path only  

TTL:
- Long (1 day or more)  

---

#### 12.2 API Requests

```
/api/user?id=123
```

Cache key:
- Include query string  

TTL:
- Short or zero  

---

#### 12.3 Personalized Content

```
/dashboard
```

Cache key:
- Include cookies  

TTL:
- Very low or disabled  

---

### 13. Common Mistakes

---

#### Mistake 1: Including too many headers

Leads to:
- Cache fragmentation  
- Low hit ratio  

---

#### Mistake 2: Ignoring query strings when needed

Leads to:
- Wrong data served  

---

#### Mistake 3: Wrong TTL settings

Leads to:
- Stale content OR excessive origin load  

---

#### Mistake 4: Mixing cache and origin logic

Leads to:
- Hard-to-debug issues  

---

### 14. Final Mental Model

> CloudFront caching depends on two core things:

1. **Cache Key** → decides if content is reused  
2. **TTL + Headers** → decide how long content stays valid  

A well-designed cache system:
- Maximizes reuse  
- Minimizes origin calls  
- Delivers fast responses globally  
