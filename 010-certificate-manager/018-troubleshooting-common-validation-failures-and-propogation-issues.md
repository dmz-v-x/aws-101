## Troubleshooting Common Validation Failures and DNS Propagation Issues

### 1. What Are We Troubleshooting?
When requesting a certificate in ACM, sometimes it stays in:

```
PENDING_VALIDATION
```

This means:
→ ACM cannot verify your domain ownership

Goal:
Identify WHY validation failed and FIX it step-by-step

---

### 2. High-Level Debugging Strategy (Mental Model)

Always think:

1. Did I create the correct DNS record?
2. Has it propagated?
3. Can ACM see it?
4. Is region/config correct?

---

### 3. Problem 1 — DNS Record Not Found

#### Symptom:
- Status: PENDING_VALIDATION
- `dig` shows no record

#### Cause:
- Record not created
- Wrong DNS provider
- Wrong hosted zone

#### Fix:
- Verify domain is using correct DNS provider
- Re-check where your domain’s nameservers point

---

### 4. Problem 2 — Wrong Record Type

#### Symptom:
- Record exists but validation fails

#### Cause:
- Used TXT instead of CNAME (ACM expects CNAME)

#### Fix:
- Delete wrong record
- Create correct CNAME record

---

### 5. Problem 3 — Incorrect Record Name

#### Symptom:
- Record exists but ACM still cannot validate

#### Cause:
- Missing underscore `_`
- Extra domain suffix added automatically

Example mistake:
```
Wrong: _abc.example.com.example.com
Correct: _abc.example.com
```

#### Fix:
- Enter exact name as provided by ACM
- Check how your DNS provider handles domain suffix

---

### 6. Problem 4 — Incorrect Record Value

#### Symptom:
- Record present but invalid

#### Cause:
- Typo in value
- Missing `.aws` suffix
- Quoted incorrectly

#### Fix:
- Copy-paste EXACT value from ACM
- Avoid manual typing

---

### 7. Problem 5 — DNS Propagation Delay

#### Symptom:
- Record looks correct locally
- ACM still not validating

#### Cause:
- DNS not propagated globally yet

#### Fix:
- Wait 5–15 minutes
- Lower TTL (60–300 seconds)

---

### 8. Problem 6 — High TTL

#### Symptom:
- Old records still being returned

#### Cause:
- High TTL (e.g., 3600 seconds)

#### Fix:
- Set TTL to low value (60–300)
- Wait for cache expiry

---

### 9. Problem 7 — Wrong Region (CloudFront Issue)

#### Symptom:
- Certificate not usable in CloudFront
- Not appearing in dropdown

#### Cause:
- Certificate created outside us-east-1

#### Fix:
- Recreate certificate in us-east-1

---

### 10. Problem 8 — Multiple DNS Zones

#### Symptom:
- Record created but not detected

#### Cause:
- Domain managed in different DNS provider than expected

Example:
- Domain registered in GoDaddy
- DNS actually hosted in Cloudflare

#### Fix:
- Check nameservers using:
```bash
dig NS example.com
```

---

### 11. Problem 9 — Validation for Multiple SANs

#### Symptom:
- Some domains validated, others not

#### Cause:
- Missing DNS record for one or more SANs

#### Fix:
- Add validation record for EACH domain

---

### 12. Problem 10 — Expired Validation Request

#### Symptom:
- Validation never completes
- Eventually fails

#### Cause:
- Too much delay
- Request expired

#### Fix:
- Request certificate again

---

### 13. Step-by-Step Debugging Checklist

#### Step 1 — Get Validation Record
```bash
aws acm describe-certificate \
  --certificate-arn <ARN>
```

---

#### Step 2 — Check DNS Record

```bash
dig CNAME _abc.example.com
```

Expected:
```
_xyz.acm-validations.aws
```

---

#### Step 3 — Check Nameservers

```bash
dig NS example.com
```

---

#### Step 4 — Wait for Propagation
- Wait at least 5–10 minutes

---

#### Step 5 — Re-check Status

```bash
aws acm describe-certificate \
  --certificate-arn <ARN> \
  --query "Certificate.Status"
```

---

### 14. Advanced Debugging Tips

- Use public DNS checkers (multiple locations)
- Try different networks (ISP caching issue)
- Flush local DNS cache:
```bash
ipconfig /flushdns   # Windows
sudo dscacheutil -flushcache  # macOS
```

---

### 15. Route53 Special Case (Easy Mode)

If using Route53:
- Use "Create records" button in ACM
- Avoid manual errors
- Validation is almost instant

---

### 16. Most Common Real-World Mistakes

1. Missing underscore `_`  
2. Wrong region (CloudFront)  
3. Wrong DNS provider  
4. Copy-paste errors  
5. Not waiting for propagation  

---

### 17. Mental Model (Golden Rule)

If ACM cannot validate:

→ Either DNS record is WRONG  
→ Or DNS record is NOT VISIBLE yet  

---

### 18. Why This Matters for ACM Mastery

In real systems:
- Most failures = DNS issues
- Debugging quickly saves hours

Mastering this makes you:
→ Reliable in production environments

---

### 19. Key Takeaways

- Validation failures are usually DNS-related
- Always verify using `dig` or `nslookup`
- Check record type, name, and value carefully
- Propagation takes time
- Region mistakes (especially CloudFront) are common
- Follow systematic debugging approach
