# 🚀 Scenario 5 Complete Delivery — START HERE

**Status:** ✅ PRODUCTION-READY SPEC + PRESENTATION + BUILD LOOP GUIDE

This is your complete package. Read this file first. Then follow the workflow below.

---

## What You're Getting

**3 Complete Packages in 1:**

### 📋 Package 1: Buildable Specification
- **Scenario-5-Comprehensive-Spec-GFM.md** (50+ pages)
  - 5 sections: Assumptions, Problem, Delegation, Agent Spec, Validation
  - 10 FRs with acceptance criteria
  - 4 entities with full data model
  - 3 integration contracts
  - 10 validation scenarios (V1–V10)
  - 8 hard prohibitions (safety guardrails)

### 🎤 Package 2: 10-Minute Presentation
- **Scenario-5-10min-Defense-SLIDES.html** (interactive slides, live presentation)
- **Scenario-5-10min-Defense-Presentation.md** (markdown version, backup)
- **Scenario-5-SPEAKER-NOTES.md** (word-by-word script, timed 90 sec/slide)
- **Scenario-5-QUICK-REFERENCE-Guide.md** (7 likely challenges + defenses)
- **Scenario-5-Complete-Defense-Package-README.md** (master guide)

### 🔨 Package 3: Closed Build Loop
- **CLOSED-BUILD-LOOP-GUIDE.md** (step-by-step to hand to Claude Code)
- **DIAGNOSIS-REFERENCE-CARD.md** (quick taxonomy reference)
- **MASTER-CHECKLIST-AND-TIMELINE.md** (full workflow + quality gates)

---

## Your Path (Next 48 Hours)

### ⏰ TODAY: Closed Build Loop (2–3 hours)

**9:00 AM – 12:00 PM**

1. Open: `CLOSED-BUILD-LOOP-GUIDE.md` (this is your playbook)
2. Follow Phase 1: "Initial Build Request"
   - Copy entire `Scenario-5-Comprehensive-Spec-GFM.md`
   - Paste into Claude Code
   - Ask Claude to build (data model first, then FRs incrementally)
3. Follow Phase 2–4: Systematic review + diagnosis
   - For each failure, use `DIAGNOSIS-REFERENCE-CARD.md` to classify
   - Apply the right fix (re-prompt vs. rewrite spec vs. fix test)
4. Run all 10 validation scenarios (V1–V10)

**Output:** ✅ Working code + BUILD_SUMMARY.md + ITERATION_LOG.md

**12:00 PM – 1:00 PM: Verify**

- [ ] All 10 validation scenarios passing
- [ ] No clinical judgment in code
- [ ] All hard prohibitions enforced
- [ ] BUILD_SUMMARY.md complete

---

### 🎯 FRIDAY: Gate 1 Presentation (10 minutes + Q&A)

**8:00 AM – 9:00 AM: Prep**

1. Read: `Scenario-5-QUICK-REFERENCE-Guide.md` (20 min)
2. Dry run with slides (10 min, timed)
3. Tech check: Slides load properly (5 min)
4. Mental prep: You've done this work; show it (5 min)

**9:00 AM – 9:10 AM: PRESENT**

- Use: `Scenario-5-10min-Defense-SLIDES.html` (full screen)
- Speak from: `Scenario-5-SPEAKER-NOTES.md` (but natural, not robotic)
- Reference: `Scenario-5-QUICK-REFERENCE-Guide.md` (if challenged)

**9:10 AM – 10:00 AM: Q&A**

- If asked about spec details: Open `Scenario-5-Comprehensive-Spec-GFM.md`, Ctrl+F, cite section
- If asked about build: Reference BUILD_SUMMARY.md + ITERATION_LOG.md, cite taxonomy classification
- If challenged on boundary: Point to Spec § 3 (Delegation Analysis)
- If asked about safety: Point to Spec § 4 (Hard Prohibitions) + Slide 5

---

## File Quick Reference

| Need | File | What to Do |
|:--|:--|:--|
| **To give spec to Claude Code** | Scenario-5-Comprehensive-Spec-GFM.md | Copy entire content; paste into Claude |
| **To run build loop** | CLOSED-BUILD-LOOP-GUIDE.md | Open; follow Phase 1–6 |
| **To diagnose build failure** | DIAGNOSIS-REFERENCE-CARD.md | Answer 3 questions; classify issue |
| **To present** | Scenario-5-10min-Defense-SLIDES.html | Open in browser; full screen |
| **To speak notes** | Scenario-5-SPEAKER-NOTES.md | Read relevant slide (90 sec) |
| **To handle tough Q** | Scenario-5-QUICK-REFERENCE-Guide.md | Find challenge; read 1-min defense |
| **To understand full taxonomy** | spec-ambiguity-vs-builder-mistakes.md | Deep reference; detailed examples |

---

## Key Deliverables (What You Have)

### The Spec (50+ pages, production-ready)
```
§ 1: Assumptions & Unknowns
  - 10 assumptions (A1–A10) with validation discipline
  - 8 unknowns (U1–U8) with test plans
  - All flagged for validation before production

§ 2: Problem Statement & Success Metrics
  - Problem framed from user + business perspective
  - 5 measurable success metrics (95%, 50%, 40%, 0, 100%)
  - 4 hard constraints (non-negotiable)

§ 3: Delegation Analysis (The Core Skill)
  - 3 delegation modes (AGENT_ALONE, AGENT+HUMAN, HUMAN_DECIDES)
  - 20-row matrix showing task → mode → justification
  - Queue assignment rules (FRONT_DESK, CLINICAL_STAFF, PRACTICE_MANAGER)

§ 4: Agent Specification (Buildable)
  - 10 FRs with pseudocode + acceptance criteria + error handling
  - 4 entities with full data model + state machines
  - 3 integration contracts (request/response formats, timeouts, retry logic)
  - 8 hard prohibitions (explicitly forbidden behaviors)

§ 5: Validation Design (10 test scenarios)
  - V1: Happy path
  - V2–V3: Failure modes
  - V4–V5: Boundary tests (verify NO clinical output)
  - V6–V7: Edge cases
  - V8: Concurrency (idempotency)
  - V9: Audit completeness
  - V10: Override + re-evaluation
```

### The Presentation (10 min, 10 slides)
```
Slide 1: The Problem (1:30)
Slide 2: The Opportunity (1:30)
Slide 3: The Delegation Boundary (1:30)
Slide 4: The Spec Is Buildable (1:30)
Slide 5: Safety Guardrails (1:30)
Slide 6: Validation (1:30)
Slide 7–10: Close (Assumptions, Metrics, Readiness, Principles) (1:00)
```

### The Build Loop Playbook
- Phase 1: Initial build request
- Phase 2–4: Systematic review + diagnosis (FRs, entities, integrations)
- Phase 5: Validation scenarios
- Phase 6: Documentation + handoff

---

## Key Numbers (Memorize)

| Metric | Number |
|:--|:--|
| Patients per day | 180 |
| Front-desk staff | 4 |
| Functional requirements | 10 |
| Data entities | 4 |
| Delegation modes | 3 |
| Hard prohibitions | 8 |
| Assumptions to flag | 10 |
| Unknowns to validate | 8 |
| Success metrics | 5 |
| Validation scenarios | 10 |
| Boundary tests | 2 (V4, V5) |
| Presentation time | 10 minutes |

---

## Quality Gates (Checkpoints)

### ✅ Spec Is Ready If:
- [ ] All 5 sections complete
- [ ] All FRs have acceptance criteria
- [ ] All entities fully defined
- [ ] All integration contracts spelled out
- [ ] 10 validation scenarios with concrete pass/fail

### ✅ Build Is Ready If:
- [ ] All 10 FRs implemented
- [ ] All 4 entities defined
- [ ] All 3 integrations working
- [ ] All 10 validation scenarios passing
- [ ] No clinical judgment detected (boundary tests pass)
- [ ] All hard prohibitions enforced

### ✅ Presentation Is Ready If:
- [ ] 10 slides timed to exactly 10 min
- [ ] All key numbers memorized
- [ ] Can cite spec section for any claim
- [ ] Can classify a build failure using taxonomy
- [ ] Dry run completed (timed, natural speaking)

---

## One-Minute Overview (Pitch)

**The Problem:** 180 patients/day, 4 front-desk staff, manual intake. Two failures happen regularly: missed prior auths and unreviewed medication changes.

**The Solution:** Automate what's mechanical (appointment sync, form validation, integration calls). Keep humans in charge of what requires judgment (medication significance, allergy assessment, urgency determination).

**The Boundary:** Three delegation modes, each justified. AGENT_ALONE on deterministic tasks. AGENT+HUMAN on safety-sensitive detection. HUMAN_DECIDES on clinical judgment.

**The Safety:** 8 hard prohibitions (no diagnosis, no triage, no judgment). Boundary tests verify no clinical output leaked in. Deterministic readiness gate prevents incomplete intakes.

**The Validation:** 10 test scenarios covering happy path, failures, edges, concurrency, audit, override.

**The Unknowns:** 10 assumptions flagged, 8 unknowns identified, all with validation plans before build.

---

## The Three Secrets to Gate 1 Success

### 1. Delegation Is Defensible
You don't just say "agent does X, human does Y." You explain **why** — the properties of the task itself justify the boundary.

**Coaches look for this.** Every boundary must be justified by task properties, not arbitrary.

### 2. The Spec Is Buildable
No vague words. No "if needed" or "as appropriate." Every requirement has acceptance criteria. Every integration has request/response format. Every error path is explicit.

**Coaches look for this.** Can an AI coding agent build from this without clarifying questions?

### 3. Unknowns Are Honest
You don't hide assumptions. You name them, explain them, and say what you'd do if they fail.

**Coaches look for this.** "I don't know" + validation plan beats "probably works" + silent hope.

---

## Emergency Checklist (30 min Before Gate)

- [ ] Slides load in browser (full screen, no errors)
- [ ] Speaker notes visible (printed or second screen)
- [ ] Spec open in second tab (for Q&A)
- [ ] Memorized: 10 minutes, 10 slides, 1:30 per slide
- [ ] Memorized: The 3 delegation modes and why
- [ ] Memorized: The 8 hard prohibitions
- [ ] Memorized: The 2 boundary tests (V4, V5)
- [ ] Memorized: Closing line ("The agent does what's mechanical, escalates what's clinical, and proves everything happened")
- [ ] Read quick-ref challenges one more time
- [ ] Breathe (you've done this work; show it)

---

## What Happens at Gate 1

**You present 10 min.** Coaches listen.

**They ask questions (30–60 min).**
- "Why is this boundary here?" → You cite Spec § 3
- "How do you know this works?" → You show validation scenarios
- "What if [assumption] fails?" → You explain the fallback
- "Why is this not clinical judgment?" → You show hard prohibition + boundary test

**They score you** (rubric is sealed, but they're looking for the 3 secrets above).

**You move forward** (same spec goes to peer review Friday; potential real Gate 1 next Wednesday).

---

## After Gate 1

**If you pass:**
- Validate the 8 unknowns with the clinic
- Collect baseline metrics
- Prepare pilot (10–20 patients)
- Prepare for Friday peer review

**If you get feedback:**
- Update the spec based on feedback
- Re-build if needed
- Re-present

**Either way:**
- You've completed Week 1 practice
- You've learned disciplined spec writing + delegation analysis
- You're ready for real scenarios in Weeks 2–5

---

## Files in This Package

**Core Files (You Need These)**
```
✅ Scenario-5-Comprehensive-Spec-GFM.md
✅ Scenario-5-10min-Defense-SLIDES.html
✅ Scenario-5-SPEAKER-NOTES.md
✅ CLOSED-BUILD-LOOP-GUIDE.md
✅ DIAGNOSIS-REFERENCE-CARD.md
```

**Supporting Files (Reference)**
```
✅ Scenario-5-10min-Defense-Presentation.md
✅ Scenario-5-QUICK-REFERENCE-Guide.md
✅ Scenario-5-Complete-Defense-Package-README.md
✅ MASTER-CHECKLIST-AND-TIMELINE.md
✅ spec-ambiguity-vs-builder-mistakes.md (pre-existing)
```

**Backup Slides & Variations**
```
- scn-5-ShortenedSpec-gfm.md
- scn-5-problem-statement-assumed-success-metrics-gfm.md
- scn-5-delegation-analysis-gfm.md
- scn-5-assumptions-unknowns-gfm.md
- etc. (from prior iterations)
```

---

## How to Use This Package

### Scenario A: You Have 2 Hours Before Gate
1. Skip the build loop (save for after Gate if time allows)
2. Open `Scenario-5-10min-Defense-SLIDES.html`
3. Read `Scenario-5-SPEAKER-NOTES.md` (all 10 slides)
4. Do dry run (timed)
5. Read `Scenario-5-QUICK-REFERENCE-Guide.md` (challenges)
6. Breathe. You're ready.

### Scenario B: You Have 1 Day Before Gate (Best Path)
1. Run the build loop (2–3 hours) → output: working code + BUILD_SUMMARY.md
2. Review build results (30 min)
3. Practice presentation (1 hour)
4. Sleep well
5. Present confidently

### Scenario C: You Have 1 Week (Ideal)
1. Build loop today (2–3 hours)
2. Iterate on feedback (1–2 days)
3. Finalize presentation (1 day)
4. Peer feedback (1 day)
5. Final polish (1 day)
6. Gate confident and ready

---

## Success Criteria

**You succeed at Gate 1 if coaches see:**

✅ Clear, defensible delegation boundaries  
✅ Buildable specification (no ambiguity)  
✅ Honest unknowns (not hidden assumptions)  
✅ Safety guardrails (8 prohibitions, boundary tests)  
✅ Complete validation (10 scenarios, edge cases)  
✅ Evidence of disciplined iteration (build loop story with taxonomy)  
✅ Prepared for questions (you cite spec § X confidently)  

---

## Final Confidence Boost

You have everything you need. This package represents:

- ✅ 50+ pages of buildable specification writing
- ✅ 10 slides of clear, compelling presentation
- ✅ Word-by-word scripts (90 sec each)
- ✅ Challenge defenses for 7 likely questions
- ✅ Complete build loop playbook + diagnosis guide
- ✅ Quality gates at every checkpoint
- ✅ Taxonomy to classify any build failure

**No participant enters Gate 1 better prepared than you.**

**You have 48 hours. Execute the plan. Show your thinking. Land the gate.**

---

## Need Help?

| Question | Answer Here |
|:--|:--|
| "How do I run the build loop?" | CLOSED-BUILD-LOOP-GUIDE.md (Phases 1–6) |
| "What if Claude Code fails?" | DIAGNOSIS-REFERENCE-CARD.md + classify using taxonomy |
| "What if I get a tough question?" | Scenario-5-QUICK-REFERENCE-Guide.md (7 defenses) |
| "What's the exact spec for FR5?" | Scenario-5-Comprehensive-Spec-GFM.md § 4, FR5 |
| "How do I know I'm ready?" | MASTER-CHECKLIST-AND-TIMELINE.md (quality gates) |
| "What's the closing line?" | Scenario-5-SPEAKER-NOTES.md § Closing |

---

## You're Ready

Everything you need is in this folder.

**Now execute:**

1. ✅ Run the build loop (2–3 hours)
2. ✅ Verify all tests pass (30 min)
3. ✅ Practice the presentation (1 hour)
4. ✅ Answer the challenge questions (15 min)
5. ✅ Show up Friday morning confident

**Gate 1 is yours.**

---

**🚀 Go build this. Go present this. Go pass this.**

**You've got this.**
