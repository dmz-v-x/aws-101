## Weighted Routing in Route 53 

### 1. Absolute Zero — What Is Weighted Routing?

At the simplest level:

**Weighted Routing means distributing traffic across multiple resources based on assigned weights.**

Example intuition:

    example.com → Server A (70%)
    example.com → Server B (30%)

Core idea:

Traffic splitting via DNS.

---

### 2. Why Weighted Routing Exists

Modern systems rarely rely on:

Single endpoint.

We often need:

• Gradual rollouts  
• Controlled experiments  
• Risk reduction  
• Traffic shaping  
• Deployment safety

Weighted routing provides controlled distribution.

---

### 3. How Weighted Routing Works (Core Mechanism)

Instead of returning:

Single DNS answer.

DNS may return:

Multiple possible answers.

Selection probability influenced by weights.

---

### 4. Important Clarification — DNS Is Probabilistic

Weights do NOT guarantee exact percentages.

They influence:

Probability of selection.

Traffic distribution is statistical over time.

---

## TRAFFIC SPLITTING — CORE APPLICATION

---

### 5. What Traffic Splitting Means

Traffic splitting means:

Sending users to different infrastructure versions.

Example:

Old system ↔ New system.

---

### 6. Why Traffic Splitting Matters

Because full cutovers are risky:

• Unknown bugs  
• Performance issues  
• Infrastructure instability  
• Unexpected failures

Gradual exposure safer.

---

### 7. Example — Gradual Rollout

Initial:

    95% → Stable version
    5%  → New version

If stable:

Increase weight progressively.

---

### 8. Why Gradual Rollout Is Critical

Reduces blast radius.

Failures affect:

Small subset instead of entire user base.

---

## BLUE-GREEN DEPLOYMENTS — CLASSIC PATTERN

---

### 9. What Blue-Green Deployment Means

Two environments:

• Blue → Current production  
• Green → New version

Both live simultaneously.

---

### 10. Weighted Routing Role in Blue-Green

DNS splits traffic:

Between Blue & Green.

Example:

    Blue → 100%
    Green → 0%

Gradually shift.

---

### 11. Why This Model Is Powerful

Enables:

• Instant rollback  
• Safe validation  
• Controlled migration  
• Minimal downtime

Infrastructure-level deployment safety.

---

### 12. Rollback Simplicity — Major Advantage

If Green fails:

Shift weight back to Blue.

No redeployment required.

---

### 13. Gotcha — DNS Caching Effects

Traffic shifts not instantaneous.

Resolvers cache responses.

TTL governs responsiveness.

---

## A/B TESTING — EXPERIMENTATION STRATEGY

---

### 14. What A/B Testing Means

Serve different variants:

To measure user behavior differences.

Example:

• UI Version A  
• UI Version B

---

### 15. Weighted Routing Role in A/B Testing

DNS distributes traffic:

Across variants.

Example:

    Variant A → 50%
    Variant B → 50%

---

### 16. Why DNS-Level A/B Testing Works

Routing occurs:

Before application logic.

No app-level traffic rules required.

---

### 17. Limitations of DNS-Based A/B Testing

DNS cannot guarantee:

Consistent user assignment.

Caching & resolver behavior introduce variability.

Better for coarse experiments.

---

## TRAFFIC SHAPING — ADVANCED VIEW

---

### 18. Weighted Routing as Traffic Control Tool

Beyond deployments:

Used for:

• Infrastructure migration  
• Region balancing  
• Cost optimization  
• Load redistribution

---

### 19. Example — Infrastructure Migration

Old Data Center → 80%
New Data Center → 20%

Gradual migration.

---

### 20. Example — Risk Mitigation

Uncertain endpoint → Small traffic exposure.

Limits failure impact.

---

## HOW WEIGHTS ACTUALLY BEHAVE

---

### 21. Weight Values Are Relative

Weights are NOT percentages.

They are ratios.

Example:

    Server A → Weight 70
    Server B → Weight 30

Equivalent to:

70:30 distribution ratio.

---

### 22. Example — Equal Distribution

    Weight 1 → Server A
    Weight 1 → Server B

Approximate 50/50 split.

---

### 23. Example — Skewed Distribution

    Weight 99 → Stable system
    Weight 1  → Canary system

Minimal exposure testing.

---

### 24. Gotcha — Small Sample Sizes Misleading

Short-term traffic may appear uneven.

Statistical smoothing occurs over time.

---

## COMMON BEGINNER MISCONCEPTIONS

---

### 25. Misconception #1 — Weights Guarantee Exact Percentages

Incorrect.

DNS routing is probabilistic.

---

### 26. Misconception #2 — Weighted Routing = Load Balancer Replacement

Incorrect.

DNS-level vs connection-level routing.

Complementary mechanisms.

---

### 27. Misconception #3 — Traffic Shifts Instantaneous

Incorrect.

TTL + caching delay transitions.

---

### 28. Misconception #4 — Weighted Routing Requires Complex Infrastructure

Incorrect.

Works with:

Any resolvable endpoints.

---

### 29. Misconception #5 — A/B Testing via DNS Always Accurate

Incorrect.

User stickiness not guaranteed.

---

## PRACTICAL DESIGN INSIGHTS

---

### 30. Weighted Routing + Health Checks = Powerful Combo

Traffic split + Failure awareness.

Enables:

Smart canary deployments.

---

### 31. Weighted Routing + Alias Records

Common AWS pattern:

Split traffic between:

Multiple AWS resources.

---

### 32. TTL Strategy — Critical Tuning Parameter

Low TTL:

• Faster traffic shifts  
• Higher DNS query volume

High TTL:

• Slower shifts  
• Lower query overhead

Trade-off decision.

---

## FINAL MENTAL MODEL

---

### 33. What Weighted Routing Truly Represents

Weighted routing is:

**Probabilistic DNS-based traffic distribution**

Used for:

• Traffic splitting  
• Deployment safety  
• Experimentation  
• Infrastructure migration  
• Risk reduction

---

### 34. Why Weighted Routing Is So Powerful

Because traffic control occurs at:

Discovery layer.

Before application logic.

Before connection establishment.

---

### 35. Elegant Big Picture

When resolving:

    example.com

DNS may dynamically decide:

Which infrastructure endpoint user receives.

Traffic patterns shaped by:

Weights + TTL + Resolver Behavior.

Simple mechanism.

Massive architectural impact.

