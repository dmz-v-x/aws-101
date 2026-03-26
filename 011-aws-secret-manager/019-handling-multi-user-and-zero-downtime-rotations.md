## Handling Multi-User / Zero-Downtime Rotations in AWS Secrets Manager (Alternating Users Strategy)

### 1. Introduction

One of the biggest challenges with credential rotation is ensuring **applications continue running while credentials change**.

If a password is rotated instantly, applications that are still using the old credential may fail.

Example problem:

```
Application → uses old database password
Secrets rotated → password changed
Application connection fails
```

This leads to **downtime**.

To avoid this issue, AWS recommends using a **multi-user rotation strategy**, also called the **alternating users strategy**.

This approach ensures **zero downtime secret rotation** by alternating between two users.

---

### 2. What is the Alternating Users Strategy?

The alternating users strategy uses **two separate users** in the target system.

Example:

```
app_user_A
app_user_B
```

Only **one user is active at a time**.

When rotation occurs, the system switches to the other user.

Example flow:

```
Current active user → app_user_A
Next rotation → switch to app_user_B
Next rotation → switch back to app_user_A
```

This approach ensures that existing connections continue working.

---

### 3. Why This Strategy Works

Applications may:

- cache credentials
- keep database connections open
- refresh credentials periodically

If a password changes suddenly, these connections break.

With alternating users:

```
Old user → still valid for existing connections
New user → used by new connections
```

Eventually, all connections migrate to the new credential.

---

### 4. Example Database Setup

Suppose we have two database users.

Example:

```
User 1 → app_user_A
User 2 → app_user_B
```

Initial state:

```
Active user → app_user_A
Password stored in Secrets Manager
```

Applications authenticate using `app_user_A`.

---

### 5. First Rotation

During rotation, Secrets Manager performs these steps.

Step 1 — Generate a new password for `app_user_B`.

Step 2 — Update database password for `app_user_B`.

Step 3 — Store new credentials in Secrets Manager.

Step 4 — Promote `app_user_B` as the active user.

Result:

```
Active user → app_user_B
Previous user → app_user_A
```

New application connections use `app_user_B`.

Existing connections using `app_user_A` continue working.

---

### 6. Second Rotation

During the next rotation cycle:

Step 1 — Generate new password for `app_user_A`.

Step 2 — Update database password.

Step 3 — Promote `app_user_A` again.

Result:

```
Active user → app_user_A
Previous user → app_user_B
```

This cycle continues indefinitely.

---

### 7. Alternating Rotation Workflow

Full rotation workflow:

```
Initial state:
Active user → A

Rotation 1:
Switch to user B

Rotation 2:
Switch back to user A

Rotation 3:
Switch to user B
```

This alternating pattern ensures seamless rotation.

---

### 8. How Secrets Manager Supports Alternating Users

AWS Secrets Manager provides built-in **rotation templates** for databases that support alternating users.

Examples include:

- Amazon RDS
- Amazon Aurora
- PostgreSQL
- MySQL
- Oracle
- Microsoft SQL Server

These templates automatically implement:

```
createSecret
setSecret
testSecret
finishSecret
```

with alternating user logic.

---

### 9. Alternating Users Rotation Flow

The rotation process typically follows this pattern:

```
Step 1: Identify inactive user
Step 2: Generate new password
Step 3: Update database user password
Step 4: Store secret in Secrets Manager
Step 5: Promote new credential
```

During this process:

```
Inactive user becomes active
Active user becomes standby
```

---

### 10. Visual Architecture

Example architecture:

```
                AWS Secrets Manager
                         |
                         v
                Rotation Lambda Function
                         |
                         v
                   Database Server
                   /            \
            app_user_A       app_user_B
```

Secrets Manager alternates between the two users during rotation.

---

### 11. Benefits of Alternating Users

This strategy provides several benefits.

#### Zero Downtime

Applications continue operating during rotation.

---

#### Connection Stability

Existing database sessions remain valid.

---

#### Safe Credential Migration

New connections gradually adopt the new credential.

---

#### Reduced Operational Risk

Credential changes do not interrupt services.

---

### 12. Example Rotation Timeline

Example scenario:

```
00:00 → app_user_A active
00:05 → rotation begins
00:10 → app_user_B becomes active
```

Existing connections:

```
Connection 1 → still using app_user_A
Connection 2 → new connection uses app_user_B
```

Eventually:

```
All connections migrate to app_user_B
```

Next rotation switches back.

---

### 13. Application Credential Refresh Strategy

Applications should refresh secrets periodically.

Example:

```
Secrets refresh interval = 5 minutes
```

This ensures applications switch to the new credential quickly.

---

### 14. Monitoring Rotation

Important monitoring tools include:

- CloudWatch Logs
- CloudTrail
- Lambda logs

Monitor for events such as:

```
Rotation failure
Authentication errors
Database update failures
```

Monitoring ensures rotation remains reliable.

---

### 15. Best Practices

Follow these best practices when implementing multi-user rotation.

#### Use alternating users for database secrets

This ensures zero downtime.

---

#### Avoid single-user rotation

Single-user rotation may break existing connections.

---

#### Use short credential caching

Applications should refresh secrets frequently.

---

#### Test rotation in staging

Always validate rotation logic before production deployment.

---

### 16. Summary

The **alternating users strategy** is the recommended approach for performing **zero-downtime secret rotation**.

Instead of rotating a single credential, the system alternates between two users.

Example:

```
User A → active
User B → standby

Rotation → switch roles
```

Key benefits include:

| Benefit | Description |
|-------|-------------|
| Zero downtime | Existing connections remain valid |
| Safer rotations | Credentials change without breaking services |
| Gradual migration | Applications switch to new credentials smoothly |

This strategy is widely used for rotating credentials in systems such as **databases, authentication services, and application backends** where maintaining continuous service availability is critical.
