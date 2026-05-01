# Routing Errors — Explained

## Definition

**A routing error = when a claim is sent to the wrong adjuster.**

---

## In Your FNOL Context

### What Happens in Correct Routing
1. Agent receives FNOL claim (email/phone/web)
2. Agent extracts claim details (incident type, damage amount, etc.)
3. Agent applies routing rules:
   - **Filter by specialisation** — Does adjuster handle this incident type?
   - **Filter by availability** — Is adjuster AVAILABLE status?
   - **Select by load** — Who has fewest open_cases?
4. Agent routes claim to selected adjuster ✅ **CORRECT ROUTE**
5. Adjuster processes claim

### What Happens in a Routing Error
Same process, but the selected adjuster is **wrong** because:

| Routing Error Type | Example |
|---|---|
| **Wrong specialisation** | VEHICLE claim routed to PROPERTY specialist |
| **Adjuster overloaded** | Agent picked adjuster with 87 open cases when others had 20 |
| **Status wrong** | Agent routed to adjuster marked UNAVAILABLE (on leave, training) |
| **Threshold mismatch** | LOW severity claim routed to HIGH-value adjuster (wasting specialist capacity) |

---

## Current Baseline: 18% Routing Error Rate

### What This Means
- **300 claims/day × 18% = 54 claims/day** routed to wrong adjuster
- **54 claims/day × 22 min/claim = 1,188 minutes = ~20 specialist-hours/day** wasted on:
  - Detecting the error (specialist opens claim, realizes it's not their specialisation)
  - Re-routing to correct adjuster
  - Context loss and rework

### Why 18% Happens Today
- Specialists manually reading unstructured text under time pressure
- Misinterpreting incident type ("burst pipe" = PROPERTY? PLUMBING?)
- Not checking adjuster availability status
- Not checking current workload
- No systematic routing logic—just best guess

---

## Why Routing Errors Are the #1 Problem You're Solving

### The Damage Cascade
1. **Claimant:** Wrong adjuster = delay, confusion, transferred again
2. **First adjuster:** Wasted time on something not their job
3. **Second adjuster:** Gets behind because they received late
4. **SLA impact:** Every re-route adds 15-20 minutes → claimant misses 2-hour ack window
5. **Business:** 20 hrs/day of specialist time on rework instead of actual claim work

---

## How Your Agent Fixes Routing Errors

### M5 — Routing Engine (Fully Agentic for LOW/MEDIUM)

**Process:**
1. **Extract incident type** → e.g., VEHICLE, PROPERTY, LIABILITY
2. **Apply deterministic filter** →
   ```
   available_adjusters = all adjusters where:
     - status = AVAILABLE
     - specialisation contains incident_type
   ```
3. **Load-balance** →
   ```
   selected = adjuster with min(open_cases)
   ```
4. **Route** → claim assigned to selected adjuster
5. **Log** → timestamp, reason, confidence

**Why this reduces errors to ≤ 5%:**
- **No judgment** — rules are lookup tables
- **Deterministic** — same claim always routes same way (unless load changes)
- **Auditable** — if adjuster says "wrong specialisation," we see exactly why agent picked them
- **Correctable** — adjuster re-assigns to correct colleague in seconds

---

## Your Success Metric for Routing Errors

### Baseline → Target
- **Today:** 18% (54 claims/day to wrong adjuster)
- **Target:** ≤ 5% (15 claims/day to wrong adjuster)
- **Measurement:** `adjuster_reassignments ÷ total_claims` over 5-day rolling window

### How You Measure It
Every time an adjuster **manually reassigns** a claim to a colleague:
- **Log it** (CRM records reassignment)
- **Count reassignments** in rolling 5-day window
- **Calculate error rate** = reassignments ÷ (total claims routed by agent)

### What 5% Means
- **15 claims/day** instead of 54
- **~3.3 specialist-hours/day** saved (instead of 20 hrs/day)
- **SLA impact:** Most claims now acked within 2-hour window

---

## Why You Explicitly Name This as a Quiet Failure Test

From your spec, Section 5.3:

> **Quiet failure:** Agent routes claim with high confidence, adjuster discovers it's wrong 30 minutes later, claim is reassigned.

**This is dangerous because:**
- No error thrown by agent
- No alert in system
- Only detected when specialist notices

**Your test:**
- Weekly audit of reassignments
- Plot trend: should stay ≤ 5%
- If trend rises above 8% in any 7-day window → investigate:
  - Are filter rules wrong?
  - Did adjuster specialisation change and CRM not updated?
  - Is load-balancing creating unexpected combinations?

---

## In Your Defense: How to Explain Routing Errors

### If Coach Asks: "What's a routing error?"

**Good answer:**
"A routing error is when the agent assigns a claim to the wrong adjuster. Today that's 18% of claims — 54/day. The adjuster opens it, sees it's not their specialisation or they're overloaded, and has to reassign. That's 20 specialist-hours/day of rework. My agent fixes this through deterministic routing: incident type → filter available adjusters → lowest open_cases → assign. Result: ≤ 5% errors, 3 specialist-hours/day saved."

**What you're demonstrating:**
- You understand the business impact (rework, SLA)
- You understand the root cause (manual judgment under pressure)
- You understand how your agent prevents it (rules-based, no judgment)
- You understand how to measure it (reassignment tracking)

---

## Routing Errors vs. Other Failure Modes

| Failure Mode | What Happens | How Agent Prevents |
|---|---|---|
| **Routing error** | Wrong adjuster selected | Deterministic rules |
| **Wrong extraction** | Policy number misread | LLM confidence gating |
| **Wrong severity** | HIGH claim marked MEDIUM | Rule 8 conservative default |
| **Silent hang** | Claim stuck in EXTRACTING | 90s watchdog timeout |
| **Duplicate** | Same claim processed twice | Duplicate detection on claim_id |

---

## Why This Matters for Your Spec Defense

### The Three Criteria Test

Your delegation boundary says: **Route LOW/MEDIUM is fully agentic**

Does it meet your three criteria?

1. ✅ **Decision logic is codifiable** → Yes, it's a lookup + comparison
2. ✅ **Errors are detectable before harm** → Yes, adjuster immediately sees wrong specialisation and reassigns
3. ✅ **No regulatory accountability** → Yes, routine routing needs no named human sign-off

✅ **Therefore: fully agentic is justified**

---

## Summary

**Routing error** = claim sent to wrong adjuster

- **Current cost:** 54/day, 20 specialist-hours/day rework, SLA breaches
- **Agent solution:** Deterministic rules (specialisation + load-balance) → ≤ 5% errors
- **Measurement:** Weekly reassignment audit
- **Defends your boundary:** Error is detectable immediately (adjuster re-assigns), so it's safe to delegate

This is your #1 success metric because it's where the biggest waste happens today.
