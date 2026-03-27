## ACM + Elastic Load Balancer (ALB/NLB) — Provision and Attach Certificate to Listeners

### 1. What Are We Doing Here?
We will:
- Use an **ACM certificate**
- Attach it to a **Load Balancer**
- Enable **HTTPS (secure traffic)**

Goal:
Client → HTTPS → Load Balancer → Backend

---

### 2. Types of Load Balancers in AWS

1. **Application Load Balancer (ALB)**  
   - Layer 7 (HTTP/HTTPS)  
   - Most commonly used  

2. **Network Load Balancer (NLB)**  
   - Layer 4 (TCP/TLS)  
   - High performance, low latency  

---

### 3. Where Certificate is Used

Certificates are attached to:
→ **Listener (port 443)**

Listener = component that handles incoming traffic

---

### 4. Prerequisites

You need:

- ACM certificate (Status = ISSUED)
- Target group (EC2 / service)
- VPC setup
- Security group allowing port 443

---

## PART A — Application Load Balancer (ALB)

---

### 5. Step 1 — Create ALB (Console)

1. Go to EC2 → Load Balancers  
2. Click **Create Load Balancer**  
3. Choose **Application Load Balancer**  

Configure:
- Name: my-alb  
- Scheme: Internet-facing  
- IP type: IPv4  
- VPC & subnets  

---

### 6. Step 2 — Configure Listener

Add listener:

- Protocol: HTTPS  
- Port: 443  

Then:
→ Select ACM certificate  

---

### 7. Step 3 — Select Certificate

Choose:

- From ACM  
- Select your certificate (example.com)

---

### 8. Step 4 — Configure Target Group

- Create or select target group  
- Protocol: HTTP  
- Port: 80  
- Register targets (EC2 instances)

---

### 9. Step 5 — Review and Create

Click:
→ **Create Load Balancer**

---

### 10. Traffic Flow (ALB)

```
Client → HTTPS (443)
        ↓
ALB (TLS termination using ACM cert)
        ↓
HTTP (80)
        ↓
Backend (EC2)
```

---

### 11. Optional — Redirect HTTP → HTTPS

Add another listener:

- Port: 80 (HTTP)
- Action: Redirect to HTTPS (443)

---

### 12. CLI Example — ALB Listener

```bash
aws elbv2 create-listener \
  --load-balancer-arn <ALB_ARN> \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=<CERT_ARN> \
  --default-actions Type=forward,TargetGroupArn=<TG_ARN>
```

---

## PART B — Network Load Balancer (NLB)

---

### 13. TLS with NLB

NLB supports:
- TCP (no TLS)
- TLS (with certificate)

Use TLS if:
→ You want encryption at load balancer

---

### 14. Step 1 — Create NLB

- Go to EC2 → Load Balancers  
- Select **Network Load Balancer**

---

### 15. Step 2 — Add Listener

- Protocol: TLS  
- Port: 443  

Attach:
→ ACM certificate  

---

### 16. Step 3 — Configure Target Group

- Protocol: TCP or TLS  
- Register targets  

---

### 17. Traffic Flow (NLB TLS)

```
Client → TLS
        ↓
NLB (TLS termination OR pass-through)
        ↓
Backend
```

---

### 18. NLB Modes (Important)

#### TLS Termination
- NLB decrypts traffic
- Backend receives plain traffic

#### TLS Pass-through
- NLB does NOT decrypt
- Backend handles TLS

---

### 19. ALB vs NLB (Certificate Usage)

| Feature          | ALB                        | NLB                        |
|------------------|----------------------------|----------------------------|
| Layer            | L7 (HTTP/HTTPS)            | L4 (TCP/TLS)               |
| TLS Termination  | Yes                        | Optional                   |
| Use Case         | Web apps                   | High-performance systems   |
| Certificate      | Attached to HTTPS listener | Attached to TLS listener   |

---

### 20. Best Practices

1. Use ALB for web apps  
2. Use HTTPS (port 443) always  
3. Redirect HTTP → HTTPS  
4. Use ACM-managed certificates  
5. Monitor certificate status  

---

### 21. Common Mistakes

- Certificate not in same region  
- Using HTTP instead of HTTPS  
- Not opening port 443 in security group  
- Wrong target group configuration  

---

### 22. Real-World Example

Scenario:
- You deploy a Node.js app on EC2  
- Use ALB + ACM  

Steps:
1. Request cert in ACM  
2. Create ALB  
3. Attach cert to HTTPS listener  
4. Route traffic to EC2  

Result:
→ Secure HTTPS website  

---

### 23. Why This Matters for ACM Mastery

This is the MOST common real-world use of ACM:

- Hosting secure web apps  
- Enabling HTTPS  
- Managing certificates at scale  

---

### 24. Key Takeaways

- Certificates are attached to load balancer listeners  
- ALB handles HTTPS at Layer 7  
- NLB supports TLS at Layer 4  
- ACM integrates directly with both  
- Always use HTTPS for secure communication  
