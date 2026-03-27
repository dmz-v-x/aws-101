## ACM Quotas & Common Limits — Where to Check and How to Request Increase

### 1. What are Quotas in AWS?
Quotas (also called limits) define:
- How many resources you can create
- How frequently you can perform certain actions

In ACM:
Quotas control how many certificates you can have and how you can use them

---

### 2. Why Quotas Matter in ACM
Without understanding quotas, you may face:

- Certificate request failures
- Deployment issues in production
- Scaling limitations in multi-domain systems

So:
→ Knowing limits is essential for real-world systems

---

### 3. Types of ACM Quotas

There are two main categories:

1. Resource Limits (how many certs you can have)  
2. API Rate Limits (how frequently you can call APIs)  

---

### 4. Common ACM Quotas (Important Ones)

#### 4.1 Certificates per Region
- Default: ~2000 certificates per account per region

This includes:
- Public certificates
- Imported certificates

---

#### 4.2 Domain Names per Certificate (SAN limit)
- Default: up to 100 domain names per certificate

Example:
- example.com
- www.example.com
- api.example.com
... up to 100

---

#### 4.3 Certificate Requests per Year
- Soft limit on how many certificates you can request annually

Used to prevent abuse (especially for public certs)

---

#### 4.4 API Rate Limits
- Limits on:
  - RequestCertificate
  - DescribeCertificate
  - ListCertificates

If exceeded:
→ You get throttling errors

---

#### 4.5 Validation Attempts
- Limited retries for domain validation

Too many failures:
→ Request may be blocked temporarily

---

### 5. ACM Private CA Quotas (Additional)

If using Private CA:

- Number of CAs per account
- Certificates issued per CA
- Revocation list limits

These are separate from public ACM quotas

---

### 6. Where to Check ACM Quotas

#### Option 1: AWS Service Quotas Console

Steps:
1. Go to AWS Console  
2. Search: "Service Quotas"  
3. Select "AWS Certificate Manager"  
4. View all limits  

---

#### Option 2: Direct ACM Documentation
- Lists all quotas and default values

---

#### Option 3: AWS CLI

```bash
aws service-quotas list-service-quotas \
  --service-code acm
```

---

### 7. Identifying When You Hit a Limit

Common signs:

- Certificate request fails
- Error like: "LimitExceededException"
- API returns throttling error

---

### 8. How to Request a Quota Increase

#### Step-by-Step (Console)

1. Go to "Service Quotas"
2. Select ACM
3. Click on the quota you want to increase
4. Click "Request quota increase"
5. Enter desired value
6. Submit request

---

### 9. Approval Process

- AWS reviews your request
- May approve automatically or manually
- Time:
  - Minutes to hours (sometimes longer)

---

### 10. Best Practices for Quota Management

1. Plan ahead for scaling  
2. Monitor certificate usage  
3. Avoid unnecessary certificate duplication  
4. Use SANs instead of multiple certs  
5. Clean up unused certificates  

---

### 11. Optimization Strategy (Important)

Instead of:
- Creating 100 certificates for 100 subdomains

Do:
- Use 1 certificate with 100 SAN entries

---

### 12. Multi-Region Consideration

Remember:
- Quotas are **per region**

So:
- ap-south-1 → separate limit  
- us-east-1 → separate limit  

---

### 13. Automation Consideration

If using Terraform/CDK:

- Add retry logic for API limits  
- Avoid mass certificate creation in loops  
- Handle throttling gracefully  

---

### 14. Common Mistakes

- Ignoring limits until production failure  
- Creating too many duplicate certs  
- Not using SAN efficiently  
- Not requesting quota increase early  

---

### 15. Real-World Scenario

You deploy a SaaS app:

- Each customer gets subdomain
- You create certificate per customer

Problem:
→ You hit 2000 cert limit

Solution:
→ Use SAN or wildcard cert

---

### 16. Why This Matters for ACM Mastery

To work at scale, you must:

- Understand limits
- Design around them
- Automate quota monitoring

This is what differentiates:
Beginner vs Production-level engineer

---

### 17. Key Takeaways

- ACM has limits on cert count, domains, and API usage
- Quotas are per region
- Check via Service Quotas console or CLI
- You can request increases easily
- Good architecture avoids hitting limits
