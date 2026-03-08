## Amazon S3 Object Size Considerations and Multipart Upload Basics

### 1. Introduction

Amazon S3 is designed to store objects of many different sizes, ranging from very small files to extremely large datasets. However, object size has an important impact on **performance, reliability, cost, and upload strategy**.

Because of this, AWS provides multiple mechanisms for uploading objects, including **multipart upload**, which is optimized for large files.

Understanding how object size affects S3 behavior helps engineers design efficient upload architectures and avoid reliability issues when transferring large data.

This guide explains:

- S3 object size limits
- why multipart uploads exist
- how multipart uploads work
- when multipart uploads should be used

---

### 2. What is an S3 Object?

In S3, data is stored as **objects**.

Each object consists of:

```
Object
 ├── Data (file content)
 ├── Metadata
 └── Object key (name/path)
```

Objects can represent many types of files such as:

- images
- videos
- log files
- backups
- datasets

The size of these objects can vary significantly depending on the workload.

---

### 3. S3 Object Size Limits

Amazon S3 supports very large objects, but there are defined limits.

Maximum object size:

```
5 TB
```

Maximum size for a single PUT upload:

```
5 GB
```

Minimum object size:

```
0 bytes
```

Because of the 5 GB limit for single PUT requests, large objects must be uploaded using **multipart upload**.

---

### 4. Small Object Uploads

Small objects can be uploaded using a simple PUT request.

Example workflow:

```
Client
   ↓
PUT request
   ↓
Object stored in S3
```

This approach works well for:

- configuration files
- small images
- log entries
- small documents

However, as object size increases, reliability problems may occur.

---

### 5. Problems with Large Single Uploads

Uploading very large objects in a single request can create several issues.

Connection failures

If the network connection drops, the entire upload must restart.

Long upload times

Large files take longer to upload in a single stream.

Limited bandwidth utilization

Single-thread uploads may not fully utilize available network bandwidth.

Memory and buffering issues

Large requests may exceed system limitations.

Multipart uploads solve these problems.

---

### 6. What is Multipart Upload?

Multipart upload allows a large object to be **split into smaller parts** before uploading.

Each part is uploaded independently.

After all parts are uploaded, S3 assembles them into a single final object.

Conceptual workflow:

```
Large file
   ↓
Split into parts
   ↓
Upload parts independently
   ↓
S3 assembles object
```

This improves reliability and performance for large uploads.

---

### 7. Multipart Upload Process

Multipart uploads follow three main steps.

```
1. Initiate multipart upload
2. Upload individual parts
3. Complete multipart upload
```

Each step is handled through specific API operations.

---

### 8. Step 1: Initiate Multipart Upload

The client begins by initiating the upload.

Example conceptual step:

```
Client → Initiate upload request
```

S3 returns an **Upload ID**.

Example:

```
UploadId = abc123
```

This ID is used for all subsequent part uploads.

---

### 9. Step 2: Upload Parts

The object is divided into multiple parts.

Each part is uploaded separately.

Example:

```
Part 1 → Upload
Part 2 → Upload
Part 3 → Upload
...
Part N → Upload
```

Each part includes:

- part number
- upload ID
- data segment

Parts can be uploaded **in parallel**.

---

### 10. Step 3: Complete Multipart Upload

After all parts are uploaded, the client sends a **complete request**.

Example conceptual workflow:

```
Client → Complete upload request
```

S3 then assembles the parts into a single object.

The final object becomes available in the bucket.

---

### 11. Multipart Upload Limits

Multipart uploads have specific limits.

Maximum object size:

```
5 TB
```

Minimum part size:

```
5 MB
```

Maximum number of parts:

```
10,000
```

Maximum part size:

```
5 GB
```

These limits allow extremely large objects to be uploaded efficiently.

---

### 12. Example Multipart Upload Calculation

Example file size:

```
10 GB
```

Part size:

```
100 MB
```

Number of parts:

```
10 GB ÷ 100 MB = 100 parts
```

Each part can be uploaded independently.

This greatly improves reliability.

---

### 13. Parallel Upload Benefits

Multipart uploads allow **parallel uploads**.

Example:

```
Thread 1 → Part 1
Thread 2 → Part 2
Thread 3 → Part 3
Thread 4 → Part 4
```

Advantages include:

- faster uploads
- better bandwidth utilization
- improved throughput

Parallelization is especially useful for large datasets.

---

### 14. Failure Recovery

Multipart uploads improve reliability.

Example:

```
Part 8 upload fails
```

Instead of restarting the entire upload:

```
Retry only Part 8
```

This significantly reduces recovery time.

---

### 15. Resumable Uploads

Multipart uploads allow **resumable uploads**.

Example scenario:

```
Parts 1–6 uploaded successfully
Network failure occurs
```

Upload can resume at:

```
Part 7
```

This prevents wasting previously uploaded data.

---

### 16. Upload Performance Optimization

Multipart uploads allow tuning performance using:

- number of parallel uploads
- part size selection
- network bandwidth usage

Typical part sizes range between:

```
8 MB – 100 MB
```

Larger part sizes reduce the number of requests.

Smaller parts allow better parallelization.

---

### 17. Multipart Upload for Large Data Pipelines

Large-scale systems commonly use multipart uploads.

Examples include:

- data lake ingestion
- video streaming platforms
- backup systems
- machine learning datasets

These systems often transfer multi-gigabyte or terabyte files.

Multipart uploads enable reliable data transfer.

---

### 18. Aborting Multipart Uploads

Sometimes uploads are interrupted before completion.

Incomplete multipart uploads leave uploaded parts stored temporarily.

Example:

```
Upload started
Only 4 of 20 parts uploaded
Upload never completed
```

These partial uploads consume storage.

S3 provides an **Abort Multipart Upload** operation to clean them up.

---

### 19. Lifecycle Rules for Multipart Cleanup

Lifecycle policies can automatically remove incomplete uploads.

Example rule:

```
Abort incomplete multipart uploads after 7 days
```

This prevents unused parts from accumulating.

---

### 20. When to Use Multipart Upload

Multipart uploads are recommended when:

```
Object size > 100 MB
```

They are required when:

```
Object size > 5 GB
```

They are also beneficial when:

- networks are unreliable
- uploads must be parallelized
- large datasets are transferred

---

### 21. When Multipart Upload Is Not Necessary

Multipart uploads may not be necessary for small files.

Example:

```
1 MB image upload
```

A simple PUT request is usually sufficient.

Multipart uploads introduce additional overhead for small files.

---

### 22. Summary

Amazon S3 supports objects up to 5 TB in size, but uploading large objects using a single request can be inefficient and unreliable.

Multipart upload solves this problem by splitting large objects into smaller parts that can be uploaded independently and in parallel.

The multipart upload process involves initiating the upload, uploading individual parts, and completing the upload so that S3 assembles the final object.

Multipart uploads improve reliability, enable parallel uploads, allow failure recovery, and support extremely large files.

For most large objects, especially those larger than 100 MB, multipart upload is the recommended upload strategy.
