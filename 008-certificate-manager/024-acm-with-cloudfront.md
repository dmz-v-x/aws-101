## ACM + CloudFront — Request/Import in us-east-1 and Attach to Distribution

### 1. What Are We Doing Here?
We will:
- Use an ACM certificate with CloudFront
- Understand the **us-east-1 requirement**
- Attach the certificate to a CloudFront distribution

Goal:
Client → HTTPS → CloudFront → Origin (S3 / ALB / EC2)

---

### 2. Important Rule (MUST REMEMBER)

CloudFront ONLY uses certificates from:

```
us-east-1 (N. Virginia)
```

Even if:
- Your backend is in another region

---

### 3. Why us-east-1?

CloudFront is a **global service**, but:
- Its control plane is tied to us-east-1
- It only looks for certificates in that region

---

### 4. Step 1 — Switch to us-east-1 Region

In AWS Console:
- Select region → **US East (N. Virginia)**

---

### 5. Step 2 — Request or Import Certificate

#### Option A: Request Public Certificate

1. Go to ACM  
2. Click **Request certificate**  
3. Add domain:

```
example.com
www.example.com
```

4. Choose:
→ DNS validation  

5. Complete validation  

---

#### Option B: Import Certificate

If you have external cert:

```bash
aws acm import-certificate \
  --certificate fileb://cert.pem \
  --private-key fileb://private.key \
  --certificate-chain fileb://chain.pem \
  --region us-east-1
```

---

### 6. Step 3 — Ensure Certificate Status = ISSUED

Check in ACM:
- Status must be **ISSUED**

Otherwise:
→ Cannot attach to CloudFront

---

### 7. Step 4 — Create or Edit CloudFront Distribution

1. Go to CloudFront  
2. Click:
   - **Create Distribution** OR  
   - Edit existing one  

---

### 8. Step 5 — Configure Domain (Alternate Domain Name)

Under:

**Alternate Domain Names (CNAMEs)**

Add:
```
example.com
www.example.com
```

---

### 9. Step 6 — Attach ACM Certificate

Under:

**Custom SSL Certificate**

Select:
→ Certificate from ACM (us-east-1)

Choose your certificate

---

### 10. Step 7 — Set Security Policy

Choose TLS version:
- Recommended: TLS 1.2 or higher

---

### 11. Step 8 — Deploy Distribution

Click:
→ Create / Save changes

Wait:
→ Deployment takes ~5–15 minutes

---

### 12. Step 9 — Update DNS

Point your domain to CloudFront:

Example (Route53):

```
example.com → CloudFront distribution domain
```

Usually:
- Use ALIAS (Route53)
- Or CNAME (external DNS)

---

### 13. Traffic Flow (Important)

```
Client → HTTPS
        ↓
CloudFront (TLS termination using ACM cert)
        ↓
Origin (S3 / ALB / EC2)
```

---

### 14. Origin Configuration (Optional HTTPS)

You can configure:

- HTTP → CloudFront → HTTP origin  
OR  
- HTTPS → CloudFront → HTTPS origin  

Best practice:
→ Use HTTPS end-to-end

---

### 15. Common Mistakes

#### Mistake 1: Wrong Region
- Certificate created outside us-east-1  
→ Not visible in CloudFront  

---

#### Mistake 2: Certificate Not ISSUED
→ Cannot select certificate  

---

#### Mistake 3: Missing Domain in Certificate
- Domain must match CloudFront CNAME  

---

#### Mistake 4: DNS Not Updated
→ Domain doesn’t point to CloudFront  

---

### 16. Verification

Check using browser:

- Visit https://example.com  
- Click lock icon  
- Verify certificate details  

---

### 17. CLI Insight (Attach via API)

CloudFront config uses certificate ARN:

```json
"ViewerCertificate": {
  "ACMCertificateArn": "arn:aws:acm:us-east-1:...",
  "SSLSupportMethod": "sni-only",
  "MinimumProtocolVersion": "TLSv1.2_2021"
}
```

---

### 18. Real-World Example

Scenario:
- React app hosted on S3  
- CloudFront as CDN  
- Custom domain example.com  

Steps:
1. Request cert in us-east-1  
2. Validate via DNS  
3. Attach to CloudFront  
4. Update DNS  

Result:
→ Secure global HTTPS website  

---

### 19. Why This Matters for ACM Mastery

CloudFront + ACM is:

- Most common production setup  
- Used for global apps  
- Critical for performance + security  

Understanding region rule:
→ Prevents major deployment issues  

---

### 20. Key Takeaways

- CloudFront requires cert in us-east-1  
- Request/import cert in that region only  
- Attach cert via distribution settings  
- Domain must match certificate  
- Always verify after deployment  
