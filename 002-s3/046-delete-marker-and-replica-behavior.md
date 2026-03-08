## Amazon S3 Delete Markers and Replica Behavior

### 1. Introduction

Amazon S3 replication copies objects from a **source bucket** to a **destination bucket**. When versioning is enabled (which is required for replication), deleting objects behaves differently than in non-versioned buckets.

Instead of permanently deleting objects, S3 creates **delete markers**.

Understanding how delete markers interact with replication is important because they affect how objects are removed, hidden, or preserved across replicated buckets.

This topic explains:

- what delete markers are  
- how they behave in versioned buckets  
- how replication handles delete markers  
- how replica objects behave during deletes  

---

### 2. Versioning and Object Deletion

In a versioned bucket, deleting an object does **not remove the data immediately**.

Instead, S3 creates a **delete marker**, which becomes the latest version of the object.

Example:

```
Object versions:

Version 1 → file.txt
Version 2 → file.txt
Delete marker → file.txt
```

The delete marker hides previous versions from normal requests.

The actual data still exists in earlier versions.

---

### 3. What is a Delete Marker?

A delete marker is a special object version that indicates an object has been deleted.

Characteristics of delete markers:

```
No data content
Unique version ID
Becomes the latest version
```

Conceptual example:

```
Object versions:

Version 1 → file.txt
Version 2 → file.txt
Version 3 → Delete marker
```

When users request the object normally, S3 returns a **404 Not Found**.

---

### 4. How Delete Markers Affect Object Visibility

When a delete marker exists:

```
GET file.txt
```

Result:

```
Object appears deleted
```

However, earlier versions still exist.

Example:

```
GET file.txt?versionId=Version1
```

The object can still be retrieved.

This behavior enables **recovery from accidental deletion**.

---

### 5. Replication and Delete Markers

Replication rules can control how delete markers behave.

Two main options exist:

```
Replicate delete markers
Do not replicate delete markers
```

The replication configuration determines how deletes propagate to the destination bucket.

---

### 6. Default Behavior

By default:

```
Delete markers are NOT replicated
```

Example scenario:

```
Object uploaded → replicated
Object deleted → delete marker created
```

Destination bucket behavior:

```
Original object still exists
```

The deletion in the source bucket does not remove the replicated object.

---

### 7. Example Without Delete Marker Replication

Example workflow:

```
Source bucket
   ↓
file.txt uploaded
   ↓
Replicated to destination
```

Later:

```
file.txt deleted in source
Delete marker created
```

Destination bucket result:

```
file.txt still exists
```

The replica remains intact.

This can be useful for backup scenarios.

---

### 8. Enabling Delete Marker Replication

Replication rules can enable delete marker replication.

Example configuration concept:

```
DeleteMarkerReplication = Enabled
```

When enabled:

```
Delete marker created in source
        ↓
Delete marker replicated
        ↓
Replica object hidden
```

This keeps both buckets synchronized.

---

### 9. Example With Delete Marker Replication

Example workflow:

```
file.txt uploaded
        ↓
Replicated to destination
```

Later:

```
file.txt deleted in source
        ↓
Delete marker replicated
```

Result:

```
Destination object appears deleted
```

Both buckets behave the same.

---

### 10. Replica Object Behavior

Replicated objects contain metadata indicating they are replicas.

Example status:

```
ReplicationStatus = REPLICA
```

Replica objects cannot be modified directly.

This prevents inconsistencies between source and destination objects.

---

### 11. Deleting Replica Objects

Replica objects cannot be deleted directly.

Example:

```
Attempt to delete replica object
```

Result:

```
Access denied
```

Reason:

Replication ensures the source bucket controls the lifecycle of replicated objects.

Deletion must occur in the source bucket.

---

### 12. Replica Modification Restrictions

Replica objects also cannot be modified directly.

Example restricted operations:

```
Metadata updates
Tag modifications
Object overwrites
```

These changes must occur in the source bucket.

The changes will then replicate automatically.

---

### 13. Delete Marker Replication Limitations

Delete marker replication applies only to **new delete markers created after replication rules are configured**.

Example:

```
Replication rule enabled today
```

Delete markers created earlier are not replicated.

This behavior ensures replication consistency.

---

### 14. Permanent Deletion Behavior

Permanent deletion of object versions behaves differently.

Example:

```
Delete specific version
```

Replication behavior:

```
Version deletion NOT replicated
```

Replication typically only copies delete markers, not version deletions.

This helps prevent accidental data loss.

---

### 15. Lifecycle Policies and Delete Markers

Lifecycle rules may also interact with delete markers.

Example lifecycle rule:

```
Expire objects after 30 days
```

Result:

```
Delete marker created
```

If delete marker replication is enabled, this marker will also replicate.

Lifecycle policies can therefore propagate deletions across buckets.

---

### 16. Delete Marker Cleanup

Delete markers themselves can accumulate.

Lifecycle rules can remove expired delete markers.

Example:

```
Remove expired delete markers
```

This helps keep bucket version history manageable.

---

### 17. Monitoring Replication Status

Replication metadata fields show object status.

Example values:

```
PENDING
COMPLETED
FAILED
REPLICA
```

Delete marker replication also generates these statuses.

Monitoring helps detect replication delays.

---

### 18. Example Backup Architecture

Example backup configuration:

```
Source bucket → production
Destination bucket → backup
```

Delete marker replication disabled.

Workflow:

```
Object uploaded → replicated
Object deleted → delete marker created
```

Destination bucket result:

```
Object still exists
```

This protects against accidental deletion.

---

### 19. Example Mirror Architecture

Example synchronization architecture:

```
Source bucket → primary
Destination bucket → secondary
```

Delete marker replication enabled.

Workflow:

```
Object uploaded → replicated
Object deleted → delete marker replicated
```

Both buckets remain synchronized.

---

### 20. Choosing Delete Marker Behavior

Choose **delete marker replication enabled** when:

```
Buckets must stay identical
Data synchronization required
```

Choose **delete marker replication disabled** when:

```
Destination bucket acts as backup
Protection from accidental deletes needed
```

The choice depends on system requirements.

---

### 21. Mental Model

A useful mental model:

```
Delete marker = logical delete
```

Replication behavior:

```
Replicate marker → delete mirrored
Do not replicate marker → backup preserved
```

Understanding this distinction is important for disaster recovery strategies.

---

### 22. Summary

In versioned S3 buckets, deleting an object creates a **delete marker** rather than permanently removing the object.

Replication rules determine whether delete markers are copied to the destination bucket.

If delete marker replication is disabled, deleting an object in the source bucket does not remove the replicated object, which allows the destination bucket to function as a backup.

If delete marker replication is enabled, the delete marker is replicated and the object appears deleted in the destination bucket as well.

Replica objects themselves cannot be modified or deleted directly, ensuring that the source bucket remains the authoritative source of data changes.

Understanding delete marker behavior is essential when designing S3 replication architectures for backup, synchronization, or disaster recovery.
