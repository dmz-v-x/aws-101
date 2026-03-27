## HTTPS, TLS Policies, Custom Domain & ACM in CloudFront

### 1. Why HTTPS is Required

#### 1.1 What HTTPS Provides

HTTPS ensures:
- Data encryption  
- Data integrity  
- Server authentication  

---

#### 1.2 Why It Matters in CloudFront

Without HTTPS:
- Data can be intercepted  
- Credentials can be exposed  
- Browsers may block requests  

---

#### 1.3 Key Insight

> HTTPS is mandatory for any production-grade application

---

### 2. TLS (Transport Layer Security)

---

#### 2.1 What is TLS?

TLS is the protocol that:
- Secures communication over HTTPS  
- Encrypts data between client and server  

---

#### 2.2 TLS Versions

- TLS 1.0 → Deprecated  
- TLS 1.1 → Deprecated  
- TLS 1.2 → Standard  
- TLS 1.3 → Latest and fastest  

---

#### 2.3 Minimum TLS Version in CloudFront

You can configure:

- Minimum TLS version allowed for viewers  

---

#### 2.4 Example Policies

- TLSv1.2_2021 (recommended)  
- TLSv1.3 (modern clients)  

---

#### 2.5 Key Insight

> Higher TLS versions = better security but less compatibility with older devices

---

### 3. TLS Security Policies in CloudFront

---

#### 3.1 What is a TLS Policy?

A **TLS policy** defines:
- Allowed TLS versions  
- Allowed cipher suites  

---

#### 3.2 Common Policies

- TLSv1 → insecure (avoid)  
- TLSv1.2_2021 → recommended  
- TLSv1.3 → modern and fastest  

---

#### 3.3 What Happens During Connection

1. Client initiates HTTPS request  
2. TLS handshake begins  
3. Protocol version negotiated  
4. Secure channel established  

---

#### 3.4 Key Insight

> TLS policy controls **how secure your HTTPS connection is**

---

### 4. Custom Domain in CloudFront

---

#### 4.1 Default Domain

CloudFront gives:

```
d123abcd.cloudfront.net
```

---

#### 4.2 Why Use Custom Domain

Instead of:

```
d123abcd.cloudfront.net
```

You use:

```
www.example.com
```

---

#### 4.3 Benefits

- Branding  
- Trust  
- SEO benefits  

---

#### 4.4 Requirement

To use custom domain:
> You MUST attach an SSL/TLS certificate

---

### 5. AWS Certificate Manager (ACM)

---

#### 5.1 What is ACM?

ACM is a service that:
- Issues SSL/TLS certificates  
- Manages renewals automatically  

---

#### 5.2 Key Requirement for CloudFront

> Certificate MUST be created in **us-east-1 (N. Virginia)**

---

#### 5.3 Why us-east-1?

CloudFront is a global service, but:
- It only reads certificates from us-east-1  

---

#### 5.4 Key Insight

> Wrong region = certificate not usable in CloudFront

---

### 6. Creating a Certificate in ACM

---

#### 6.1 Step 1: Request Certificate

- Go to ACM  
- Choose **Request a certificate**  
- Select **Public certificate**  

---

#### 6.2 Step 2: Add Domain Names

Examples:

```
example.com
www.example.com
*.example.com
```

---

#### 6.3 Step 3: Choose Validation Method

- DNS validation (recommended)  
- Email validation  

---

### 7. DNS Validation (Recommended)

---

#### 7.1 How It Works

ACM gives you:

- A DNS record (CNAME)

You must:
- Add it to your domain DNS

---

#### 7.2 Flow

1. ACM provides CNAME  
2. You add it in DNS (Route53 or other)  
3. ACM verifies ownership  
4. Certificate is issued  

---

#### 7.3 Benefits

- Automatic renewal  
- No manual action required later  

---

#### 7.4 Example

```
Name: _abc.example.com
Value: _xyz.acm-validations.aws
```

---

#### 7.5 Key Insight

> DNS validation is **fully automated and preferred**

---

### 8. Email Validation (Legacy Method)

---

#### 8.1 How It Works

ACM sends emails to:

- admin@domain.com  
- webmaster@domain.com  

---

#### 8.2 Flow

1. Receive email  
2. Click approval link  
3. Certificate issued  

---

#### 8.3 Drawbacks

- Manual process  
- Renewal requires re-validation  
- Email dependency  

---

#### 8.4 Key Insight

> Avoid email validation for production systems

---

### 9. Attaching Certificate to CloudFront

---

#### 9.1 Steps

1. Open CloudFront distribution  
2. Go to **Settings**  
3. Add **Alternate Domain Name (CNAME)**  
4. Select ACM certificate  
5. Save and deploy  

---

#### 9.2 Example

- Domain: `www.example.com`  
- Certificate: ACM-issued  

---

### 10. DNS Configuration for Custom Domain

---

#### 10.1 Add DNS Record

Point domain to CloudFront:

```
Type: CNAME or A (Alias)
Name: www.example.com
Value: d123abcd.cloudfront.net
```

---

#### 10.2 Route53 (Recommended)

Use:
- **A record (Alias)** instead of CNAME  

---

#### 10.3 Result

```
User → www.example.com → CloudFront
```

---

### 11. Enforcing HTTPS in CloudFront

---

#### 11.1 Viewer Protocol Policy

Options:

- HTTP and HTTPS  
- Redirect HTTP → HTTPS  
- HTTPS only  

---

#### 11.2 Best Practice

> Use **Redirect HTTP to HTTPS**

---

### 12. End-to-End Secure Setup

```
User → HTTPS → CloudFront → HTTPS → Origin
```

---

### 13. Common Mistakes

---

#### Mistake 1: Creating certificate in wrong region

- CloudFront cannot use it  

---

#### Mistake 2: Not validating certificate

- Certificate stays pending  

---

#### Mistake 3: Missing DNS configuration

- Domain does not resolve  

---

#### Mistake 4: Not enforcing HTTPS

- Security vulnerabilities  

---

### 14. Debugging Issues

---

#### 14.1 SSL Certificate Not Working

- Check region (must be us-east-1)  
- Check validation status  

---

#### 14.2 Domain Not Resolving

- Verify DNS records  
- Check propagation  

---

#### 14.3 HTTPS Errors

- Certificate mismatch  
- Domain not included in cert  

---

### 15. Best Practices

---

#### 15.1 Always Use ACM

- Free  
- Auto-renewed  
- Easy integration  

---

#### 15.2 Use DNS Validation

- Automated  
- Reliable  

---

#### 15.3 Enforce HTTPS

- Redirect all traffic  
- Use strong TLS policy  

---

#### 15.4 Use Modern TLS

- TLS 1.2 or higher  

---

### 16. Final Mental Model

> HTTPS in CloudFront is enabled by combining:

- TLS (encryption protocol)  
- ACM (certificate provider)  
- Custom domain (user-facing URL)  

CloudFront ensures:
- Secure delivery  
- Trusted identity  
- Encrypted communication across the entire request path  
