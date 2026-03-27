## Request Certificate via AWS Console — Step-by-Step (Public Cert + SANs)

### 1. What Are We Doing in This Step?
We will:
- Request a **public SSL/TLS certificate**
- Add **multiple domain names (SANs)**
- Use **DNS validation** (recommended)

Goal:
Get a certificate ready to attach to services like ALB or CloudFront

---

### 2. Prerequisites (Must Have Before Starting)

You need:
- AWS account
- Access to AWS Console
- A domain name (example.com)
- Access to DNS (Route53 or external provider like GoDaddy)

---

### 3. Step 1 — Open ACM Console

1. Go to AWS Console  
2. Search: **Certificate Manager**  
3. Click: **AWS Certificate Manager (ACM)**  

Make sure:
- You are in the correct region  
  - For CloudFront → use **us-east-1**

---

### 4. Step 2 — Click “Request Certificate”

You’ll see two options:

- Request a public certificate  
- Request a private certificate  

Select:
→ **Request a public certificate**

Click:
→ **Next**

---

### 5. Step 3 — Add Domain Names (Important)

Now you define:
Which domains this certificate should cover

#### Option A: Single Domain
```
example.com
```

#### Option B: Multiple Domains (SANs)
Click **“Add another name to this certificate”**

Example:
```
example.com
www.example.com
api.example.com
```

---

### 6. Step 4 — Wildcard Domains (Optional)

You can also use wildcard:

```
*.example.com
```

This covers:
- app.example.com
- api.example.com
- any subdomain

Note:
- Does NOT cover root domain automatically  
  → You must still add `example.com`

---

### 7. Step 5 — Choose Validation Method

You will see two options:

1. DNS validation (RECOMMENDED)  
2. Email validation  

Select:
→ **DNS validation**

Why?
- Easier
- Supports auto-renewal
- No email dependency

Click:
→ **Next**

---

### 8. Step 6 — Add Tags (Optional)

You can add tags like:

- Name = MyAppCert
- Environment = Production

This helps in:
- Organization
- Cost tracking
- Automation

Click:
→ **Next**

---

### 9. Step 7 — Review and Request

Check:
- Domain names (SANs)
- Validation method = DNS

Click:
→ **Request**

---

### 10. Step 8 — DNS Validation (Critical Step)

After request:

You will see:
- Status: Pending validation
- DNS records (CNAME)

Example:
```
_name.example.com → _xyz.acm-validations.aws
```

---

### 11. Step 9 — Add DNS Records

Go to your DNS provider:

#### If using Route53:
- Click **“Create records in Route53”** (auto option)

#### If using external DNS:
- Manually create CNAME record

Example:
- Name: `_name.example.com`
- Type: CNAME
- Value: `_xyz.acm-validations.aws`

---

### 12. Step 10 — Wait for Validation

- Takes: few seconds to few minutes
- ACM checks DNS record

Once verified:
→ Status becomes **Issued**

---

### 13. Step 11 — Certificate Ready to Use

Now you can:
- Attach to ALB
- Attach to CloudFront
- Use in API Gateway

---

### 14. Important Notes

- DNS validation enables **auto-renewal**
- Same DNS record can be reused across regions
- Certificate ARN is used to attach it to services

---

### 15. Common Mistakes

- Wrong region (especially for CloudFront)
- Missing DNS record
- Wrong record type (must be CNAME)
- Not waiting for DNS propagation
- Forgetting root domain when using wildcard

---

### 16. Quick Recap (Mental Flow)

1. Open ACM  
2. Request public cert  
3. Add domains (SANs)  
4. Choose DNS validation  
5. Add DNS record  
6. Wait → Issued  

---

### 17. Hands-On Task (Do This Now)

Try this:

- Request certificate for:
  - example.com
  - www.example.com

- Add DNS validation record
- Wait until status = **Issued**

---

### 18. Key Takeaways

- Public cert request is simple via console
- SANs allow multiple domains in one cert
- DNS validation is best practice
- Certificate becomes usable after validation
