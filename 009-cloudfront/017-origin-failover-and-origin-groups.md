## Origin Failover and Origin Groups in CloudFront

### 1. Why Origin Failover is Needed

#### 1.1 Problem

Your origin can fail due to:

- Server downtime  
- Network issues  
- High load  
- Regional outages  

---

#### 1.2 Impact

Without failover:

```
Origin Down → Users get errors (5xx) ❌
```

---

#### 1.3 Solution

CloudFront provides:

> **Origin Failover using Origin Groups**

---

#### 1.4 Key Insight

> Failover ensures **high availability and reliability**

---

### 2. What is an Origin Group?

---

#### 2.1 Definition

An **origin group** is a configuration that includes:

- **Primary origin**
- **Secondary (failover) origin**

---

#### 2.2 Behavior

CloudFront:

1. Sends request to primary origin  
2. If failure occurs → switches to secondary origin  

---

#### 2.3 Architecture

```
            → Primary Origin
User → CloudFront
            → Secondary Origin (on failure)
```

---

### 3. How Failover Works

---

#### 3.1 Normal Flow

```
User → CloudFront → Primary Origin → Response
```

---

#### 3.2 Failure Flow

```
User → CloudFront → Primary Origin (fails)
                      ↓
                 Secondary Origin → Response
```

---

#### 3.3 Key Insight

> Failover happens **automatically at request time**

---

### 4. Failover Conditions

---

#### 4.1 What Triggers Failover

CloudFront switches to secondary origin when it receives:

- HTTP 500 (Internal Server Error)  
- HTTP 502 (Bad Gateway)  
- HTTP 503 (Service Unavailable)  
- HTTP 504 (Gateway Timeout)  

---

#### 4.2 Optional Status Codes

You can configure:
- Which status codes trigger failover  

---

#### 4.3 Key Insight

> Failover is based on **response status codes**, not health checks

---

### 5. Important Limitation

---

#### 5.1 No Active Health Checks

CloudFront does NOT:

- Continuously monitor origin health  

---

#### 5.2 Behavior

- It only reacts **after a failed request**

---

#### 5.3 Key Insight

> Failover is **reactive**, not proactive

---

### 6. Types of Origins in Failover

---

#### 6.1 Supported Combinations

- S3 → S3  
- S3 → Custom origin  
- Custom → Custom  

---

#### 6.2 Example

- Primary: S3 bucket  
- Secondary: ALB  

---

### 7. Setting Up Origin Groups (Step-by-Step)

---

### 7.1 Step 1: Define Origins

Create two origins:

- Primary origin  
- Secondary origin  

---

### 7.2 Step 2: Create Origin Group

- Select both origins  
- Define failover order  

---

### 7.3 Step 3: Configure Failover Criteria

Choose status codes:

```
500, 502, 503, 504
```

---

### 7.4 Step 4: Attach to Cache Behavior

- Select origin group instead of single origin  

---

### 7.5 Step 5: Deploy

- Wait for CloudFront propagation  

---

### 8. Real-World Example

---

#### Scenario: Static Website

---

Primary:
```
S3 bucket (main region)
```

Secondary:
```
S3 bucket (backup region)
```

---

#### Flow

- Normal → Primary serves content  
- Failure → Secondary serves content  

---

### 9. Advanced Use Cases

---

### 9.1 Disaster Recovery

- Primary region failure  
- Automatic fallback to backup region  

---

### 9.2 Multi-Region Architecture

- Improve reliability  
- Reduce downtime  

---

### 9.3 Hybrid Setup

- Primary: ALB (dynamic content)  
- Secondary: S3 (static fallback page)  

---

### 10. Custom Error Responses with Failover

---

#### 10.1 Combine with Error Pages

- Serve custom error pages from secondary origin  

---

#### 10.2 Example

- Primary fails  
- Secondary serves "maintenance page"  

---

### 11. Performance Considerations

---

#### 11.1 First Request Delay

- Failover adds latency on failure  

---

#### 11.2 No Pre-Warming

- Secondary origin not used unless needed  

---

### 12. Common Mistakes

---

#### Mistake 1: Expecting health checks

- CloudFront does not proactively check origin  

---

#### Mistake 2: Not configuring correct status codes

- Failover not triggered  

---

#### Mistake 3: Misconfigured secondary origin

- Failover also fails  

---

#### Mistake 4: Using same failing infrastructure

- Both origins fail  

---

### 13. Best Practices

---

#### 13.1 Use Different Regions

- Avoid single point of failure  

---

#### 13.2 Ensure Secondary is Fully Functional

- Same content or fallback content  

---

#### 13.3 Configure Proper Status Codes

- Include all relevant 5xx errors  

---

#### 13.4 Test Failover

- Simulate failures  

---

### 14. Testing Failover

---

#### 14.1 Method

- Stop primary origin  
- Send request  

---

#### 14.2 Expected Result

- Response served from secondary origin  

---

#### 14.3 Verification

- Check logs  
- Add headers to identify origin  

---

### 15. Final Mental Model

> CloudFront origin failover provides **automatic fallback** when the primary origin fails.

- Primary origin = default  
- Secondary origin = backup  

Failover is:
- Reactive  
- Status-code based  
- Per-request decision  

This ensures:
- High availability  
- Better user experience  
- Resilient architecture  
