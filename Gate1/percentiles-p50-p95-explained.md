# Percentiles (p50, p95, p99) — Explained

## The Core Idea

When you measure **latency** (how long something takes), you don't get one number. You get a **distribution** of measurements.

**Percentile** = "At what number are X% of measurements below this value?"

---

## Simple Example: SOAP Response Times

Imagine you make 100 calls to your SOAP API and measure how long each takes:

```
Call 1:   2ms
Call 2:   3ms
Call 3:   5ms
Call 4:   4ms
Call 5:   180ms  ← slow one
Call 6:   3ms
...
Call 100: 4ms
```

Now sort all 100 measurements from fastest to slowest and look at different positions:

---

## The Percentiles

| Percentile | What It Means | Example |
|---|---|---|
| **p50** | 50% of calls are faster, 50% are slower | The **median** response time |
| **p95** | 95% of calls are faster, 5% are slower | The tail — worst 1 in 20 |
| **p99** | 99% of calls are faster, 1% are slower | The extreme tail — worst 1 in 100 |

### Visualized

```
0ms ──────────────────────── 200ms
└─ p50 (median)
         └─ p95 (the long tail starts)
                  └─ p99 (the worst outliers)

Example sorted measurements (100 calls):
Position 1:   2ms    ← p1
Position 50:  4ms    ← p50 (median)
Position 95:  15ms   ← p95
Position 99:  180ms  ← p99
Position 100: 500ms  ← max (one terrible outlier)
```

---

## Why Percentiles Matter (Not Just Average)

### The Average Can Lie

Imagine these 10 SOAP calls:

```
Call 1:  5ms
Call 2:  5ms
Call 3:  5ms
Call 4:  5ms
Call 5:  5ms
Call 6:  5ms
Call 7:  5ms
Call 8:  5ms
Call 9:  5ms
Call 10: 500ms  ← one database timeout
```

**Average = 55ms**

But 9 calls out of 10 are 5ms. That average is useless.

### With Percentiles You See the Truth

```
p50 (median):  5ms
p95:           5ms
p99:           500ms
max:           500ms
```

Now you can see: **95% of calls are fast (5ms), but 1% are terrible (500ms).**

---

## In Your Spec (A.3)

### Your Assumption
> "SOAP responds in ≤ 5 seconds"

### What That Really Means (With Percentiles)

You're assuming:
- **p95 latency ≤ 5 seconds**

Why p95? Because:
- You need most calls to be fast (95% fast = good system)
- You can tolerate some slow calls (5% slow = network hiccups, temporary load)
- But 5% slow at 5 seconds is acceptable

### What You're Worried About

If SOAP is actually:
```
p95:  30 seconds
p99:  60 seconds
```

Then:
- 1 in 20 calls hangs for 30 seconds
- Every batch of 300 claims has ~15 calls that wait 30 seconds
- **15 claims × 30s = 450 seconds = 7.5 minutes added latency per batch**
- **SLA is 120 minutes per claim, but latency alone is eating 7.5 minutes on just SOAP delays**

If SOAP also times out (retries), that could be 60-90 seconds per failed call. You lose your SLA entirely.

---

## How to Test This (Sprint 1)

### The Test from Your Spec

> **Test:** 50 staging calls before committing to 2-retry pattern

Here's what that means:

1. **Make 50 calls** to the SOAP API in staging environment
2. **Record every response time** (milliseconds)
3. **Sort the 50 measurements**
4. **Calculate:**
   - p50 (position 25): Is it ≤ 1 second? Good.
   - p95 (position ~47-48): Is it ≤ 5 seconds? Good.
   - p99 (position ~49-50): Is it ≤ 10 seconds? Tolerable.

### If Results Show

| Scenario | Meaning | Decision |
|---|---|---|
| p95 = 800ms | SOAP is fast | ✅ Assumption confirmed. Build as spec. |
| p95 = 5 seconds | SOAP is on edge | 🟡 Acceptable but risky. Add cache layer. |
| p95 = 30 seconds | SOAP is slow | 🔴 Assumption false. Re-design validation. |

---

## Real-World SOAP Example

Let's say you run 50 test calls and get these times:

```
Call 1:    412ms
Call 2:    385ms
Call 3:    410ms
Call 4:    390ms
Call 5:    415ms
...
Call 45:   2100ms  ← first slow outlier
Call 46:   400ms
Call 47:   420ms
Call 48:   2500ms  ← second slow outlier
Call 49:   410ms
Call 50:   2800ms  ← third slow outlier
```

**Results:**
- **p50 (median, position 25):** ~410ms ✅
- **p95 (position 47-48):** ~2,500ms 🟡 (ouch, beyond your 5s assumption, but only 2-3 outliers)
- **p99 (position 49-50):** ~2,800ms 🔴 (your assumption is WRONG)

**What you'd do:**
"My assumption of ≤5 seconds is mostly right, but worst 5% are hitting 2.5+ seconds. If I have 300 claims/day with SOAP calls, that's 15 claims hitting 2.5s delays. Not fatal, but I need to add a timeout + cache layer or accept slightly higher SLA breach rate."

---

## Why You Say "p95" Not Just "Average"

### In Your Defense

**Don't say:** "SOAP responds in ≤ 5 seconds on average"

**Do say:** "I'm assuming SOAP p95 latency is ≤ 5 seconds. That means 95% of calls are fast. Confidence is LOW because I have no data. Test is 50 staging calls. If p95 exceeds 10 seconds, the latency budget fails and I need a cache layer."

### Why the Precision Matters

Saying "on average" hides tail risk. Saying "p95" shows you understand:
- Most calls will be fast
- But some will be slow
- You've planned for that

---

## Common Percentiles You'll See

| Percentile | Common Use | Example |
|---|---|---|
| **p50** | Median response time | "Half my users experience this speed" |
| **p75** | Three-quarters fast | Start seeing complaints here |
| **p90** | 90% good, 10% bad | SLA target zone |
| **p95** | 95% good, 5% bad | What you're assuming in A.3 |
| **p99** | 99% good, 1% really bad | Worst outliers |
| **p99.9** | 99.9% good, 0.1% terrible | AWS/Google use this for uptime |

---

## In Your 3-Minute Defense

### If Coach Asks: "What do you mean by 'p95 latency'?"

**Good answer:**
"p95 means the 95th percentile — the point where 95% of calls are faster and 5% are slower. I'm not assuming average or best-case. I'm saying: 95 out of 100 SOAP calls will complete in ≤5 seconds. The worst 5% might be slower, which I handle with retries. Confidence is LOW because 'legacy SOAP' could have variable performance. Test is 50 staging calls to measure the actual distribution."

**What you're demonstrating:**
- You understand tail risk (not hiding it with averages)
- You're planning for real-world variability
- You've defined a measurable test

---

## Why This Matters for Your Spec

### The SLA Math

Your claim SLA is **2 hours = 120 minutes**.

If you have:
- M1 (normalise): 200ms
- M2 (extract): 15 seconds (LLM call)
- **M3 (validate via SOAP): p95 = ?**
- M4 (classify): 100ms
- M5 (route): 100ms
- M6 (ack): 500ms

**Total latency budget: 120 minutes minus overhead**

If SOAP p95 is 5 seconds:
- Most claims routed in ~20 seconds ✅

If SOAP p95 is 30 seconds:
- Claims start hitting SLA limits 🔴

That's why A.3 is marked **LOW confidence** and is a **sprint 1 blocker**.

---

## Summary

**p95 = the 95th percentile = 95% of measurements are below this number**

In your spec:
- **Assumption:** SOAP p95 latency ≤ 5 seconds
- **Why LOW confidence:** You have no data on legacy system
- **Why it matters:** If wrong, SLA is unreachable
- **Test:** 50 staging calls, measure distribution
- **Decision point:** If p95 > 10 seconds, add cache layer or re-design

This shows you understand:
1. Tail risk (not hiding it with averages)
2. Real-world variability (not assuming best-case)
3. Measurable validation (not vague testing)
