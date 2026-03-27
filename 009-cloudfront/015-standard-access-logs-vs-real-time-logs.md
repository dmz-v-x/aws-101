## Standard Access Logs vs Real-Time Logs in CloudFront

### 1. Why Logging is Important

#### 1.1 Purpose of Logs

CloudFront logs help you:

- Analyze traffic  
- Debug issues  
- Monitor performance  
- Detect security threats  

---

#### 1.2 Types of Logs in CloudFront

- **Standard Access Logs**
- **Real-Time Logs**

---

#### 1.3 Key Insight

> Both serve different purposes:  
- Standard → historical analysis  
- Real-time → instant insights  

---

## PART 1 — Standard Access Logs

---

### 2. What are Standard Access Logs?

---

#### 2.1 Definition

Standard logs are:

> Batch logs delivered to S3 periodically

---

#### 2.2 Characteristics

- Delivered every few minutes (not instant)  
- Stored in S3  
- Low cost  
- Good for analytics  

---

### 3. Standard Logs Flow

```
User Request
     ↓
CloudFront Processes Request
     ↓
Logs Collected
     ↓
Logs Batched
     ↓
Delivered to S3 Bucket
```

---

### 4. What Data is Logged

---

Each log entry includes:

- Timestamp  
- Client IP  
- Request path  
- HTTP method  
- Status code  
- Bytes transferred  
- Referrer  
- User-Agent  

---

### 5. Setting Up Standard Logs (Step-by-Step)

---

#### 5.1 Step 1: Create S3 Bucket

- Create a bucket (e.g., `cf-logs-bucket`)  
- Same or different region  

---

#### 5.2 Step 2: Enable Logging in CloudFront

- Go to distribution  
- Edit settings  
- Enable **Standard Logging**  

---

#### 5.3 Step 3: Configure Logging Options

- Select S3 bucket  
- Set prefix (optional)

Example:

```
logs/cloudfront/
```

---

#### 5.4 Step 4: Permissions

CloudFront must be able to write to S3.

---

#### 5.5 Step 5: Save & Deploy

- Wait for distribution update  

---

### 6. Output Format

---

Logs are stored as:

```
S3://bucket-name/logs/cloudfront/...
```

File format:
- Text (space-separated fields)

---

### 7. Use Cases of Standard Logs

---

- Historical traffic analysis  
- Cost analysis  
- Debugging errors  
- Reporting  

---

## PART 2 — Real-Time Logs

---

### 8. What are Real-Time Logs?

---

#### 8.1 Definition

Real-time logs are:

> Logs delivered within seconds of request

---

#### 8.2 Characteristics

- Near real-time (1–2 seconds delay)  
- Streamed continuously  
- Requires streaming service  
- Higher cost  

---

### 9. Real-Time Logs Flow

```
User Request
     ↓
CloudFront Processes Request
     ↓
Log Generated Instantly
     ↓
Streamed to Kinesis Data Stream
     ↓
Processed / Stored / Analyzed
```

---

### 10. Components Required

---

To use real-time logs, you need:

- Kinesis Data Streams  
- IAM role for CloudFront  
- Real-time log configuration  

---

### 11. Setting Up Real-Time Logs (Step-by-Step)

---

### 11.1 Step 1: Create Kinesis Data Stream

- Go to Kinesis  
- Create stream (e.g., `cf-realtime-stream`)  
- Choose shard capacity  

---

### 11.2 Step 2: Create IAM Role

Allow CloudFront to write to Kinesis:

```
{
  "Effect": "Allow",
  "Action": "kinesis:PutRecords",
  "Resource": "arn:aws:kinesis:region:account-id:stream/cf-realtime-stream"
}
```

---

### 11.3 Step 3: Create Real-Time Log Configuration

In CloudFront:

- Define log fields  
- Select sampling rate (1%–100%)  
- Attach Kinesis stream  
- Attach IAM role  

---

### 11.4 Step 4: Attach to Cache Behavior

- Open distribution  
- Select behavior  
- Attach real-time log config  

---

### 11.5 Step 5: Deploy

- Wait for propagation  

---

### 12. Log Fields in Real-Time Logs

---

You can select fields like:

- Timestamp  
- Client IP  
- URI  
- Status code  
- Cache status  
- Bytes sent  
- Headers  

---

### 13. Sampling Rate

---

#### 13.1 What It Means

Percentage of requests logged:

- 100% → all requests  
- 10% → 1 out of 10  

---

#### 13.2 Why Use Sampling

- Reduce cost  
- Control data volume  

---

### 14. Processing Real-Time Logs

---

#### 14.1 Options

- Kinesis Data Firehose → S3  
- Lambda → real-time processing  
- Analytics tools (Athena, Elasticsearch)  

---

#### 14.2 Example Pipeline

```
CloudFront → Kinesis → Lambda → S3 → Athena
```

---

### 15. Standard vs Real-Time Logs (Comparison)

---

| Feature | Standard Logs | Real-Time Logs |
|--------|-------------|----------------|
| Delivery time | Minutes delay | Seconds |
| Destination | S3 | Kinesis |
| Cost | Low | Higher |
| Use case | Analytics | Monitoring |
| Setup | Simple | Complex |

---

### 16. When to Use What

---

#### 16.1 Use Standard Logs When

- You need historical analysis  
- Cost is a concern  
- Real-time data not required  

---

#### 16.2 Use Real-Time Logs When

- Need instant monitoring  
- Detect attacks quickly  
- Real-time dashboards  

---

### 17. Real-World Setup

---

#### Scenario

E-commerce platform:

---

##### Standard Logs

- Stored in S3  
- Used for:
  - Daily reports  
  - Traffic analysis  

---

##### Real-Time Logs

- Streamed via Kinesis  
- Used for:
  - Fraud detection  
  - Monitoring spikes  

---

### 18. Common Mistakes

---

#### Mistake 1: Using real-time logs unnecessarily

- High cost  

---

#### Mistake 2: Not setting sampling rate

- Data overload  

---

#### Mistake 3: Missing IAM permissions

- Logs not delivered  

---

#### Mistake 4: Not analyzing logs

- Wasted data  

---

### 19. Best Practices

---

#### 19.1 Use Standard Logs by Default

- Cost-effective  
- Sufficient for most use cases  

---

#### 19.2 Use Real-Time Logs Selectively

- Critical systems  
- Security monitoring  

---

#### 19.3 Use Sampling

- Reduce cost  

---

#### 19.4 Store Logs Properly

- Organize with prefixes  
- Use lifecycle policies  

---

### 20. Final Mental Model

> CloudFront logging provides two levels of visibility:

- **Standard Logs** → batch, delayed, cost-efficient  
- **Real-Time Logs** → instant, streaming, powerful  

Choose based on:
- Speed requirement  
- Cost tolerance  
- Complexity you can manage  
