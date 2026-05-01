# COPY THIS PROMPT INTO CLAUDE CODE

---

## STEP 1: Open Claude.ai or Cursor IDE

## STEP 2: Start New Conversation

## STEP 3: Paste Everything Below (The Prompt First, Then Your Spec)

---

```
You are building a Patient Intake Coordination Agent for a family medicine practice.

HARD RULES (non-negotiable):
- Do NOT perform clinical judgment anywhere
- Do NOT create clinical output (no diagnosis, urgency, significance assessment)
- Every action must be logged
- Fail closed on errors (escalate, never guess data)
- All decisions rule-based (never infer)

Read the full specification below and implement everything:

[PASTE ENTIRE TEXT OF Scenario-5-Comprehensive-Spec-GFM.md HERE]

YOUR IMPLEMENTATION TASK:

1. Implement all 10 Functional Requirements (FR1-FR10)
   - Each as a method/function with the pseudocode logic shown
   - Include error handling per spec
   - Include escalation triggers as specified

2. Implement the 4 data entities
   - Appointment (read-only from athenahealth)
   - IntakeWorkItem (state machine with full enums)
   - EscalationEvent (immutable, with status tracking)
   - AuditLog (append-only, immutable)

3. Implement 3 mock integration adapters
   - athenahealth (mock appointments, medications, allergies)
   - Insurance eligibility tool (mock responses with timeout handling)
   - Prior-auth rules source (rule lookup, cache)

4. Create test suite for V1-V10 validation scenarios
   - Each scenario as a test function
   - Clear setup, execution, assertions
   - V4 and V5 must verify NO clinical keywords in escalation contexts

5. Run all tests and provide results
   - PASS/FAIL for each V1-V10
   - Details on any failures
   - Summary of test execution

BUILD ORDER (do in this sequence):
Phase 1 (Core Workflow): FR1, FR2, FR3 → test with V1, V2, V3
Phase 2 (Safety-Critical): FR4, FR5, FR6, FR7 → test with V4-V7
Phase 3 (Operations): FR8, FR9, FR10 → test with V8-V10
Phase 4 (Validation): Run all V1-V10, report results

ACCEPTANCE CRITERIA (ALL must be true for completion):
✅ All 10 FRs implemented with code
✅ All 4 entities fully defined with state machines and enums
✅ All 10 validation tests pass (V1-V10: PASS)
✅ Boundary tests (V4, V5) explicitly verify NO clinical keywords in escalation contexts
✅ Audit logging complete (all 17 action types from FR10 present)
✅ No clinical judgment anywhere in codebase
✅ Fail-closed behavior verified (timeouts → escalate, not guess)

CODE STYLE:
- Use Python 3.8+
- Use dataclasses for entities (or equivalent)
- Use enums for all state machines
- Use type hints (no bare dict)
- Use explicit error handling (no silent failures)
- Use docstrings for functions/methods

TESTING:
- Use pytest for test suite
- Each scenario V1-V10 is one test function
- Test setup includes mock data and mock integrations
- Test assertions verify expected state, escalations, and logs
- Boundary tests (V4, V5) scan escalation context for prohibited keywords

PROHIBITED KEYWORDS (for boundary test assertions):
diagnosis, diagnose, clinical, urgent, severity, significant, important, 
dangerous, safe, unsafe, recommend, suggest, concern, risk_level, urgency_score

If ANY of these keywords appear in V4 or V5 escalation context → TEST FAILS

CLARIFICATION QUESTIONS:
If the spec is unclear on ANY point, ASK ME before implementing.
Do NOT guess or make assumptions about unclear sections.

Examples of things to ask me about:
- "FR4 says 'never infer prior-auth.' Should I set status=UNDETERMINED or status=ERROR?"
- "FR5: Should escalation be created even if no mismatch is found?"
- "FR8: Are all 7 readiness conditions AND (all true = ready) or OR (any true = ready)?"

DELIVERABLES:
1. Full implementation code (Python)
2. Test suite with results showing PASS/FAIL for each scenario
3. List of any clarifications you needed to ask for
4. Any implementation notes or decisions

START NOW.
```

---

## STEP 4: After "START NOW" — Paste Your Full Spec

Copy the entire text from: **Scenario-5-Comprehensive-Spec-GFM.md**

Paste it after the prompt above, replacing [PASTE ENTIRE TEXT...]

---

## STEP 5: Hit Enter

Claude will build for 10–20 minutes.

---

## STEP 6: When Claude Finishes

Claude will provide:
- Python implementation code
- Test suite code
- Test results (PASS/FAIL for each scenario)
- Any questions/clarifications

---

## STEP 7: Next Steps

1. **Save Claude's code** to a file: `scenario5_implementation.py`
2. **Save test suite** to a file: `scenario5_tests.py`
3. **Run tests locally:**
   ```bash
   pip install pytest
   pytest scenario5_tests.py -v
   ```
4. **Note failures** and reference: **CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md**
5. **Iterate** until all V1–V10 pass

---

## THAT'S IT

Ready? Copy the prompt. Append your spec. Send to Claude.

Come back when tests return and we'll diagnose any failures.

**GO.**
