## Detect Expiring Certificates and Rotate Safely — Deploy New Cert, Update Resources, Retire Old

### 1. What Are We Solving?
Even though ACM auto-renews managed certificates, in real-world systems you still need to:

- Detect expiring certificates
- Rotate certificates safely
- Avoid downtime or security risks

This is especially important for:
- Imported certificates
- Compliance requirements
- Zero-downtime deployments

---

### 2. What is Certificate Rotation?
Rotation means:

Replacing an old certificate with a new one **before it expires**

Goal:
→ No downtime  
→ No broken HTTPS  
→ No user impact  

---

### 3. Step 1 — Detect Expiring Certificates

#### Option 1: AWS Console
- Go to ACM → Certificates
- Check "Expiration" column

---

#### Option 2: AWS CLI

```bash
aws acm list-certificates
```

Then:

```bash
aws acm describe-certificate \
  --certificate-arn <ARN> \
  --query "Certificate.NotAfter"
```

---

#### Option 3: Filter Expiring Certs

```bash
aws acm list-certificates \
  --query "CertificateSummaryList[].CertificateArn"
```

Then loop and check expiry dates

---

### 4. Monitoring Best Practice

Set alerts for:
- Certificates expiring in 30 days
- Certificates expiring in 15 days

Use:
- CloudWatch
- AWS Config rules
- Custom scripts

---

### 5. Step 2 — Request New Certificate

Before old cert expires:

- Request new certificate (same domain/SANs)

```bash
aws acm request-certificate \
  --domain-name example.com \
  --validation-method DNS
```

---

### 6. Step 3 — Validate New Certificate

- Add DNS validation record
- Wait until status = **ISSUED**

---

### 7. Step 4 — Deploy New Certificate (Safe Switch)

#### For ALB:

1. Go to Load Balancer → Listeners  
2. Edit HTTPS listener  
3. Add new certificate  
4. Set as default  

---

#### For CloudFront:

1. Edit distribution  
2. Select new certificate  
3. Deploy changes  

---

### 8. Zero-Downtime Rotation Strategy

Best practice:

- Attach BOTH old and new certificates (if supported)
- Gradually switch to new certificate
- Monitor traffic

---

### 9. Step 5 — Verify New Certificate in Production

Check using browser or CLI:

```bash
openssl s_client -connect example.com:443
```

Verify:
- New certificate is being served
- Correct expiry date

---

### 10. Step 6 — Retire Old Certificate

After confirming:

- Remove old certificate from service
- Delete from ACM (optional cleanup)

---

### 11. Rotation for Imported Certificates

Important:

- ACM does NOT auto-renew imported certs
- You must:
  1. Generate new cert externally
  2. Import into ACM
  3. Replace old cert

---

### 12. Automation Strategy (Advanced)

In production systems:

Automate:

1. Detect expiring certs  
2. Request new cert  
3. Validate via DNS  
4. Update resources  
5. Remove old cert  

Tools:
- Terraform
- AWS CDK
- CI/CD pipelines

---

### 13. Common Mistakes

- Waiting until last day to rotate  
- Not validating new cert before switching  
- Deleting old cert too early  
- Forgetting to update all services  
- Ignoring imported cert renewal  

---

### 14. Real-World Example

Scenario:
- Certificate expires in 20 days

Steps:
1. Request new cert  
2. Validate DNS  
3. Attach to ALB  
4. Test HTTPS  
5. Remove old cert  

No downtime  
Users unaffected  

---

### 15. When Rotation is Critical

- Imported certificates  
- Compliance (PCI, SOC2)  
- Key compromise  
- Changing CA/provider  

---

### 16. ACM Managed vs Rotation

Managed cert:
- ACM auto-renews → minimal rotation needed

Imported cert:
- Manual rotation → critical skill

---

### 17. Mental Model

Think:

Never replace instantly  
→ Always overlap old + new  

---

### 18. Why This Matters for ACM Mastery

In real production:

- Expired cert = outage  
- Bad rotation = downtime  

Mastering rotation:
→ Makes you production-ready engineer  

---

### 19. Key Takeaways

- Always detect expiry early  
- Rotate before expiration  
- Use zero-downtime strategy  
- Validate before switching  
- Clean up old certificates  
- Automate in production systems  
