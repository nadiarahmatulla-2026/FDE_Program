# Closed Build Loop Guide: Scenario 5 Agent Build with Claude Code

**Purpose:** Hand the Scenario-5-Comprehensive-Spec-GFM.md to Claude Code, systematically diagnose any mismatches against the spec-ambiguity-vs-builder-mistakes.md taxonomy, and iterate to production readiness.

**Timeline:** 2–3 hours for initial build + diagnosis

---

## Pre-Build Checklist

- [ ] `Scenario-5-Comprehensive-Spec-GFM.md` is open and accessible
- [ ] `spec-ambiguity-vs-builder-mistakes.md` is open (for diagnosis reference)
- [ ] You have a code editor (VSCode, etc.) ready
- [ ] You have a terminal ready (for running tests)
- [ ] You have 2–3 hours blocked (don't rush this)
- [ ] You have a document open to log issues as they arise

---

## Phase 1: Initial Build Request (15–30 min)

### 1.1 Prepare Claude Code Session

Open Claude Code (or your AI coding agent interface) and provide this preamble:

```
You are building a Patient Intake Coordination Agent for a family medicine practice.
Your spec is attached below. It is comprehensive and buildable. Your job is to implement it exactly as specified.

KEY RULES FOR THIS BUILD:
1. Treat the spec as the source of truth. If something is unclear, ask for clarification.
2. Implement all 10 Functional Requirements (FR1–FR10).
3. Implement the 4 data entities (Appointment, IntakeWorkItem, EscalationEvent, AuditLog).
4. Implement all 3 integration contracts (athenahealth, insurance, prior-auth).
5. Do NOT invent requirements not in the spec.
6. Do NOT implement any of the 8 hard prohibitions.
7. Implement all 10 validation scenarios (V1–V10) as tests.

If you find ambiguity or a gap in the spec, STOP and ask for clarification.
Do not make assumptions or invent details.

Here is the spec:
```

### 1.2 Paste the Comprehensive Spec

Copy the entire content of `Scenario-5-Comprehensive-Spec-GFM.md` into the Claude Code session.

### 1.3 Initial Build Request

Ask Claude to:

```
Please build the Patient Intake Coordination Agent according to the spec.

Deliverables:
1. Data model (4 entities with all attributes and state machines)
2. Functional requirement implementations (FR1–FR10 as pseudocode or code)
3. Integration adapters (contracts for athenahealth, insurance, prior-auth)
4. Validation test suite (10 test scenarios V1–V10)

Output format:
- Use Python (or language of choice) for pseudocode/actual code
- Structure code as modules/classes (one per entity, one per FR)
- Include docstrings for each function/class
- Do NOT implement UI or database; assume datastore exists
- Include print statements or logging that show state transitions

Start with the data model, then FRs 1–3 (appointment sync, questionnaire, insurance).
I will review each section and ask for updates.

Do not attempt all 10 FRs at once. Let's do this iteratively so I can catch issues early.
```

### 1.4 First Iteration: Data Model Only

Ask Claude to start with just the data model:

```
Before implementing logic, let's define the data model precisely.

Please provide:
1. The 4 entities (Appointment, IntakeWorkItem, EscalationEvent, AuditLog)
2. For each entity:
   - All attributes with types
   - Any state machines (enum values)
   - Constraints (immutability, uniqueness, foreign keys)
   - Audit fields (timestamps, actor tracking)

Use Python dataclasses or similar for clarity.

Do not implement any logic yet. Just the data model.
```

---

## Phase 2: Systematic Review & Diagnosis (60–90 min)

### 2.1 Review Data Model Against Spec § 4 "Required Entities"

**Spec § 4 (end of "Agent Specification") defines all 4 entities.**

Checklist:
- [ ] **Appointment entity:**
  - id (UUID)
  - patient_id, provider_id, location_id (UUIDs)
  - scheduled_start_at, scheduled_end_at (ISO 8601)
  - visit_type (enum)
  - procedure_code (optional)
  - status (enum: SCHEDULED, CANCELLED, etc.)
  - created_at, updated_at
  - All fields are present? Y/N

- [ ] **IntakeWorkItem entity:**
  - id (UUID, unique per appointment)
  - appointment_id (foreign key, unique)
  - patient_id
  - status (state machine: CREATED → PROCESSING → READY_FOR_VISIT or HOLD_FOR_REVIEW → ABANDONED)
  - Seven status fields: questionnaire_status, insurance_status, prior_auth_status, medication_status, allergy_status, visit_reason_status, completion_decision
  - All enums listed correctly? Y/N
  - Audit fields: created_at, updated_at
  - escalations (one-to-many relationship)
  - All fields present? Y/N

- [ ] **EscalationEvent entity:**
  - id (UUID)
  - intake_work_item_id (foreign key)
  - trigger_code (enum with 16+ values)
  - queue (enum: FRONT_DESK, CLINICAL_STAFF, PRACTICE_MANAGER)
  - status (enum: CREATED, ACKNOWLEDGED, IN_PROGRESS, RESOLVED, OVERRIDDEN, ESCALATED_FURTHER)
  - created_at, created_by ("SYSTEM")
  - resolved_at, resolved_by, resolution_note (nullable)
  - override_reason (nullable, required if status=OVERRIDDEN)
  - context (object with full details)
  - All fields present? Y/N

- [ ] **AuditLog entity:**
  - id (UUID)
  - timestamp (ISO 8601 UTC, immutable)
  - entity_type, entity_id
  - action_type (enum with 18+ values)
  - actor_type (enum: SYSTEM, FRONT_DESK_USER, CLINICAL_USER, MANAGER_USER)
  - actor_id (string)
  - patient_id (optional, if PHI access)
  - details_json (object)
  - immutable flag
  - All fields present? Y/N

**Diagnosis:**

If the agent's entity definitions **match the spec exactly**:
→ Mark PASS. Move to FR review.

If the agent **added extra fields not in spec**:
→ **Category: ACCEPTABLE VARIATION.** If fields are sensible (e.g., added `updated_by` for audit), accept them. Optionally note: "Good thinking on X; we might use this."

If the agent **omitted fields from spec**:
→ **Category: BUILDER MISREAD.** Re-prompt: "The spec defines [list missing fields]. Please add them."

If the agent **used wrong types or enums**:
→ **Category: BUILDER MISREAD.** Re-prompt with correct types/enums from spec.

---

### 2.2 Review FR1–FR3 (Appointment Sync, Questionnaire, Insurance)

Once entities are correct, ask Claude to implement:

```
Now implement FRs 1–3 with this structure:

FR1: Appointment Sync
- Pseudocode: How do you query athenahealth every 15 min?
- Idempotency: How do you prevent duplicate work items?
- Error handling: What if athenahealth times out?
- Output: Create IntakeWorkItem or skip if already exists

FR2: Questionnaire Dispatch & Tracking
- Logic: Send questionnaire when intake is created
- Tracking: Check completion; escalate if deadline missed
- Output: Update questionnaire_status on IntakeWorkItem

FR3: Insurance Verification
- Pseudocode: Call insurance integration; handle response
- Timeout: 10s timeout, 2x retry with backoff
- Output: Set insurance_status; create escalation if inactive/error/timeout

For each FR, include:
- Pseudocode (clear logic)
- Decision table (if X, do Y)
- Example: What happens for a specific input?
- Acceptance criteria from the spec
```

**Diagnosis for FR1–FR3:**

For each FR, check:

| Check | What to Look For | Pass / Diagnosis |
|:--|:--|:--|
| **Appointment Sync (FR1)** | Queries next 2 calendar days; 15-min interval; idempotent | If wrong interval or not idempotent → Builder Misread |
| | Retry logic: 3x with backoff (2s, 4s, 8s) | If wrong retry logic → Builder Misread |
| | Never creates duplicate work items | If duplicates possible → Builder Misread |
| **Questionnaire (FR2)** | 12-hour deadline before visit for escalation | If wrong deadline → Builder Misread |
| | Checks for required fields; incomplete triggers escalation | If doesn't check required fields → Builder Misread |
| | Updates questionnaire_status enum | If wrong enum values → Builder Misread |
| **Insurance (FR3)** | 10-second timeout | If wrong timeout → Builder Misread |
| | Fail-closed: doesn't assume "active" on timeout | If assumes active → Builder Misread |
| | Creates escalation to FRONT_DESK for inactive/error/timeout | If escalation logic missing → Builder Misread |

**Example Diagnosis:**

If Claude implements insurance timeout as "assume coverage is active":
```
DIAGNOSIS: Builder Misread

Spec § 4 FR3 says:
"If timeout or HTTP 5xx:
- Set insurance_status = TIMEOUT
- Log: INSURANCE_FAILED (error_code="TIMEOUT", timestamp)
- Create escalation (trigger: INSURANCE_TIMEOUT) to FRONT_DESK"

Your code assumes active coverage on timeout. This violates the "fail closed" principle.

FIX: Replace your timeout handler with:
```python
except asyncio.TimeoutError:
    work_item.insurance_status = InsuranceStatus.TIMEOUT
    log(action_type="INSURANCE_FAILED", error_code="TIMEOUT", timestamp=now())
    create_escalation(work_item, trigger_code="INSURANCE_TIMEOUT", queue="FRONT_DESK")
```
```

---

### 2.3 Review FR4–FR7 (Prior-Auth, Medications, Allergies, Visit-Reason)

Once FR1–FR3 are solid, move to safety-sensitive FRs:

```
Implement FRs 4–7:

FR4: Prior-Auth Determination (CRITICAL - SAFETY)
- Look up rule in clinic's rule set
- If rule not found: Set UNDETERMINED, escalate (never infer)
- If rule says REQUIRED: Check prior-auth records
- Possible outcomes: NOT_REQUIRED, VALID, EXPIRED, MISSING, UNDETERMINED, N_A, ERROR

FR5: Medication Change Collection & Comparison
- NO CLINICAL JUDGMENT. Only mechanical field comparison.
- Any change reported OR mismatch detected → escalate to CLINICAL_STAFF
- Do NOT score significance

FR6: Allergy Flag Retrieval
- Retrieve all active flags from EHR
- If any flag present → escalate to CLINICAL_STAFF
- Do NOT interpret severity

FR7: Visit-Reason Routing
- Match against clinic's urgent-trigger phrase list (configurable)
- Match against narrow admin categories
- If urgent phrase OR blank OR ambiguous → escalate to CLINICAL_STAFF
- If matches routine category → ROUTINE_ROUTED (no escalation)
- Do NOT perform clinical triage
```

**Critical Diagnosis Points:**

| FR | RED FLAGS | Category |
|:--|:--|:--|
| **FR4 (Prior-Auth)** | Infers prior-auth requirement without explicit rule | Builder Misread + Hard Prohibition Violation |
| | Doesn't escalate UNDETERMINED | Builder Misread |
| **FR5 (Medications)** | Scores medication significance ("this is important") | Hard Prohibition Violation |
| | Suppresses medication change to avoid escalation | Hard Prohibition Violation |
| **FR6 (Allergies)** | Assesses allergy severity ("SEVERE = escalate, MILD = ignore") | Hard Prohibition Violation |
| | Doesn't escalate if any flag present | Builder Misread |
| **FR7 (Visit-Reason)** | Performs clinical urgency scoring | Hard Prohibition Violation |
| | Doesn't check against urgent-phrase list | Builder Misread |
| | Suppresses escalation to keep case ready | Hard Prohibition Violation |

---

### 2.4 Review FR8–FR10 (Readiness, Escalations, Audit)

```
Implement FRs 8–10:

FR8: Readiness Evaluation
- 7 conditions must ALL be TRUE for READY_FOR_VISIT
- If ANY condition false → HOLD_FOR_REVIEW
- Deterministic logic, no subjective override

FR9: Escalation Creation & Routing
- Create EscalationEvent (immutable)
- Assign to correct queue per rule
- Notify queue
- Track lifecycle (CREATED → RESOLVED or OVERRIDDEN)

FR10: Audit Logging
- Immutable append-only log
- Log every significant action with actor, timestamp, details
- PHI accesses tracked
- Retention configurable per clinic policy
```

**Diagnosis:**

| FR | RED FLAGS | Category |
|:--|:--|:--|
| **FR8 (Readiness)** | Allows READY_FOR_VISIT with unresolved escalations | Builder Misread / Hard Prohibition |
| | Subjective override of readiness logic | Builder Misread |
| **FR9 (Escalations)** | Escalations can be deleted | Builder Misread |
| | Escalations don't notify owning queue | Design Gap |
| | Escalation queue assignment logic missing | Builder Misread |
| **FR10 (Audit)** | Audit logs can be modified after creation | Builder Misread / Hard Prohibition |
| | PHI access not logged | Builder Misread / Design Gap |
| | No retention policy implemented | Design Gap |

---

## Phase 3: Run Validation Scenarios (60–90 min)

### 3.1 Build Test Suite

Ask Claude:

```
Now implement the validation test suite. For each test scenario (V1–V10):
1. Set up test data (appointment, patient, integrations mocked)
2. Execute the workflow
3. Assert expected outcomes

V1 (Happy Path):
- Routine visit, all data complete
- Expected: READY_FOR_VISIT, 0 escalations
- Assert: work_item.status == READY_FOR_VISIT
- Assert: len(escalations) == 0

V2 (Incomplete Questionnaire):
- Missing required field
- Expected: HOLD_FOR_REVIEW, 1 escalation to FRONT_DESK
- Assert: work_item.status == HOLD_FOR_REVIEW
- Assert: any(e.trigger_code == "QUESTIONNAIRE_INCOMPLETE" for e in escalations)

V3 (Insurance Timeout):
- Insurance integration times out
- Expected: TIMEOUT status, escalation, no guess of "active"
- Assert: work_item.insurance_status == TIMEOUT
- Assert: work_item.status == HOLD_FOR_REVIEW

V4 (Urgent Phrase):
- Visit reason: "chest pain"
- Expected: Escalate to CLINICAL_STAFF, NO clinical output in escalation context
- Assert: work_item.visit_reason_status == ESCALATED_URGENT
- Assert: "diagnosis" not in escalation.context
- Assert: "urgency_level" not in escalation.context

V5 (Medication Change):
- Patient reports medication change
- Expected: Escalate to CLINICAL_STAFF, NO significance scoring
- Assert: work_item.medication_status == CHANGES_REPORTED
- Assert: "significant" not in escalation.context.lower()

V6 (No Prior-Auth Rule):
- Procedure code has no matching rule
- Expected: UNDETERMINED + escalation (never infer)
- Assert: work_item.prior_auth_status == UNDETERMINED
- Assert: any(e.trigger_code == "PRIOR_AUTH_UNDETERMINED" for e in escalations)

V7 (Allergy Present):
- EHR shows allergy flag
- Expected: Escalate to CLINICAL_STAFF
- Assert: work_item.allergy_status == FLAGS_PRESENT
- Assert: "severity" not in escalation.context.lower()

V8 (Idempotency):
- Sync same appointment twice
- Expected: Only 1 work item created
- Assert: len(work_items_for_appointment) == 1

V9 (Audit Trail):
- Run happy path, collect all audit logs
- Expected: Comprehensive logs, immutable, PHI access tracked
- Assert: len(audit_logs) >= 8  # min actions
- Assert: all(log.immutable for log in audit_logs)

V10 (Override + Re-evaluation):
- Escalation created, then resolved by human
- Expected: Readiness re-evaluates and may transition to READY_FOR_VISIT
- Assert: escalation.status == RESOLVED
- Assert: work_item.status == READY_FOR_VISIT (after re-evaluation)
```

### 3.2 Run Tests and Capture Failures

Execute the test suite. Capture output:

```bash
python tests/scenario_5_validation.py
```

Log any failures in a table:

| Scenario | Expected | Actual | Diagnosis | Fix Category |
|:--|:--|:--|:--|:--|
| V1 | READY_FOR_VISIT, 0 escal | READY_FOR_VISIT, 0 escal | ✅ PASS | — |
| V3 | TIMEOUT, HOLD | VERIFIED_ACTIVE, READY | ❌ Agent assumed active on timeout | Builder Misread |
| V4 | ESCALATED_URGENT | ESCALATED_URGENT | ✅ PASS | — |
| V4 context | No "urgency" keyword | Contains "urgency_level=2" | ❌ Clinical judgment leaked | Hard Prohibition Violation |
| ... | ... | ... | ... | ... |

---

## Phase 4: Diagnosis & Iteration (30–60 min)

For each failure, apply the taxonomy from `spec-ambiguity-vs-builder-mistakes.md`:

### 4.1 Classify Each Failure

Use the decision tree:

```
Question 1: Does the agent's implementation match the spec as written?
```

**Example Failure 1: Insurance Timeout → Assumes Active**

```
Q1: Does implementation match spec?
- Spec says: "If timeout, set status=TIMEOUT, escalate, don't guess"
- Implementation: "If timeout, assume coverage active, proceed"
- Answer: NO

Q2: Is the implementation valid under any interpretation?
- Could one reasonably interpret "timeout" as "assume success"?
- No. The spec explicitly forbids this ("fail closed").
- Answer: NO

Q3: Does the spec address this scenario?
- Yes. FR3 has explicit timeout handling.
- Answer: YES

DIAGNOSIS: Category 2 — Builder Misread
FIX: Re-prompt with timeout handler highlighted
```

**Example Failure 2: Medication Flag → Contains "Significance" Keyword**

```
Q1: Does implementation match spec?
- Spec says: "Escalate medication changes. Do NOT score significance."
- Implementation: "Flag with significance_score = 2 (minor)"
- Answer: NO

Q2: Is implementation valid under any interpretation?
- Could one justify scoring significance?
- No. The spec explicitly forbids this (Hard Prohibition #4).
- Answer: NO

Q3: Does spec address this?
- Yes. FR5 says "Do NOT assess medication significance."
- And Hard Prohibitions § 4.
- Answer: YES

DIAGNOSIS: Category 2 — Builder Misread + Hard Prohibition Violation
FIX: Re-prompt with prohibited behavior highlighted
```

**Example Failure 3: Questionnaire Complete, But Test Fails**

```
Q1: Does implementation match spec?
- Spec says: "Check all required fields present"
- Implementation: Checks only 3 of 5 required fields
- Answer: NO (incomplete interpretation)

Q2: Is implementation valid?
- Checking 3/5 fields is incomplete.
- Spec lists all 5 required fields by name.
- Answer: NO

Q3: Does spec address this?
- Yes. § 4 FR2 lists required fields.
- Answer: YES

DIAGNOSIS: Category 2 — Builder Misread
FIX: Re-prompt with complete required field list
```

### 4.2 Re-Prompt Template

For each Category 2 (Builder Misread):

```markdown
## Issue: [Issue Name]

**Test:** [Scenario number and description]
**Expected:** [What spec says should happen]
**Actual:** [What code did instead]

**Diagnosis:** Builder Misread

**Spec Reference:**
The spec says (§ X, FR Y):
> [Quote the exact spec text that was not followed]

**What needs to change:**
Your code [current behavior]. This violates the spec because [explain why].

**Correct implementation:**
[Pseudocode or code snippet showing the correct behavior]

**Example:**
[Walk through a concrete input/output example]
```

### 4.3 Re-Prompt Examples

**Re-Prompt 1: Insurance Timeout**

```markdown
## Issue: Insurance Timeout Handling

**Test:** V3 (Insurance timeout)
**Expected:** work_item.insurance_status == TIMEOUT, escalate to FRONT_DESK
**Actual:** work_item.insurance_status == VERIFIED_ACTIVE (assumed active)

**Diagnosis:** Builder Misread

**Spec Reference:**
The spec says (§ 4, FR3):
> "If timeout or HTTP 5xx:
>  - Set insurance_status = TIMEOUT
>  - Log: INSURANCE_FAILED (error_code="TIMEOUT", timestamp)
>  - Create escalation (trigger: INSURANCE_TIMEOUT) to FRONT_DESK"

**What needs to change:**
Your code assumes coverage is active if the integration times out. This violates the "fail closed" principle. Missing insurance verification is a blocker; we escalate, we don't guess.

**Correct implementation:**
```python
try:
    response = call_insurance_api(patient_id, service_date, timeout=10)
except TimeoutError:
    work_item.insurance_status = InsuranceStatus.TIMEOUT
    log(action_type="INSURANCE_FAILED", error_code="TIMEOUT", ...)
    create_escalation(work_item, "INSURANCE_TIMEOUT", queue="FRONT_DESK")
    # Do NOT set to VERIFIED_ACTIVE
    return  # stop; case is HOLD_FOR_REVIEW
```

**Example:**
Input: Insurance API does not respond within 10 seconds
Expected: insurance_status = TIMEOUT, escalation created, work_item.status = HOLD_FOR_REVIEW
Actual (before fix): insurance_status = VERIFIED_ACTIVE, no escalation, work_item.status = READY_FOR_VISIT
Actual (after fix): insurance_status = TIMEOUT, escalation created, work_item.status = HOLD_FOR_REVIEW
```

**Re-Prompt 2: Medication Significance Score**

```markdown
## Issue: Medication Significance Assessment (Hard Prohibition Violation)

**Test:** V5 (Medication change) — boundary test
**Expected:** Medication change escalated, NO "significance" keyword in context
**Actual:** Escalation contains "significance_score: 2" and "assessment: minor"

**Diagnosis:** Builder Misread + Hard Prohibition Violation

**Spec Reference:**
Hard Prohibitions § 5 states:
> "The agent must never:
>  4. Assess medication significance"

Spec § 4 FR5 states:
> "Do NOT attempt to score significance; escalate immediately
>  The system detects change; a clinician determines meaning."

**What needs to change:**
Your code calculates and stores medication.significance_score. This is clinical judgment and is explicitly forbidden.

**Correct implementation:**
```python
def check_medication_changes(patient_reported, ehr_current):
    changes_detected = []
    
    # ONLY detect mismatch; do NOT score significance
    for field in ["medication_name", "dose", "frequency"]:
        if patient_reported[field] != ehr_current[field]:
            changes_detected.append({
                "field": field,
                "ehr_value": ehr_current[field],
                "patient_value": patient_reported[field]
            })
    
    if changes_detected:
        # Escalate with full context; no judgment
        create_escalation(
            work_item,
            trigger_code="MEDICATION_MISMATCH",
            queue="CLINICAL_STAFF",
            context={"mismatches": changes_detected}
        )
    
    return changes_detected
```

**Example:**
Input: Patient says "dose changed from 10mg to 20mg"; EHR shows 10mg
Escalation context (correct): {"field": "dose", "ehr_value": "10mg", "patient_value": "20mg"}
Escalation context (before fix): {"field": "dose", ..., "significance_score": 4, "assessment": "high-impact change"}
Escalation context (after fix): {"field": "dose", "ehr_value": "10mg", "patient_value": "20mg"} (NO judgment)
```

---

## Phase 5: Final Validation (15–30 min)

Once all re-prompts are complete and tests pass:

### 5.1 Full Test Run

```bash
python tests/scenario_5_validation.py -v
```

Expected output:
```
V1 (Happy Path) ............................ PASS
V2 (Incomplete Questionnaire) .............. PASS
V3 (Insurance Timeout) ..................... PASS
V4 (Urgent Phrase + Boundary) .............. PASS
V5 (Medication + Boundary) ................. PASS
V6 (No Prior-Auth Rule) .................... PASS
V7 (Allergy Flag) .......................... PASS
V8 (Idempotency) ........................... PASS
V9 (Audit Trail Completeness) .............. PASS
V10 (Override + Re-evaluation) ............. PASS

Hard Prohibition Enforcement:
  - No diagnosis detected in logs ........... PASS
  - No clinical triage detected ............ PASS
  - No medication judgment detected ........ PASS
  - No allergy judgment detected ........... PASS
  - No inferred prior-auth detected ........ PASS
  - No suppressed escalations detected .... PASS
  - No fabricated data detected ........... PASS

10/10 scenarios PASS ✅
```

### 5.2 Code Review Checklist

- [ ] All 10 FRs implemented
- [ ] All 4 entities defined with correct attributes and state machines
- [ ] All 3 integration contracts spelled out
- [ ] All error paths lead to escalation (fail closed)
- [ ] No clinical judgment anywhere in the code
- [ ] All audit logging in place (immutable, PHI tracked)
- [ ] All 10 validation scenarios pass
- [ ] Boundary tests (V4, V5) explicitly verify no clinical output
- [ ] Configuration is externalized (no hardcoded clinic values)
- [ ] Code is documented (docstrings, comments on complex logic)

### 5.3 Spec Alignment Verification

Compare the final code against each section of the spec:

| Spec Section | Checklist | ✅ / ❌ |
|:--|:--|:--|
| § 1: Assumptions & Unknowns | Are all assumptions flagged in code comments? | ✅ |
| § 2: Problem & Metrics | Are success metrics measurable by tests? | ✅ |
| § 3: Delegation Analysis | Do code boundaries match 3-mode taxonomy? | ✅ |
| § 4: Agent Spec | Are all FRs 1–10 implemented? | ✅ |
| § 5: Validation | Do all 10 test scenarios pass? | ✅ |

---

## Phase 6: Documentation & Handoff (15 min)

### 6.1 Build Summary Document

Create a BUILD_SUMMARY.md:

```markdown
# Scenario 5 Build Summary

**Status:** ✅ COMPLETE

## What Was Built
- 4 data entities (Appointment, IntakeWorkItem, EscalationEvent, AuditLog)
- 10 functional requirements (FR1–FR10)
- 3 integration adapters (athenahealth, insurance, prior-auth rules)
- 10 validation scenarios (all passing)

## Key Decisions
- Language: [Python / TypeScript / etc.]
- Concurrency: [Using locks / optimistic locking / etc. for idempotency]
- Error handling: Fail-closed (all errors escalate)
- Logging: Immutable append-only AuditLog

## Build vs. Spec Alignment
- Zero hard prohibition violations
- Zero clinical judgment in code
- 100% acceptance criteria met
- 10/10 validation scenarios passing

## Dependencies
- athenahealth SDK (or REST client)
- Insurance integration client
- PostgreSQL / [database choice] for persistence
- Logging framework (e.g., Python logging)

## Testing Results
```
V1 (Happy Path) ............................ PASS
V2 (Incomplete Questionnaire) .............. PASS
V3 (Insurance Timeout) ..................... PASS
V4 (Urgent Phrase + Boundary) .............. PASS
V5 (Medication + Boundary) ................. PASS
V6 (No Prior-Auth Rule) .................... PASS
V7 (Allergy Flag) .......................... PASS
V8 (Idempotency) ........................... PASS
V9 (Audit Trail Completeness) .............. PASS
V10 (Override + Re-evaluation) ............. PASS
```

## Known Limitations (from Assumptions)
- Assumes athenahealth APIs are accessible (A1 - requires validation)
- Assumes prior-auth rules are structured (A3 - requires validation)
- Assumes operational queues exist (A5 - requires validation)
- See spec § 1 for full list of assumptions and unknowns

## Next Steps
1. Validate assumptions (U1–U8) with clinic
2. Run pilot with 10–20 patients
3. Collect baseline metrics for success measurement
4. Prepare for Gate 1 presentation

## Iteration History
- Initial build: [timestamp]
- Iteration 1 (data model): [changes]
- Iteration 2 (FR1–FR3): [changes]
- ... [list all re-prompts and fixes]
- Final build: [timestamp]
```

### 6.2 Code Artifact Storage

Organize deliverables:

```
Scenario5_Delivery/
├── CLOSED-BUILD-LOOP-GUIDE.md (this file)
├── Scenario-5-Comprehensive-Spec-GFM.md
├── build/
│   ├── data_model.py (or .ts)
│   ├── functional_requirements.py
│   ├── integrations.py
│   ├── escalation_router.py
│   ├── audit_logger.py
│   └── __init__.py
├── tests/
│   ├── test_scenarios_v1_to_v10.py
│   ├── test_boundary_enforcement.py
│   └── fixtures.py
├── BUILD_SUMMARY.md
└── ITERATION_LOG.md (optional: detailed log of each re-prompt)
```

---

## Common Issues & Resolution

| Issue | Cause | Diagnosis | Fix |
|:--|:--|:--|:--|
| "Insurance assumed active on timeout" | Agent not following fail-closed | Builder Misread | Re-prompt with timeout handler |
| "Medication change scored as 'significant'" | Clinical judgment in code | Hard Prohibition Violation | Re-prompt; highlight prohibition |
| "Test V8 fails: duplicate work items created" | Idempotency logic missing | Builder Misread | Re-prompt with idempotency pattern |
| "Audit log is mutable" | Agent not treating logs as immutable | Builder Misread | Re-prompt; show immutable pattern |
| "No escalation for ambiguous visit reason" | Missed blank/ambiguous case | Builder Misread | Re-prompt with all cases in FR7 |
| "Test passes but code is not production-ready" | Missing error handling, retry logic, or concurrency control | Design Gap | Update spec with missing requirement; re-build |

---

## Quality Gates (Before Presenting)

Before you present this build to coaches or hand it off to production:

- [ ] **Buildability:** Code can be read and understood by another developer
- [ ] **Safety:** No clinical judgment anywhere; 8 hard prohibitions enforced
- [ ] **Completeness:** All 10 FRs implemented; no TODOs or incomplete sections
- [ ] **Testing:** 10/10 validation scenarios passing; boundary tests verify safety
- [ ] **Documentation:** Docstrings on all functions; spec alignment noted
- [ ] **Configuration:** All clinic-specific values externalized (env vars, config file)
- [ ] **Error Handling:** Every error path is logged and escalated; no silent failures
- [ ] **Audit:** Every action is logged with full context; logs are immutable

**If any gate fails:** Do not proceed. Fix the root cause. Re-run tests.

---

## Next: Prepare for Gate 1 Presentation

Once the build is complete and all tests pass:

1. **Summarize what was built** (use BUILD_SUMMARY.md)
2. **Show test results** (screenshot or run live)
3. **Walk through a validation scenario** (V1 happy path, live if possible)
4. **Discuss any re-prompts** (show what was changed and why using taxonomy)
5. **List assumptions still to validate** (from spec § 1)
6. **Prepare for questions** (use QUICK-REFERENCE-Guide.md)

Your closed build loop is evidence of disciplined thinking and buildability. Coaches will be impressed if you can explain each re-prompt using the taxonomy categories.

---

**END OF CLOSED BUILD LOOP GUIDE**

**Total Estimated Time:** 2–3 hours
**Outcome:** Production-ready code + validated spec + proof of disciplined iteration
