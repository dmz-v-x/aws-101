## Amazon S3 Multipart Upload API Flow: When It Helps, Completing and Aborting Uploads

### 1. Introduction

Amazon S3 multipart upload is designed to efficiently upload large objects by splitting them into smaller parts. Instead of sending the entire file in a single request, the file is divided into pieces that are uploaded independently.

This method improves reliability, performance, and scalability when handling large objects.

Multipart uploads are widely used in applications such as:

- video storage platforms
- data lake ingestion pipelines
- backup systems
- large dataset transfers

Understanding the **API flow** of multipart uploads is important for designing systems that handle large files efficiently.

---

### 2. Multipart Upload Overview

Multipart upload allows a large object to be divided into multiple parts before uploading.

Conceptual workflow:

```
Large File
   ↓
Split into smaller parts
   ↓
Upload parts independently
   ↓
S3 assembles parts into final object
```

Each uploaded part is stored temporarily until the upload is completed.

---

### 3. Multipart Upload API Flow

The multipart upload process involves three main API steps.

```
1. InitiateMultipartUpload
2. UploadPart
3. CompleteMultipartUpload
```

There is also a cleanup operation:

```
AbortMultipartUpload
```

These operations form the complete multipart upload workflow.

---

### 4. Step 1: Initiate Multipart Upload

The first step is initiating the multipart upload.

API operation:

```
CreateMultipartUpload
```

This request tells S3 that a multipart upload session is starting.

Example conceptual request:

```
POST /bucket/object?uploads
```

Response:

```
UploadId = unique-upload-id
```

The **UploadId** uniquely identifies this multipart upload session.

All subsequent requests must include this ID.

---

### 5. Purpose of Upload ID

The Upload ID allows S3 to track which parts belong to the same upload session.

Example:

```
UploadId: abc123
```

Parts uploaded with this ID will be grouped together when the final object is assembled.

Without the Upload ID, S3 cannot associate parts with the correct upload.

---

### 6. Step 2: Upload Individual Parts

After initiating the upload, the file is split into smaller parts.

Each part is uploaded separately using the **UploadPart API**.

Example conceptual request:

```
PUT /bucket/object?partNumber=1&uploadId=abc123
```

Each request contains:

- part number
- upload ID
- data for that part

Parts can be uploaded sequentially or in parallel.

---

### 7. Part Number Requirements

Each uploaded part must have a unique part number.

Part numbers must follow these rules:

```
Minimum: 1
Maximum: 10,000
```

Example sequence:

```
Part 1
Part 2
Part 3
...
Part N
```

The part numbers determine the final order of the assembled object.

---

### 8. Part Size Requirements

Multipart uploads have minimum size requirements.

Minimum size for each part:

```
5 MB
```

Exception:

```
Last part can be smaller
```

Maximum part size:

```
5 GB
```

These limits ensure efficient processing by S3.

---

### 9. Parallel Uploads

One major advantage of multipart upload is **parallel uploads**.

Example:

```
Thread 1 → Upload Part 1
Thread 2 → Upload Part 2
Thread 3 → Upload Part 3
Thread 4 → Upload Part 4
```

Parallel uploads improve:

- throughput
- upload speed
- network utilization

This is especially useful for large datasets.

---

### 10. Upload Part Response

Each UploadPart request returns a response containing an **ETag**.

Example:

```
Part 1 → ETag: "abc123"
Part 2 → ETag: "def456"
```

These ETags must be saved by the client.

They are required when completing the multipart upload.

---

### 11. Step 3: Complete Multipart Upload

After all parts are uploaded, the client sends a **CompleteMultipartUpload** request.

Example conceptual request:

```
POST /bucket/object?uploadId=abc123
```

The request includes:

```
List of parts
Part numbers
ETags
```

Example conceptual structure:

```
Part 1 → ETag "abc123"
Part 2 → ETag "def456"
Part 3 → ETag "ghi789"
```

S3 uses this list to assemble the final object.

---

### 12. Object Assembly

After receiving the completion request:

```
S3 retrieves uploaded parts
        ↓
Parts are ordered by part number
        ↓
Parts are concatenated
        ↓
Final object is created
```

Once assembled, the object becomes available in the bucket.

The temporary parts are removed automatically.

---

### 13. Multipart Upload Diagram

Conceptual multipart upload process:

```
Initiate Upload
      ↓
Upload Part 1
Upload Part 2
Upload Part 3
Upload Part 4
      ↓
Complete Upload
      ↓
S3 assembles final object
```

This allows efficient upload of very large objects.

---

### 14. When Multipart Upload Helps

Multipart upload is particularly useful in several scenarios.

Large files

Objects larger than 100 MB benefit significantly.

Unstable network connections

If a connection fails, only the failed part needs to be retried.

High bandwidth environments

Parallel uploads maximize throughput.

Distributed systems

Multiple workers can upload different parts.

These advantages make multipart uploads ideal for large-scale systems.

---

### 15. Performance Benefits

Multipart uploads improve performance in several ways.

Parallel uploads increase throughput.

Smaller parts reduce retry cost.

Uploads can resume from the failed part.

Better utilization of network bandwidth.

These improvements are significant for large files.

---

### 16. Resumable Upload Behavior

Multipart uploads allow uploads to be resumed.

Example scenario:

```
Parts 1–5 uploaded
Network interruption occurs
```

Instead of restarting the entire upload:

```
Resume at Part 6
```

This saves time and network bandwidth.

---

### 17. Incomplete Multipart Uploads

If an upload is started but never completed, uploaded parts remain stored in S3.

Example scenario:

```
Upload initiated
Only 3 parts uploaded
Completion request never sent
```

The uploaded parts still consume storage.

This is known as an **incomplete multipart upload**.

---

### 18. Aborting Multipart Uploads

To remove incomplete uploads, the **AbortMultipartUpload API** is used.

Example conceptual request:

```
DELETE /bucket/object?uploadId=abc123
```

Result:

```
All uploaded parts deleted
Multipart upload session removed
```

This frees the storage consumed by uploaded parts.

---

### 19. Lifecycle Rules for Multipart Cleanup

S3 provides lifecycle policies to clean up incomplete uploads automatically.

Example rule:

```
Abort incomplete multipart uploads after 7 days
```

This prevents unused parts from accumulating in storage.

Lifecycle cleanup is strongly recommended for production systems.

---

### 20. Monitoring Multipart Uploads

Administrators can monitor multipart uploads using:

S3 console

Lists active multipart uploads.

AWS CLI

Commands such as:

```
list-multipart-uploads
```

Monitoring helps detect abandoned uploads.

---

### 21. Best Practices

Recommended practices for multipart uploads include:

Use multipart uploads for files larger than 100 MB.

Choose appropriate part sizes (8–100 MB typical).

Upload parts in parallel to maximize throughput.

Store part ETags until completion.

Configure lifecycle rules to clean up incomplete uploads.

These practices ensure reliable and efficient file uploads.

---

### 22. Summary

Amazon S3 multipart upload allows large objects to be uploaded as multiple independent parts. This improves reliability, performance, and scalability when transferring large files.

The multipart upload API workflow consists of three primary operations:

InitiateMultipartUpload starts the upload session and returns an upload ID.

UploadPart uploads individual parts using that upload ID.

CompleteMultipartUpload assembles the uploaded parts into the final object.

If an upload fails or is abandoned, AbortMultipartUpload can remove the uploaded parts.

Multipart uploads are especially beneficial for large objects, unreliable networks, and high-performance upload systems.
