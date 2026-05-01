# Gate 1 Defense — 3-Minute Walkthrough (Bullet Points)
## First Notice of Loss - FNOL Claims Processing Agent | Nadia Rahmatulla

Our client is a mid-size Insurance company exploring the option of building an AI Agent solution to process their 'First Notice of Loss claims' 
 The client processes 300 claims a day with 12 specialists, 22 minutes per claim. Issues are they're hitting 18% routing errors where a claim goes to the wrong adjustor, and 31% SLA breaches. 
 
Claims can be received by email, phone transcript, web form and must be: triaged by severity, validated against policy coverage, 
routed to the appropriate adjuster, and acknowledged to the claimant — all within 2 hours of receipt

The client is open to full automation where appropriate but insist on human oversight for high-value or ambiguous claims. 

They have a modern CRM (client data system) with APIs, a legacy policy administration system with SOAP endpoints, and a document management system. They have no AI infrastructure today.

## Flow: Following the Spec Structure

### **Section 1: Assumptions & Unknowns (45 seconds)**

#### Why This Section Comes First
- Hidden assumptions are the primary failure mode at this stage
- Every non-trivial claim in the spec depends on at least one assumption
- Marking them here means reviewer sees exactly where confidence is load-bearing


#### Three LOW-Confidence Assumptions (Load-Bearing)

**A.3 — SOAP (Policy Admin) responds in ≤ 5 seconds** - we don't know how quickly that responds & so could hugely affect SLA's
- If actually 30s typical → latency budget collapses, SLA at risk
- **Confidence: LOW** — "legacy SOAP" is a red flag, no performance data provided
- **Test:** 50 staging calls before committing to 2-retry pattern
- **Why it matters:** Gates the validation module design

**A.4 — "High-value" means £50,000**
- If client means £10k → entire delegation boundary shifts
- **Confidence: LOW** — placeholder, not client-confirmed
- **Test:** Client threshold workshop, written sign-off in sprint 0
- **Why it matters:** Load-bearing for severity classifier and delegation design

**A.8 — LLM API access within build timeline**
- Insurance = strict PII + data residency restrictions
- **Confidence: LOW** — could take 3+ months
- **Test:** Sprint 0 blocker — written provider approval required
- **If this fails:** Re-scope extraction to regex/rules, 15% escalation rate instead of 5%

#### Eight Critical Unknowns (Named with Resolution Path)
- **U.1:** "High-value" threshold → Sprint 0 blocker, client workshop
- **U.2:** CRM (centralize client data) API schema → Sprint 1, API docs + sandbox
- **U.3:** SOAP latency profile → Sprint 1, WSDL + 90-day uptime logs
- **U.7:** FCA automated decline rules → Sprint 0 blocker, legal review


**Principle:** Honest uncertainty beats plausible-sounding guess. If I don't know, I say so, why it matters, and how I'll resolve it.

---

### **Section 2: Problem Statement & Success Metrics (30 seconds)**

#### Current State — What's Broken
- **300 FNOLs/day, 12 specialists, 22 min/claim** = 110 person-hours/day, zero slack
- **18% routing errors** = 54 claims/day to wrong adjuster → rework
- **31% SLA breaches** = 93 claimants/day not acknowledged within 2 hours
- **Root cause:** Specialists doing structured, repeatable work manually from unstructured inputs under time pressure

#### Why This Matters — Claimant Perspective
- FNOL is the highest-stakes moment in the relationship
- Distressed claimant expects rapid acknowledgment
- Failing here breaks trust at the worst possible time
- Not an efficiency problem — it's a **trust problem**

#### Success Metrics (Measurable Against Baseline)
- **Routing error:** 18% → ≤ 5% (adjuster re-assignments ÷ total claims)
- **SLA compliance:** 69% → ≥ 90% (routed_at − received_at ≤ 120 min)
- **Ack time:** Unknown baseline → ≤ 15 min for LOW/MEDIUM
- **Business case threshold:** System pays for itself if routing < 8% AND ack SLA ≥ 90% within 60 days

---

### **Section 3: Delegation Analysis (60 seconds)**

#### Delegation Framework — Three Criteria
A task is **fully agentic** when:
1. Decision logic can be fully coded
2. Errors are detectable before downstream harm
3. No regulatory requirement for a human to authorise

#### The Boundary — LOW/MEDIUM vs. HIGH/CRITICAL

**✅ Fully Agentic (70-85% of claims):**
- **Ingest & normalise** → no judgment, text → structured record
- **Extract entities** → LLM + confidence gating, below threshold = escalate loudly
- **Validate policy** → deterministic rules, SOAP timeout = escalate
- **Triage LOW/MEDIUM** → Rules 1-7 deterministic, Rule 8 defaults MEDIUM (conservative)
- **Route LOW/MEDIUM** → incident type → adjuster pool → lowest open_cases
- **Acknowledge LOW/MEDIUM** → template + field substitution, ≤ 15 min

**🟡 Agent-Led, Human Oversight (15-30% of claims):**
- **Triage HIGH/CRITICAL** → agent classifies, specialist reviews
- **Route HIGH/CRITICAL** → agent prepares dossier, specialist routes + approves ack
- **Decline notices** → agent drafts, specialist approves before send
- **System failures** → agent retries 2x, specialist resolves

#### Why This Boundary Is Defensible

**LOW/MEDIUM meets all three criteria:**
- Routing logic = lookup + load-balance (codifiable)
- Wrong route = adjuster re-assigns immediately (detectable)
- No legal accountability requirement for routine intake

**HIGH/CRITICAL fails criterion 3:**
- Client explicitly requires specialist oversight for high-value claims
- Even if agent *could* route accurately, accountability requirement keeps task human
- Not a technical limitation — it's a business and legal boundary

**Test of delegation thinking:** I can explain why every task is at its level with named criteria, not "feels right"

#### Rule 8 Design Decision
- Default is **MEDIUM, not LOW**
- **Why:** LOW = auto-route without review
- Wrong LOW classification on HIGH claim = unreviewed high-stakes claim
- **Conservative choice:** False positive on escalation preferred over false negative

---

### **Section 4: Agent Specification (60 seconds)**

#### State Machine — Executable Contract
- Claim in exactly one state at all times
- All transitions logged: timestamp, actor (AGENT or specialist_id), reason
- **Max time in state defined:** EXTRACTING > 90s → watchdog → SPECIALIST_QUEUE
- **Zero silent hangs**

#### Module Precision — M4 Severity Classifier Example

**8 rules applied in priority order, first match wins:**
1. Bodily injury mentioned → CRITICAL
2. Incident type = LIABILITY → HIGH
3. Damage ≥ £50k → HIGH (threshold — client to confirm)
4. Third party + damage ≥ £10k → HIGH
5. VEHICLE + damage ≥ £5k → MEDIUM
6. PROPERTY + damage ≥ £10k → MEDIUM
7. Damage < £5k, no third party, no injury → LOW
8. No match → **MEDIUM** (conservative default)

**Why Rule 8 = MEDIUM:**
- If damage value absent or no rule matches
- LOW = auto-route without review = highest risk error
- Conservative when uncertain

**LLM fallback (Rule 8 only):**
- If LLM returns HIGH/CRITICAL → treat as HIGH → queue
- If LLM returns LOW/MEDIUM → keep MEDIUM
- **Cannot downgrade without specialist confirmation**

#### Agent Rules — Never Violate
1. Never process without writing raw to DMS first
2. Never auto-route HIGH/CRITICAL
3. Never send decline without specialist approval
4. Never retry SOAP > 2 times
5. claim_id set at ingestion, immutable

#### Integration Contracts — All Unknowns Named
- 5 external systems, 5 scope-outs required (U.2, U.3, U.4, U.5)
- **SOAP:** Unknown WSDL, latency profile → Sprint 1 blocker
- **CRM:** Unknown API schema → Sprint 1 blocker
- **DMS:** Unknown rate limits → Sprint 1 blocker
- **Notification:** Unknown provider → Sprint 0 blocker
- **Every unknown has resolution path and sprint assignment**

**Test of precision:** Could an AI coding agent start building without clarifying questions?

---

### **Section 5: Validation Design (45 seconds)**

#### "Working" = 5 Testable Conditions (5-day rolling window, ≥ 500 claims)
1. Routing error ≤ 5% (re-assignments ÷ total)
2. SLA compliance ≥ 90% (routed_at − received_at ≤ 120 min)
3. Zero silent failures (complete audit trail)
4. Zero unacknowledged routed claims (every ROUTED has notification)
5. Escalation rate 15-35% (outside band → investigate)

#### Quiet Failure Tests (Most Dangerous)

> **Quiet failure:** Agent processes, no error raised, output is wrong, no one notices

| Failure | Test |
|---|---|
| Wrong policy # extracted with high confidence | Weekly 5% random audit. Target ≥ 97% accuracy |
| Rule 8 defaults MEDIUM but HIGH in prose | 20 synthetic high-value narrative claims. Verify LLM flags HIGH |
| Duplicate processed as new | 10 reformulated duplicate pairs. Target ≥ 8/10 flagged |
| Claim stuck in EXTRACTING (LLM hang) | Watchdog test: mock hang, verify moves to queue after 90s |

#### Failure Modes Tied to Spec Decisions
- **£50k threshold too high?** → HIGH claims slip through as MEDIUM
- **SOAP 2x retry during outage?** → 30s/claim × 300 = queue backup
- **Monitor:** If specialist "should be LOW" override rate > 20% on MEDIUM → recalibrate

#### 30-Day Pilot Plan
- **Week 1-2:** Shadow mode, 60 claims/day. Pass: zero state errors, extraction ≥ 95%
- **Week 3:** LOW live only. Pass: routing error ≤ 5%, ack ≤ 15 min
- **Week 4:** LOW + MEDIUM live. Pass: combined routing error ≤ 5%, SLA breach ≤ 12%
- **Rollback trigger:** If routing error > 10% in any 48hr window → auto-move all to queue

---

## Close (15 seconds)

### Why This Spec Is Defensible

**Four things:**
1. **Delegation boundaries justified** — I can explain why every task is at its level with named criteria
2. **Assumptions honest** — 3 LOW confidence, 8 sprint blockers, no filler
3. **Spec build-ready** — State machine executable, modules unambiguous, AI could build without clarification
4. **Validation testable** — 5 numeric conditions, quiet failures named and tested

**What I'm here to defend:** If challenged on a boundary, I have reasoning. If challenged on assumption, I have test. If challenged on precision, I have contracts.

---

## Delivery Notes

### Pacing (Total: 3 minutes 45 seconds)
- **0:00-0:45** — Section 1: Assumptions
- **0:45-1:15** — Section 2: Problem
- **1:15-2:15** — Section 3: Delegation
- **2:15-3:15** — Section 4: Specification
- **3:15-3:45** — Section 5: Validation + Close

### Tone
- **Confident, not defensive** — presenting reasoning, not justifying
- **Concrete, not abstract** — "Rule 8 defaults MEDIUM" not "we have fallback logic"
- **Own the unknowns** — "Confidence is LOW" is strength, not weakness

### Physical Delivery
- **No slides** — walking through document they have
- **Reference sections by number** — "Section 1 comes first for a reason..."
- **Point to specific examples** — "M4, Rule 8, defaults MEDIUM..."
- **Pause after load-bearing statements** — "Confidence is LOW." [beat] "The test is..."

### What to Have
- ✅ Full spec (open, sections bookmarked)
- ✅ One notecard with timing checkpoints
- ✅ Water

---

## Defending Under Pressure — Response Scripts

### Challenge: "Why is £50k in the spec if you don't know it?"
**Response:**
"That's why it's in the assumptions section marked LOW confidence. Hiding it would be worse. The spec shows where confidence is load-bearing. U.1 is marked sprint 0 blocker — resolution is client threshold workshop with written sign-off before any code. Honest uncertainty beats plausible guess."

### Challenge: "What if SOAP is down 2 hours during business hours?"
**Response:**
"Addressed in A.3, confidence LOW, and U.8. Test is request 90-day uptime logs in sprint 1. If SOAP is unreliable, options are policy data cache layer or accept validation failure spike during outages. I've named the dependency; I haven't hidden it. This is why A.3 is LOW confidence."

### Challenge: "Why not auto-route HIGH claims?"
**Response:**
"Which of my three criteria doesn't HIGH claims meet? The logic is codifiable — I could write the rules. Errors are detectable — wrong adjuster re-assigns. But criterion 3: client explicitly requires specialist oversight for high-value claims. That's a legal and accountability requirement regardless of technical capability. If you're saying HIGH should be agentic, I need evidence the accountability requirement doesn't apply."

### Challenge: "How prevent routing HIGH as MEDIUM?"
**Response:**
"Rules 1-3 are deterministic — bodily injury, liability, £50k. No agent discretion. Rule 8 defaults MEDIUM when uncertain, not LOW. LLM fallback on Rule 8 can escalate from MEDIUM to HIGH, but cannot downgrade without specialist sign-off. Severity escalation is one-way: agent can raise, cannot lower. Conservative by design."

### Challenge: "Spec is too detailed / not detailed enough"
**Response:**
"Show me where. If module M4 could be interpreted two ways, that's my bug and I'll fix it. But 'severity classifier has 8 rules in priority order, first match wins, default MEDIUM' — where's the ambiguity? The test of precision is: could an AI coding agent build from this without clarifying questions? If you see an ambiguity, I want to fix it."

### Challenge: "Which failure mode are you missing?"
**Response:**
"Walk me through it. The five conditions cover routing accuracy, SLA, silent failures, acknowledgment, and escalation rate. Quiet failure tests cover wrong extraction, wrong classification, and system hangs. If there's a sixth failure mode with material impact, I want to know it and add it to 5.3."

### Challenge: "You're wrong about X"
**Response:**
"Walk me through your reasoning. If I've misunderstood a constraint, I'll update the spec cleanly right now. But if it's a judgment call — like where to draw the delegation line — I need to understand what criteria you're using that I'm not."

---

## Updating Cleanly Under Pressure

### If You Realize an Error Mid-Defense

**Don't say:**
- "Oh yeah, I guess that doesn't work..."
- "Hmm, I didn't think of that..."
- "Maybe I should change..."

**Do say:**
- "That's a valid catch. [State the fix]. I'm updating it to [new decision] because [reasoning]. This doesn't cascade to [X], but it does affect [Y], which means [update]."

**Example:**
"That's a valid catch. A.4 should be sprint 0, not sprint 1, because the £50k threshold gates the entire severity classifier. Without client sign-off, I can't finalize Rules 3 and 4. I'm updating it to a sprint 0 blocker now. This also means U.1 moves from sprint 1 to sprint 0."

**Show the update:**
- Note it on spec in front of you
- State the change clearly
- Name any cascade effects
- Explain why it doesn't break other parts (or why it does and how you'll handle it)

---

## The Mindset

You're not defending a finished product. You're defending a **way of thinking.**

### Four Core Questions (Self-Check Before Walking In)

1. ✅ **Can I defend every delegation boundary with named criteria?**
   - Not "feels right" but "codifiable logic + detectable errors + no accountability requirement"

2. ✅ **Can I name three LOW-confidence assumptions and explain why they're load-bearing?**
   - A.3 (SOAP latency), A.4 (£50k threshold), A.8 (LLM procurement)
   - Each has hypothesis, test, and sprint gate

3. ✅ **Can I point to a module contract and explain why it's unambiguous?**
   - M4: 8 rules, priority order, first match wins, default MEDIUM
   - LLM prompt is pinned, not "extract relevant info"

4. ✅ **Can I name a quiet failure and the test that catches it?**
   - Wrong policy # with high confidence → weekly 5% audit, ≥ 97% accuracy
   - HIGH in prose but MEDIUM by rule → 20 synthetic narratives

**If yes to all four: you're ready.**

---

## What Success Looks Like in This Defense

### **Good:**
- "That task is agentic because decision logic is a lookup against the policy table, error is visible immediately, and there's no legal accountability requirement"
- "Confidence is LOW because I have no SOAP latency data. Test is 50 staging calls. If p95 > 10s, I need a cache layer"
- "The extractor runs LLM with JSON schema. If confidence < 0.85, re-prompt once. Still below? EXTRACTION_FAILED, escalate with dossier"

### **Bad:**
- "I think that task should be agentic" [no criteria]
- "I'm pretty sure SOAP will be fast enough" [confidence hidden]
- "The agent will extract the policy number" [no failure case specified]

---

## Final Check

### Before You Walk In, Can You Answer:

1. **"Why is the delegation line there, not somewhere else?"**
   → Three criteria, client requirement for HIGH/CRITICAL oversight

2. **"What are you least confident about?"**
   → A.3 (SOAP latency), A.4 (threshold), A.8 (LLM procurement) — all marked LOW, all have tests

3. **"Could an AI build this without asking questions?"**
   → State machine is executable, modules have I/O contracts, agent rules are non-negotiable

4. **"How do you know it's working?"**
   → 5 numeric conditions, quiet failure tests named, pilot has pass conditions + auto-rollback

**If you can answer all four with specifics from the spec: you're ready to defend.**

---

*This is the test of your way of thinking. The document demonstrates it. This walkthrough defends it.*
