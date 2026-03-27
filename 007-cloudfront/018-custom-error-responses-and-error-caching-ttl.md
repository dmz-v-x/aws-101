## Custom Error Responses and Error Caching TTL in CloudFront

### 1. Why Error Handling Matters

#### 1.1 Problem

When something goes wrong:

- Origin returns errors (4xx / 5xx)  
- Users see generic, ugly error pages  

---

#### 1.2 Impact

- Poor user experience  
- No branding  
- Confusing messages  

---

#### 1.3 Solution

CloudFront allows you to:

- Serve **custom error pages**  
- Control **error caching behavior (TTL)**  

---

### 2. Understanding Error Types

---

#### 2.1 4xx Errors (Client Errors)

- 400 → Bad request  
- 403 → Forbidden  
- 404 → Not found  

---

#### 2.2 5xx Errors (Server Errors)

- 500 → Internal error  
- 502 → Bad gateway  
- 503 → Service unavailable  
- 504 → Timeout  

---

#### 2.3 Key Insight

> 4xx = user/request issue  
> 5xx = server/origin issue  

---

### 3. What are Custom Error Responses?

---

#### 3.1 Definition

Custom error responses allow you to:

> Replace default CloudFront error pages with your own content

---

#### 3.2 Example

Instead of:

```
403 Forbidden (plain text)
```

You show:

```
/error-pages/403.html
```

---

#### 3.3 Benefits

- Better UX  
- Branding  
- Helpful messages  
- SEO-friendly pages  

---

### 4. How Custom Error Responses Work

---

#### 4.1 Flow

```
User Request
     ↓
CloudFront → Origin returns error (e.g., 404)
     ↓
CloudFront intercepts error
     ↓
Fetches custom error page
     ↓
Returns custom response to user
```

---

#### 4.2 Key Insight

> CloudFront can **override error responses before reaching the user**

---

### 5. Configuring Custom Error Responses

---

### 5.1 Step 1: Create Error Pages

Store files like:

```
/errors/404.html
/errors/500.html
```

In:
- S3 bucket  
- Or origin server  

---

### 5.2 Step 2: Configure in CloudFront

For each error code:

- Specify HTTP error code (e.g., 404)  
- Set custom response page path  
- Define response code (optional)  

---

### 5.3 Step 3: Choose Response Code

You can:

- Return original error code (404)  
- OR override (e.g., return 200)

---

#### Example

```
Error: 404  
Custom Page: /errors/404.html  
Response Code: 200
```

---

#### 5.4 Key Insight

> You can **mask errors** by returning different status codes

---

### 6. Error Caching TTL

---

#### 6.1 What is Error TTL?

Error TTL defines:

> How long CloudFront caches error responses

---

#### 6.2 Default Behavior

CloudFront caches errors for a default duration (usually a few seconds)

---

#### 6.3 Why Error Caching Exists

- Reduce repeated origin requests  
- Improve performance  

---

### 7. How Error TTL Works

---

#### 7.1 Flow

```
Error occurs
     ↓
CloudFront caches error
     ↓
Subsequent requests served from cache
     ↓
TTL expires → new origin request
```

---

#### 7.2 Example

```
Error TTL = 60 seconds
```

- First request → origin returns 503  
- Next 60 seconds → CloudFront serves cached error  
- After 60 sec → retry origin  

---

### 8. Configuring Error TTL

---

#### 8.1 Per Error Code

You can set TTL individually:

- 404 → 300 seconds  
- 503 → 10 seconds  

---

#### 8.2 Strategy

- Short TTL for temporary errors  
- Longer TTL for stable errors  

---

### 9. Choosing TTL Values (Important)

---

#### 9.1 4xx Errors (Client Errors)

Example:
- 404 (Not Found)

Recommended TTL:
- Medium to long (e.g., 300–600 sec)

---

#### 9.2 5xx Errors (Server Errors)

Example:
- 503 (Service Unavailable)

Recommended TTL:
- Short (e.g., 5–30 sec)

---

#### 9.3 Key Insight

> Cache server errors briefly to allow quick recovery

---

### 10. Real-World Example

---

#### Scenario: Website with Custom Errors

---

##### Setup

- `/errors/404.html`  
- `/errors/500.html`  

---

##### Configuration

| Error Code | Custom Page | TTL |
|----------|------------|-----|
| 404 | /errors/404.html | 300 sec |
| 500 | /errors/500.html | 10 sec |

---

##### Result

- Friendly user experience  
- Efficient caching  
- Fast recovery from failures  

---

### 11. Advanced Behavior

---

#### 11.1 Origin Failover + Error Pages

- Primary origin fails  
- Secondary origin serves fallback  
- Custom error page used if both fail  

---

#### 11.2 Serving Static Fallback Content

Example:
- API fails → show static "maintenance page"  

---

### 12. SEO Considerations

---

#### 12.1 Returning 200 for Errors

Bad practice if misused:
- Search engines think page is valid  

---

#### 12.2 Best Practice

- Keep correct status codes for SEO-critical pages  

---

### 13. Common Mistakes

---

#### Mistake 1: Long TTL for 5xx errors

- Delays recovery  

---

#### Mistake 2: Not using custom error pages

- Poor user experience  

---

#### Mistake 3: Incorrect error paths

- CloudFront cannot find error page  

---

#### Mistake 4: Returning 200 for all errors

- SEO issues  

---

### 14. Debugging Error Responses

---

#### 14.1 Check Headers

```
X-Cache
Status Code
```

---

#### 14.2 Verify Error Page Path

- Ensure file exists  
- Correct permissions  

---

#### 14.3 Test Origin Directly

- Confirm error source  

---

### 15. Best Practices

---

#### 15.1 Use Custom Error Pages

- Improve UX  

---

#### 15.2 Tune TTL Carefully

- Short for 5xx  
- Longer for 4xx  

---

#### 15.3 Store Error Pages in Reliable Origin

- Prefer S3  

---

#### 15.4 Avoid Masking Errors Incorrectly

- Preserve correct status codes when needed  

---

### 16. Final Mental Model

> CloudFront error handling has two layers:

1. **Custom Error Responses** → control what user sees  
2. **Error Caching TTL** → control how long errors persist  

Together, they help:
- Improve user experience  
- Reduce origin load  
- Maintain system resilience  
