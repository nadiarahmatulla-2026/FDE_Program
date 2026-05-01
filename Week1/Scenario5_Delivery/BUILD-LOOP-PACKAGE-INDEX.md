# Scenario 5: Complete Build Loop Package — INDEX

**Status:** Ready to hand to Claude Code and iterate

**Created:** For Week 1 practice (Gate 1 preparation)

---

## FILES IN THIS PACKAGE

### **PRIMARY SPEC**
📄 **Scenario-5-Comprehensive-Spec-GFM.md** (50+ pages)
- The authoritative specification
- 10 Functional Requirements with pseudocode
- 4 Data Entities with state machines
- 3 Integration Contracts
- 10 Validation Scenarios (V1–V10)
- 8 Hard Prohibitions
- 10 Assumptions + 8 Unknowns
- Success Metrics & Delegation Analysis
- **USE FOR:** Handing to Claude, reference during diagnosis

---

### **BUILD LOOP GUIDES**

📋 **CLOSED-BUILD-LOOP-INSTRUCTIONS.md** (Quick reference)
- 5-minute read
- What you're about to do
- The 4-part taxonomy (TL;DR)
- Common issues checklist
- **USE FOR:** Quick setup before starting

📋 **CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md** (Full toolkit)
- 15-minute read
- Complete diagnostic taxonomy with examples
- Step-by-step actions for each category
- Common Scenario 5 issues and fixes
- Iteration log template
- **USE FOR:** Diagnosing issues, applying fixes

📋 **CLOSED-BUILD-LOOP-COMPLETE-PACKAGE.md** (Overview)
- This package's summary
- Iteration workflow example
- Success criteria
- Pro tips and pitfalls
- **USE FOR:** Getting oriented, understanding the flow

📋 **BUILD-LOOP-SETUP.txt** (Minimal instructions)
- 2-minute reference
- Copy-paste prompt for Claude
- Common issues table
- **USE FOR:** Quick reminder before starting

---

### **PRESENTATION MATERIALS** (If defending your solution)

🎨 **Scenario-5-10min-Defense-SLIDES.html** (Interactive)
- 10 slides, fully formatted
- Color-coded, timed
- Designed for screen sharing
- **USE FOR:** Live presentation

📝 **Scenario-5-10min-Defense-Presentation.md** (Text version)
- Same 10 slides as markdown
- Talking points for each slide
- **USE FOR:** Backup or print copy

📝 **Scenario-5-SPEAKER-NOTES.md** (Full script)
- Word-by-word what to say
- Tone guidance for each slide
- Timing breakdowns
- Q&A strategies
- **USE FOR:** Rehearsal, exact pacing

📝 **Scenario-5-QUICK-REFERENCE-Guide.md** (Q&A Defense)
- 7 likely coach challenges + defenses
- Key phrases coaches like
- Confidence boost section
- **USE FOR:** Last-minute prep, Q&A

---

## HOW TO USE THIS PACKAGE

### **Scenario 1: You're Ready to Start the Build Loop**

1. **Open:** CLOSED-BUILD-LOOP-INSTRUCTIONS.md (5 min)
2. **Copy:** Prompt from that file
3. **Append:** Full text of Scenario-5-Comprehensive-Spec-GFM.md
4. **Send:** To Claude Code
5. **Wait:** 10–20 minutes for Claude's build
6. **Run:** Tests locally (`pytest scenario5_tests.py -v`)
7. **Reference:** CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md for each failure

---

### **Scenario 2: Tests Failed and You Need to Diagnose**

1. **Open:** CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md
2. **Find:** The section matching your failing test (see "Common Issues" table)
3. **Read:** The diagnostic section for that issue
4. **Classify:** Using the 4-part taxonomy
5. **Apply:** The recommended fix
6. **Re-run:** Tests
7. **Document:** In Scenario-5-BuildLoop-Issues.md (template provided in guide)

---

### **Scenario 3: You Need to Defend Your Solution (Presentation)**

1. **Open:** Scenario-5-10min-Defense-SLIDES.html (in browser, full screen)
2. **Read:** Scenario-5-SPEAKER-NOTES.md for talking points
3. **Reference:** Scenario-5-QUICK-REFERENCE-Guide.md for Q&A
4. **Have open:** Scenario-5-Comprehensive-Spec-GFM.md (on second screen)
5. **Present:** 10 minutes, hit all key points
6. **Answer:** Questions using the spec and diagnostic guide

---

## QUICK START (3 STEPS)

### **Step 1: Prepare (5 minutes)**
```
1. Read: CLOSED-BUILD-LOOP-INSTRUCTIONS.md
2. Copy: The Claude prompt
3. Copy: Full text of Scenario-5-Comprehensive-Spec-GFM.md
```

### **Step 2: Hand to Claude (1 minute)**
```
1. Open Claude.ai or Cursor
2. Paste prompt + spec
3. Hit enter
```

### **Step 3: Iterate (30–60 minutes)**
```
1. Wait for Claude's build (10–20 min)
2. Run tests locally
3. For each failure:
   - Reference CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md
   - Classify the issue
   - Apply the fix
4. Re-run until all pass
```

---

## THE 4-PART TAXONOMY (REFERENCE)

When a test fails, classify the issue:

| Type | Signal | Fix | Owner |
|:--|:--|:--|:--|
| **SPEC AMBIGUITY** | Spec unclear; Claude's interpretation reasonable but not intended | Rewrite spec to be explicit | You |
| **BUILDER MISREAD** | Spec clear; Claude's code doesn't match | Show Claude exact line, ask for fix | Claude |
| **TEST PROBLEM** | Code correct per spec; test expectations wrong | Fix the test, not the code | You |
| **DESIGN GAP** | Spec clear but incomplete; missing requirement | Add requirement to spec | You |

---

## EXPECTED OUTCOMES

### **After Iteration 1 (Initial Build)**
- Some tests pass, some fail (expected)
- Failures are likely BUILDER MISREAD (Claude didn't read everything)
- Takes ~20 min to diagnose issues

### **After Iteration 2 (After First Fixes)**
- More tests pass
- Remaining failures mix of MISREAD and AMBIGUITY
- Takes ~10 min to diagnose and fix

### **After Iteration 3 (Final Polish)**
- Most tests pass
- Remaining issues are DESIGN GAP or AMBIGUITY (rare)
- Takes ~15 min total to finalize

### **Final State (Build Loop Complete)**
- ✅ All V1–V10 tests pass
- ✅ Boundary tests (V4, V5) verify no clinical output
- ✅ Audit logs complete
- ✅ Zero unresolved issues
- ✅ You can explain every design decision

---

## SUCCESS CRITERIA (When to Stop Iterating)

**All of these must be true:**

- ✅ V1: Happy path → READY_FOR_VISIT, 0 escalations
- ✅ V2: Incomplete form → HOLD_FOR_REVIEW, escalation created
- ✅ V3: Insurance timeout → BLOCK, no guessed coverage
- ✅ V4: Urgent phrase → ESCALATE, NO clinical keywords
- ✅ V5: Med change → ESCALATE, NO significance scoring
- ✅ V6: No rule match → UNDETERMINED + escalate (no inference)
- ✅ V7: Allergy flag → ESCALATE, NO severity assessment
- ✅ V8: Duplicate sync → Idempotent (1 work item, not 2)
- ✅ V9: Audit trail → Complete, immutable, all actions logged
- ✅ V10: Override + re-eval → Readiness re-evaluates after resolution

---

## COMMON ISSUES (Quick Reference)

| Issue | Diagnosis | Fix |
|:--|:--|:--|
| Insurance ACTIVE gets escalated | BUILDER MISREAD | Show FR3 step 2a |
| Med changes don't escalate | BUILDER MISREAD | Show FR5 step c |
| Readiness allows open escalations | BUILDER MISREAD | Show FR8 step 1d |
| Audit logs incomplete | BUILDER MISREAD | Show all 17 actions in FR10 |
| Urgency score in escalations | BUILDER MISREAD | Show Hard Prohibitions |
| Questionnaire deadline wrong | SPEC AMBIGUITY | Clarify exact 12-hour threshold |
| Reconciliation state unclear | SPEC AMBIGUITY | Define conditions for each status |

---

## ITERATION LOG TEMPLATE

Create file: `Scenario-5-BuildLoop-Issues.md`

```markdown
# Build Loop Issues Log

## Iteration 1
- Issue 1: [V# FAIL] — [Diagnosis] — [Fix applied]
- Issue 2: [V# FAIL] — [Diagnosis] — [Fix applied]

## Iteration 2
- Issue 3: [V# FAIL] — [Diagnosis] — [Fix applied]

## Final
- All V1–V10: ✅ PASS
- Build loop: ✅ COMPLETE
```

---

## FILES YOU MIGHT CREATE

During the build loop, you'll create:

1. **scenario5_implementation.py** (Claude's code, saved locally)
2. **scenario5_tests.py** (Claude's tests, saved locally)
3. **Scenario-5-BuildLoop-Issues.md** (Your iteration log, template in diagnostic guide)
4. **Scenario-5-BuildLoop-Results.txt** (Final test results snapshot)

---

## PRO TIPS

✅ **DO:**
- Diagnose before fixing (wrong diagnosis = wrong fix)
- Keep the spec as source of truth (update it when needed)
- Document every issue (you're learning the domain)
- Expect 2–4 iterations (this is normal)
- Re-run all tests after each fix (don't assume others still pass)

❌ **DON'T:**
- Re-prompt Claude without diagnosing (wastes time)
- Assume Claude is always wrong (often the spec is unclear)
- Change spec without telling Claude (spec drift)
- Skip the boundary tests (V4, V5 — these verify safety)
- Accept "might work" (build loop proves correctness)

---

## NEXT STEPS AFTER BUILD LOOP

When build loop is complete (all V1–V10 pass):

1. **Document** your iteration log
2. **Review** your spec one final time (any improvements?)
3. **Prepare** for integration testing (with real EHR data)
4. **Defend** your solution (use presentation materials)
5. **Submit** for Gate 1 peer review

---

## SUPPORT

**If you get stuck:**
1. Re-read the diagnostic guide (common issues table)
2. Check the spec (is it clear on this point?)
3. Show Claude the exact spec line (re-prompting often helps)
4. Take a 10-minute break (fresh eyes help)

**If you're not sure about a diagnosis:**
- Ask yourself: "Is the spec clear about this?" (YES → BUILDER MISREAD | NO → SPEC AMBIGUITY)
- If still unsure, go with SPEC AMBIGUITY (safer, doesn't blame Claude)
- Clarify the spec, re-prompt with updated section

---

## YOU'RE READY

You have everything needed to:
1. ✅ Hand a complete spec to Claude
2. ✅ Understand what Claude builds back
3. ✅ Diagnose any mismatches
4. ✅ Apply the right fix
5. ✅ Iterate to completion

**Ready to start? Open CLOSED-BUILD-LOOP-INSTRUCTIONS.md and go.**

---

**Questions?** Check the diagnostic guide. **Success?** Document it. **Learning?** That's the whole point.

**Go build this thing.**
