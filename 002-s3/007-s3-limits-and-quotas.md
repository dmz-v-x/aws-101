## Amazon S3 Limits and Quotas Explained: Object Size Limits, Bucket Limits, and Request Rate Basics

### 1. Introduction

Amazon S3 is designed to be a massively scalable storage system capable of storing enormous amounts of data. Many organizations store petabytes or even exabytes of data in S3.

Even though S3 is highly scalable, it still operates under certain **limits and quotas** that define how the service behaves and how applications should be designed.

Understanding these limits is important because it helps engineers design systems that:

- scale properly
- avoid performance bottlenecks
- handle very large datasets
- follow AWS service constraints

The most important limits involve:

- Object size limits
- Bucket limits
- Request rate performance

---

### 2. What Are Service Limits and Quotas?

In cloud services, limits define the boundaries within which the system operates.

These limits exist for several reasons:

- system stability
- fair usage across customers
- infrastructure management
- predictable performance

AWS categorizes limits into two types:

Hard limits

These cannot be changed.

Soft limits

These may be increased by requesting AWS support.

Some S3 limits are fixed by design, while others can be adjusted.

---

### 3. S3 Scalability Philosophy

Before discussing the specific limits, it is important to understand an important design principle of S3.

Amazon S3 is designed to provide **virtually unlimited storage capacity**.

This means there is no fixed maximum amount of data you can store in S3.

Organizations commonly store:

- billions of objects
- petabytes of data
- extremely large datasets

The limits that exist are primarily related to **individual objects, buckets, and request patterns**, not total storage.

---

### 4. Object Size Limits

One of the most important limits in S3 relates to the size of individual objects.

S3 defines both minimum and maximum object sizes.

Minimum object size

```
0 bytes
```

This means an object can technically be empty.

Maximum object size

```
5 TB
```

This is the largest object that can be stored in S3.

This limit is very large and is sufficient for most use cases such as large video files, backups, and datasets.

---

### 5. Single Upload Size Limit

Although the maximum object size is 5 TB, there is an important constraint.

A single upload request can only upload objects up to:

```
5 GB
```

If an object is larger than 5 GB, it must be uploaded using a technique called **multipart upload**.

Multipart upload allows large files to be uploaded in smaller pieces.

---

### 6. Multipart Upload Concept

Multipart upload breaks a large file into smaller parts.

Each part is uploaded independently.

After all parts are uploaded, S3 assembles them into the final object.

Example workflow:

Large file:

```
10 GB video file
```

Split into:

```
Part 1 → 2 GB
Part 2 → 2 GB
Part 3 → 2 GB
Part 4 → 2 GB
Part 5 → 2 GB
```

After uploading all parts, S3 combines them into a single object.

Multipart uploads improve reliability and performance.

---

### 7. Benefits of Multipart Upload

Multipart uploads provide several advantages.

Reliability

If a part fails to upload, only that part must be retried.

Parallel uploads

Multiple parts can be uploaded simultaneously.

Network efficiency

Large files can resume uploading if interrupted.

Because of these advantages, multipart uploads are commonly used for large files.

---

### 8. Minimum Multipart Part Size

When performing multipart uploads, each part must meet a minimum size requirement.

Minimum part size:

```
5 MB
```

Except for the final part, which can be smaller.

Maximum number of parts allowed:

```
10,000 parts
```

These constraints determine how large objects are divided.

---

### 9. Bucket Limits

Buckets are the top-level containers in S3.

AWS places limits on the number of buckets that an AWS account can create.

Default bucket limit per account:

```
100 buckets
```

This is usually sufficient for most applications.

If more buckets are required, AWS allows requesting an increase.

---

### 10. Why Bucket Limits Exist

Bucket limits exist primarily for organizational reasons.

Buckets are intended to represent major storage boundaries such as:

- environments
- applications
- teams
- data categories

Because buckets contain large numbers of objects, most organizations store large datasets within a smaller number of buckets.

Instead of creating thousands of buckets, objects are typically organized using **prefixes inside a bucket**.

---

### 11. Objects Per Bucket

One important property of S3 is that **there is no fixed limit on the number of objects in a bucket**.

A bucket can contain:

- millions of objects
- billions of objects
- trillions of objects

S3 automatically scales storage infrastructure to support large object counts.

This makes S3 suitable for massive datasets such as:

- log storage
- backups
- data lakes
- analytics datasets

---

### 12. Request Rate Basics

Applications interact with S3 by sending requests.

Examples of requests include:

GET requests

Retrieve objects.

PUT requests

Upload objects.

DELETE requests

Remove objects.

LIST requests

Retrieve object listings.

Each request interacts with the S3 infrastructure.

---

### 13. Historical Request Rate Limitations

In older versions of S3, request performance was limited by key naming patterns.

Applications had to distribute objects across multiple prefixes to achieve higher request rates.

For example:

```
logs/a/
logs/b/
logs/c/
```

This distributed load across different storage partitions.

---

### 14. Modern S3 Request Rate Performance

Amazon significantly improved S3 performance.

Modern S3 automatically scales to support very high request rates.

Current performance guidance includes:

At least:

```
3,500 PUT requests per second per prefix
```

and

```
5,500 GET requests per second per prefix
```

These are baseline performance levels.

Applications can achieve even higher throughput by using multiple prefixes.

---

### 15. Understanding Prefix-Based Scaling

Because S3 distributes objects internally using prefixes, request performance scales with prefix diversity.

Example structure:

```
logs/server1/
logs/server2/
logs/server3/
```

Each prefix allows additional parallel request capacity.

Large-scale systems often design object keys intentionally to distribute load.

---

### 16. Example of High Throughput Architecture

Consider a logging system storing millions of log files.

Example key structure:

```
logs/server1/2025/01/log1.txt
logs/server2/2025/01/log2.txt
logs/server3/2025/01/log3.txt
```

Because multiple prefixes exist, S3 can distribute requests across multiple partitions.

This allows higher request throughput.

---

### 17. S3 Listing Limitations

When listing objects in a bucket, S3 returns results in batches.

Maximum objects returned per listing request:

```
1,000 objects
```

If a bucket contains more objects, additional requests must be made using pagination.

Applications must handle this when scanning large datasets.

---

### 18. Large Dataset Considerations

When dealing with billions of objects, applications must design carefully.

Common strategies include:

Using structured prefixes

Avoiding extremely large single listings

Using inventory reports

Using analytics tools like Athena

These techniques improve scalability and performance.

---

### 19. Monitoring Request Performance

S3 provides monitoring capabilities through services such as:

CloudWatch metrics

S3 Storage Lens

Access logs

These tools allow engineers to observe request patterns and identify bottlenecks.

Monitoring is important for maintaining high performance systems.

---

### 20. Common Mistakes Related to Limits

Some common mistakes developers make include:

Uploading large files without multipart upload

Creating too many buckets unnecessarily

Designing poor object key structures

Performing large listing operations inefficiently

Ignoring request rate distribution

Understanding S3 limits helps avoid these issues.

---

### 21. Real World Scale Examples

Many large organizations rely heavily on S3.

Examples include:

Streaming platforms storing massive video libraries

Machine learning systems storing training datasets

Backup systems storing enterprise data

Data lakes storing analytics datasets

These systems rely on S3's scalability combined with proper architecture.

---

### 22. Summary

Amazon S3 is designed to scale to extremely large workloads, but it still operates within certain limits.

Important limits include:

Object size

Objects can be up to 5 TB in size.

Single upload size

Uploads larger than 5 GB require multipart upload.

Bucket limits

Each account can create up to 100 buckets by default.

Objects per bucket

There is no fixed limit on the number of objects per bucket.

Request rate

S3 supports thousands of requests per second per prefix and scales automatically.

Understanding these limits helps engineers design scalable systems that fully leverage the power of Amazon S3.
