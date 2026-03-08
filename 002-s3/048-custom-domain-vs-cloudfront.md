## Using a Custom Domain with Amazon S3 and CloudFront: TLS/HTTPS, OAI and OAC Access Methods

### 1. Introduction

When hosting websites or distributing content stored in Amazon S3, organizations often want to use a **custom domain name** instead of the default AWS endpoints.

Example:

Default S3 endpoint:

```
https://bucket-name.s3.amazonaws.com
```

Custom domain:

```
https://www.example.com
```

To enable HTTPS, caching, and global distribution, Amazon **CloudFront** is typically placed in front of the S3 bucket.

CloudFront provides:

- TLS/HTTPS support
- global edge caching
- custom domain integration
- secure access control to S3

This architecture is widely used for production static websites and content delivery systems.

---

### 2. Basic Architecture

A typical architecture using CloudFront and S3 looks like this:

```
User
  ↓
DNS (example.com)
  ↓
CloudFront Distribution
  ↓
S3 Bucket (origin)
```

The S3 bucket stores the content, while CloudFront handles content delivery and HTTPS.

---

### 3. Why CloudFront Is Required for HTTPS

S3 static website endpoints do **not support HTTPS**.

Example website endpoint:

```
http://bucket-name.s3-website-us-east-1.amazonaws.com
```

Only HTTP is supported.

CloudFront solves this by terminating TLS connections.

Architecture:

```
User → HTTPS → CloudFront → S3
```

CloudFront provides secure TLS encryption for all client connections.

---

### 4. Custom Domain Configuration

To use a custom domain, CloudFront must be configured with an **alternate domain name (CNAME)**.

Example configuration:

```
Alternate Domain Name:
www.example.com
```

Once configured, CloudFront can serve content using that domain.

DNS must then point the domain to the CloudFront distribution.

---

### 5. DNS Configuration

DNS records connect the domain to CloudFront.

Example DNS record:

```
www.example.com → CloudFront distribution
```

In Amazon Route 53, this is typically an **Alias record**.

Example:

```
Record type: A
Alias target: CloudFront distribution
```

Other DNS providers can use CNAME records.

---

### 6. TLS Certificates

CloudFront requires an SSL/TLS certificate to serve HTTPS traffic.

Certificates are issued through **AWS Certificate Manager (ACM)**.

Example certificate domain:

```
www.example.com
```

After creating the certificate, it is attached to the CloudFront distribution.

CloudFront then uses the certificate to secure HTTPS connections.

---

### 7. HTTPS Connection Flow

Connection workflow:

```
User requests https://www.example.com
        ↓
DNS resolves to CloudFront
        ↓
CloudFront presents TLS certificate
        ↓
Secure HTTPS connection established
        ↓
CloudFront retrieves object from S3
```

This ensures encrypted communication between users and the CDN.

---

### 8. The Problem with Public S3 Buckets

Without additional configuration, the S3 bucket must be **public** to serve content.

Example architecture:

```
User → CloudFront → Public S3 bucket
```

Problem:

```
Users could bypass CloudFront
Access S3 directly
```

This reduces security and bypasses caching and WAF protections.

To solve this, S3 buckets are usually kept **private**.

CloudFront is granted exclusive access to the bucket.

---

### 9. CloudFront Origin Access Control

CloudFront provides two mechanisms for secure S3 access:

```
OAI (Origin Access Identity)
OAC (Origin Access Control)
```

Both methods allow CloudFront to access private S3 buckets.

This prevents direct public access.

---

### 10. Origin Access Identity (OAI)

OAI is the **older access method** for CloudFront.

Architecture:

```
User
 ↓
CloudFront
 ↓
OAI identity
 ↓
Private S3 bucket
```

The OAI acts as a virtual user that CloudFront uses to access the bucket.

---

### 11. OAI Bucket Policy

The S3 bucket policy allows the OAI to read objects.

Example concept:

```
Principal: CloudFront OAI
Action: s3:GetObject
Resource: bucket/*
```

This ensures:

```
Only CloudFront can access objects
```

Public access is blocked.

---

### 12. OAI Limitations

Although widely used, OAI has several limitations.

Limitations include:

```
Legacy architecture
Limited integration with modern security features
No support for advanced request signing
```

Because of these limitations, AWS introduced **Origin Access Control (OAC)**.

---

### 13. Origin Access Control (OAC)

OAC is the **modern replacement for OAI**.

Architecture:

```
User
 ↓
CloudFront
 ↓
OAC request signing
 ↓
Private S3 bucket
```

CloudFront signs requests using **SigV4 authentication** before sending them to S3.

S3 verifies the signature before allowing access.

---

### 14. OAC Request Signing

When CloudFront uses OAC:

```
CloudFront signs request
        ↓
Request sent to S3
        ↓
S3 validates signature
        ↓
Object returned
```

This ensures only authorized CloudFront distributions can access the bucket.

---

### 15. OAC Bucket Policy

Bucket policies must allow the CloudFront distribution to access objects.

Example concept:

```
Principal: CloudFront service
Condition: SourceArn = CloudFront distribution
Action: s3:GetObject
```

This tightly restricts access to a specific distribution.

---

### 16. OAI vs OAC Comparison

| Feature | OAI | OAC |
|------|------|------|
| Generation | Older | New |
| Authentication | IAM-style identity | SigV4 request signing |
| Security model | Basic | Stronger |
| AWS recommendation | Legacy | Recommended |
| CloudFront integration | Limited | Full |

AWS now recommends **OAC for new architectures**.

---

### 17. Example Secure Architecture

Recommended production architecture:

```
User
 ↓
HTTPS
 ↓
CloudFront CDN
 ↓
Origin Access Control
 ↓
Private S3 bucket
```

Benefits:

```
HTTPS support
Edge caching
Private bucket
Secure origin access
```

Users cannot bypass CloudFront.

---

### 18. Preventing Direct S3 Access

With OAC configured correctly:

```
Direct request to S3
```

Result:

```
Access denied
```

Only requests signed by CloudFront are allowed.

This prevents unauthorized downloads.

---

### 19. Performance Benefits of CloudFront

CloudFront improves performance by caching content at edge locations.

Example workflow:

```
User requests object
        ↓
CloudFront edge cache
        ↓
Object returned
```

If object is not cached:

```
CloudFront fetches object from S3
Stores in cache
```

Future requests are served faster.

---

### 20. Security Benefits

CloudFront also provides additional security features.

Examples include:

```
AWS WAF integration
DDoS protection (AWS Shield)
Signed URLs
Access logs
```

These capabilities are not available when serving content directly from S3.

---

### 21. Example Production Setup

Typical modern setup:

```
Custom domain → DNS (Route 53)
        ↓
CloudFront distribution
        ↓
TLS certificate (ACM)
        ↓
Origin Access Control
        ↓
Private S3 bucket
```

This architecture is used by many production static websites and media platforms.

---

### 22. Summary

When serving content stored in Amazon S3 using a custom domain, CloudFront is typically used as a content delivery layer.

CloudFront provides HTTPS support through TLS certificates, global caching through edge locations, and integration with DNS for custom domains.

To prevent direct access to S3 buckets, CloudFront can use **Origin Access Identity (OAI)** or the newer **Origin Access Control (OAC)** mechanism.

OAI uses a special identity that is granted bucket access, while OAC signs requests using modern AWS request authentication.

Today, OAC is the recommended approach because it offers stronger security and better integration with CloudFront.

By combining S3, CloudFront, TLS certificates, and OAC, organizations can build secure, scalable, and globally distributed content delivery systems.
