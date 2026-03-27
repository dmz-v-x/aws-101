## Issuing Private Certificates via ACM — Requesting a Private Certificate

### 1. What Are We Doing Here?
We will:
- Use **AWS Private CA**
- Request a **private certificate via ACM**
- Understand how it gets issued and used

Goal:
Issue internal certificates like:
```
api.internal
service.company.local
```

---

### 2. How This Flow Works (Big Picture)

```
ACM → Requests certificate
        ↓
Private CA → Signs certificate
        ↓
ACM → Stores certificate
        ↓
You → Attach to services (ALB, etc.)
```

---

### 3. Prerequisites

You need:

- Private CA (Status = ACTIVE)  
- ACM access  
- IAM permissions:
  - acm:RequestCertificate
  - acm-pca:IssueCertificate  

---

### 4. Step 1 — Open ACM Console

1. Go to AWS Console  
2. Search: **Certificate Manager (ACM)**  
3. Click: Open ACM  

---

### 5. Step 2 — Click “Request Certificate”

You will see two options:

- Request a public certificate  
- Request a private certificate  

Select:
→ **Request a private certificate**

Click:
→ **Next**

---

### 6. Step 3 — Select Private CA

Choose:
→ Your **Private CA ARN**

This tells ACM:
→ Which CA should sign the certificate

---

### 7. Step 4 — Add Domain Name

Enter internal domain:

Examples:
```
api.internal
service.company.local
```

---

### 8. Step 5 — Add SANs (Optional)

You can add multiple domains:

```
api.internal
service.internal
db.internal
```

---

### 9. Step 6 — Choose Key Algorithm

Default:
- RSA 2048 (recommended)

---

### 10. Step 7 — Configure Additional Options

You can define:

- Certificate template  
- Key usage  
- Extended key usage  

Default template:
→ End-entity certificate (TLS server)

---

### 11. Step 8 — Review and Request

Click:
→ **Request**

---

### 12. Step 9 — Certificate Issued Instantly

Unlike public certs:

- No DNS validation  
- No email validation  

Because:
→ You control the CA  

Status:
→ **ISSUED immediately**

---

### 13. Step 10 — Use the Certificate

You can now:

- Attach to ALB (internal load balancer)  
- Use in EKS ingress  
- Use for mTLS setups  

---

### 14. CLI Example

```bash
aws acm request-certificate \
  --domain-name api.internal \
  --certificate-authority-arn <CA_ARN>
```

---

### 15. Important Difference from Public Certs

| Feature            | Public Cert           | Private Cert            |
|--------------------|----------------------|--------------------------|
| Validation         | DNS/Email required   | No validation            |
| Trust              | Public               | Internal only            |
| Issuance time      | Minutes              | Instant                  |

---

### 16. Where Can You Use Private Certificates?

- Internal ALB  
- EKS services  
- Microservices  
- Internal APIs  
- mTLS authentication  

---

### 17. Trust Requirement (Very Important)

Clients must trust your CA:

- Install root CA certificate  
- Configure trust store  

Otherwise:
→ Connection will fail  

---

### 18. Common Mistakes

- Using private cert for public website  
- Not configuring trust on clients  
- Wrong CA selected  
- Forgetting SAN entries  

---

### 19. Real-World Example

Scenario:
- Microservices in EKS  
- Need secure communication  

Steps:
1. Create Private CA  
2. Request private cert  
3. Attach to ALB  
4. Enable mTLS  

Result:
→ Secure internal system  

---

### 20. Mental Model

Public cert:
→ "Prove to the world"  

Private cert:
→ "Prove within my system"  

---

### 21. Why This Matters for ACM Mastery

Private certificate issuance is used in:

- Enterprise architectures  
- Kubernetes systems  
- Zero-trust networks  

---

### 22. Key Takeaways

- Private certs are issued via ACM + Private CA  
- No validation needed  
- Issued instantly  
- Used for internal systems  
- Clients must trust your CA  
```
