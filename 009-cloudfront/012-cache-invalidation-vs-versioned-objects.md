## Cache Invalidation vs Versioned Objects in CloudFront

### 1. Why This Topic Matters

#### 1.1 The Core Problem

CloudFront caches content to improve performance.

But when you update a file:

```
app.js (updated)
```

Users may still receive:
```
OLD cached version ❌
```

---

#### 1.2 Two Solutions

To handle updated content:

- **Cache Invalidation**
- **Versioned Objects**

---

#### 1.3 Key Insight

> This is a trade-off between **control vs efficiency**

---

### 2. Cache Invalidation

---

#### 2.1 What is Invalidation?

Invalidation means:

> Force CloudFront to remove specific files from cache immediately

---

#### 2.2 How It Works

1. You request invalidation  
2. CloudFront removes cached object  
3. Next request → fetched from origin  

---

#### 2.3 Example

Invalidate:

```
/app.js
/images/*
```

---

#### 2.4 Result

- Old cache removed  
- New content served  

---

### 3. Invalidation Flow

```
Cached Object Exists
        ↓
Invalidate Request
        ↓
Cache Removed
        ↓
Next Request → Origin Fetch
```

---

### 4. Invalidation Quotas & Limits

---

#### 4.1 Free Tier

- **1000 invalidation paths per month** (free)

---

#### 4.2 Beyond Free Limit

- Charged per additional invalidation path  

---

#### 4.3 Concurrent Requests Limit

- Limited number of simultaneous invalidation requests  
- Large invalidations may take time  

---

#### 4.4 Wildcard Invalidation

Example:

```
/images/*
```

Counts as:
- **1 path** (not per file)

---

#### 4.5 Key Insight

> Invalidation is **not free at scale**

---

### 5. Problems with Invalidation

---

#### 5.1 Cost

- Frequent invalidations → higher cost  

---

#### 5.2 Performance Delay

- Takes time to propagate globally  

---

#### 5.3 Operational Overhead

- Requires manual or automated triggers  

---

#### 5.4 Not Scalable for Frequent Updates

- Especially bad for CI/CD pipelines  

---

### 6. Versioned Objects (Best Practice)

---

#### 6.1 What is Versioning?

Instead of replacing a file:

```
app.js
```

You create a new version:

```
app.v2.js
```

---

#### 6.2 How It Works

1. New file uploaded with new name  
2. CloudFront treats it as new object  
3. No cache conflict  

---

#### 6.3 Example

Old:

```
/app.js
```

New:

```
/app.abc123.js
```

---

#### 6.4 Result

- No invalidation needed  
- Immediate availability  

---

### 7. Versioning Flow

```
New File Uploaded
        ↓
New URL Requested
        ↓
Cache MISS (new object)
        ↓
Fetched from origin
        ↓
Cached normally
```

---

### 8. Types of Versioning

---

#### 8.1 Manual Versioning

```
app.v1.js
app.v2.js
```

---

#### 8.2 Hash-Based Versioning (Recommended)

```
app.3f4k2l.js
```

---

#### 8.3 Build Tool Integration

Tools like:
- Webpack  
- Vite  

Automatically generate hashed filenames

---

### 9. Invalidation vs Versioning (Comparison)

---

| Feature | Invalidation | Versioning |
|--------|-------------|------------|
| Cost | Can increase | No extra cost |
| Speed | Delayed | Immediate |
| Complexity | Operational overhead | Simple once setup |
| Best for | Rare updates | Frequent updates |

---

### 10. When to Use Invalidation

---

Use invalidation when:

- File name cannot change  
- Critical fix needed immediately  
- Small number of files  

---

### 11. When to Use Versioning

---

Use versioning when:

- Frequent deployments  
- CI/CD pipelines  
- Static assets (JS, CSS, images)  

---

### 12. Best Practice Strategy

---

#### 12.1 Combine Both

- Use **versioning for most cases**  
- Use **invalidation for emergencies**  

---

#### 12.2 Example Strategy

- Static files → versioned  
- HTML entry file → invalidated  

---

### 13. Real-World Example

---

#### Scenario: React App Deployment

---

##### Step 1: Build generates:

```
app.abc123.js
style.xyz456.css
```

---

##### Step 2: Upload to S3

---

##### Step 3: Update HTML file

```
index.html → references new file names
```

---

##### Step 4: Invalidate only:

```
/index.html
```

---

#### Result

- New assets loaded  
- Minimal invalidation cost  

---

### 14. Common Mistakes

---

#### Mistake 1: Using invalidation for every deploy

- Expensive  
- Slow  

---

#### Mistake 2: Not versioning assets

- Leads to stale content  

---

#### Mistake 3: Invalidating entire distribution

```
/*
```

- Expensive  
- Unnecessary  

---

#### Mistake 4: Forgetting HTML cache

- Old HTML points to old assets  

---

### 15. Cost Optimization Tips

---

#### 15.1 Use Wildcards Smartly

```
/assets/*
```

Instead of multiple paths  

---

#### 15.2 Minimize Invalidations

- Only invalidate what's needed  

---

#### 15.3 Prefer Versioning

- Avoid repeated invalidations  

---

### 16. Final Mental Model

> There are two ways to update cached content:

- **Invalidation** → Remove old content  
- **Versioning** → Create new content  

Best practice:

> Never fight the cache — **work with it using versioning**

Use invalidation only when:
- Necessary  
- Minimal  
- Controlled  
