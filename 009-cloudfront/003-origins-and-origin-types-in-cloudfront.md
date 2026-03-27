## Origins & Origin Types in CloudFront

### 1. What is an Origin?

#### 1.1 Definition

An **origin** is the backend source from which CloudFront fetches content when it is not available in cache.

---

#### 1.2 Key Responsibility

The origin:
- Stores the actual content
- Responds to CloudFront requests
- Acts as the **source of truth**

---

#### 1.3 Request Flow with Origin

```
User → Edge Location → Regional Cache → Origin → Response → Cached → User
```

---

### 2. Types of Origins in CloudFront

CloudFront supports multiple origin types depending on your use case.

---

### 3. S3 Bucket Origin (Static Content)

#### 3.1 What it is

Using an **Amazon S3 bucket** as the origin where:
- Files are stored as objects
- Access is typically restricted (private)

---

#### 3.2 Two Ways to Use S3

##### Option 1: S3 REST Endpoint (Recommended)

```
bucket-name.s3.amazonaws.com
```

- Works with **Origin Access Control (OAC)**
- Supports private buckets
- Secure (no public access required)

---

##### Option 2: S3 Website Endpoint

```
bucket-name.s3-website-region.amazonaws.com
```

- Used for static website hosting
- Requires public access
- Behaves like a traditional web server

---

#### 3.3 When to Use S3 Origin

Use when:
- Serving static assets (HTML, CSS, JS, images)
- Building static websites
- Need high scalability with minimal maintenance

---

#### 3.4 Advantages

- Fully managed
- Highly scalable
- Low cost
- Integrates with OAC for security

---

#### 3.5 Key Insight

> Prefer **S3 REST endpoint + OAC** over website endpoint for secure production setups

---

### 4. S3 Object Lambda Origin

#### 4.1 What it is

Allows you to:
> Modify S3 objects dynamically using Lambda before sending them to users

---

#### 4.2 How it Works

1. CloudFront requests object  
2. S3 Object Lambda triggers Lambda function  
3. Lambda modifies object  
4. Modified response returned  

---

#### 4.3 Use Cases

- Image resizing on-the-fly  
- Redacting sensitive data  
- Dynamic content transformation  

---

#### 4.4 Key Insight

> Adds **dynamic processing on top of static storage**

---

### 5. Custom Origins

A **custom origin** is any HTTP server that is NOT a standard S3 REST endpoint.

---

### 6. EC2 as Origin

#### 6.1 What it is

Your application running on an EC2 instance.

---

#### 6.2 When to Use

- Backend servers
- SSR apps (Next.js, Node.js)
- APIs

---

#### 6.3 Considerations

- Must handle scaling
- Requires load balancing for production

---

### 7. Application Load Balancer (ALB)

#### 7.1 What it is

ALB distributes traffic across multiple EC2 instances.

---

#### 7.2 Why Use ALB with CloudFront

- High availability
- Auto scaling support
- Handles dynamic traffic

---

#### 7.3 Key Insight

> ALB is the **recommended custom origin for scalable backend systems**

---

### 8. NGINX / Custom HTTP Servers

#### 8.1 What it is

Any custom web server:
- NGINX
- Apache
- Node.js server

---

#### 8.2 Use Cases

- Custom routing logic
- Reverse proxy setups
- Microservices gateway

---

#### 8.3 Considerations

- Must manage infrastructure
- Ensure proper caching headers

---

### 9. S3 Website Endpoint (as Custom Origin)

#### 9.1 What it is

When S3 static website hosting is enabled, it behaves like a custom origin.

---

#### 9.2 Limitations

- No OAC support
- Must be public
- Limited security

---

#### 9.3 When to Use

- Simple public websites
- No strict security requirements

---

### 10. API Gateway as Origin

#### 10.1 What it is

Using API Gateway endpoints behind CloudFront.

---

#### 10.2 Use Cases

- REST APIs
- GraphQL APIs
- Serverless backends

---

#### 10.3 Benefits

- Global caching for APIs
- Reduced latency
- Offloads API Gateway

---

#### 10.4 Key Insight

> CloudFront + API Gateway = **faster and cheaper API delivery**

---

### 11. AWS Elemental MediaStore

#### 11.1 What it is

A storage service optimized for:
- Video streaming
- Low-latency media delivery

---

#### 11.2 Use Cases

- Live streaming
- Video-on-demand platforms

---

#### 11.3 Key Insight

> Designed for **high-throughput media workloads**

---

### 12. S3 Origin vs Custom Origin (Critical Comparison)

---

#### 12.1 S3 Origin

Use when:
- Content is static
- No backend logic required
- Want low cost and simplicity

---

#### 12.2 Custom Origin

Use when:
- Dynamic content required
- Backend processing needed
- APIs or SSR involved

---

#### 12.3 Decision Table

| Use Case | Recommended Origin |
|--------|------------------|
| Static website | S3 (REST endpoint) |
| Private content | S3 + OAC |
| Dynamic app | ALB / EC2 |
| APIs | API Gateway / ALB |
| Streaming | MediaStore |

---

### 13. Origin Configuration Concepts

---

#### 13.1 Origin Domain

The DNS name of your origin:
- S3 bucket endpoint
- ALB DNS
- API Gateway URL

---

#### 13.2 Origin Protocol Policy

Defines how CloudFront connects to origin:
- HTTP only  
- HTTPS only  
- Match viewer  

---

#### 13.3 Origin Path

Adds a prefix to requests:

Example:
```
Origin: example.com
Origin Path: /v1
```

Request:
```
/users → example.com/v1/users
```

---

### 14. CORS (Cross-Origin Resource Sharing)

---

#### 14.1 What is CORS?

CORS controls:
> Whether a browser allows requests between different domains

---

#### 14.2 Why It Matters in CloudFront

Scenario:
- Frontend on `app.com`
- API on `api.com`

Browser blocks request unless CORS is configured.

---

#### 14.3 Where to Configure CORS

- At origin (recommended)
- OR via CloudFront (less common)

---

#### 14.4 Required Headers

```
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
```

---

#### 14.5 S3 CORS Example

```
[
  {
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["GET"],
    "AllowedHeaders": ["*"]
  }
]
```

---

#### 14.6 Common Issues

- Missing headers → blocked requests  
- Preflight failure (OPTIONS request)  
- Credentials + wildcard conflict  

---

### 15. Origin Failures & Behavior

---

#### 15.1 When Origin is Down

CloudFront:
- Returns error (5xx)
- Can use failover origin (if configured)

---

#### 15.2 Timeout Settings

- Connection timeout
- Response timeout

---

### 16. Security Considerations

---

#### 16.1 S3 Security

- Use OAC (recommended)
- Block public access

---

#### 16.2 Custom Origin Security

- Use HTTPS
- Restrict access via:
  - Security groups
  - WAF
  - IP allowlists

---

#### 16.3 Origin Shield (Advanced)

- Additional centralized caching layer
- Reduces origin load further

---

### 17. Final Mental Model

> The origin is the **source of truth**, and CloudFront acts as a smart caching layer in front of it.

Choosing the correct origin:
- Impacts performance  
- Determines scalability  
- Affects security  
- Controls cost  

Correct design = efficient, scalable, and secure system.
