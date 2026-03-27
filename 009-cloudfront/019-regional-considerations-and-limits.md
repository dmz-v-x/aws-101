## Regional Considerations & Limits in CloudFront

### 1. Understanding CloudFront Scope

---

#### 1.1 Global Service Nature

CloudFront is a **global service**:

- Distributions are deployed worldwide  
- Edge locations exist across multiple regions  
- Users are served from nearest edge  

---

#### 1.2 Key Insight

> CloudFront itself is **global**, but it interacts with many **regional services**

---

### 2. Global vs Regional Components

---

### 2.1 Global Components (CloudFront Side)

These are **global**:

- Distributions  
- Edge locations  
- Cache behaviors  
- Cache policies  
- Origin request policies  
- CloudFront Functions  
- Lambda@Edge (deployed globally)

---

### 2.2 Regional Components (AWS Services)

These are **region-specific**:

- S3 buckets  
- EC2 instances  
- ALB/NLB  
- API Gateway  
- Kinesis (for real-time logs)  
- ACM certificates (must be in us-east-1 for CloudFront)

---

#### 2.3 Key Insight

> CloudFront is global, but your **origins and integrations are regional**

---

### 3. ACM Certificate Region Requirement

---

#### 3.1 Critical Rule

For CloudFront:

> Certificates MUST be created in **us-east-1**

---

#### 3.2 Why?

- CloudFront only reads certificates from us-east-1  
- Even though distribution is global  

---

#### 3.3 Common Mistake

- Creating certificate in another region → not usable  

---

### 4. Lambda@Edge Regional Behavior

---

#### 4.1 Development Region

Lambda@Edge functions must be created in:

```
us-east-1
```

---

#### 4.2 Deployment

- Automatically replicated globally  
- Runs at edge locations  

---

#### 4.3 Key Insight

> Created in one region → executed globally

---

### 5. CloudFront Functions Scope

---

#### 5.1 Execution

- Runs at edge locations  
- Fully global  

---

#### 5.2 No Regional Constraints

- No region-specific setup required  

---

### 6. Origin Region Considerations

---

#### 6.1 S3 Origins

- Can be in any region  
- CloudFront fetches from that region  

---

#### 6.2 Latency Impact

- Far origin → higher latency on cache miss  

---

#### 6.3 Best Practice

> Use origin region close to primary users OR rely on caching heavily

---

### 7. Logging Regional Considerations

---

#### 7.1 Standard Logs

- Stored in S3 (any region)  

---

#### 7.2 Real-Time Logs

- Require Kinesis Data Streams  
- Must be created in a specific region  

---

#### 7.3 Key Insight

> Logging pipelines are **region-dependent**

---

### 8. CloudFront Quotas (Important Limits)

---

### 8.1 Distribution Limits

- Default: ~200 distributions per account (can increase)  

---

### 8.2 Cache Behaviors per Distribution

- Default: 25 behaviors (can request increase)  

---

### 8.3 Origins per Distribution

- Default: 25 origins  

---

### 8.4 Key Groups

- Limited number per account  

---

### 8.5 Invalidations

- 1000 free paths/month  
- Concurrent invalidation limits  

---

### 9. Request & Performance Limits

---

#### 9.1 File Size Limits

- Max object size supported (very large, TB scale)  

---

#### 9.2 Header Limits

- Max header size  
- Max number of headers  

---

#### 9.3 URL Length

- Max URL length constraints  

---

#### 9.4 Key Insight

> CloudFront handles massive scale but still has practical limits

---

### 10. Rate Limits

---

#### 10.1 API Rate Limits

- CloudFront API has request limits  
- Excess → throttling  

---

#### 10.2 Real-Time Logs Throughput

- Depends on Kinesis shard capacity  

---

### 11. Propagation & Deployment Limits

---

#### 11.1 Deployment Time

- Changes take ~5–20 minutes  

---

#### 11.2 Global Propagation

- Config must reach all edge locations  

---

#### 11.3 Key Insight

> CloudFront changes are **not instant**

---

### 12. Regional Restrictions & Compliance

---

#### 12.1 Data Residency

- Data may pass through multiple regions  
- Important for compliance  

---

#### 12.2 Geo Restrictions

- Can block/allow specific countries  

---

### 13. Multi-Region Architecture

---

#### 13.1 Use Case

- High availability  
- Disaster recovery  

---

#### 13.2 Setup

- Multiple origins in different regions  
- Use origin failover  

---

#### 13.3 Key Insight

> CloudFront enables **multi-region resilience**

---

### 14. Cost Considerations by Region

---

#### 14.1 Data Transfer Pricing

- Varies by region  
- Edge location pricing differs  

---

#### 14.2 Origin Transfer Costs

- Data fetched from origin incurs cost  

---

#### 14.3 Key Insight

> Geography impacts cost and performance

---

### 15. Common Mistakes

---

#### Mistake 1: Ignoring us-east-1 requirement

- Breaks certificate and Lambda@Edge setup  

---

#### Mistake 2: Assuming everything is global

- Some services are regional  

---

#### Mistake 3: Not planning for quotas

- Deployment failures  

---

#### Mistake 4: Ignoring propagation delay

- Confusion during updates  

---

### 16. Best Practices

---

#### 16.1 Understand Global vs Regional Split

- Design architecture accordingly  

---

#### 16.2 Use us-east-1 for Required Services

- ACM  
- Lambda@Edge  

---

#### 16.3 Monitor Quotas

- Request increases when needed  

---

#### 16.4 Optimize Origin Placement

- Balance latency and cost  

---

### 17. Final Mental Model

> CloudFront is a **global distribution layer**, but it depends on **regional AWS services**.

To design correctly:

- Know what is global  
- Know what is regional  
- Respect quotas and limits  

This ensures:
- Scalable architecture  
- Reliable deployments  
- Optimal performance  
