# Scenario 5 Closed Build Loop: Complete Diagnostic Guide

**Status:** Ready to hand to Claude Code and iterate
**Taxonomy Source:** spec-ambiguity-vs-builder-mistakes.md (full reference included below)

---

## Quick Start: What You're About To Do

1. **Give Claude your spec** (`Scenario-5-Comprehensive-Spec-GFM.md`)
2. **Claude builds** all 10 FRs, 4 entities, mock integrations, and runs 10 tests
3. **You run tests locally** and note which ones fail
4. **You diagnose each failure** using the 4-part taxonomy below
5. **You fix the root cause** (spec, code, test, or missing requirement)
6. **Loop until all tests pass**

---

## The 4-Part Diagnostic Taxonomy (Executive Summary)

When Claude's build doesn't match expectations, **classify the issue** before fixing it:

### **Type 1: SPEC AMBIGUITY** ← You own the fix
- **Signal:** Spec is unclear; Claude's interpretation is defensible but not what you meant
- **Fix:** Rewrite the spec to be explicit
- **Example:** Spec says "validate address is valid" → Claude checks if non-empty. You meant "geocode in USPS." Spec ambiguity.

### **Type 2: BUILDER MISREAD** ← Claude owns the fix
- **Signal:** Spec is clear; Claude didn't read it carefully
- **Fix:** Point Claude to exact spec line, ask for code fix
- **Example:** Spec says "escalate if insurance ≠ active" → Claude doesn't escalate on ERROR. Builder misread.

### **Type 3: TEST PROBLEM** ← Test owns the fix
- **Signal:** Code is correct per spec; test expectations are wrong
- **Fix:** Update test, not code
- **Example:** Spec says "round half up" → Claude implements correctly → test expects floor behavior. Test wrong.

### **Type 4: DESIGN GAP** ← Spec owns the fix
- **Signal:** Spec is clear but incomplete; Claude built what was asked but missed something obvious
- **Fix:** Add missing requirement to spec, re-prompt Claude
- **Example:** Spec says "save order" → Claude saves with no error handling. Spec missing error-handling requirement.

---

## How to Diagnose

**When a test fails:**

1. **Read the failure message** (what expected vs. actual)
2. **Open the spec** to the relevant FR section
3. **Ask: Is the spec clear about this?**
   - YES, clear → **BUILDER MISREAD** (Step 5 below)
   - NO, unclear → **SPEC AMBIGUITY** (Step 4 below)
4. **Ask: Did Claude build what was asked?**
   - YES, built correctly → **TEST PROBLEM** (Step 3 below) OR **DESIGN GAP** (Step 4 below)
   - NO, didn't build it → **BUILDER MISREAD** (Step 5 below)

---

## Step-by-Step Actions for Each Category

### **Step 1: SPEC AMBIGUITY (You Fix)**

**Diagnosis:** Spec is unclear; Claude's choice is reasonable but not your intent.

**Example:**
- Spec: "Escalate if insurance ≠ active"
- Claude: Escalates only on INACTIVE (treats ERROR, TIMEOUT as non-escalating)
- You meant: Escalate on ANY non-active state (INACTIVE, ERROR, TIMEOUT all escalate)
- Issue: Spec didn't list all non-active states

**Action:**
1. Rewrite spec to be explicit
2. List all possible values
3. Specify which ones trigger escalation

**Updated spec example:**
```
FR3, Step 2: Handle Response

If response = {status: "active"}:
   Set insurance_status = VERIFIED_ACTIVE
   No escalation

If response status is ANYTHING ELSE (inactive, error, timeout):
   Set insurance_status accordingly
   Create escalation to FRONT_DESK

Specifically:
- status: "inactive" → insurance_status = VERIFIED_INACTIVE → escalate
- status: "error" → insurance_status = ERROR → escalate
- timeout/no response → insurance_status = TIMEOUT → escalate
```

**Send to Claude:**
"I found a spec ambiguity. The original FR3 said 'Escalate if insurance ≠ active.' I meant this to include ALL non-active outcomes (INACTIVE, ERROR, TIMEOUT). Here's the clarified version: [paste updated spec]. Please revise your implementation."

---

### **Step 2: BUILDER MISREAD (Claude Fixes)**

**Diagnosis:** Spec is clear, but Claude's code doesn't match.

**Example:**
- Spec FR5, step c: "If patient answer = 'Yes': Set medication_status = CHANGES_REPORTED, create escalation"
- Claude's code: Only escalates if there's a mismatch detected
- Issue: Claude didn't read the full FR5; only read the comparison logic

**Action:**
1. Find the exact spec line Claude missed
2. Show Claude the line
3. Show Claude what the code does vs. what spec says
4. Ask for specific code fix

**Message to Claude:**
```
I found a builder misread in FR5 (Medication Changes).

Spec says (FR5, section c):
"If patient answer = 'Yes':
   Set medication_status = CHANGES_REPORTED
   Create escalation (trigger: MEDICATION_CHANGE_REPORTED) to CLINICAL_STAFF"

Your code: Only creates escalation if comparison detects a mismatch.

Problem: The spec says escalate on ANY "Yes" answer, not just on mismatches.

Fix: Create the escalation when patient answers "Yes", regardless of comparison results.
```

---

### **Step 3: TEST PROBLEM (You Fix)**

**Diagnosis:** Claude's code is correct per spec; test expectations are wrong.

**Example:**
- Spec: "Escalate if any allergy flag exists"
- Claude: Detects all flags correctly
- Test: Only checks for specific allergen name (but order might vary)
- Issue: Test too brittle; assumes specific order

**Action:**
1. Verify Claude's code actually matches the spec
2. Verify Claude's logic is correct
3. Fix the test, not the code
4. Re-run

**Test fix example:**
```python
# BEFORE (wrong):
assert escalation.context['allergies'][0]['name'] == 'Penicillin'

# AFTER (correct):
assert len(escalation.context['allergies']) > 0
assert any(a['name'] == 'Penicillin' for a in escalation.context['allergies'])
```

---

### **Step 4: DESIGN GAP (You Fix)**

**Diagnosis:** Spec is clear but incomplete; missing entire requirement.

**Example:**
- Spec says: "Log QUESTIONNAIRE_SENT"
- Claude logs: timestamp, patient_id, action_type
- Missing: What channel was used (email/SMS/portal)?
- Issue: Spec didn't specify to log the channel

**Action:**
1. Identify what's missing
2. Add it to the spec explicitly
3. Re-prompt Claude with updated spec

**Updated spec:**
```
FR10, Audit Logging

When: Questionnaire is sent

Log:
- timestamp: ISO 8601 UTC
- entity_id: intake_work_item_id
- action_type: QUESTIONNAIRE_SENT
- details:
    channel: "email" | "sms" | "patient_portal"
    recipient: patient contact info
    patient_id: UUID
    message_id: if applicable
```

**Send to Claude:**
"I found a design gap in FR10. When we log QUESTIONNAIRE_SENT, we need to include which channel was used (email, SMS, or patient portal). Here's the updated FR10: [paste]. Please revise logging to include this."

---

## Common Issues in Scenario 5 Build

These are likely mismatches you'll find (and how to fix them):

| Issue | Root Cause | Diagnosis | Fix |
|:--|:--|:--|:--|
| Insurance ACTIVE status escalates | Claude escalates all statuses | BUILDER MISREAD | Show FR3 step 2a (active = no escalation) |
| Med changes don't escalate | Claude only escalates if mismatch | BUILDER MISREAD | Show FR5 step c (Yes = always escalate) |
| Readiness allows open escalations | Claude checks fields, not escalations array | BUILDER MISREAD | Show FR8 step 1d (query escalations array) |
| Audit logs missing 17 actions | Claude logs only state changes | BUILDER MISREAD or DESIGN GAP | List all 17 actions, show FR10 |
| Urgency scoring in escalations | Claude adds urgency fields | BUILDER MISREAD | Show Hard Prohibitions (no urgency scoring) |
| Questionnaire escalation wrong | Deadline threshold misunderstood | SPEC AMBIGUITY | Clarify: escalate at 12-hour mark exactly |
| Reconciliation state unclear | "Medication mismatch" vs "change reported" | SPEC AMBIGUITY | Define exact conditions for each status |

---

## Document Your Iterations

Create file: `Scenario-5-BuildLoop-Issues.md`

```markdown
# Build Loop Iteration Log

## Iteration 1: Initial Build

**Claude delivered:** Code + test results

### Test Results
- V1 (Happy Path): ✅ PASS
- V2 (Incomplete Form): ❌ FAIL
- V3 (Insurance Timeout): ✅ PASS
- V4 (Urgent Phrase): ❌ FAIL
- V5 (Med Change): ❌ FAIL
- [continue V6-V10]

### Issue 1: V2 fails
**Test:** test_incomplete_questionnaire
**Expected:** questionnaire_status=INCOMPLETE, escalation to FRONT_DESK
**Actual:** questionnaire_status=INCOMPLETE, but no escalation created
**Spec Reference:** FR2, step 2b
**Diagnosis:** BUILDER MISREAD
**Spec excerpt:** "If any required field missing: set questionnaire_status = INCOMPLETE and escalate"
**Fix Applied:** Sent Claude FR2 step 2b with instruction to add escalation logic

### Issue 2: V4 fails (Urgent Phrase)
**Test:** test_urgent_phrase_escalation
**Expected:** Escalation context has NO prohibited keywords
**Actual:** Escalation context includes "urgency_level": "high"
**Spec Reference:** FR7 + Hard Prohibitions § 4
**Diagnosis:** BUILDER MISREAD
**Spec excerpt:** "Do NOT attempt to score medical urgency"
**Fix Applied:** Sent Claude Hard Prohibitions list with instruction to remove urgency field

### Issue 3: V5 fails (Med Change)
**Test:** test_medication_change_escalation
**Expected:** medication_status=CHANGES_REPORTED, escalation created
**Actual:** medication_status=NO_CHANGES_REPORTED, no escalation
**Spec Reference:** FR5, step c
**Diagnosis:** BUILDER MISREAD
**Spec excerpt:** "If patient answer = 'Yes': Set medication_status = CHANGES_REPORTED, Create escalation"
**Fix Applied:** Sent Claude exact FR5 step c with instruction to escalate on "Yes" answer

## Iteration 2: After Fixes

**Claude re-implemented:** FR2, FR7, FR5

### Test Results After Fixes
- V2 (Incomplete Form): ✅ PASS
- V4 (Urgent Phrase): ✅ PASS
- V5 (Med Change): ✅ PASS
- [continue with remaining tests]

### New Issue: V8 fails (Idempotency)
**Test:** test_duplicate_appointment_sync
**Expected:** Only 1 work item for same appointment
**Actual:** 2 work items created
**Spec Reference:** FR1 (Appointment Sync)
**Diagnosis:** BUILDER MISREAD
**Spec excerpt:** "Idempotent: re-running sync with same data produces no duplicates"
**Fix Applied:** Added uniqueness constraint check before work-item creation

## Iteration 3: After Additional Fixes

### Final Test Results
- V1: ✅ PASS
- V2: ✅ PASS
- V3: ✅ PASS
- V4: ✅ PASS
- V5: ✅ PASS
- V6: ✅ PASS
- V7: ✅ PASS
- V8: ✅ PASS
- V9: ✅ PASS
- V10: ✅ PASS

### Build Loop Status: ✅ COMPLETE

Total iterations: 3
Total issues found: 5
Issues by category:
- BUILDER MISREAD: 5
- SPEC AMBIGUITY: 0
- TEST PROBLEM: 0
- DESIGN GAP: 0
```

---

## The Prompt to Give Claude

```
You are building a Patient Intake Coordination Agent for a family medicine practice.

HARD RULES (non-negotiable):
- Do NOT perform clinical judgment
- Do NOT create clinical output
- Every action must be logged
- Fail closed (escalate, never guess)
- All decisions rule-based (never infer)

Read the full specification below and implement everything:

[PASTE ENTIRE Scenario-5-Comprehensive-Spec-GFM.md HERE]

IMPLEMENTATION TASK:
1. Implement all 10 Functional Requirements (FR1-FR10)
2. Implement the 4 data entities (Appointment, IntakeWorkItem, EscalationEvent, AuditLog)
3. Implement 3 mock integrations (athenahealth, insurance tool, prior-auth rules)
4. Create test suite for V1-V10 validation scenarios
5. Run all tests and report PASS/FAIL

BUILD ORDER:
Phase 1: FR1, FR2, FR3 (core workflow)
Phase 2: FR4, FR5, FR6, FR7 (safety-critical)
Phase 3: FR8, FR9, FR10 (operations/audit)
Phase 4: Test suite (V1-V10)

ACCEPTANCE CRITERIA (ALL must be true):
✅ All 10 FRs implemented
✅ All 4 entities with state machines and enums
✅ All 10 validation tests pass
✅ V4, V5 verify NO clinical keywords in escalation contexts
✅ Audit logging complete (all 17 action types from FR10)
✅ No clinical judgment anywhere

PROVIDE:
- Full Python implementation
- Test suite with results
- Any clarifications you needed

If the spec is unclear, ASK ME before implementing. Do NOT guess.
```

---

## When Done

✅ All V1-V10 tests pass
✅ Boundary tests (V4, V5) verify no clinical output
✅ Audit logs complete
✅ No open issues (all spec ambiguities resolved, all builder misreads fixed)

**Implementation is build-loop ready and can move to integration testing.**

---

## Reference: Full Taxonomy (From spec-ambiguity-vs-builder-mistakes.md)

### **Category 1: Spec Ambiguity**

Spec is unclear or interpretable multiple ways. Builder chose a valid interpretation that wasn't intended.

**Signal:** Agent's code matches spec as written but not as you intended.

**Examples:**
- Spec: "Validate address is valid" → Agent: Checks non-empty fields | You meant: Geocode in USPS
- Spec: "Round tax to nearest cent" → Agent: Uses round() | You meant: Use floor()
- Spec: "If user doesn't provide phone, skip validation" → Agent: Sets to null | You meant: Use default phone

**Fix:** Rewrite spec to be explicit. Don't re-prompt agent.

---

### **Category 2: Builder Misread**

Spec is clear. Agent didn't read it carefully.

**Signal:** Agent's code contradicts explicit spec statement.

**Examples:**
- Spec: "DO NOT process payment until approval" → Agent: Processes immediately
- Spec: "customer_id: immutable" → Agent: Allows modifying it
- Spec: "status: enum[OPEN, CONFIRMED, CANCELLED], default OPEN" → Agent: Uses CONFIRMED

**Fix:** Re-prompt agent with exact spec line highlighted. Don't change spec.

---

### **Category 3: Test Problem**

Build matches spec. Test expectations are wrong.

**Signal:** Code and spec align; test fails due to bad assertions or brittle assumptions.

**Examples:**
- Spec: "Round half up" → Agent: Implements correctly → Test: Expects floor
- Spec: "Sort by date, most recent first" → Agent: Correct → Test: Assumes specific ID on tied dates

**Fix:** Update test logic. Don't change code.

---

### **Category 4: Design Gap**

Spec is clear but incomplete. Missing entire requirement category.

**Signal:** Implementation is "correct" per spec but obviously incomplete.

**Examples:**
- Spec: "Validate and save order" → Missing: Error handling for save failures
- Spec: "Create assignment" → Missing: Concurrency control (prevent duplicates)
- Spec: "Process payment" → Missing: Audit logging

**Fix:** Add missing requirement to spec explicitly. Re-prompt agent. Don't accept as "implied best practice."

---

## You're Ready

1. **Copy your spec** (Scenario-5-Comprehensive-Spec-GFM.md)
2. **Paste into Claude** with the prompt above
3. **Wait for Claude's build** (code + test results)
4. **Run locally:** `pytest scenario5_tests.py -v`
5. **Diagnose each failure** using the 4-part taxonomy
6. **Apply fixes** (spec, code, test, or requirements)
7. **Re-run and iterate** until all V1-V10 pass
8. **Document** the journey in Scenario-5-BuildLoop-Issues.md

**Go get it built.**
