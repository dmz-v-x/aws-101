## Compression (Gzip / Brotli), File Formats, and Range Requests in CloudFront

### 1. Why Compression Matters

#### 1.1 Problem

Large files:
- Take longer to transfer  
- Increase bandwidth cost  
- Slow down user experience  

---

#### 1.2 Solution

Compression reduces file size before sending to user.

---

#### 1.3 Key Insight

> Smaller payload = faster delivery + lower cost

---

### 2. Types of Compression in CloudFront

---

#### 2.1 Gzip Compression

- Older but widely supported  
- Works on almost all browsers  

---

#### 2.2 Brotli Compression

- Newer and more efficient  
- Better compression ratio than gzip  

---

#### 2.3 Key Insight

> Brotli > Gzip in efficiency, but depends on browser support

---

### 3. How CloudFront Decides Compression

---

#### 3.1 Based on Viewer Request

Browser sends:

```
Accept-Encoding: gzip, br
```

---

#### 3.2 CloudFront Behavior

- Checks if compression is enabled  
- Checks file type  
- Checks if object size qualifies  
- Returns compressed version if supported  

---

#### 3.3 Key Insight

> Compression is **viewer-driven + CloudFront-controlled**

---

### 4. Enabling Compression in CloudFront

---

#### 4.1 Setting

In cache behavior:

- Enable:
  - **Compress objects automatically**

---

#### 4.2 Result

CloudFront:
- Compresses eligible files  
- Stores compressed version in cache  

---

### 5. Supported File Types for Compression

---

#### 5.1 Compressible Types

- HTML  
- CSS  
- JavaScript  
- JSON  
- XML  
- Text files  

---

#### 5.2 Non-Compressible Types

- Images (JPEG, PNG, GIF)  
- Videos (MP4, etc.)  
- Already compressed formats  

---

#### 5.3 Key Insight

> Compression is useful only for **text-based content**

---

### 6. Size Requirements for Compression

---

#### 6.1 Minimum Size

- Objects must be **> 1 KB**

---

#### 6.2 Maximum Size

- Typically up to **10 MB** for automatic compression  

---

#### 6.3 Key Insight

> Very small or very large files may not be compressed

---

### 7. Pre-Compressed Objects

---

#### 7.1 What It Means

You upload files already compressed:

```
app.js.gz
app.js.br
```

---

#### 7.2 How It Works

- Origin sends compressed file  
- Includes header:

```
Content-Encoding: gzip
```

---

#### 7.3 CloudFront Behavior

- Does NOT recompress  
- Passes through as-is  

---

#### 7.4 When to Use Pre-Compression

- Full control over compression  
- Optimized build pipelines  
- Better compression than CloudFront defaults  

---

#### 7.5 Key Insight

> Pre-compressed objects give **maximum control and efficiency**

---

### 8. Automatic vs Pre-Compressed (Comparison)

---

| Feature | Automatic Compression | Pre-Compressed |
|--------|---------------------|---------------|
| Setup | Simple | Requires build setup |
| Control | Limited | Full control |
| Performance | Good | Best |
| Flexibility | Low | High |

---

### 9. Brotli vs Gzip Behavior

---

#### 9.1 Priority

If browser supports Brotli:

→ CloudFront prefers **Brotli**

Else:

→ Falls back to **gzip**

---

#### 9.2 Example

```
Accept-Encoding: br, gzip
```

Response:
```
Content-Encoding: br
```

---

#### 9.3 Key Insight

> CloudFront automatically selects the **best supported encoding**

---

### 10. Caching and Compression

---

#### 10.1 Separate Cache Entries

CloudFront stores:

- Gzip version  
- Brotli version  

Separately

---

#### 10.2 Why?

Because:
- Different encodings = different responses  

---

#### 10.3 Key Insight

> Compression affects cache key implicitly

---

### 11. Viewer Support Caveats

---

#### 11.1 Older Browsers

- May not support Brotli  
- Fall back to gzip or no compression  

---

#### 11.2 Missing Headers

If browser does NOT send:

```
Accept-Encoding
```

→ No compression applied  

---

#### 11.3 HTTPS Requirement for Brotli

- Brotli is supported mainly over HTTPS  

---

#### 11.4 Key Insight

> Compression depends on **client capabilities**

---

### 12. Range Requests

---

#### 12.1 What is a Range Request?

A request for a **portion of a file** instead of the whole file.

---

#### 12.2 Example

```
Range: bytes=0-1023
```

---

#### 12.3 Use Cases

- Video streaming  
- Resume downloads  
- Large file handling  

---

### 13. How CloudFront Handles Range Requests

---

#### 13.1 Flow

1. Viewer requests partial content  
2. CloudFront checks cache  
3. If available → serves requested range  
4. Else → fetches from origin  

---

#### 13.2 Response

```
HTTP 206 Partial Content
```

---

#### 13.3 Key Insight

> Range requests improve efficiency for large files

---

### 14. Compression + Range Requests

---

#### 14.1 Important Interaction

- Range requests typically work with **uncompressed content**

---

#### 14.2 Why?

Compression changes byte offsets:
- Makes partial delivery complex  

---

#### 14.3 Result

- CloudFront may disable compression for range requests  

---

#### 14.4 Key Insight

> Compression and range requests are **not always compatible**

---

### 15. Performance Implications

---

#### 15.1 Benefits of Compression

- Faster page loads  
- Reduced bandwidth usage  
- Lower cost  

---

#### 15.2 Benefits of Range Requests

- Efficient large file delivery  
- Better streaming performance  

---

### 16. Best Practices

---

#### 16.1 Enable Compression

- Always enable automatic compression  

---

#### 16.2 Use Brotli

- Ensure HTTPS  
- Let CloudFront handle negotiation  

---

#### 16.3 Use Pre-Compression for Critical Assets

- JS/CSS bundles  
- Large text files  

---

#### 16.4 Optimize File Types

- Compress only text-based files  

---

#### 16.5 Handle Range Requests Carefully

- Especially for video/media delivery  

---

### 17. Common Mistakes

---

#### Mistake 1: Expecting images to compress

- Already compressed  

---

#### Mistake 2: Ignoring Accept-Encoding header

- Leads to no compression  

---

#### Mistake 3: Not enabling compression

- Wastes bandwidth  

---

#### Mistake 4: Mixing compression with range-heavy workloads

- Can cause unexpected behavior  

---

### 18. Final Mental Model

> CloudFront optimizes delivery using:

- **Compression** → reduces size of responses  
- **Content negotiation** → serves best format (Brotli/gzip)  
- **Range requests** → delivers partial content efficiently  

Together, they ensure:
- Faster performance  
- Lower costs  
- Better user experience  
