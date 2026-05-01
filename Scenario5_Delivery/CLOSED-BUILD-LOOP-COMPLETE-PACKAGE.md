# Scenario 5 Complete Build Loop Package — Final Summary

**Ready to hand to Claude Code and iterate.**

---

## What You Now Have

### **Core Spec (Primary Reference)**
- **Scenario-5-Comprehensive-Spec-GFM.md** 
  - Full 50+ page buildable specification
  - 10 Functional Requirements with pseudocode
  - 4 Data Entities with state machines
  - 3 Integration Contracts
  - 10 Validation Scenarios (V1–V10)
  - 8 Hard Prohibitions (clinical boundary)
  - 10 Assumptions + 8 Unknowns (honest risk register)

### **Build Loop Guides**
1. **CLOSED-BUILD-LOOP-INSTRUCTIONS.md** (Quick reference, 5-min read)
2. **CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md** (Full diagnostic toolkit, 15-min read)

### **Presentation Materials** (If needed for defense)
- Scenario-5-10min-Defense-SLIDES.html (interactive slides)
- Scenario-5-10min-Defense-Presentation.md (text version)
- Scenario-5-SPEAKER-NOTES.md (full script)
- Scenario-5-QUICK-REFERENCE-Guide.md (Q&A defense)

---

## The Closed Build Loop (Step-by-Step)

### **Step 1: Prepare Claude Input** (5 minutes)

1. Open `Scenario-5-Comprehensive-Spec-GFM.md`
2. Copy entire text
3. Open Claude.ai or Cursor IDE
4. Start new conversation

### **Step 2: Send the Build Prompt** (Copy from CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md)

```
You are building a Patient Intake Coordination Agent...
[Full prompt in guide]
[Paste your entire spec]
```

### **Step 3: Claude Builds** (Expect: 10–20 min)

Claude will provide:
- Python data models (4 entities)
- Implementation code (10 FRs as methods)
- Mock integrations (3 adapters)
- Test suite (V1–V10 scenarios)
- Test results (PASS/FAIL for each)

### **Step 4: Run Tests Locally** (10 minutes)

```bash
# Save Claude's code to file
# Install dependencies
pip install pytest dataclasses python-dateutil

# Run tests
pytest scenario5_tests.py -v

# See which tests pass/fail
```

### **Step 5: Diagnose Failures** (Use 4-Part Taxonomy)

For each failing test:

1. **Read failure message** (expected vs actual)
2. **Open spec** to relevant FR
3. **Ask: Is spec clear?**
   - YES → **BUILDER MISREAD** (Claude fix needed)
   - NO → **SPEC AMBIGUITY** (Spec fix needed)
4. **Apply fix** (see diagnostic guide)
5. **Re-run tests**

### **Step 6: Iterate Until All Pass**

Loop:
- Classify issue
- Apply fix
- Re-run tests
- Document in `Scenario-5-BuildLoop-Issues.md`

### **Step 7: Success Criteria**

✅ All V1–V10 tests pass
✅ Boundary tests (V4, V5) verify NO clinical keywords
✅ Audit logs complete
✅ No open issues

---

## The 4-Part Diagnostic Taxonomy (TL;DR)

| Category | Signal | Fix | Owner |
|:--|:--|:--|:--|
| **SPEC AMBIGUITY** | Spec unclear; Claude's choice reasonable but not intended | Rewrite spec | You |
| **BUILDER MISREAD** | Spec clear; Claude's code contradicts it | Re-prompt Claude | Claude |
| **TEST PROBLEM** | Code correct per spec; test wrong | Fix test | You |
| **DESIGN GAP** | Spec incomplete; missing entire category | Add to spec | You |

---

## Expected Issues in Scenario 5

These are **likely** mismatches you'll find:

### **Issue 1: Insurance Escalation Logic**
- **Problem:** Claude escalates insurance status that should not escalate
- **Diagnosis:** BUILDER MISREAD (FR3 is clear)
- **Fix:** Show Claude FR3, step 2a: "If status='active', no escalation"

### **Issue 2: Medication Change Escalation**
- **Problem:** Claude doesn't escalate med changes, only escalates if mismatch
- **Diagnosis:** BUILDER MISREAD (FR5 is clear)
- **Fix:** Show Claude FR5, step c: "If patient answer='Yes', create escalation"

### **Issue 3: Readiness Gate**
- **Problem:** Claude marks READY_FOR_VISIT even with open escalations
- **Diagnosis:** BUILDER MISREAD (FR8 is clear)
- **Fix:** Show Claude FR8, step 1d: "Check escalations array, not just status fields"

### **Issue 4: Audit Logs**
- **Problem:** Audit logs incomplete (missing some action types)
- **Diagnosis:** BUILDER MISREAD or DESIGN GAP
- **Fix:** List all 17 action types from FR10, ask Claude to add all

### **Issue 5: Clinical Output**
- **Problem:** Escalation context includes urgency score or significance assessment
- **Diagnosis:** BUILDER MISREAD (Hard Prohibitions are clear)
- **Fix:** Show Claude prohibitions: "Do NOT score urgency or assess significance"

---

## File Quick Reference

| File | Purpose | When to Use |
|:--|:--|:--|
| **Scenario-5-Comprehensive-Spec-GFM.md** | Primary spec | Hand to Claude, reference during diagnosis |
| **CLOSED-BUILD-LOOP-INSTRUCTIONS.md** | Quick setup | Before you start |
| **CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md** | Full toolkit | Diagnose issues, fix them |
| **CLOSED-BUILD-LOOP-ISSUES.md** | (You create) | Document iterations |
| Scenario-5-10min-Defense-SLIDES.html | Presentation | If you need to defend the solution |

---

## Iteration Workflow Example

### **Iteration 1**
```
Claude builds → Run tests → Results:
  ✅ V1: PASS
  ❌ V2: FAIL (no escalation on incomplete form)
  ✅ V3: PASS
  ❌ V5: FAIL (med change not escalated)
  
Diagnose V2:
  - Spec says: "If incomplete, escalate"
  - Claude's code: No escalation
  - Diagnosis: BUILDER MISREAD
  - Fix: Show Claude FR2 step 2b
  
Document: Issue 1 (V2) — BUILDER MISREAD — FR2 step 2b — Fixed
```

### **Iteration 2**
```
Claude revises → Run tests → Results:
  ✅ V1: PASS
  ✅ V2: PASS (fixed)
  ✅ V3: PASS
  ✅ V4: PASS
  ❌ V5: FAIL (urgency score in escalation)
  
Diagnose V5:
  - Spec says: "Do NOT score urgency"
  - Claude's code: Includes urgency_level field
  - Diagnosis: BUILDER MISREAD
  - Fix: Show Claude Hard Prohibitions
  
Document: Issue 2 (V5) — BUILDER MISREAD — Hard Prohibitions — Fixed
```

### **Iteration 3**
```
Claude revises → Run tests → Results:
  ✅ V1: PASS
  ✅ V2: PASS
  ✅ V3: PASS
  ✅ V4: PASS
  ✅ V5: PASS
  ✅ V6: PASS
  ✅ V7: PASS
  ✅ V8: PASS
  ✅ V9: PASS
  ✅ V10: PASS

BUILD LOOP COMPLETE ✅
```

---

## When to Stop Iterating

Stop when **ALL** of these are true:

- ✅ V1–V10 all show PASS
- ✅ Boundary tests (V4, V5) confirm NO clinical keywords in escalation contexts
- ✅ Audit logs include all 17 action types from FR10
- ✅ Zero open issues (no unresolved spec ambiguities)
- ✅ You can explain each design decision

**At this point:** Implementation is build-loop ready. Ready for integration with real EHR data.

---

## Pro Tips for Success

### **Before You Start**
- [ ] Spec is complete and internally consistent (no contradictions)
- [ ] You understand all 10 FRs yourself (can explain to Claude)
- [ ] You have time for 2–4 iterations (build → test → diagnose → fix → repeat)
- [ ] You have Claude access (Claude.ai or Cursor)

### **During Build Loop**
- [ ] Don't blame Claude immediately; diagnose first
- [ ] If spec is unclear, own that (don't re-prompt Claude to guess)
- [ ] Document every issue (category, root cause, fix applied)
- [ ] Keep the spec as the source of truth (update it when needed)

### **Common Pitfalls**
- ❌ Re-prompting Claude without diagnosing first (wastes time)
- ❌ Changing the spec without telling Claude (spec drift)
- ❌ Treating SPEC AMBIGUITY as BUILDER MISREAD (wrong fix)
- ❌ Not running tests locally (only trusting Claude's results)
- ❌ Giving up after first iteration (usually takes 2–4)

### **Success Indicators**
- ✅ Issues trend from BUILDER MISREAD → DESIGN GAP → SPEC AMBIGUITY (progressively cleaner builds)
- ✅ Iterations get faster (less debug, more understand)
- ✅ Final code matches spec precisely
- ✅ All boundary tests pass (clinical safety verified)

---

## The Diagnostic Checklist

When a test fails, use this checklist:

- [ ] Test name and failure message recorded
- [ ] Spec section (FR#) identified
- [ ] Spec section read carefully (not skimmed)
- [ ] Claude's code reviewed (understand what it does)
- [ ] Question asked: "Does spec address this?" (YES or NO)
- [ ] Category classified: AMBIGUITY | MISREAD | TEST | GAP
- [ ] Root cause identified (not symptom)
- [ ] Fix planned (who does it: you, Claude, or test)
- [ ] Fix applied
- [ ] Tests re-run
- [ ] Issue documented in Issues log

---

## You're Ready

**You have:**
- ✅ A complete, buildable specification
- ✅ A diagnostic taxonomy to classify issues
- ✅ Instructions for Claude (copy-paste ready)
- ✅ A clear iteration workflow
- ✅ Expected issues and fixes

**Next step:** Send the spec to Claude Code. Come back when you have test results.

---

## Support Files (For Reference)

If you need additional context:

- **Week1-Thinking-Discipline-Primer.md** — Foundation thinking (why this all matters)
- **production-spec-checklist.md** — Spec quality standards (reference while diagnosing)
- **claude-md-examples-guide.md** — Implementation guidance (if Claude asks clarifying questions)

---

## Final Checklist Before You Start

- [ ] Scenario-5-Comprehensive-Spec-GFM.md is finalized
- [ ] You've read it entirely (not skimmed)
- [ ] You understand all 10 FRs
- [ ] You know the 8 Hard Prohibitions by heart
- [ ] You have Claude access (Claude.ai or Cursor IDE)
- [ ] You have 30–60 minutes available for initial build + first iteration
- [ ] You have this diagnostic guide bookmarked or printed
- [ ] You're ready to iterate (not expecting perfect build on first try)

---

**Ready?** Copy your spec. Go build this. Document the journey. Learn what this feels like.

**This is the work of Week 1 practice: spec discipline, diagnosis discipline, iteration discipline.**

**Go.**
