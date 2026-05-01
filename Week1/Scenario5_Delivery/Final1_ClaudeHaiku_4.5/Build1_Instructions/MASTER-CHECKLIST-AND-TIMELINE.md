# Scenario 5 Complete Delivery Package — Master Checklist

**Status:** 🟢 READY FOR CLOSED BUILD LOOP & PRESENTATION

This document ties together all deliverables and provides the complete workflow.

---

## What You Have (Complete File List)

### Specification & Design Documents

| File | Purpose | When to Use |
|:--|:--|:--|
| **Scenario-5-Comprehensive-Spec-GFM.md** | Full buildable spec (50+ pages) | Hand to Claude Code; reference during diagnosis |
| **Scenario-5-10min-Defense-Presentation.md** | Slides as text with talking points | Script reference; backup version |
| **Scenario-5-10min-Defense-SLIDES.html** | Interactive visual slides (10 slides) | Live presentation; screen sharing |
| **Scenario-5-Complete-Defense-Package-README.md** | Master guide to presentation files | Before presenting (orientation) |
| **Scenario-5-SPEAKER-NOTES.md** | Word-by-word script + timing | Rehearsal; speaking confidence |
| **Scenario-5-QUICK-REFERENCE-Guide.md** | Likely challenges + defenses | Last 10 min before presenting |

### Build & Testing Guides

| File | Purpose | When to Use |
|:--|:--|:--|
| **CLOSED-BUILD-LOOP-GUIDE.md** | Step-by-step for Claude Code → build | Run this after spec is final |
| **DIAGNOSIS-REFERENCE-CARD.md** | Quick taxonomy reference during build | When tests fail; classify issues |
| **spec-ambiguity-vs-builder-mistakes.md** | (Pre-existing) Full taxonomy doc | Deep reference; detailed examples |

---

## Your Timeline (Next 48 Hours)

### Thursday (Today / Now)

**9:00 AM – 12:00 PM: Closed Build Loop**
1. Open `CLOSED-BUILD-LOOP-GUIDE.md` (this is your playbook)
2. Open `Scenario-5-Comprehensive-Spec-GFM.md` (give to Claude Code)
3. Follow Phase 1 (initial build request) → Phase 6 (documentation)
4. Time: 2–3 hours

**Output:**
- ✅ Working code (4 entities, 10 FRs, 3 integrations)
- ✅ All 10 validation scenarios passing
- ✅ BUILD_SUMMARY.md documenting what was built
- ✅ ITERATION_LOG.md documenting each fix (with taxonomy classification)

**12:00 PM – 1:00 PM: Review Build Results**
- Read BUILD_SUMMARY.md
- Verify all tests pass (10/10 ✅)
- Check that no clinical judgment leaked into code
- Confirm all hard prohibitions enforced

---

### Friday (Gate 1 Presentation Day)

**8:00 AM – 9:00 AM: Presentation Prep**

1. Read `Scenario-5-QUICK-REFERENCE-Guide.md` (20 min)
   - Refresh 7 likely challenges + your defenses
   - Memorize closing line
   
2. Dry run (10 min)
   - Open slides in browser (full screen)
   - Speak aloud through all 10 slides
   - Time yourself (should be 10 min ±1 min)
   
3. Mental prep (10 min)
   - You've done serious work
   - Coaches will see it
   - Answer questions confidently using taxonomy
   - Breathe

4. Tech check (10 min)
   - Slides load properly
   - Have backup: PDF or printed copy
   - Spec open on second screen

**9:00 AM – 9:10 AM: GATE 1 PRESENTATION**

Use `Scenario-5-10min-Defense-SLIDES.html` (full screen).
Speak from `Scenario-5-SPEAKER-NOTES.md` (but don't read it out loud).
Reference `Scenario-5-QUICK-REFERENCE-Guide.md` if challenged.

**After presentation (30–60 min): Q&A / Deep Dive**

Coaches will ask detailed questions. Use this strategy:
1. If they ask about assumptions: "See Spec § 1, Assumptions A1–A10"
2. If they ask about validation: "See Spec § 5, Scenarios V1–V10"
3. If they ask about build decisions: "See ITERATION_LOG.md, where we classified each fix using the taxonomy"
4. If they ask about clinical boundary: "See Hard Prohibitions § 5; boundary tests V4 and V5 verify it"

---

## File Usage Matrix (Quick Reference)

| Situation | Open File | Then Do |
|:--|:--|:--|
| **Want to give spec to Claude Code** | Scenario-5-Comprehensive-Spec-GFM.md | Copy entire content; paste into Claude |
| **Want to diagnose a build failure** | DIAGNOSIS-REFERENCE-CARD.md | Answer 3 questions; classify issue; choose fix |
| **Want to understand full diagnosis process** | CLOSED-BUILD-LOOP-GUIDE.md | Sections 2.1–2.4 (systematic review) |
| **Preparing to present (1 hour before)** | Scenario-5-QUICK-REFERENCE-Guide.md | Read challenges + defenses; boost confidence |
| **Presenting now (live)** | Scenario-5-10min-Defense-SLIDES.html | Full screen in browser |
| **Need exact words for each slide** | Scenario-5-SPEAKER-NOTES.md | Read relevant slide section (90 sec each) |
| **During Q&A about hard requirements** | Scenario-5-Comprehensive-Spec-GFM.md | Ctrl+F for keyword; read relevant section aloud |
| **Need to explain a build decision** | BUILD_SUMMARY.md + ITERATION_LOG.md | Show what issue was found, how it was classified, what was fixed |

---

## Key Numbers (Memorize These)

| Metric | Number | Why It Matters |
|:--|:--|:--|
| **Patients/day** | 180 | Scale of the problem |
| **Front-desk staff** | 4 | Workload pain point |
| **Physicians** | 6 | Stakeholder count |
| **Locations** | 2 | Distributed system |
| **Common failures** | 2 (expired prior auths, med changes) | Core problem solved |
| **Assumptions flagged** | 10 (A1–A10) | Honest risk register |
| **Unknowns to validate** | 8 (U1–U8) | Before-build requirements |
| **Functional requirements** | 10 (FR1–FR10) | Scope of build |
| **Data entities** | 4 (Appointment, IntakeWorkItem, EscalationEvent, AuditLog) | Data model |
| **Integration contracts** | 3 (athenahealth, insurance, prior-auth) | External dependencies |
| **Delegation modes** | 3 (AGENT_ALONE, AGENT+HUMAN, HUMAN_DECIDES) | Boundary structure |
| **Hard prohibitions** | 8 | Safety guardrails |
| **Success metrics** | 5 | Measurement framework |
| **Validation scenarios** | 10 (V1–V10) | Test coverage |
| **Boundary tests** | 2 (V4, V5) | Safety verification |
| **Readiness conditions** | 7 | READY_FOR_VISIT gate |
| **Escalation queues** | 3 (FRONT_DESK, CLINICAL_STAFF, PRACTICE_MANAGER) | Escalation routing |
| **Presentation minutes** | 10 | Gate 1 time limit |

---

## Taxonomy Shortcuts (For Diagnosis)

When you hit a build issue, use this:

```
BUILDER MISREAD signals:
- Agent ignored explicit spec constraint
- Code contradicts clear spec text
- Agent used wrong default
- Agent missed required field
→ FIX: Re-prompt with relevant section highlighted

SPEC AMBIGUITY signals:
- Agent's interpretation is defensible
- Spec uses fuzzy words
- Two smart people could disagree
- Your intent ≠ agent's interpretation
→ FIX: Rewrite spec to remove ambiguity

DESIGN GAP signals:
- Spec is silent on something critical
- Code built exactly what was asked for but is incomplete
- Something "production-obvious" is missing
→ FIX: Add requirement to spec; rebuild

TEST PROBLEM signals:
- Code matches spec
- Test checks brittle values
- Test makes assumptions not in spec
→ FIX: Update test (not code)
```

---

## Red Lines (Don't Cross These)

🚫 **Hard Prohibitions** — If code violates any of these, it fails immediately:

1. ❌ Code diagnoses or infers diagnoses
2. ❌ Code performs clinical triage
3. ❌ Code recommends treatment
4. ❌ Code assesses medication significance
5. ❌ Code assesses allergy severity
6. ❌ Code infers prior-auth without rule
7. ❌ Code suppresses escalations
8. ❌ Code fabricates data on failure

**If any violation is found:**
- Classify as: Hard Prohibition Violation (subcategory of Builder Misread)
- Re-prompt immediately
- Include spec section that forbids this behavior

---

## Gates (Quality Checkpoints Before Proceeding)

### Before Handing to Claude Code
- [ ] Spec is complete (all 5 sections: assumptions, problem, delegation, agent spec, validation)
- [ ] Spec has acceptance criteria for each FR
- [ ] Spec lists all prohibited behaviors
- [ ] Spec includes 10 validation scenarios

→ **If not all ✅: Don't proceed. Refine spec first.**

### After Initial Build (Phase 1)
- [ ] Data model matches Spec § 4 exactly
- [ ] No clinical content in data model
- [ ] All state machines defined correctly

→ **If not all ✅: Re-prompt before moving to FRs.**

### After FR1–FR3 Implementation
- [ ] FR1 idempotent (no duplicates)
- [ ] FR2 handles deadline correctly
- [ ] FR3 fails closed (no guessing on integration failure)

→ **If not all ✅: Re-prompt with examples.**

### After FR4–FR7 Implementation (Safety-Critical)
- [ ] FR4 never infers prior-auth (always consults rules)
- [ ] FR5 never scores medication significance
- [ ] FR6 never assesses allergy severity
- [ ] FR7 never performs clinical triage

→ **If not all ✅: Hard prohibition violation. Re-prompt immediately.**

### Before Final Validation Run
- [ ] All 10 FRs implemented
- [ ] All 3 integrations spelled out
- [ ] All 4 entities defined
- [ ] All audit logging in place

→ **If not all ✅: Add missing components.**

### Final Test Run
- [ ] V1–V10 all passing
- [ ] Boundary tests (V4, V5) verify NO clinical output
- [ ] Idempotency test (V8) verified
- [ ] Audit test (V9) verified

→ **If not all ✅: Diagnose and fix failures before Gate 1.**

---

## The Narrative You'll Tell Coaches

**Opening (Slide 1–2):**
"The practice sees 180 patients/day with 4 staff doing intake manually. Two things fail regularly: expired prior authorizations and unreviewed medication changes. We're solving this by automating what's mechanical and keeping humans in charge of what requires judgment."

**Middle (Slide 3–6):**
"The delegation is clear: three modes. Agent works alone on deterministic tasks. Agent flags + human reviews on safety-sensitive detection. Humans keep all clinical decisions. The spec is buildable: 10 FRs, 4 entities, 3 integrations. Every requirement has acceptance criteria. We've tested 10 scenarios including boundary tests that verify no clinical judgment leaked in."

**Close (Slide 10):**
"Three principles: fail closed (escalate ambiguity, never guess), route and flag (agent is a router, not a clinician), audit everything (every action logged, immutable). This isn't just a spec; it's a safety boundary written into code."

**If asked about build:**
"We ran a closed loop with Claude Code. Any failures were classified using the spec-ambiguity-vs-builder-mistakes taxonomy. Each re-prompt fixed a specific category issue. No hard prohibitions were violated. All 10 test scenarios passing."

---

## Success Criteria for Gate 1

You pass Gate 1 if coaches see:

✅ **Clear delegation boundaries** (Spec § 3 defends each mode)  
✅ **Buildable spec** (10 FRs with acceptance criteria, integration contracts defined)  
✅ **Honest unknowns** (10 assumptions + 8 unknowns with validation plans)  
✅ **Safety guardrails** (8 prohibitions, boundary tests, deterministic readiness)  
✅ **Complete validation** (10 scenarios, edge cases, failure modes, concurrency)  
✅ **Disciplined iteration** (build loop shows classification of each fix)  
✅ **Prepared for questions** (you can cite spec § X for any claim)  

---

## Quick Pre-Gate Confidence Check

Answer these 5 questions:

1. **Can you explain in 1 sentence why the delegation boundary is at AGENT_ALONE vs. AGENT+HUMAN vs. HUMAN_DECIDES?**
   - Yes → ✅
   - No → Review Spec § 3 + Slide 3

2. **Can you point to the spec section that proves each of the 8 hard prohibitions are enforced?**
   - Yes → ✅
   - No → Review Spec § 4 (end) + Slide 5

3. **Can you explain what tests V4 and V5 do and why they verify safety?**
   - Yes → ✅
   - No → Review Spec § 5 (V4, V5) + Slide 6

4. **Can you list 3 assumptions and explain what you'd do if each one failed?**
   - Yes → ✅
   - No → Review Spec § 1 (Assumptions A1–A10)

5. **Can you describe a build failure, classify it by taxonomy, and explain the fix?**
   - Yes → ✅
   - No → Review DIAGNOSIS-REFERENCE-CARD.md

**If all 5 ✅: You're ready.**  
**If <5: Spend 15 min on the gap. You'll be fine.**

---

## Emergency Resources (If Things Go Wrong)

| Problem | Resource | What to Do |
|:--|:--|:--|
| Claude Code produces bad code | DIAGNOSIS-REFERENCE-CARD.md | Classify issue; re-prompt correctly |
| You get defensive questions at Gate | QUICK-REFERENCE-Guide.md | Find challenge; read 1-min defense |
| You need to explain a delegation choice | Spec § 3 + Slide 3 | Quote the justification |
| You need to prove no clinical judgment | Spec § 5 (V4, V5) | Show boundary tests |
| You need to prove safety | Spec § 4 (Hard Prohibitions) + Slide 5 | Show the 8 prohibitions |
| You need to prove buildability | Spec § 4 (FRs 1–10) + Slide 4 | Show acceptance criteria |
| You got behind on timing | Scenario-5-SPEAKER-NOTES.md | Compress later slides; land on closing strong |

---

## Checklist: Ready to Present?

**Spec:**
- [ ] Comprehensive-Spec-GFM.md is complete and built
- [ ] All 10 FRs have acceptance criteria
- [ ] All hard prohibitions listed
- [ ] All 10 validation scenarios described

**Build:**
- [ ] Claude Code implementation complete
- [ ] All 10 validation scenarios passing
- [ ] Boundary tests (V4, V5) verify no clinical output
- [ ] BUILD_SUMMARY.md documents what was built

**Presentation:**
- [ ] Slides load properly (HTML or PDF)
- [ ] Speaker notes memorized (10 min timing)
- [ ] Quick-ref guide opened (challenge defenses)
- [ ] Spec open on second screen (for Q&A)

**Confidence:**
- [ ] Can explain delegation in 1 sentence per mode
- [ ] Can cite spec section for any claim
- [ ] Can classify a build failure using taxonomy
- [ ] Have gone through dry run (timed, full script)

**All ✅?** → **You're ready. Go present.**

---

## After Gate 1

Whether you pass or get feedback:

**If approved:**
1. Validate the 8 unknowns (U1–U8) with the clinic
2. Collect baseline metrics for success measurement
3. Prepare pilot plan (10–20 patients)
4. Prepare for Friday peer review (same spec, different reviewers)

**If asked for revisions:**
1. Note specific feedback
2. Determine which component(s) need work (delegation? assumptions? validation?)
3. Update the spec
4. Re-build if needed
5. Re-present

**Either way:**
- You've completed Week 1 practice scenario
- You've learned to write a buildable spec
- You've practiced disciplined iteration
- You're ready for the real Gate 1 scenario next Wednesday

---

## Final Motivational Note

You have:

✅ A comprehensive, defensible spec (50+ pages of thinking)  
✅ A clear 10-minute presentation (10 slides, tight script)  
✅ Complete guidance for building (closed build loop playbook)  
✅ Safety guardrails (8 prohibitions, boundary tests)  
✅ Honest unknowns (10 assumptions, 8 unknowns, validation plans)  
✅ Methodology for diagnosing build issues (taxonomy + reference card)  

**This is not a practice submission. This is production-grade thinking.**

Coaches will see:
- Serious delegation analysis
- Buildable specification writing
- Clinical boundary discipline
- Honest risk assessment
- Systematic validation approach
- Iteration rigor

**You're more prepared than 80% of Week 1 participants will be.**

Go present this. Defend it confidently. You've done the work.

---

**Everything you need is in this folder. You're ready.**

**🚀 Go execute.**
