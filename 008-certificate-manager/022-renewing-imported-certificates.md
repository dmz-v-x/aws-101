## Renewing Imported Certificates — Manual Re-Import Process

### 1. Why Imported Certificates Need Manual Renewal
When you **import a certificate into ACM**, AWS does NOT manage its lifecycle.

That means:
- No automatic renewal  
- No automatic rotation  
- No alerts by default  

You are responsible for:
→ Tracking expiry  
→ Renewing certificate  
→ Re-importing into ACM  

---

### 2. Key Difference (Very Important)

| Feature                | ACM Managed Cert | Imported Cert        |
|------------------------|------------------|---------------------|
| Renewal                | Automatic        | Manual              |
| Private key access     | No               | Yes                 |
| Responsibility         | AWS              | You                 |

---

### 3. High-Level Renewal Flow

1. Detect certificate is expiring  
2. Generate new private key + CSR  
3. Get new certificate from CA  
4. Import new certificate into ACM  
5. Update AWS resources  
6. Remove old certificate  

---

### 4. Step 1 — Detect Expiring Certificate

Use CLI:

```bash
aws acm describe-certificate \
  --certificate-arn <OLD_CERT_ARN> \
  --query "Certificate.NotAfter"
```

Best practice:
→ Start renewal at least 30 days before expiry

---

### 5. Step 2 — Generate New Key and CSR

```bash
openssl genrsa -out private.key 2048
```

```bash
openssl req -new -key private.key -out request.csr
```

---

### 6. Step 3 — Get Certificate from CA

Send CSR to:
- External CA (DigiCert, Let’s Encrypt, etc.)

Receive:
- Certificate (cert.pem)
- Certificate chain (chain.pem)

---

### 7. Step 4 — Import New Certificate into ACM

```bash
aws acm import-certificate \
  --certificate fileb://cert.pem \
  --private-key fileb://private.key \
  --certificate-chain fileb://chain.pem
```

Output:
```json
{
  "CertificateArn": "arn:aws:acm:region:account:certificate/NEW_CERT_ID"
}
```

---

### 8. Important Behavior — New ARN Created

When you re-import:

- ACM creates a NEW certificate
- OLD certificate is NOT replaced automatically

So:
→ You must update resources manually

---

### 9. Step 5 — Update AWS Resources

Attach new certificate:

#### For ALB:
- Go to Listener → HTTPS
- Replace old cert with new cert ARN

#### For CloudFront:
- Edit distribution
- Select new certificate
- Deploy changes

---

### 10. Step 6 — Verify New Certificate

Check using:

```bash
openssl s_client -connect example.com:443
```

Verify:
- New expiry date
- Correct certificate chain

---

### 11. Step 7 — Remove Old Certificate

After verification:

- Detach old certificate
- Delete from ACM (optional but recommended)

---

### 12. Zero-Downtime Strategy (Important)

Best practice:

1. Import new certificate  
2. Attach alongside old certificate (if possible)  
3. Switch traffic  
4. Remove old certificate  

---

### 13. Automation Strategy (Advanced)

In production:

Automate:
- Expiry detection  
- CSR generation  
- Certificate issuance  
- Import process  
- Resource update  

Tools:
- CI/CD pipelines  
- Terraform / CDK  
- Scripts (boto3, Node.js)

---

### 14. Common Mistakes

- Waiting until certificate expires  
- Overwriting files incorrectly  
- Using wrong private key  
- Forgetting to update resources  
- Not verifying new certificate  

---

### 15. Security Best Practices

- Protect private key (`chmod 600 private.key`)  
- Rotate keys during renewal  
- Never reuse compromised keys  
- Store keys securely  

---

### 16. Real-World Example

Scenario:
- Imported cert expires in 20 days  

Steps:
1. Generate new CSR  
2. Get cert from CA  
3. Import into ACM  
4. Update ALB  
5. Verify HTTPS  
6. Delete old cert  

---

### 17. When You Must Use Imported Certificates

- External CA required  
- Need private key access  
- Hybrid cloud/on-prem usage  
- Compliance requirements  

---

### 18. Mental Model

Think:

Managed cert → "Set and forget"  
Imported cert → "You own everything"  

---

### 19. Why This Matters for ACM Mastery

In real systems:
- Many orgs use external CAs  
- Manual renewal is common  

If you don’t master this:
→ You risk outages due to expired certs  

---

### 20. Key Takeaways

- Imported certificates require manual renewal  
- New certificate = new ARN  
- Must update resources manually  
- Always rotate before expiry  
- Automation is critical at scale  
