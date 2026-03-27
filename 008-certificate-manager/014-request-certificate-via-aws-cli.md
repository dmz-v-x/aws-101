## Request Certificate via AWS CLI — `aws acm request-certificate`

### 1. What Are We Doing Here?
We will request a **public SSL/TLS certificate** using the AWS CLI instead of the console.

Why CLI?
- Automation (CI/CD pipelines)
- Infrastructure as Code workflows
- Faster and scriptable

---

### 2. Prerequisites (Must Complete First)

You need:

1. AWS CLI installed  
   ```bash
   aws --version
   ```

2. AWS credentials configured  
   ```bash
   aws configure
   ```

3. Proper IAM permissions:
   - `acm:RequestCertificate`
   - `acm:DescribeCertificate`

4. A domain name (example.com)

---

### 3. Basic Command (Minimum Required)

```bash
aws acm request-certificate \
  --domain-name example.com \
  --validation-method DNS
```

---

### 4. What This Command Does

- Requests a **public certificate**
- Uses **DNS validation**
- Creates certificate in **current CLI region**

Output:
```json
{
  "CertificateArn": "arn:aws:acm:region:account:certificate/1234..."
}
```

Save this ARN → VERY IMPORTANT

---

### 5. Important: Region Matters

By default:
- CLI uses your configured region

To specify region:

```bash
aws acm request-certificate \
  --domain-name example.com \
  --validation-method DNS \
  --region us-east-1
```

Use:
- us-east-1 → for CloudFront
- your region → for ALB/API Gateway

---

### 6. Adding Multiple Domains (SANs)

You can include multiple domains:

```bash
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names www.example.com api.example.com \
  --validation-method DNS
```

---

### 7. Using Wildcard Domains

```bash
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names "*.example.com" \
  --validation-method DNS
```

---

### 8. Step 2 — Get DNS Validation Records

After requesting, certificate status = **PENDING_VALIDATION**

Now fetch DNS records:

```bash
aws acm describe-certificate \
  --certificate-arn <CERTIFICATE_ARN>
```

Look for:
```json
"DomainValidationOptions": [
  {
    "ResourceRecord": {
      "Name": "_abc.example.com",
      "Type": "CNAME",
      "Value": "_xyz.acm-validations.aws"
    }
  }
]
```

---

### 9. Step 3 — Add DNS Record

Go to your DNS provider:

Add CNAME:
```
Name:  _abc.example.com
Type:  CNAME
Value: _xyz.acm-validations.aws
```

---

### 10. Step 4 — Wait for Validation

ACM checks DNS record automatically.

Check status:

```bash
aws acm describe-certificate \
  --certificate-arn <CERTIFICATE_ARN> \
  --query "Certificate.Status"
```

Output:
```
"ISSUED"
```

---

### 11. Optional — Add Tags

```bash
aws acm request-certificate \
  --domain-name example.com \
  --validation-method DNS \
  --tags Key=Name,Value=MyCert Key=Env,Value=Prod
```

---

### 12. Full Example (Best Practice)

```bash
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names www.example.com api.example.com \
  --validation-method DNS \
  --region us-east-1 \
  --tags Key=Project,Value=MyApp
```

---

### 13. Common Mistakes

- Forgetting region (CloudFront requires us-east-1)
- Not saving CertificateArn
- Not adding DNS record
- Using wrong record type (must be CNAME)
- Not waiting for DNS propagation

---

### 14. Automation Insight (Important)

In production:
- You automate:
  - Certificate request
  - DNS record creation (Route53)
  - Validation polling

This is done using:
- Terraform
- AWS CDK
- Scripts (boto3 / Node.js SDK)

---

### 15. Real-World Flow (CLI-Based)

1. Run request command  
2. Get ARN  
3. Fetch validation records  
4. Create DNS record  
5. Wait → certificate becomes ISSUED  
6. Attach to service  

---

### 16. Why This Matters for ACM Mastery

CLI usage is critical for:
- DevOps workflows
- Automation pipelines
- Multi-region deployments

Without CLI:
→ You cannot scale certificate management

---

### 17. Key Takeaways

- `request-certificate` creates a public cert
- DNS validation is recommended
- Always store Certificate ARN
- Use `describe-certificate` to get validation details
- Region selection is critical
- CLI enables automation and scalability
