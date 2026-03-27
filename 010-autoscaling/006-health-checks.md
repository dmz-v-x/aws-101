## Health Checks in AWS Auto Scaling — EC2 Health Checks, ELB Health Checks, and Custom Health Checks

### 1. Introduction

In a scalable cloud environment, simply launching servers is not enough. The system must also ensure that those servers are **healthy and capable of serving application traffic**.

Servers can become unhealthy for many reasons:

- Operating system crashes
- Application failures
- Memory leaks
- Network issues
- Configuration problems
- Dependency failures

If unhealthy servers continue receiving traffic, users may experience:

```
Application errors
Slow response times
Failed requests
Service downtime
```

To prevent this, AWS Auto Scaling uses **health checks**.

Health checks allow the system to continuously monitor the state of instances and automatically replace instances that become unhealthy.

This capability is often called:

```
Auto healing infrastructure
```

Health checks are a fundamental component of building **reliable and fault-tolerant cloud architectures**.

---

### 2. What Are Health Checks?

A **health check** is a mechanism that determines whether an instance is functioning correctly.

If the system detects that an instance is unhealthy:

```
Auto Scaling terminates the instance
Auto Scaling launches a replacement instance
```

The replacement instance is launched using the **Launch Template or Launch Configuration** defined in the Auto Scaling Group.

The goal of health checks is to ensure that the Auto Scaling Group always maintains:

```
Healthy
Available
Operational
Instances
```

---

### 3. Why Health Checks Are Important

Without health checks, an Auto Scaling Group may still maintain the desired number of instances, but some of those instances might not be functioning properly.

Example:

```
Desired capacity = 4
```

But:

```
Instance 1 → healthy
Instance 2 → healthy
Instance 3 → application crashed
Instance 4 → network issue
```

Technically, the Auto Scaling Group still has **4 instances running**, but only **2 are actually usable**.

Health checks detect these problems and trigger **automatic recovery**.

---

### 4. Types of Health Checks in Auto Scaling

AWS Auto Scaling supports several types of health checks.

The main ones are:

```
EC2 Health Checks
Elastic Load Balancer (ELB) Health Checks
Custom Health Checks
```

Each type verifies a different aspect of instance health.

---

### 5. EC2 Health Checks

EC2 health checks verify the **basic operational state of the instance at the infrastructure level**.

They are performed by the EC2 service itself.

These checks focus on whether the instance is:

```
Running
Reachable
Functioning at the system level
```

---

### 5.1 What EC2 Health Checks Monitor

EC2 health checks monitor two main components:

```
System status checks
Instance status checks
```

---

#### System Status Checks

System status checks monitor the **underlying AWS infrastructure** that hosts the instance.

This includes:

```
Physical host hardware
Network infrastructure
Hypervisor
Power supply
```

Example problems detected:

```
Hardware failure
Network outage in data center
Host-level issues
```

If AWS detects a system-level problem, the instance may be marked as **unhealthy**.

---

#### Instance Status Checks

Instance status checks monitor the **instance operating system**.

Examples of issues detected include:

```
Operating system crashes
Kernel panic
Network misconfiguration
System boot failure
```

These checks verify that the instance itself is functioning correctly.

---

### 5.2 Limitations of EC2 Health Checks

EC2 health checks only verify whether the instance is **running and reachable**.

They do **not verify whether the application inside the instance is functioning correctly**.

Example scenario:

```
Instance running normally
Operating system healthy
But application server crashed
```

In this case:

```
EC2 health check = healthy
Application = unhealthy
```

To detect application-level failures, additional health checks are required.

---

### 6. Elastic Load Balancer (ELB) Health Checks

Elastic Load Balancer health checks verify the **health of the application running inside instances**.

These checks are performed by the load balancer.

Instead of checking infrastructure status, ELB health checks verify whether the **application endpoint responds correctly**.

---

### 6.1 How ELB Health Checks Work

A load balancer periodically sends a request to each registered instance.

Example request:

```
GET /health
```

If the instance responds successfully:

```
HTTP 200 OK
```

The instance is considered **healthy**.

If the instance fails to respond correctly:

```
HTTP 500
Timeout
Connection refused
```

The instance is marked **unhealthy**.

---

### 6.2 Example ELB Health Check Configuration

Typical health check settings include:

```
Protocol → HTTP
Path → /health
Port → 80
Healthy threshold → 3 successful responses
Unhealthy threshold → 2 failures
Interval → 30 seconds
Timeout → 5 seconds
```

This means:

```
Load balancer checks instance every 30 seconds
If 2 checks fail → instance unhealthy
If 3 checks succeed → instance healthy
```

---

### 6.3 Integration with Auto Scaling

When ELB health checks detect an unhealthy instance:

```
Load balancer stops sending traffic
Auto Scaling terminates the instance
Auto Scaling launches a replacement
```

This process ensures that only **healthy instances receive traffic**.

---

### 7. Custom Health Checks

Sometimes application health cannot be determined by simple HTTP responses.

In such cases, organizations implement **custom health checks**.

Custom health checks allow applications to report their own health status.

---

### 7.1 Why Custom Health Checks Are Needed

Modern applications often depend on multiple components.

Example dependencies:

```
Database
Cache system
External APIs
Message queues
Authentication services
```

An application may appear healthy at the infrastructure level but fail due to dependency issues.

Example scenario:

```
Application server running
Database connection broken
Application cannot serve requests
```

EC2 and ELB health checks may not detect this situation.

Custom health checks can.

---

### 7.2 Example Custom Health Endpoint

Applications often implement a dedicated endpoint such as:

```
/health
/status
/ready
```

Example response:

```
{
  "status": "healthy",
  "database": "connected",
  "cache": "connected",
  "messageQueue": "connected"
}
```

If any dependency fails, the endpoint returns an error.

Example:

```
{
  "status": "unhealthy",
  "database": "connection_failed"
}
```

The load balancer can then mark the instance unhealthy.

---

### 7.3 Custom Health Check Workflow

Example process:

```
Application dependency fails
↓
Health endpoint reports unhealthy status
↓
Load balancer health check fails
↓
Instance marked unhealthy
↓
Auto Scaling replaces instance
```

This ensures deeper monitoring of application health.

---

### 8. Instance Replacement Process

When Auto Scaling detects an unhealthy instance, it performs a replacement sequence.

Example workflow:

```
Instance becomes unhealthy
↓
Auto Scaling marks instance unhealthy
↓
Instance removed from load balancer
↓
Instance terminated
↓
New instance launched
↓
Instance passes health checks
↓
Instance added to load balancer
```

This process maintains **service availability**.

---

### 9. Health Check Grace Period

When a new instance launches, it may take time for the application to start.

Examples include:

```
Application startup
Database initialization
Service registration
Cache warming
```

During this startup phase, the instance may temporarily fail health checks.

To prevent premature termination, Auto Scaling uses a **health check grace period**.

Example configuration:

```
Health check grace period = 300 seconds
```

During this time:

```
Health check failures are ignored
```

This allows the application to fully initialize.

---

### 10. Choosing the Right Health Check Type

Different health checks serve different purposes.

---

#### EC2 Health Checks

Best for detecting:

```
Infrastructure failures
Operating system crashes
Network issues
```

---

#### ELB Health Checks

Best for detecting:

```
Application failures
HTTP endpoint issues
Service unavailability
```

---

#### Custom Health Checks

Best for detecting:

```
Dependency failures
Business logic errors
Complex application health conditions
```

---

### 11. Best Practices for Health Checks

To design reliable systems, several best practices should be followed.

---

#### Use Load Balancer Health Checks

Always enable ELB health checks when using Auto Scaling with load balancers.

---

#### Implement Application Health Endpoints

Create dedicated endpoints such as:

```
/health
/ready
/status
```

---

#### Configure Reasonable Health Check Intervals

Health checks should run frequently enough to detect failures quickly but not so frequently that they cause unnecessary load.

---

#### Use Grace Periods

Allow enough startup time for instances before evaluating health.

---

### 12. Example End-to-End Health Monitoring Flow

A typical health monitoring process may look like this:

```
Instance launches
↓
Application starts
↓
Health endpoint becomes available
↓
Load balancer begins health checks
↓
Instance marked healthy
↓
Traffic routed to instance
```

If a failure occurs:

```
Application crash
↓
Health check fails
↓
Instance marked unhealthy
↓
Auto Scaling terminates instance
↓
Replacement instance launched
```

---

### 13. Summary

Health checks are a critical component of AWS Auto Scaling.

They ensure that only **healthy instances serve application traffic** and that failed instances are automatically replaced.

AWS Auto Scaling supports multiple types of health checks:

```
EC2 Health Checks → infrastructure level
ELB Health Checks → application level
Custom Health Checks → dependency and business logic level
```

By combining these health checks with Auto Scaling Groups and load balancers, AWS enables systems to become **self-healing, resilient, and highly available**.
