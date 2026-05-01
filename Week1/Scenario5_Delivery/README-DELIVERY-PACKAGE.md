# SCENARIO 5: YOUR COMPLETE DELIVERY PACKAGE — FINAL SUMMARY

**Status:** Ready for Gate 1 (Week 1 practice)

**What you have:** Everything needed to spec, defend, and build the Patient Intake Coordination Agent.

---

## 📦 COMPLETE PACKAGE CONTENTS

### **TIER 1: THE SPEC (Primary Work)**

📄 **Scenario-5-Comprehensive-Spec-GFM.md** ⭐ START HERE
- 50+ page buildable specification
- 10 Functional Requirements (FRs 1–10) with pseudocode
- 4 Data Entities with full state machines
- 3 Integration Contracts (athenahealth, insurance, prior-auth)
- 10 Validation Scenarios (V1–V10)
- 8 Hard Prohibitions (clinical safety)
- 10 Assumptions + 8 Unknowns (honest risk)
- 5 Success Metrics
- Delegation Analysis (3 modes, fully justified)

**Use for:** Handing to Claude Code, reference during disputes

---

### **TIER 2: CLOSED BUILD LOOP (Implementation & Iteration)**

📋 **QUICK-START-CARD.txt** ⭐ READ FIRST
- 1-page tearsheet
- The flow (5 steps)
- 4-part taxonomy
- Common issues table
- Criteria for "done"

📋 **CLOSED-BUILD-LOOP-INSTRUCTIONS.md**
- Quick setup (5 min read)
- Copy-paste prompt for Claude
- What to expect from Claude
- Common issues checklist

📋 **CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md** ⭐ YOUR BIBLE FOR BUILD LOOP
- Full 4-part taxonomy (with examples)
- Step-by-step actions for each category
- Common Scenario 5 issues & fixes
- Iteration log template
- When to stop iterating

📋 **CLOSED-BUILD-LOOP-COMPLETE-PACKAGE.md**
- Package overview
- Expected outcomes per iteration
- Pro tips & pitfalls
- Diagnostic checklist
- Success criteria

📋 **BUILD-LOOP-PACKAGE-INDEX.md**
- Index of all build loop files
- How to use each file
- Quick reference tables

---

### **TIER 3: PRESENTATION & DEFENSE (If needed for Gate 1)**

🎨 **Scenario-5-10min-Defense-SLIDES.html** ⭐ PRESENT WITH THIS
- 10 interactive slides
- Color-coded, timed
- Ready to screen-share
- Covers: Problem → Opportunity → Boundary → Spec → Safety → Validation → Assumptions → Metrics → Readiness → Principles

📝 **Scenario-5-SPEAKER-NOTES.md** ⭐ READ THIS BEFORE PRESENTING
- Word-for-word script
- 90 seconds per slide
- Tone guidance
- Timing breakdowns
- Q&A strategies

📝 **Scenario-5-10min-Defense-Presentation.md**
- Same 10 slides as markdown
- Talking points
- Backup reference

📝 **Scenario-5-QUICK-REFERENCE-Guide.md**
- 7 likely coach challenges
- Your 1-minute defense for each
- Key phrases coaches like
- Confidence boost

📝 **Scenario-5-Complete-Defense-Package-README.md**
- How to prepare for presentation
- Timing breakdown
- Backup answers
- Troubleshooting

---

## 🚀 HOW TO USE THIS PACKAGE

### **Scenario A: You're Ready to Build**

1. **Open:** QUICK-START-CARD.txt (1 min read)
2. **Open:** Scenario-5-Comprehensive-Spec-GFM.md (copy full text)
3. **Open:** CLOSED-BUILD-LOOP-INSTRUCTIONS.md (get the prompt)
4. **Go to:** Claude.ai or Cursor IDE
5. **Paste:** Prompt + your spec
6. **Wait:** 10–20 min for build
7. **Run tests:** `pytest scenario5_tests.py -v`
8. **Reference:** CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md (for each failure)
9. **Iterate:** Until all V1–V10 pass

---

### **Scenario B: Tests Failed, Need to Diagnose**

1. **Open:** QUICK-START-CARD.txt (see 4-part taxonomy)
2. **Open:** CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md
3. **Find:** Your issue in the "Common Issues" table
4. **Read:** The diagnostic section
5. **Classify:** Using the taxonomy
6. **Apply:** The recommended fix
7. **Re-run:** Tests
8. **Document:** In iteration log

---

### **Scenario C: Need to Defend Your Solution**

1. **Open:** Scenario-5-10min-Defense-SLIDES.html (in browser, full screen)
2. **Read:** Scenario-5-SPEAKER-NOTES.md (for talking points)
3. **Have open:** QUICK-REFERENCE-Guide.md (for Q&A)
4. **Have open:** Scenario-5-Comprehensive-Spec-GFM.md (on 2nd screen for reference)
5. **Present:** 10 minutes
6. **Defend:** Challenges using spec + quick reference

---

## ✅ SUCCESS MILESTONES

### **Milestone 1: Spec Complete** ✓ (You're here)
- ✅ 50+ page comprehensive spec
- ✅ 10 FRs with pseudocode
- ✅ 4 entities with state machines
- ✅ 3 integration contracts
- ✅ 10 validation scenarios
- ✅ 8 hard prohibitions
- ✅ Honest unknowns documented

### **Milestone 2: Build Loop Complete** (Next: 1–2 hours)
- Run tests with Claude's code
- All V1–V10 pass
- Boundary tests verify no clinical output
- Audit logs complete
- Zero open spec ambiguities

### **Milestone 3: Ready for Gate 1** (After build loop)
- ✅ Can defend the solution (10-min presentation)
- ✅ Can explain every design decision
- ✅ Can diagnose build issues
- ✅ Implementation proven with tests

---

## 🎯 WHAT COACHES ARE LOOKING FOR

**Three things:**

1. **Defensible delegation boundaries** ← Your spec has this (3 modes, each justified)
2. **Buildable spec (no ambiguity)** ← Your spec has this (10 FRs with pseudocode)
3. **Honest unknowns (5+)** ← Your spec has this (10 assumptions + 8 unknowns)

**Plus:**

- ✅ Safety baked in (8 hard prohibitions, boundary tests)
- ✅ Validation that's concrete (10 test scenarios)
- ✅ Problem understood (success metrics, stakeholder focus)
- ✅ Risk awareness (assumptions discipline, unknowns logged)

---

## 📊 FILE QUICK REFERENCE

| When | Read This | Minutes |
|:--|:--|:--|
| About to start | QUICK-START-CARD.txt | 2 |
| Before handing to Claude | CLOSED-BUILD-LOOP-INSTRUCTIONS.md | 5 |
| Diagnosing a failure | CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md | 10–15 |
| Before presenting | Scenario-5-SPEAKER-NOTES.md | 10 |
| Q&A prep | Scenario-5-QUICK-REFERENCE-Guide.md | 5 |
| Confused about workflow | BUILD-LOOP-PACKAGE-INDEX.md | 10 |

---

## 🔧 THE CLOSED BUILD LOOP (In 30 Seconds)

```
1. HAND spec to Claude Code
2. Claude builds: code + tests
3. You run tests locally
4. Tests fail (expected)
5. For each failure:
   • Classify using 4-part taxonomy
   • Fix root cause (spec, code, test, or requirement)
   • Re-run
6. Loop until all pass
7. Documentation: You learned to diagnose builds
```

---

## ⚙️ THE 4-PART TAXONOMY (Your Decision Tree)

**When test fails → Ask: "Is spec clear?"**

| Spec Clear? | Classification | Fix | Owner |
|:--|:--|:--|:--|
| YES | BUILDER MISREAD | Point Claude to exact line | Claude |
| NO | SPEC AMBIGUITY | Rewrite spec to be explicit | You |
| (Special) | TEST PROBLEM | Fix test logic | You |
| (Missing) | DESIGN GAP | Add requirement to spec | You |

---

## 📋 EXPECTED JOURNEY

### **Hour 1: Prepare & Hand to Claude**
- [ ] Read: QUICK-START-CARD.txt (2 min)
- [ ] Prepare: Copy spec + prompt (5 min)
- [ ] Hand to Claude: Paste into chat (1 min)
- [ ] Wait: Claude builds (10–20 min)
- [ ] Download: Claude's code (1 min)

### **Hour 2: First Iteration**
- [ ] Run tests locally (2 min)
- [ ] See failures: 2–5 likely (2 min)
- [ ] Diagnose each failure (5–10 min)
- [ ] Apply fixes (10 min)
- [ ] Re-run tests (2 min)
- [ ] Document: What was wrong, how fixed (5 min)

### **Hour 3: Second Iteration** (Often final)
- [ ] Run tests (2 min)
- [ ] See fewer failures (2 min)
- [ ] Diagnose remaining issues (5 min)
- [ ] Apply fixes (5 min)
- [ ] Re-run (2 min)
- [ ] All pass (likely)

### **After: Prep to Defend**
- [ ] Read: SPEAKER-NOTES.md (10 min)
- [ ] Practice: Say it out loud (10 min)
- [ ] Prep: QUICK-REFERENCE-Guide.md (5 min)
- [ ] Present: 10-min defense

---

## 🏆 WHAT YOU'LL HAVE AFTER THIS

✅ A **comprehensive, buildable spec** that Claude can implement
✅ A **tested, validated implementation** (all V1–V10 pass)
✅ An **honest understanding** of what's certain vs. uncertain
✅ **Skill at diagnosing builds** (the 4-part taxonomy)
✅ The ability to **defend your solution** (10-min presentation)
✅ **Documentation** of your thinking process

---

## 🎓 WHAT THIS IS TEACHING YOU

This is Week 1 FDE core skill development:

1. **Spec Discipline** — Can you write a spec precise enough that an AI doesn't need to ask clarifying questions?
2. **Delegation Clarity** — Can you draw a defensible boundary between what humans decide and what agents do?
3. **Diagnosis Discipline** — When a build fails, can you figure out **why** (not just blame the builder)?
4. **Honest Unknowns** — Can you separate what you know, assume, and need to validate?
5. **Safety-First Thinking** — Can you bake safety requirements into the design, not bolt them on after?

---

## 🚨 CRITICAL REMINDERS

- ✅ The spec is your source of truth (don't let Claude drift the design)
- ✅ Diagnose before fixing (wrong diagnosis = wrong fix)
- ✅ Boundary tests (V4, V5) are non-negotiable (they prove safety)
- ✅ Document your iterations (you're learning the domain)
- ✅ Expect 2–4 iterations (this is normal, not a failure)

---

## 📌 NEXT STEPS

### **Immediately:**
1. Read: QUICK-START-CARD.txt (print it, put on desk)
2. Review: The 4-part taxonomy (memorize it)
3. Prepare: Copy your spec + prompt

### **Now:**
1. Open Claude
2. Paste prompt + spec
3. Wait for build

### **When tests return:**
1. Run locally
2. Reference: CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md
3. Iterate until all pass

### **After build loop passes:**
1. Read: SPEAKER-NOTES.md
2. Practice: 10-minute presentation
3. Defend: Your solution

---

## 🎯 SUCCESS = All Tests Pass

When you see:
```
V1: PASS ✅
V2: PASS ✅
V3: PASS ✅
V4: PASS ✅ (no clinical keywords)
V5: PASS ✅ (no significance scoring)
V6: PASS ✅
V7: PASS ✅
V8: PASS ✅ (idempotent)
V9: PASS ✅ (audit complete)
V10: PASS ✅ (override works)

BUILD LOOP COMPLETE
```

**You're done.** Implementation is proven. Ready for Gate 1.

---

## 📞 STUCK? HERE'S HOW TO UNSTICK YOURSELF

| Problem | Solution |
|:--|:--|
| "What do I do first?" | Read QUICK-START-CARD.txt (2 min) |
| "How do I diagnose a failure?" | Read CLOSED-BUILD-LOOP-DIAGNOSTIC-GUIDE.md (10 min) |
| "Is the spec ambiguous or Claude's fault?" | Use 4-part taxonomy (see above) |
| "What do I say when presenting?" | Read SPEAKER-NOTES.md (10 min) |
| "What if Claude says the spec is unclear?" | That's honest feedback; clarify the spec |
| "What if I'm not sure how to fix something?" | Diagnose first (taxonomy), fix second (docs) |

---

## 🏁 YOU'RE READY

You have:
- ✅ A complete, defensible specification
- ✅ A diagnostic framework for build issues
- ✅ Clear instructions for Claude
- ✅ Validation that proves correctness
- ✅ A presentation that defends your thinking

**Everything you need to succeed at Week 1 practice → Gate 1.**

**Go build this.**

---

**Questions?** Everything is documented. Start with QUICK-START-CARD.txt.

**Ready?** Copy your spec. Hand to Claude. Come back when tests return. We'll diagnose together.

**Let's go.**
