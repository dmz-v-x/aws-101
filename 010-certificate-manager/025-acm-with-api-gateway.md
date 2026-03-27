## ACM + API Gateway (REST & HTTP) — Attach Certificates and Custom Domain Considerations

### 1. What Are We Doing Here?
We will:
- Attach an **ACM certificate** to API Gateway
- Use a **custom domain name** (e.g., api.example.com)
- Understand differences between REST API and HTTP API

Goal:
Client → HTTPS → Custom Domain → API Gateway → Backend

---

### 2. Why Do We Need ACM with API Gateway?

By default:
- API Gateway gives you a URL like:
```
https://abc123.execute-api.ap-south-1.amazonaws.com
```

But in production:
You want:
```
https://api.example.com
```

To enable HTTPS on your custom domain:
→ You must attach an ACM certificate

---

### 3. Two Types of API Gateway

1. REST API (older, more features)  
2. HTTP API (newer, simpler, cheaper)  

Both support custom domains + ACM certificates

---

### 4. Regional vs Edge-Optimized (Important Concept)

When creating custom domain:

#### Option 1: Regional Endpoint
- Traffic goes directly to API Gateway in that region
- Certificate must be in SAME region

#### Option 2: Edge-Optimized Endpoint
- Uses CloudFront internally
- Certificate MUST be in **us-east-1**

---

### 5. Step 1 — Request Certificate

#### For Regional API:
- Create certificate in SAME region as API

#### For Edge-Optimized API:
- Create certificate in **us-east-1**

---

### 6. Step 2 — Create Custom Domain

Go to:
- API Gateway Console → Custom Domain Names

Click:
→ **Create custom domain name**

---

### 7. Step 3 — Configure Domain

Enter:
```
api.example.com
```

Select:
- Endpoint type:
  - Regional OR Edge-Optimized

---

### 8. Step 4 — Attach ACM Certificate

Choose:
→ ACM certificate (must match domain)

Important:
- Certificate must be ISSUED
- Domain must match (SAN included)

---

### 9. Step 5 — Configure Base Path Mapping

Map domain to API:

Example:

```
api.example.com → / → MyAPI (stage: prod)
```

Meaning:
- Root path goes to your API

---

### 10. Step 6 — Deploy API

Ensure:
- API is deployed to a stage (e.g., prod)

---

### 11. Step 7 — Update DNS

Point domain to API Gateway:

#### For Regional:
- Use A record (ALIAS) → API Gateway domain

#### For Edge-Optimized:
- Use CloudFront domain provided

---

### 12. Traffic Flow

#### Regional:

```
Client → HTTPS
        ↓
API Gateway (regional)
        ↓
Backend (Lambda / service)
```

---

#### Edge-Optimized:

```
Client → HTTPS
        ↓
CloudFront (edge)
        ↓
API Gateway
        ↓
Backend
```

---

### 13. REST API vs HTTP API (Certificate Use)

| Feature          | REST API              | HTTP API             |
|------------------|----------------------|----------------------|
| Custom Domain    | Supported            | Supported            |
| ACM Cert         | Required             | Required             |
| Complexity       | Higher               | Lower                |
| Cost             | Higher               | Lower                |

---

### 14. Important Rules

1. Domain must match certificate  
2. Certificate must be in correct region  
3. Certificate must be ISSUED  
4. DNS must point correctly  

---

### 15. Common Mistakes

#### Mistake 1: Wrong Region
- Regional API → cert must be same region  
- Edge → cert must be us-east-1  

---

#### Mistake 2: Missing SAN
- Certificate doesn’t include api.example.com  

---

#### Mistake 3: DNS Not Updated
- Domain not pointing to API Gateway  

---

#### Mistake 4: API Not Deployed
- No stage → domain won’t work  

---

### 16. Verification

Test:

```bash
curl https://api.example.com
```

Or open in browser:
→ Check HTTPS lock icon  

---

### 17. CLI Example (Create Domain)

```bash
aws apigatewayv2 create-domain-name \
  --domain-name api.example.com \
  --domain-name-configurations CertificateArn=<CERT_ARN>
```

---

### 18. Real-World Example

Scenario:
- Backend: Lambda  
- API Gateway: HTTP API  
- Domain: api.example.com  

Steps:
1. Request cert in region  
2. Create custom domain  
3. Attach cert  
4. Map API  
5. Update DNS  

Result:
→ Secure API endpoint  

---

### 19. Why This Matters for ACM Mastery

API Gateway + ACM is used in:

- Serverless apps  
- Microservices  
- Public APIs  

Understanding region + domain mapping:
→ Prevents deployment failures  

---

### 20. Key Takeaways

- ACM cert is required for custom domains  
- Regional API → cert in same region  
- Edge API → cert in us-east-1  
- Must map domain to API stage  
- DNS configuration is critical  
