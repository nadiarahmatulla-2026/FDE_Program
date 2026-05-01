# Scenario 5: Patient Intake Agent — 10-Minute Defense Presentation

**Format:** PowerPoint Talking Points (one slide per 1 min 30 sec)

---

## SLIDE 1: The Problem (1:30)
### Title: "180 Intakes/Day, 4 People, Manual Process"

**Talking Points:**
- 6 physicians, 2 locations, ~180 patients per day
- 4-person front-desk team manually managing intake
- **Two critical failures happen regularly:**
  - Expired prior authorizations are missed before visits start
  - Medication changes aren't reviewed by physicians
- **Result:** Visit disruptions, rework, compliance gaps

**Visual:** Simple graphic of overworked front desk; arrow pointing to 2 common failures

**Why it matters:** This is the real problem the agent solves. Not just "process automation," but specific operational pain.

---

## SLIDE 2: The Opportunity (1:30)
### Title: "Automate What's Mechanical, Keep Humans in Control"

**Talking Points:**
- The agent handles **purely administrative** intake tasks:
  - Appointment sync (mechanical)
  - Questionnaire dispatch (administrative)
  - Insurance verification (integration call)
  - Prior-auth checking (rule lookup)
  - Medication mismatch detection (field comparison)
  - Allergy flag retrieval (data pull)
  - Visit-reason routing (category matching)
  
- **Humans decide everything that requires clinical judgment:**
  - Is a medication change significant? → Human clinician
  - Is this allergy important? → Human clinician
  - Is this visit really urgent? → Human clinician

**Visual:** 2-column layout: "Agent Automates" vs "Human Decides"

**Why it matters:** Establishes the hard delegation boundary upfront. No ambiguity about where clinical judgment stays human.

---

## SLIDE 3: The Delegation Boundary (1:30)
### Title: "Three Delegation Modes — Defensible"

**Talking Points:**
- **AGENT_ALONE (7 tasks):** Deterministic, no clinical content
  - Example: Appointment sync every 15 min, check questionnaire completion
  - Safe because: structured data, clear rules, no judgment
  
- **AGENT + HUMAN REVIEW (8 tasks):** Agent detects, human confirms
  - Example: Flag medication changes → clinical staff reviews
  - Safe because: detection is mechanical, significance judgment stays human
  
- **HUMAN_DECIDES (5 tasks):** Agent provides data, human decides
  - Example: Is this urgent? Determine urgency → human only
  - Safe because: clinical judgment cannot be automated

**Visual:** Three boxes showing task examples in each mode

**Why it matters:** Coaches are looking for this. Boundary can't be arbitrary; it must be justified. I'm showing the **principle** behind each boundary.

---

## SLIDE 4: The Spec Is Buildable (1:30)
### Title: "10 Functional Requirements + 4 Entities + 3 Integrations"

**Talking Points:**
- **10 FRs** (Appointment Sync, Questionnaire, Insurance, Prior-Auth, Medications, Allergies, Visit-Reason, Readiness, Escalations, Audit Logs)
- **Each FR has:**
  - Pseudocode / algorithmic logic
  - Acceptance criteria (testable)
  - Error handling (no guesses)
  - Escalation triggers (deterministic)
  
- **Data Model:** 4 entities (Appointment, IntakeWorkItem, EscalationEvent, AuditLog)
  - All enums defined
  - State machines explicit
  - No ambiguity about what data goes where
  
- **Integration Contracts:** athenahealth, Insurance Tool, Prior-Auth Rules
  - Request/response formats shown (JSON examples)
  - Timeout: 10s, Retry: 2x backoff
  - Fallback behavior: fail closed (escalate, don't guess)

**Visual:** Three columns: FRs, Data Model, Integrations

**Why it matters:** An AI coding agent can start building from this immediately. No "what should we do if..." questions needed.

---

## SLIDE 5: Safety Guardrails (1:30)
### Title: "8 Hard Prohibitions — Non-Negotiable"

**Talking Points:**
- The system **must never**:
  1. Diagnose
  2. Perform clinical triage
  3. Recommend treatment
  4. Assess medication significance
  5. Assess allergy severity
  6. Infer prior-auth without explicit rules
  7. Suppress escalations to "appear efficient"
  8. Fabricate data when integration fails
  
- **If any of these tempt you in the build:** Don't implement it. Escalate or remove the feature.

- **Enforcement:** Boundary tests verify no clinical output in escalation contexts

**Visual:** Red warning icon; list of 8 prohibitions

**Why it matters:** This is the non-negotiable clinical safety boundary. Scenario 5 explicitly requires this.

---

## SLIDE 6: Validation — Happy Path (1:30)
### Title: "10 Concrete Test Scenarios"

**Talking Points:**
- **V1 (Happy Path):** Routine visit, complete intake → READY_FOR_VISIT
  - No escalations
  - Expected: All statuses GREEN, 0 escalations
  
- **V2–V3 (Failure Modes):**
  - V2: Incomplete questionnaire → Hold, escalate to front-desk
  - V3: Insurance timeout → Don't guess "active," escalate instead
  
- **V4–V5 (Boundary Tests):** 
  - V4: Urgent-phrase visit reason → Escalate, verify NO clinical output
  - V5: Medication change → Escalate, verify NO significance scoring
  
- **V6–V10 (Edge Cases, Concurrency, Audit, Override):**
  - V8: Sync same appointment twice → Idempotent (1 work item, not 2)
  - V9: All audit logs complete + immutable
  - V10: Human resolves escalation → readiness re-evaluates

**Visual:** Icons: ✅ PASS, ⚠️ ESCALATE, 🛑 BLOCK, 🔒 HOLD

**Why it matters:** Coaches want to see that I've thought about edge cases, not just the happy path. Boundary tests specifically verify clinical safety.

---

## SLIDE 7: Assumptions & Unknowns (1:30)
### Title: "10 Assumptions + 8 Unknowns = Honest Risk Register"

**Talking Points:**
- **Top 3 Assumptions:**
  - A1: athenahealth has machine-accessible APIs (MEDIUM confidence)
  - A3: Prior-auth rules exist in structured form (MEDIUM confidence)
  - A5: Operational queues exist and are monitored (MEDIUM confidence)
  - Impact if wrong: Design shifts; requires fallback approach
  
- **Top 3 Unknowns to validate:**
  - U1: Exact athenahealth API availability in this clinic
  - U5: HIPAA logging requirements (compliance blocker if undefined)
  - U6: Current baseline metrics (needed to set realistic targets)
  
- **Why this matters:** I'm not pretending certainty I don't have. I've done the thinking to separate "I'm confident about this" from "I need to validate this."

**Visual:** Risk matrix: HIGH/MEDIUM/LOW confidence vs HIGH/MEDIUM/LOW impact

**Why it matters:** Coaches specifically ask for "at least 5 genuine unknowns, not filler." This shows disciplined thinking.

---

## SLIDE 8: Success Metrics (1:30)
### Title: "Measurable Outcomes — Before & After"

**Talking Points:**
- **Metric 1:** Intake Completeness Before Visit ≥95%
  - What it measures: Are physicians actually starting visits with complete intake info?
  - Current baseline: Unknown (need to collect)
  
- **Metric 2:** Missed Prior-Auth & Medication Defects ≥50% reduction
  - Directly addresses the two failures named in scenario
  
- **Metric 3:** Front-Desk Workload 30–40% reduction
  - Measure: intake handling time per patient
  - Why: Justifies the AI investment
  
- **Metric 4:** Clinical Boundary Violations = 0
  - Non-negotiable safety metric
  
- **Metric 5:** Audit Logging = 100% coverage
  - Compliance requirement

**Visual:** Dashboard-style mockup showing 5 KPIs

**Why it matters:** Coaches want to see that I know how the solution will be measured. Not vague ("better"), specific (95%, 50%, 40%).

---

## SLIDE 9: Readiness Discipline (1:30)
### Title: "Deterministic Gate: 7 Conditions Must All Be TRUE"

**Talking Points:**
- A work item can only move to **READY_FOR_VISIT** if ALL of these are true:
  1. Questionnaire = COMPLETE
  2. Insurance = VERIFIED_ACTIVE
  3. Prior-Auth = VALID or NOT_REQUIRED
  4. Medications = No open escalations
  5. Allergies = No open escalations
  6. Visit Reason = ROUTINE_ROUTED (or escalation resolved)
  7. No recent integration failures
  
- If **ANY** condition is false → **HOLD_FOR_REVIEW**
- **Key:** No subjective override of readiness logic
  - Humans can resolve individual escalations
  - Once resolved, readiness re-evaluates automatically
  - But logic itself is deterministic, not opinionated

**Visual:** Checklist with 7 conditions; ALL checked = READY; ANY unchecked = HOLD

**Why it matters:** This prevents the system from appearing "ready" while hiding unresolved issues. Safety is enforced by logic, not by hope.

---

## SLIDE 10: Closing (1:30)
### Title: "Why This Works — Three Core Principles"

**Talking Points:**
1. **Fail Closed, Not Open**
   - Missing data? Escalate.
   - Integration timeout? Escalate.
   - Ambiguous rule? Escalate.
   - Never guess or assume success.

2. **Automate What's Mechanical, Escalate What Requires Judgment**
   - Agent is a **router and flagging system**, not a clinician
   - Every escalation includes full context for human decision
   - Zero clinical judgment anywhere in the system

3. **Audit Everything**
   - Every action logged with actor, timestamp, details
   - Logs immutable
   - If there's a question later, the audit trail answers it

**Visual:** Three pillars: Fail Closed | Route & Flag | Audit Everything

**Final Statement:**
"This isn't just a spec. It's a safety boundary written into code. The agent does what's mechanical, escalates what's clinical, and proves everything happened."

---

## Presenter Notes (What You Say Under Each Slide)

**SLIDE 1:** "The practice is doing intake manually. 180 patients a day, 4 staff. And they're missing things — expired prior authorizations slip through, medication changes don't get reviewed before the physician walks in. That's the problem we're solving."

**SLIDE 2:** "Here's the key insight: most of intake is just moving data around. Check the appointment time. Did the patient fill out the form? Is insurance active? Did medications change? These are questions we can ask and answer mechanically. The clinical judgment — is this change important? — that stays with the clinician. The agent collects and routes; humans decide."

**SLIDE 3:** "I've drawn three clear delegation lines. The agent works alone on deterministic tasks — sync appointments, check forms. The agent + human together flag things that might need attention — medication mismatch, allergy present. And humans keep the clinical decisions — is this urgent, is this medication significant? Each boundary is justified by the work itself, not by arbitrary 'let's automate this one because we can.'"

**SLIDE 4:** "The spec isn't prose. It's 10 functional requirements with pseudocode, decision trees, error handling. Four data entities with all states defined. Three integration contracts with exact request/response formats, timeouts, retry logic. An AI coding agent can take this and build it. No 'what if' questions needed."

**SLIDE 5:** "And we've locked down the safety boundary. Eight things the system will never do. Diagnose? No. Triage medically? No. Assess medication significance? No. If the temptation comes up in the build, we explicitly say 'don't implement it.' That boundary is hard."

**SLIDE 6:** "We have 10 test scenarios. Happy path, failure modes, edge cases. We sync the same appointment twice and verify it doesn't create two work items. We trigger urgent-phrase routing and verify that the escalation context contains zero clinical judgment. We test that when insurance times out, we escalate, we don't guess. These aren't hypotheticals; they're concrete."

**SLIDE 7:** "I've listed 10 assumptions — athenahealth has APIs, prior-auth rules exist in structured form, queues are operationally owned — and I've said what I'm confident about and what I'm not. I need to validate these before build. That's not weakness; that's discipline."

**SLIDE 8:** "We'll measure: Are intakes actually complete before visits? Did we reduce missed prior-auths and medication issues by at least 50%? Did we free up the front desk? Did we maintain a zero clinical judgment boundary? These are the metrics. Not 'did the system run,' but 'did it solve the problem safely?'"

**SLIDE 9:** "Readiness isn't a gut call. It's a deterministic gate. Seven conditions. All true = ready. Any false = hold. Humans resolve individual escalations and readiness re-evaluates. But the logic itself is mechanical. That's how we prevent incomplete intakes from slipping through."

**SLIDE 10:** "At the heart: Fail closed. Route and flag. Audit everything. The agent doesn't think it's a clinician. It collects data, escalates safely, and logs every step. That's how we automate what's mechanical and keep clinical judgment human."

---

## Backup Slides (If Asked)

### Backup A: "What if an integration fails?"
- Insurance timeout? Set status=TIMEOUT, create escalation to FRONT_DESK, log it.
- Don't assume "active" coverage.
- Fail closed: the case waits for human review or manual override.

### Backup B: "How do you know no clinical judgment leaked in?"
- Boundary tests (V4, V5) scan escalation contexts for prohibited keywords: "diagnosis," "urgency_level," "significant," "recommend."
- If any keyword found, test fails.
- We enforce the boundary in code.

### Backup C: "What if the clinic doesn't have structured prior-auth rules?"
- That's Assumption A3, and I've flagged it as MEDIUM confidence.
- If rules don't exist, the design shifts: we escalate all prior-auth questions to FRONT_DESK (human decides).
- No inference, ever.

### Backup D: "How many escalations will actually happen?"
- In the happy path (V1): 0 escalations, case goes to READY_FOR_VISIT.
- In real-world scenarios: Depends on data quality and baseline rates.
- That's why we need current baseline metrics (Assumption U6).

### Backup E: "Who resolves escalations and how fast?"
- FRONT_DESK resolves: questionnaire incomplete, insurance inactive, prior-auth issues
- CLINICAL_STAFF resolves: medication changes, allergies, ambiguous visit reasons
- PRACTICE_MANAGER resolves: system outages
- Deadline: 4 hours before visit (time buffer for readiness before appointment)
- Expected response time: configurable per clinic workflow

---

## Visual Design Tips (If Creating Actual Slides)

- **Colors:** Green for "READY," Red for "HOLD," Yellow for "ESCALATE"
- **Icons:** Checkmark for autonomous tasks, Person + Checkmark for human review, Person for human decides
- **Charts:** Risk matrix (Assumption slide), Dashboard KPIs (Metrics slide), Checklist (Readiness slide)
- **Font:** Large (minimum 28pt for titles, 18pt for bullets) — you're in a room, not a document
- **Animation:** Minimal. Transitions only if they help (e.g., conditions checking off one by one for Readiness slide)
- **Backup:** Have the full spec PDF open on your laptop in case someone asks a detailed question

---

## Timing Breakdown
- Slide 1 (Problem): 0:00–1:30 (1:30)
- Slide 2 (Opportunity): 1:30–3:00 (1:30)
- Slide 3 (Delegation): 3:00–4:30 (1:30)
- Slide 4 (Buildable): 4:30–6:00 (1:30)
- Slide 5 (Safety): 6:00–7:30 (1:30)
- Slide 6 (Validation): 7:30–9:00 (1:30)
- Slide 7 (Assumptions): 9:00–10:30 (1:30)
- **BREAK / BUFFER:** 10:30–11:00 (optional, for questions mid-presentation)

**If you have exactly 10 minutes and no break:**
- Compress Slides 7–10 into a final "Closing" slide
- Hit: Assumptions, Metrics, Readiness, Principles
- Use backup slides for detailed Q&A

---

## Likely Questions & Quick Answers

**Q: "Won't the agent make mistakes on visit-reason routing?"**
- A: Yes, which is why we escalate ambiguous or urgent-looking reasons to clinical staff. The agent's job is to catch obvious routine cases (saves time) and flag uncertain ones (doesn't suppress safety).

**Q: "What if a patient doesn't complete the questionnaire?"**
- A: At 12 hours before visit, we escalate to FRONT_DESK. They can manually collect or proceed with clinical review. The case is marked HOLD_FOR_REVIEW until resolved.

**Q: "How is this different from just hiring one more front-desk person?"**
- A: One more person costs ~$40K/year fully loaded. The agent handles 30–40% of intake volume without scaling headcount. After 1–2 years, it pays for itself. Plus it's consistent (no human variation) and auditable (every action logged).

**Q: "What if the clinic doesn't have the infrastructure you assume?"**
- A: That's why I've listed 10 assumptions and 8 unknowns. We validate these before building. If athenahealth APIs don't exist, we adapt the design. If prior-auth rules are tribal knowledge, we escalate. The spec is flexible on tactics, rigid on safety.

**Q: "Is this HIPAA compliant?"**
- A: Yes. Every PHI access is logged. Logs are immutable. Logs are retained per clinic policy. Audit trail is complete. But the exact compliance requirements are clinic-specific (Assumption U5), so we confirm with their privacy officer before build.

---

## What This Presentation Shows Coaches

✅ **You understand the problem** — not "automate everything," but specific, named failures (expired prior auths, medication changes)

✅ **You've drawn defensible delegation lines** — not arbitrary, justified by task properties

✅ **The spec is buildable** — pseudocode, entities, integration contracts, no ambiguity

✅ **Safety is enforced in design** — hard prohibitions, boundary tests, deterministic gates

✅ **You've tested edge cases** — not just happy path, but failure modes and boundary enforcement

✅ **You're honest about unknowns** — 10 assumptions + 8 unknowns + validation plans

✅ **You know how success is measured** — 5 metrics, specific targets, data collection plan

---

**END OF PRESENTATION NOTES**
