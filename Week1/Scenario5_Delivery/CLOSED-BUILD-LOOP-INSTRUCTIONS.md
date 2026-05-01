# Closed Build Loop: Patient Intake Agent

## What You're About To Do

Send your spec (`Scenario-5-Comprehensive-Spec-GFM.md`) to Claude Code and have it build the agent, then diagnose any mismatches using a taxonomy.

---

## The 4-Part Diagnostic Taxonomy

When Claude builds something that doesn't match the spec, classify the issue:

### **Type 1: SPEC AMBIGUITY**
- **Problem:** Spec is unclear; Claude interpreted reasonably but not as you meant
- **Who fixes it:** YOU
- **How:** Rewrite the spec to be explicit, send Claude the corrected section

### **Type 2: BUILDER MISREAD**  
- **Problem:** Spec is clear; Claude didn't read it carefully
- **Who fixes it:** CLAUDE
- **How:** Point Claude to exact spec line, ask for code fix

### **Type 3: TEST PROBLEM**
- **Problem:** Claude's code is right; test logic is wrong
- **Who fixes it:** YOU
- **How:** Fix the test, not the code

### **Type 4: DESIGN GAP**
- **Problem:** Spec doesn't specify requirement; Claude couldn't implement what's not there
- **Who fixes it:** YOU
- **How:** Add requirement to spec, send updated section to Claude

---

## How to Use the Taxonomy

**When Claude's output doesn't match:**

1. Look at the failing test (e.g., V2)
2. Read what the spec says about that FR
3. Ask: **Is the spec clear?**
   - YES → **BUILDER MISREAD** (Claude's code needs fixing)
   - NO → **SPEC AMBIGUITY** (spec needs clarifying)
4. Apply the fix (update spec OR correct Claude's code)
5. Re-run tests
6. Repeat until all pass

---

## The Prompt to Give Claude

Copy and paste this into Claude Code, then append the full spec:

```
You are building a Patient Intake Coordination Agent for a family medicine practice.

HARD RULES (non-negotiable):
- Do NOT perform clinical judgment anywhere
- Do NOT create clinical output
- Every action must be logged
- Fail closed (escalate, never guess)
- All decisions rule-based (never infer)

Read the specification below and implement:

[PASTE ENTIRE Scenario-5-Comprehensive-Spec-GFM.md HERE]

IMPLEMENTATION TASK:
1. Code all 10 Functional Requirements (FR1-FR10)
2. Code the 4 data entities (Appointment, IntakeWorkItem, EscalationEvent, AuditLog)
3. Code mock integrations (athenahealth, insurance, prior-auth rules)
4. Create test suite for V1-V10 validation scenarios
5. Run all tests, report PASS/FAIL

BUILD ORDER:
- Phase 1: FR1, FR2, FR3 (core)
- Phase 2: FR4, FR5, FR6, FR7 (safety)
- Phase 3: FR8, FR9, FR10 (operations)
- Phase 4: Test V1-V10

ACCEPTANCE CRITERIA (ALL must be true):
✅ All 10 FRs implemented
✅ All 4 entities with state machines
✅ All V1-V10 tests pass
✅ V4, V5 verify NO clinical keywords in escalation contexts
✅ Audit logging complete (all 17 action types from FR10)
✅ No clinical judgment anywhere

PROVIDE:
- Python implementation
- Test suite with results
- Any clarifications you needed

If spec is unclear, ASK ME before implementing. Do NOT guess.
```

---

## After Claude Delivers

1. **Run the code locally**
   ```bash
   pytest scenario5_tests.py -v
   ```

2. **For each failing test:**
   - Check the error message (expected vs actual)
   - Open the spec to the relevant FR
   - Classify using the 4-part taxonomy
   - Document the issue

3. **Apply fixes:**
   - SPEC AMBIGUITY → Update spec, send to Claude
   - BUILDER MISREAD → Show Claude the exact line, ask for fix
   - TEST PROBLEM → Fix test yourself
   - DESIGN GAP → Add requirement to spec, send to Claude

4. **Re-run tests**

5. **Loop until all pass**

---

## Common Issues You'll Likely Find

| Issue | Root Cause | Fix |
|:--|:--|:--|
| Insurance ACTIVE gets escalated | Claude escalates all statuses | Show FR3 step 2a (active → no escalation) |
| Med changes not escalated | Claude only escalates if mismatch | Show FR5 step c (yes → always escalate) |
| Readiness allows open escalations | Claude checks fields, not array | Show FR8 step 1d (check escalations array) |
| Audit logs incomplete | Claude only logs state changes | List all 17 actions from FR10 |
| Urgency scoring in escalations | Claude adds clinical fields | Show Hard Prohibitions (no urgency scoring) |

---

## Success = All Tests Pass

When all V1-V10 show PASS and boundary tests (V4, V5) verify no clinical output:

✅ Build loop is complete
✅ Implementation ready for integration
✅ You can explain every design decision

---

## Document Your Work

Create file: `Scenario-5-BuildLoop-Issues.md`

```markdown
# Build Loop Issues & Resolutions

## Iteration 1

### Test Failures
- V2: FAIL
- V5: FAIL

### Issue 1: V2 fails (Medication change not escalated)
- **Expected:** medication_status=CHANGES_REPORTED, escalation created
- **Actual:** medication_status=NO_CHANGES_REPORTED, no escalation
- **Spec Reference:** FR5, step c
- **Diagnosis:** BUILDER MISREAD
- **Fix Applied:** Showed Claude FR5 step c, asked for escalation logic

### Issue 2: V5 fails (Readiness allows open escalations)
- **Expected:** status=HOLD_FOR_REVIEW (escalation open)
- **Actual:** status=READY_FOR_VISIT (escalation open)
- **Spec Reference:** FR8, step 1d
- **Diagnosis:** BUILDER MISREAD
- **Fix Applied:** Explained escalations array query logic

## Iteration 2

### Test Results After Fixes
- V2: ✅ PASS
- V5: ✅ PASS
- [Continue for all V1-V10]

## Final Status
- All V1-V10: ✅ PASS
- Build loop: ✅ COMPLETE
```

---

## You're Ready

1. Copy spec
2. Paste into Claude with the prompt
3. Wait for Claude to build
4. Run tests locally
5. Diagnose any failures
6. Apply fixes
7. Iterate until all pass
8. Document the journey

**That's the closed build loop.**

Go ahead and send it to Claude Code. When you have results, come back and we'll diagnose together.
