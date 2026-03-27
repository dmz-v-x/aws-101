## Troubleshooting Common CloudFront Problems

### 1. Why Troubleshooting Matters

CloudFront sits between users and your origin, so issues can arise at multiple layers:

```
Viewer → CloudFront → Origin
```

Problems can come from:
- Cache behavior  
- Origin configuration  
- Security settings  
- Headers  

---

### 2. Understanding `X-Cache` Header (Very Important)

---

#### 2.1 What is `X-Cache`?

A response header that tells you:

> Whether the response came from cache or origin

---

#### 2.2 Common Values

- `Hit from cloudfront` → served from cache  
- `Miss from cloudfront` → fetched from origin  
- `Error from cloudfront` → something failed  

---

### 3. X-Cache: Error (Critical Issue)

---

#### 3.1 What It Means

```
X-Cache: Error from cloudfront
```

CloudFront could not successfully serve the request.

---

#### 3.2 Common Causes

- Origin unreachable  
- Permission denied  
- Invalid response from origin  
- SSL/TLS handshake failure  

---

#### 3.3 Debug Steps

1. Check origin availability  
2. Verify DNS resolution  
3. Check security groups / firewall  
4. Inspect CloudFront logs  
5. Test origin directly (curl/Postman)

---

#### 3.4 Example Debug

```
curl -I https://origin-domain.com/file
```

---

### 4. Certificate Issues

---

#### 4.1 Common Problems

- Certificate not attached  
- Wrong region (not us-east-1)  
- Domain mismatch  
- Expired certificate  

---

#### 4.2 Symptoms

- HTTPS not working  
- Browser security warnings  
- SSL handshake failure  

---

#### 4.3 Debug Steps

- Verify certificate in ACM (must be us-east-1)  
- Ensure domain is included in certificate  
- Check validation status (Issued)  
- Confirm CloudFront distribution is using correct cert  

---

#### 4.4 Key Insight

> CloudFront ONLY uses certificates from **us-east-1**

---

### 5. CORS Misconfiguration

---

#### 5.1 What is the Issue

Browser blocks request due to missing CORS headers.

---

#### 5.2 Symptoms

- Errors in browser console:
  ```
  Access-Control-Allow-Origin missing
  ```

---

#### 5.3 Common Causes

- Missing CORS headers at origin  
- Incorrect allowed origins  
- Preflight (OPTIONS) failure  

---

#### 5.4 Fix Steps

- Add headers at origin:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: *
```

---

#### 5.5 Preflight Issue

Ensure origin handles:

```
OPTIONS request
```

---

#### 5.6 Key Insight

> CORS is enforced by browser, not CloudFront

---

### 6. Cache-Control Conflicts

---

#### 6.1 Problem

Unexpected caching behavior:
- Content not updating  
- Content expiring too fast  

---

#### 6.2 Causes

- Conflicting TTL settings  
- Incorrect Cache-Control headers  
- CloudFront overrides  

---

#### 6.3 Example

```
Cache-Control: max-age=10
Minimum TTL: 60
```

Result:
→ Cached for 60 seconds (unexpected)

---

#### 6.4 Debug Steps

1. Inspect response headers  
2. Check cache policy settings  
3. Compare with origin headers  

---

#### 6.5 Key Insight

> CloudFront TTL settings can override origin headers

---

### 7. Origin Permission Errors (403 Access Denied)

---

#### 7.1 Common Scenario

```
403 Forbidden
```

---

#### 7.2 Causes (S3 Origin)

- Missing OAC/OAI permissions  
- Incorrect bucket policy  
- Public access blocked incorrectly  

---

#### 7.3 Example Bucket Policy Fix (OAC)

```
{
  "Effect": "Allow",
  "Principal": {
    "Service": "cloudfront.amazonaws.com"
  },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket/*",
  "Condition": {
    "StringEquals": {
      "AWS:SourceArn": "distribution ARN"
    }
  }
}
```

---

#### 7.4 Causes (Custom Origin)

- Security group blocking CloudFront  
- Firewall restrictions  
- Missing headers  

---

#### 7.5 Debug Steps

- Try accessing origin directly  
- Check bucket policy / server config  
- Verify OAC/OAI setup  

---

#### 7.6 Key Insight

> Most 403 errors are **permission misconfigurations**

---

### 8. 502 / 503 Errors

---

#### 8.1 What They Mean

- 502 → Bad gateway  
- 503 → Service unavailable  

---

#### 8.2 Causes

- Origin not responding  
- Timeout issues  
- Invalid origin response  

---

#### 8.3 Debug Steps

- Check origin health  
- Increase timeout settings  
- Verify origin response format  

---

### 9. DNS & Routing Issues

---

#### 9.1 Symptoms

- Domain not resolving  
- Incorrect endpoint reached  

---

#### 9.2 Causes

- Missing DNS record  
- Wrong CNAME/Alias  
- Propagation delay  

---

#### 9.3 Fix

- Verify DNS configuration  
- Use correct CloudFront domain  
- Wait for propagation  

---

### 10. Cache Issues (Stale Content)

---

#### 10.1 Problem

Old content still being served

---

#### 10.2 Causes

- High TTL  
- No invalidation  
- No versioning  

---

#### 10.3 Fix

- Invalidate cache  
- Use versioned files  
- Adjust TTL  

---

### 11. Debugging Tools

---

#### 11.1 Curl

```
curl -I https://your-domain.com/file
```

Check:
- `X-Cache`
- Headers  

---

#### 11.2 Browser DevTools

- Network tab  
- Inspect headers  
- Check CORS errors  

---

#### 11.3 CloudFront Logs

- Standard logs (S3)  
- Real-time logs  

---

#### 11.4 CloudWatch Metrics

- 4xx errors  
- 5xx errors  
- Cache hit rate  

---

### 12. Systematic Debugging Approach

---

#### Step 1: Identify Layer

- Viewer issue  
- CloudFront issue  
- Origin issue  

---

#### Step 2: Check Headers

- `X-Cache`  
- Cache-Control  
- CORS headers  

---

#### Step 3: Test Origin Directly

- Bypass CloudFront  

---

#### Step 4: Check Logs

- CloudFront logs  
- Origin logs  

---

### 13. Common Mistakes

---

#### Mistake 1: Debugging only CloudFront

- Issue may be at origin  

---

#### Mistake 2: Ignoring headers

- Headers reveal root cause  

---

#### Mistake 3: Not checking permissions

- Most issues are access-related  

---

#### Mistake 4: Forgetting cache layer

- Cache may hide updates  

---

### 14. Final Mental Model

> Troubleshooting CloudFront requires thinking in layers:

```
Viewer → CloudFront → Cache → Origin
```

To debug effectively:

- Identify where failure occurs  
- Inspect headers  
- Validate configurations  
- Test each layer independently  

Most issues fall into:
- Permissions  
- Caching  
- Configuration mismatches  
- Network connectivity  
