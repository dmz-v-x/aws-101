## Viewer vs Origin Protocols, Encryption, TLS, and HTTP/2/3 in CloudFront

### 1. Viewer vs Origin — Core Concept

#### 1.1 Two Separate Connections

In CloudFront, every request involves **two different connections**:

```
Viewer (User) ↔ CloudFront ↔ Origin
```

---

#### 1.2 Viewer Connection

This is the connection between:
- User (browser/client)
- CloudFront edge location

---

#### 1.3 Origin Connection

This is the connection between:
- CloudFront
- Your origin (S3, ALB, EC2, etc.)

---

#### 1.4 Key Insight

> Viewer and origin connections are **completely independent** and can use different protocols and security settings

---

### 2. Viewer Protocol Policy

#### 2.1 What It Controls

Defines how users connect to CloudFront.

---

#### 2.2 Options

##### HTTP and HTTPS
- Allows both
- Not secure (HTTP allowed)

---

##### Redirect HTTP to HTTPS
- Automatically redirects HTTP → HTTPS
- Recommended for most use cases

---

##### HTTPS Only
- Rejects HTTP requests
- Strict security

---

#### 2.3 Best Practice

> Use **Redirect HTTP to HTTPS** or **HTTPS Only** in production

---

### 3. Origin Protocol Policy

#### 3.1 What It Controls

Defines how CloudFront connects to your origin.

---

#### 3.2 Options

##### HTTP Only
- No encryption
- Faster but insecure

---

##### HTTPS Only
- Fully encrypted connection
- Recommended

---

##### Match Viewer
- Uses same protocol as viewer request

---

#### 3.3 Best Practice

> Use **HTTPS Only** for secure end-to-end encryption

---

### 4. End-to-End Encryption

#### 4.1 What It Means

```
User (HTTPS) → CloudFront (HTTPS) → Origin (HTTPS)
```

---

#### 4.2 Why It Matters

- Prevents data interception
- Required for sensitive data (auth, payments)

---

#### 4.3 Key Insight

> True security requires encryption on **both legs** of the request

---

### 5. TLS (Transport Layer Security)

#### 5.1 What is TLS?

TLS is the protocol that:
- Encrypts data
- Secures communication over HTTPS

---

#### 5.2 TLS Versions

- TLS 1.0 (deprecated)
- TLS 1.1 (deprecated)
- TLS 1.2 (widely used)
- TLS 1.3 (latest, fastest, most secure)

---

#### 5.3 Minimum TLS Version in CloudFront

You can configure:
- Minimum TLS version allowed for viewers

---

#### 5.4 Example Settings

- TLSv1.2_2021 (recommended)
- TLSv1.3 (if supported by client)

---

#### 5.5 Key Insight

> Setting a higher minimum TLS version improves security but may drop support for older clients

---

### 6. SSL/TLS Certificates in CloudFront

---

### 6.1 Why Certificates Are Needed

Certificates enable:
- HTTPS connections
- Identity verification
- Encrypted communication

---

### 7. ACM-Managed Certificates (Recommended)

#### 7.1 What is ACM?

AWS Certificate Manager (ACM) provides:
- Free SSL/TLS certificates
- Automatic renewal

---

#### 7.2 Key Requirement

> Certificate MUST be created in **us-east-1 (N. Virginia)** for CloudFront

---

#### 7.3 Benefits

- Free
- Auto-renewed
- Easy integration with CloudFront

---

#### 7.4 Use Case

- Custom domains (example.com)
- Production applications

---

### 8. Custom Certificates

#### 8.1 What It Means

You upload your own certificate instead of using ACM.

---

#### 8.2 When to Use

- External certificate providers
- Compliance requirements
- Existing enterprise setups

---

#### 8.3 Drawbacks

- Manual renewal
- More operational effort

---

### 9. SNI (Server Name Indication)

#### 9.1 What is SNI?

SNI allows:
> Multiple SSL certificates to be served from the same IP address

---

#### 9.2 Why It Exists

CloudFront hosts many domains on shared infrastructure.

Without SNI:
- Each domain would need a dedicated IP

---

#### 9.3 How It Works

Client sends:
- Domain name during TLS handshake

Server responds with:
- Correct certificate

---

#### 9.4 Benefits

- Efficient
- Cost-effective
- Default in modern browsers

---

#### 9.5 Limitation

Very old clients (rare) may not support SNI.

---

### 10. Dedicated IP (Legacy Option)

#### 10.1 What It Is

Alternative to SNI:
- Dedicated IP per certificate

---

#### 10.2 When Needed

- Legacy clients without SNI support

---

#### 10.3 Drawback

- Expensive
- Rarely needed today

---

### 11. HTTP Protocol Versions

---

### 11.1 HTTP/1.1

- Older protocol
- One request per connection (mostly)
- Higher latency

---

### 11.2 HTTP/2

#### Features:
- Multiplexing (multiple requests on one connection)
- Header compression
- Faster page loads

---

#### CloudFront Support

- Supported for viewer connections
- Enabled by default

---

### 11.3 HTTP/3 (QUIC)

#### Features:
- Uses UDP instead of TCP
- Faster connection setup
- Better performance on unreliable networks

---

#### CloudFront Support

- Supported for viewer connections
- Improves performance on mobile networks

---

#### Key Insight

> HTTP/3 reduces latency significantly compared to HTTP/2

---

### 12. Viewer vs Origin Protocol Support

| Connection | Supported Protocols |
|----------|-------------------|
| Viewer → CloudFront | HTTP/1.1, HTTP/2, HTTP/3 |
| CloudFront → Origin | HTTP/1.1, HTTP/2 |

---

### 13. Protocol Selection Flow

---

#### 13.1 Viewer Side

- Browser chooses best protocol (HTTP/2 or HTTP/3)
- Based on support and network conditions

---

#### 13.2 Origin Side

- CloudFront connects using:
  - HTTP/1.1 (default)
  - HTTP/2 (if enabled and supported)

---

### 14. Performance Implications

---

#### 14.1 HTTP/2 Benefits

- Reduced latency
- Better throughput

---

#### 14.2 HTTP/3 Benefits

- Faster handshakes
- Better performance on mobile/unstable networks

---

#### 14.3 HTTPS Overhead

- Slight CPU cost for encryption
- Negligible compared to benefits

---

### 15. Security Best Practices

---

#### 15.1 Viewer Side

- Enforce HTTPS  
- Set minimum TLS to 1.2 or higher  

---

#### 15.2 Origin Side

- Use HTTPS only  
- Validate certificates  

---

#### 15.3 Certificates

- Prefer ACM-managed certificates  
- Use DNS validation  

---

#### 15.4 Avoid

- HTTP-only configurations  
- Deprecated TLS versions  

---

### 16. Common Mistakes

---

#### Mistake 1: Using HTTP for origin

Leads to:
- Unencrypted backend traffic  

---

#### Mistake 2: Not enforcing HTTPS

Leads to:
- Security vulnerabilities  

---

#### Mistake 3: Wrong certificate region

Leads to:
- CloudFront not accepting certificate  

---

#### Mistake 4: Ignoring TLS versions

Leads to:
- Weak security posture  

---

### 17. Final Mental Model

> CloudFront acts as a **secure bridge**:

- Viewer side → optimized (HTTP/2/3 + TLS)  
- Origin side → controlled (HTTP/1.1/2 + TLS policies)  

Security and performance depend on:
- Protocol selection  
- TLS configuration  
- Certificate management  
