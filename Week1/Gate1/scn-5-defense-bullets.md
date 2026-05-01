# Gate 1 Defense — FNOL Claims Processing Agent
## Nadia Rahmatulla | 10-Minute Defense

---

## 1. THE PROBLEM (90 seconds)

### Current State
- **300 FNOL claims/day** through email, phone, web
- **12 specialists** spending **22 min/claim** on manual extraction, lookup, routing
- **110 person-hours/day consumed** — zero capacity slack
- **18% routing error rate** (54 claims/day to wrong adjuster)
- **31% SLA breach rate** (93 claimants/day not acknowledged in 2 hours)

### Why This Matters
- **Business:** Zero headroom for volume spikes; routing errors cause rework; SLA breaches risk regulatory attention
- **Claimant:** FNOL is the highest-stakes moment in the relationship — distressed customer, promise of support. Failing here breaks trust at the worst possible time
- **Root cause:** Specialists doing structured, repeatable work manually from unstructured inputs under time pressure. This is a systems problem, not a people problem.

---

## 2. THE SOLUTION BOUNDARIES (2 minutes)

### What the Agent Does (Fully Agentic)
- **Ingest & normalise** unstructured text from all 3 channels
- **Extract** structured fields with LLM + confidence scoring (policy #, claimant, incident details)
- **Validate** policy coverage via SOAP call to legacy system
- **Triage** severity using 7 deterministic rules + LLM fallback
- **Route** LOW/MEDIUM claims to best-match adjuster (specialisation + load balance)
- **Acknowledge** claimant within 15 minutes via template-driven email/SMS

### What Requires Human Oversight
- **HIGH/CRITICAL severity** — bodily injury, liability, damage ≥ £50k → specialist reviews agent dossier, routes, approves ack
- **Coverage declines** — agent drafts notice, specialist approves before send
- **Low-confidence extraction** — any required field below threshold → specialist queue
- **System failures** — SOAP timeout, CRM down → escalate with full context, don't guess

### The Delegation Principle
> **A task is fully agentic when:** decision logic is codifiable, errors are detectable before harm, no regulatory requirement for named human accountability.
> **Why this boundary is defensible:** LOW/MEDIUM claims have deterministic routing rules; HIGH/CRITICAL carry legal/financial stakes that require professional accountability regardless of technical capability.

---

## 3. HONEST ASSUMPTIONS & UNKNOWNS (90 seconds)

### Critical Assumptions (If Wrong, System Breaks)
1. **A.4 — "High-value" threshold = £50k** *(placeholder — client confirmation required)*
   - If client means £10k → entire delegation boundary shifts
   - **Test:** Present threshold table in sprint 0, get written sign-off

2. **A.3 — SOAP responds in ≤ 5 seconds at p95**
   - If it takes 30s regularly → latency budget collapses, SLA at risk
   - **Confidence: LOW** — "legacy SOAP system" is a red flag. No perf data provided.
   - **Test:** 50 test calls in staging, measure p95/p99 before committing to 2-retry pattern

3. **A.8 — LLM API access within build timeline**
   - Insurance = strict data residency + PII restrictions
   - If 3-month procurement delay → re-scope extraction to regex/rules for sprint 1
   - **Test:** Sprint 0 blocker — written confirmation of provider, data residency, PII approval

### Genuine Unknowns (Must Resolve Before Build)
- **U.1:** What does "high-value" mean to this client numerically? *(loads severity rules)*
- **U.2:** CRM API: base URL, auth, adjuster schema? *(blocks routing module)*
- **U.3:** SOAP WSDL, auth, response schema, latency? *(blocks validation module)*
- **U.7:** FCA constraints on automated declines? *(may force redesign of NOT_COVERED flow)*

**No filler.** Every unknown listed has a blocker attached and a resolution path named.

---

## 4. SPEC PRECISION — COULD AN AI BUILD FROM THIS? (2 minutes)

### State Machine (Executable Contract)
- Claim is in **exactly one state** at all times: `RECEIVED → EXTRACTING → VALIDATING → TRIAGING → AUTO_ROUTING → ROUTED`
- Every transition logged: timestamp, actor (AGENT or specialist_id), reason
- **Max time in state defined** — watchdog moves to `SPECIALIST_QUEUE` on timeout, not silent hang
- **Example:** If claim in `EXTRACTING` > 90 seconds → auto-escalate, specialist sees "LLM call hung or low-confidence field"

### Module Contracts Are Unambiguous
**M2 — Extractor:**
- 9 required fields, each with confidence threshold (0.75–0.90)
- Re-prompt logic: if field below threshold, run structured clarification once
- Still below? → `EXTRACTION_FAILED` → specialist queue with full dossier
- **LLM prompt is pinned in spec** — not "extract relevant info", but exact instruction with JSON schema

**M4 — Severity Classifier:**
- 8 rules applied in priority order; first match wins
- Rule 8 default is **MEDIUM** (not LOW) — conservative when uncertain
- **Rationale:** LOW = auto-route without review. Wrong LOW classification on HIGH claim = unreviewed high-stakes claim. False positive on escalation preferred.

**M5 — Routing Engine:**
- Filter adjusters by specialisation + AVAILABLE status
- Select: lowest `open_cases`; tie-break on earliest `last_assigned_at`
- If filtered list empty → `SPECIALIST_QUEUE` (cannot auto-route)
- **No "best judgment" steps** — every decision is a lookup or comparison

### Agent Rules (Never Violate)
1. Never process without writing raw to DMS first
2. Never auto-route HIGH/CRITICAL
3. Never send decline without specialist approval
4. Never retry SOAP > 2 times
5. `claim_id` set at ingestion, immutable

**Test of precision:** Closed build loop with Claude Code. Every unintended builder output classified: spec ambiguity (I fix), builder misread (builder fixes), or unjustified addition (rollback).

---

## 5. VALIDATION — HOW DO WE KNOW IT'S WORKING? (2 minutes)

### "Working" = 5 Testable Conditions (5-day rolling window, ≥ 500 claims)
| # | Condition | Measurement |
|---|---|---|
| 1 | Routing error ≤ 5% | Adjuster re-assignment ÷ total claims |
| 2 | SLA compliance ≥ 90% | `routed_at` − `received_at` ≤ 120 min |
| 3 | Zero silent failures | Every terminal-state claim has complete audit trail |
| 4 | Zero unacknowledged routed claims | Every `ROUTED` has notification record |
| 5 | Escalation rate 15–35% | Outside band → investigation |

### Quiet Failure Tests (Most Dangerous)
> **Quiet failure:** Agent processes claim, no error raised, output is wrong, no one notices.

| Failure Mode | Detection Test |
|---|---|
| Wrong policy # extracted with high confidence | Weekly 5% random audit by specialist. Target: ≥ 97% accuracy |
| Rule 8 defaults MEDIUM but HIGH damage in prose | 20 synthetic high-value narrative claims. Verify: LLM flags HIGH → queue |
| Duplicate processed as new | 10 reformulated duplicate pairs. Target: ≥ 8/10 flagged |
| Claim stuck in EXTRACTING (LLM hang) | Watchdog test: mock LLM hang. Verify: moves to queue after 90s |

### Failure Modes Tied to Spec Decisions
- **£50k threshold too high?** → HIGH claims slip through as MEDIUM
- **SOAP 2x retry during outage?** → 30s delay/claim × 300 = queue backup
- **Monitor:** If specialist "should be LOW" override rate > 20% on MEDIUM → recalibrate

### 30-Day Pilot Plan
- **Week 1–2:** Shadow mode, 60 claims/day. Pass: zero state machine errors, extraction ≥ 95%
- **Week 3:** LOW live only. Pass: routing error ≤ 5%, ack ≤ 15 min
- **Week 4:** LOW + MEDIUM live. Pass: combined routing error ≤ 5%, SLA breach ≤ 12%
- **Rollback trigger:** If routing error > 10% in any 48-hour window → auto-move all claims to specialist queue

---

## 6. WHY THIS SPEC IS DEFENSIBLE (90 seconds)

### 1. Delegation Boundaries Are Justified, Not Arbitrary
- LOW/MEDIUM: deterministic rules + detectable errors + no legal accountability requirement = fully agentic
- HIGH/CRITICAL: financial/legal stakes + client-stated requirement = human oversight
- **I can explain why every task is at its delegation level** — not "feels right", but named criteria

### 2. Assumptions Are Honest
- 10 assumptions, each with hypothesis + test + confidence level
- 3 marked LOW confidence (SOAP latency, £50k threshold, LLM procurement) — these are load-bearing
- **8 critical unknowns** with blocker status and resolution path
- No "we'll figure it out later" — every unknown is named and scheduled

### 3. Spec Is Build-Ready
- State machine is executable
- Module I/O schemas are complete
- Integration contracts list what's known and **what's unknown** (5 systems, 5 scope-outs required)
- Agent rules are non-negotiable constraints
- **If Claude Code misreads this spec, that's a spec bug I own**

### 4. Validation Is Testable
- "Working" has 5 numeric conditions
- Quiet failure tests named (the ones that don't throw errors)
- Pilot has pass conditions and auto-rollback trigger
- **Not "we'll write tests" — these ARE the tests**

---

## EXPECTED COACH CHALLENGES

### "Why is the £50k threshold in the spec if you don't know it?"
- Because hiding it is worse. The spec shows where confidence is load-bearing.
- U.1 is marked as sprint 0 blocker — resolution path is client threshold workshop.
- **Honest uncertainty beats plausible-sounding guess.**

### "What if SOAP is down for 2 hours during business hours?"
- Addressed in assumptions (A.3, confidence LOW) and unknowns (U.8).
- Test: request 90-day uptime logs before sprint 1.
- If SOAP is unreliable → introduce policy data cache layer or accept that validation failures spike during outages.
- **I've named the dependency; I haven't hidden it.**

### "How do you prevent the agent from routing HIGH claims as MEDIUM?"
- Rules 1–3 (bodily injury, liability, £50k) are deterministic — no agent discretion.
- Rule 8 defaults MEDIUM when uncertain, not LOW — conservative choice.
- LLM fallback cannot downgrade MEDIUM to LOW without specialist confirmation.
- **Severity escalation is one-way: agent can raise, cannot lower without human sign-off.**

### "What's the single biggest risk to this system?"
- **A.8 — LLM procurement delay.** If PII or data residency blocks third-party LLM API for 3+ months, extraction module must be re-scoped to regex/rules for sprint 1. That changes the accuracy assumption and likely increases EXTRACTION_FAILED escalation rate from ~5% to ~15%.
- **Mitigation:** Sprint 0 blocker. No code written until LLM provider is confirmed in writing.

---

## CLOSE (30 seconds)

**This spec is ready for critique because:**
1. I can defend every delegation boundary with named criteria
2. Assumptions and unknowns are honest — 3 are marked low confidence, 8 are sprint blockers
3. The spec is precise enough that an AI coding agent could start building without clarifying questions
4. Validation is testable — "working" has 5 numeric conditions, quiet failures are named

**What I'm asking for feedback on:**
- Are the severity thresholds the right conversation to have with the client in sprint 0?
- Is SOAP latency unknown (A.3, U.8) a sprint 0 gate or acceptable sprint 1 risk?
- Have I missed a quiet failure mode?

---

*Prepared for Gate 1 Critique Session*
