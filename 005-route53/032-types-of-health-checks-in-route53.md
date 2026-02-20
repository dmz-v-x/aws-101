## Types of Health Checks in Route 53

### 1. Absolute Zero — Why Multiple Health Check Types Exist

At first glance, you might think:

“A health check just checks if a server is alive.”

Reality:

Modern systems are far more complex.

Failures may involve:

• Infrastructure  
• Network  
• Application logic  
• Dependencies  
• Internal metrics  
• Composite conditions

One check type cannot cover everything.

---

### 2. Big Picture Mental Model

Route 53 health checks operate at:

**Different layers of system health visibility**

From:

Simple reachability → Complex logic → Metric-driven decisions.

---

## ENDPOINT HEALTH CHECKS — MOST BASIC & COMMON

---

### 3. Absolute Zero — What Is an Endpoint Health Check?

**Endpoint health checks monitor whether a specific endpoint is reachable and responsive.**

Think:

“Can I talk to this server?”

---

### 4. What Route 53 Actually Does

Route 53 sends probes:

To your configured endpoint.

Valid response → Healthy  
Failure/no response → Unhealthy

---

### 5. Supported Endpoint Protocols

Typical checks:

• HTTP / HTTPS  
• TCP

Each tests different connectivity assumptions.

---

### 6. HTTP Health Checks — Behavior

Route 53 performs:

HTTP request → Evaluates response.

Healthy if:

• Valid status code  
• Response within timeout  
• Meets configured criteria

---

### 7. TCP Health Checks — Behavior

Simpler logic:

Can TCP connection be established?

Yes → Healthy  
No → Unhealthy

---

### 8. What Endpoint Checks Are Good At

Detecting:

• Server downtime  
• Network failures  
• Service unreachability  
• Basic availability problems

---

### 9. What Endpoint Checks CANNOT Detect

They do NOT detect:

• Logical application errors  
• Slow performance  
• Dependency failures  
• Partial degradation  
• Business logic issues

Reachability ≠ Correctness.

---

### 10. Gotcha — “Always 200 OK” Endpoints

If endpoint always returns success:

Health check becomes meaningless.

Must reflect:

Real service health.

---

## CALCULATED HEALTH CHECKS — LOGIC-LEVEL MONITORING

---

### 11. Absolute Zero — What Is a Calculated Health Check?

**Calculated health checks combine multiple health checks into a single logical decision.**

Think:

Composite health evaluation.

---

### 12. Why Calculated Checks Exist

Real systems often depend on:

Multiple components.

Example:

• Web server  
• Database  
• Cache  
• API dependency

Single endpoint insufficient.

---

### 13. Example Scenario

Application healthy only if:

• Web server healthy  
• Database healthy  
• API dependency healthy

Calculated check aggregates signals.

---

### 14. Conceptual Logic Model

Calculated health check evaluates:

Boolean logic across checks.

Example:

Healthy if:

• At least 2 of 3 checks pass  
OR  
• All checks pass  
OR  
• Specific combinations succeed

---

### 15. Why This Mechanism Is Powerful

Enables:

• Multi-component health modeling  
• Dependency-aware routing  
• Failure tolerance logic  
• Sophisticated availability decisions

---

### 16. Example — Redundant Systems

Healthy if:

Any 1 of multiple servers alive.

Avoids unnecessary failovers.

---

### 17. Example — Critical Dependencies

Healthy only if:

ALL required services healthy.

Strict correctness enforcement.

---

### 18. Gotcha — Cascading Failures

Improper logic design may:

Trigger excessive unhealthy states.

Design logic carefully.

---

### 19. Gotcha — Complexity Explosion

Too many combined checks:

Hard to reason about.

Keep logic understandable.

---

## CLOUDWATCH ALARM-BASED CHECKS — METRIC-LEVEL HEALTH

---

### 20. Absolute Zero — What Is a CloudWatch-Based Health Check?

**CloudWatch alarm-based checks use AWS metrics instead of network probes.**

Health defined by:

System metrics.

Not reachability.

---

### 21. Why Metric-Based Checks Exist

Some failures are invisible to:

Network-level probing.

Example:

• High CPU usage  
• Memory exhaustion  
• Error rate spikes  
• Latency degradation  
• Queue saturation

Endpoint still responds → But system unhealthy.

---

### 22. How This Works Conceptually

CloudWatch monitors metrics → Alarm triggered → Route 53 health check reacts.

Healthy ↔ Unhealthy driven by metrics.

---

### 23. Example — CPU-Based Health

Healthy if:

CPU < Threshold.

High CPU → Mark unhealthy → DNS reroutes traffic.

---

### 24. Example — Error Rate Monitoring

Healthy if:

Error rate low.

Error spike → Traffic redirected.

---

### 25. Example — Latency Thresholds

Healthy if:

Latency acceptable.

Performance-aware routing decisions.

---

### 26. Why This Mechanism Is Extremely Powerful

Enables:

• Performance-driven routing  
• Predictive failure mitigation  
• Load-shedding strategies  
• Early failure detection  
• Deep system visibility

---

### 27. Gotcha — Metric Selection Matters

Wrong metric → Wrong routing decisions.

Health definition critical.

---

### 28. Gotcha — Alarm Sensitivity

Overly sensitive alarms:

Cause routing instability.

Poor UX.

---

### 29. Gotcha — CloudWatch ≠ External Availability

Metric-based checks require:

Correct AWS monitoring setup.

---

## COMPARING HEALTH CHECK TYPES — CLEAR DISTINCTIONS

---

### 30. Endpoint Health Checks

Measure:

Reachability & basic responsiveness.

Best for:

Infrastructure/network failures.

---

### 31. Calculated Health Checks

Measure:

Logical aggregation of multiple checks.

Best for:

Complex system health modeling.

---

### 32. CloudWatch Alarm-Based Checks

Measure:

Metric-defined health state.

Best for:

Performance & internal system behavior.

---

## WHEN TO USE WHICH TYPE

---

### 33. Use Endpoint Checks When

You need:

Basic availability detection.

---

### 34. Use Calculated Checks When

System health depends on:

Multiple components.

---

### 35. Use CloudWatch Checks When

Health depends on:

Metrics & performance indicators.

---

### 36. Combine Types for Sophisticated Architectures

Example:

Endpoint checks + Metric checks + Calculated logic.

Multi-layer health modeling.

---

## COMMON BEGINNER MISCONCEPTIONS

---

### 37. Misconception #1 — All Health Checks Are Same

Incorrect.

Different visibility layers.

---

### 38. Misconception #2 — Endpoint Checks Enough for Everything

Incorrect.

Cannot detect performance degradation.

---

### 39. Misconception #3 — Metric Checks Replace Monitoring Systems

Incorrect.

Monitoring & routing serve different roles.

---

### 40. Misconception #4 — More Checks Always Better

Incorrect.

Complexity & instability risks increase.

---

### 41. Misconception #5 — Health Checks Guarantee Perfect Decisions

Incorrect.

Depend on design & configuration.

---

## FINAL MENTAL MODEL

---

### 42. What Health Check Types Truly Represent

Health check types represent:

**Different layers of system health intelligence**

Endpoint → Connectivity health  
Calculated → Logical/system health  
CloudWatch → Metric/performance health

---

### 43. Why This Matters for Architecture Design

Choosing correct health check type affects:

• Reliability  
• Stability  
• Performance  
• Routing correctness  
• Failure response behavior

---

### 44. Elegant Big Picture

Route 53 health checks evolve from:

“Is server alive?”

To:

“Is system functioning correctly according to infrastructure, logic, and metrics?”

DNS becomes:

Intelligent reliability control plane.
