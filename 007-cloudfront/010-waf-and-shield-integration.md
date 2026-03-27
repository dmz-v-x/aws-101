## AWS WAF & Shield Integration with CloudFront

### 1. Why Security Layer is Needed

#### 1.1 Problem

CloudFront distributions are public endpoints:

- Anyone on the internet can send requests  
- Vulnerable to attacks like:
  - DDoS  
  - SQL injection  
  - XSS  
  - Bot traffic  

---

#### 1.2 Solution

AWS provides:

- **AWS WAF (Web Application Firewall)** → Application layer protection  
- **AWS Shield** → DDoS protection  

---

#### 1.3 Key Insight

> WAF + Shield together protect CloudFront at **network + application layers**

---

### 2. AWS Shield

---

#### 2.1 What is AWS Shield?

AWS Shield is a managed service that protects against:

> Distributed Denial of Service (DDoS) attacks

---

#### 2.2 Types of Shield

---

##### Shield Standard (Default)

- Automatically enabled  
- No cost  
- Protects against:
  - Layer 3 (network) attacks  
  - Layer 4 (transport) attacks  

---

##### Shield Advanced

- Paid service  
- Enhanced protection  
- Real-time monitoring  
- DDoS response team access  

---

#### 2.3 Key Insight

> Every CloudFront distribution is automatically protected by **Shield Standard**

---

### 3. AWS WAF (Web Application Firewall)

---

#### 3.1 What is AWS WAF?

WAF allows you to:

> Define rules to allow, block, or monitor HTTP requests

---

#### 3.2 Where It Operates

- Works at **Layer 7 (application layer)**  
- Attached directly to CloudFront distribution  

---

#### 3.3 What WAF Can Inspect

- IP address  
- Headers  
- Query strings  
- Cookies  
- Request body  

---

### 4. WAF Web ACL (Access Control List)

---

#### 4.1 What is a Web ACL?

A **Web ACL** is a collection of rules applied to CloudFront.

---

#### 4.2 Rule Actions

- Allow  
- Block  
- Count (monitor only)  

---

#### 4.3 Evaluation Order

- Rules evaluated in order  
- First match decides action  

---

### 5. Basic WAF Rules

---

### 5.1 IP Block List

#### What it does

Blocks requests from specific IPs.

---

#### Example Use Cases

- Block malicious IPs  
- Block specific countries  
- Restrict admin access  

---

#### Example Rule

```
If IP in blocked list → Block request
```

---

### 5.2 Allow List (Whitelist)

#### What it does

Only allows specific IPs.

---

#### Use Case

- Internal tools  
- Admin dashboards  

---

### 5.3 Rate Limiting Rule

---

#### What it does

Blocks IPs sending too many requests.

---

#### Example

```
If requests > 1000 in 5 minutes → Block IP
```

---

#### Use Cases

- Prevent brute-force attacks  
- Stop bots  
- Control API abuse  

---

#### Key Insight

> Rate limiting is one of the most powerful protections

---

### 6. Managed Rule Groups (OWASP Protection)

---

#### 6.1 What are Managed Rules?

Prebuilt security rules provided by AWS.

---

#### 6.2 OWASP Top 10 Protection

Protects against:

- SQL Injection  
- Cross-Site Scripting (XSS)  
- Command Injection  
- Local File Inclusion  

---

#### 6.3 AWS Managed Rule Groups

Examples:

- AWSManagedRulesCommonRuleSet  
- AWSManagedRulesSQLiRuleSet  
- AWSManagedRulesKnownBadInputsRuleSet  

---

#### 6.4 Benefits

- No need to write rules manually  
- Continuously updated by AWS  

---

#### 6.5 Key Insight

> Always enable managed rules for baseline protection

---

### 7. Custom WAF Rules

---

#### 7.1 What You Can Customize

- Block specific paths  
- Match headers  
- Match query parameters  
- Regex-based filtering  

---

#### 7.2 Example

```
Block requests where:
User-Agent contains "curl"
```

---

#### 7.3 Advanced Matching

- String match  
- Regex match  
- Geo match (country-based)  

---

### 8. Attaching WAF to CloudFront

---

#### 8.1 Steps

1. Create Web ACL in WAF  
2. Add rules (IP, rate limit, OWASP, etc.)  
3. Associate Web ACL with CloudFront distribution  

---

#### 8.2 Result

All incoming requests:
→ Evaluated by WAF before reaching CloudFront cache  

---

### 9. Request Flow with WAF & Shield

```
User Request
     ↓
AWS Shield (DDoS protection)
     ↓
AWS WAF (rule evaluation)
     ↓
CloudFront Edge (cache)
     ↓
Origin (if needed)
```

---

### 10. Real-World Security Setup

---

#### Example Configuration

- Shield Standard enabled  
- WAF Web ACL:
  - Block known malicious IPs  
  - Rate limit (1000 req / 5 min)  
  - AWS Managed Rules enabled  

---

#### Result

- DDoS protected  
- Application protected  
- Bot traffic reduced  

---

### 11. Monitoring & Logging

---

#### 11.1 WAF Logs

- Store logs in:
  - S3  
  - CloudWatch  
  - Kinesis  

---

#### 11.2 Metrics

- Allowed requests  
- Blocked requests  
- Rate-limited requests  

---

#### 11.3 Alerts

- Trigger alarms for:
  - Sudden traffic spikes  
  - Increased blocked requests  

---

### 12. Common Mistakes

---

#### Mistake 1: Not enabling WAF

- Leaves app exposed  

---

#### Mistake 2: Over-blocking traffic

- Blocks legitimate users  

---

#### Mistake 3: No rate limiting

- Vulnerable to bot attacks  

---

#### Mistake 4: Ignoring logs

- Hard to debug issues  

---

### 13. Best Practices

---

#### 13.1 Always Use Managed Rules

- Quick baseline protection  

---

#### 13.2 Add Rate Limiting

- Prevent abuse  

---

#### 13.3 Use IP Restrictions Carefully

- Avoid blocking valid users  

---

#### 13.4 Monitor Traffic

- Use logs and metrics  

---

#### 13.5 Combine with Shield

- Network + application protection  

---

### 14. Final Mental Model

> Security in CloudFront is layered:

- **Shield** → protects against large-scale network attacks  
- **WAF** → filters and controls application-level traffic  

Together, they ensure:
- Availability  
- Security  
- Reliability of your application  
