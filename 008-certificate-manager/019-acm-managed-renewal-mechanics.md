## ACM Managed Renewal Mechanics — Why DNS Validation is Recommended

### 1. What is Certificate Renewal?
Every SSL/TLS certificate has an expiry date.

Before it expires:
→ It must be renewed  
→ Otherwise HTTPS breaks (browser shows security warning)

---

### 2. What is ACM Managed Renewal?
ACM automatically renews **Amazon-issued (managed) certificates** for you.

You DO NOT need to:
- Generate new CSR
- Reinstall certificate
- Manually track expiry

ACM handles everything

---

### 3. When Does ACM Renew a Certificate?

ACM starts renewal:
→ **Before expiration (typically ~60 days prior)**

Process:
1. Attempt validation
2. Issue new certificate
3. Replace old certificate seamlessly

---

### 4. Key Requirement for Renewal

ACM can ONLY auto-renew if:

- Certificate is **in use** (attached to a service)
- Domain validation still passes

---

### 5. DNS Validation — Why It’s Recommended

If you used DNS validation:

- ACM already has DNS record
- It reuses SAME record for renewal
- No human action required

Result:
→ Fully automatic renewal

---

### 6. Email Validation — Problem with Renewal

If you used email validation:

- ACM sends renewal emails
- You must approve manually

If you miss email:
→ Certificate expires

---

### 7. Renewal Flow (DNS Validation)

Step-by-step:

1. Certificate is nearing expiry  
2. ACM checks DNS validation record  
3. If record exists → validation succeeds  
4. ACM issues new certificate  
5. Updates attached services  

No downtime  
No manual steps  

---

### 8. Renewal Flow (Email Validation)

1. ACM sends validation emails  
2. You must click approval link  
3. If approved → certificate renewed  
4. If ignored → certificate expires  

---

### 9. Important Condition — Certificate Must Be In Use

ACM renews only if:

- Certificate is attached to:
  - ALB
  - CloudFront
  - API Gateway

If NOT in use:
→ ACM may NOT renew

---

### 10. What Happens After Renewal?

- New certificate is deployed automatically
- Services continue using updated certificate
- No interruption in HTTPS traffic

---

### 11. DNS Record Must Not Be Deleted

Critical rule:

DO NOT delete validation CNAME record

If deleted:
- Renewal fails
- Certificate expires

---

### 12. How to Check Renewal Status

Using CLI:

```bash
aws acm describe-certificate \
  --certificate-arn <ARN>
```

Look for:
- RenewalEligibility
- Status

---

### 13. Common Renewal Failures

#### Problem 1: DNS record deleted
→ Recreate record

#### Problem 2: Domain no longer valid
→ Update certificate

#### Problem 3: Certificate not in use
→ Attach to service

#### Problem 4: Email validation not approved
→ Switch to DNS validation

---

### 14. Best Practices (Very Important)

1. Always use DNS validation  
2. Never delete validation records  
3. Keep certificates attached to services  
4. Monitor certificate status  
5. Use Route53 for easiest automation  

---

### 15. Real-World Example

You create certificate for:
```
example.com
```

You add DNS record:
```
_abc.example.com → _xyz.acm-validations.aws
```

After ~11 months:

- ACM checks DNS record  
- Valid → renews automatically  
- Your app keeps working  

---

### 16. Migration Tip (Email → DNS)

If you used email validation:

Best practice:
→ Recreate certificate using DNS validation

Why?
→ Avoid future manual renewals

---

### 17. Why This Matters for ACM Mastery

In production:
- Expired cert = downtime
- Downtime = business impact

DNS validation ensures:
→ Zero-maintenance certificate lifecycle

---

### 18. Mental Model

Think like this:

DNS validation = Permanent proof  
Email validation = Temporary approval  

---

### 19. Key Takeaways

- ACM auto-renews managed certificates
- Renewal starts before expiry (~60 days)
- DNS validation enables full automation
- Email validation requires manual approval
- Certificate must remain in use
- Never delete validation DNS record
