# Scenario 5 Complete Defense Package — File Index & How to Use

You now have **everything** needed to defend your Scenario 5 solution in 10 minutes and answer detailed questions afterward.

---

## Files Created (in Scenario5_Delivery folder)

### 1. **Scenario-5-Comprehensive-Spec-GFM.md** (MAIN SPEC)
- **What it is:** Full 50+ page buildable specification
- **Use it for:** Production reference, backup details, exact acceptance criteria
- **How to use in presentation:** Open on second screen; reference specific sections when challenged
- **Key sections:**
  - § 1: 10 Assumptions + 8 Unknowns (validation discipline)
  - § 2: Problem Statement & Success Metrics (why it matters)
  - § 3: Delegation Analysis (defensible boundaries)
  - § 4: Agent Specification (buildable, precise)
  - § 5: Validation Design (10 test scenarios)

---

### 2. **Scenario-5-10min-Defense-Presentation.md** (TEXT VERSION)
- **What it is:** 10 slides as markdown with talking points
- **Use it for:** Script reference, backup speaker notes, print for handouts
- **How to use:** Read the "What You Say" section for each slide during presentation

---

### 3. **Scenario-5-10min-Defense-SLIDES.html** (VISUAL SLIDES)
- **What it is:** Interactive HTML presentation (10 slides, timed, color-coded)
- **Use it for:** Live presentation, screen sharing
- **How to use:**
  1. Open in browser (Chrome, Firefox, Safari all work)
  2. Go full screen (F11 or Cmd+Ctrl+F)
  3. Click through slides or use arrow keys
  4. Print to PDF if needed (for backup or archiving)

**Printing to PowerPoint:**
- Option A: Print HTML to PDF, then import images into PowerPoint
- Option B: Copy slide content from HTML and paste into PowerPoint manually
- Option C: Use directly in browser (simpler, fewer technical failures)

---

### 4. **Scenario-5-QUICK-REFERENCE-Guide.md** (BEFORE YOU PRESENT)
- **What it is:** Compact reference with likely challenges and your defenses
- **Use it for:** Last 10 minutes before presentation, during Q&A
- **Key content:**
  - 7 likely coach challenges + your 1-minute defense for each
  - Key phrases coaches like ("Fail closed, not open")
  - Handling off-track moments
  - Confidence boost section

---

### 5. **Scenario-5-SPEAKER-NOTES.md** (DETAILED TALKING POINTS)
- **What it is:** Full word-for-word script for each slide
- **Use it for:** Rehearsal, exact pacing (90 seconds/slide), confidence building
- **Key content:**
  - What you say for each slide (word-by-word)
  - Tone guidance (empathetic, confident, analytical, etc.)
  - Timing breakdowns
  - Q&A strategies
  - Backup examples if you get questions

---

## 10-Minute Presentation Flow

**0:00–1:30 | SLIDE 1:** The Problem
- 180 intakes/day, 4 staff, manual process
- Two failures: missed prior auths, medication changes

**1:30–3:00 | SLIDE 2:** The Opportunity
- Automate mechanical (form checks, integration calls)
- Escalate clinical (significance, urgency)

**3:00–4:30 | SLIDE 3:** The Delegation Boundary
- AGENT_ALONE (7 tasks) – deterministic
- AGENT + HUMAN (8 tasks) – detection + review
- HUMAN_DECIDES (5 tasks) – clinical only

**4:30–6:00 | SLIDE 4:** The Spec Is Buildable
- 10 FRs with pseudocode + acceptance criteria
- 4 data entities with state machines
- 3 integration contracts (JSON examples)

**6:00–7:30 | SLIDE 5:** Safety Guardrails
- 8 hard prohibitions (no diagnosis, no triage, etc.)
- Boundary tests enforce compliance

**7:30–9:00 | SLIDE 6:** Validation
- 10 test scenarios (happy path, failures, edges, concurrency, audit, override)
- Boundary tests verify NO clinical output

**9:00–10:30 | SLIDES 7–10:** Close
- 10 assumptions + 8 unknowns (honest risk register)
- 5 success metrics (specific targets, not vague)
- Readiness discipline (7 conditions, all must be TRUE)
- 3 core principles (Fail Closed, Route & Flag, Audit Everything)

---

## How to Prepare (in Order)

### **72 Hours Before**
1. Read `Scenario-5-SPEAKER-NOTES.md` word-by-word (30 min)
2. Read `Scenario-5-Comprehensive-Spec-GFM.md` sections 1–3 (45 min)
3. Time yourself reading speaker notes aloud (should be ~10 min, not more)

### **24 Hours Before**
1. Read `Scenario-5-QUICK-REFERENCE-Guide.md` (20 min)
2. Drill the 7 likely challenges + your defenses (15 min)
3. Open `Scenario-5-10min-Defense-SLIDES.html` in browser, go full screen, practice (10 min)

### **1 Hour Before**
1. Open both files:
   - HTML slides on main screen (full screen)
   - Speaker notes on second screen (or printed)
2. Walk through once, no talking (2 min)
3. Walk through with talking (10 min)
4. Deep breath. You're ready.

### **Immediately Before**
1. Check browser is full screen, slides load properly
2. Bathroom break
3. Have backup: printed slides or PDF open
4. Read the closing line aloud: "The agent does what's mechanical, escalates what's clinical, and proves everything happened."
5. Start.

---

## Handling Different Presentation Formats

### **If You're Screen Sharing**
- Use `Scenario-5-10min-Defense-SLIDES.html` full screen
- Have speaker notes on second monitor (read, don't just click)
- Have the spec PDF open in background in case of deep questions

### **If You're Printing**
- Print `Scenario-5-10min-Defense-SLIDES.html` to PDF (print to PDF option in browser)
- Print `Scenario-5-SPEAKER-NOTES.md` as backup notes
- Bring color printouts if possible (helps readability)

### **If You're Presenting in Person (Projector)**
- Load HTML in browser on laptop connected to projector
- Have speaker notes on your laptop screen only (use Presenter Mode if available)
- Backup: Have printed slides and notes on table

### **If Interrupted or Asked to Deep-Dive**
- Have `Scenario-5-Comprehensive-Spec-GFM.md` open
- Say "Great question, let me show you exactly where we address that in the spec"
- Find the section (use Ctrl+F), read the relevant passage
- Resume presentation

---

## What Coaches Are Looking For (And How Your Package Addresses It)

| Coach Expectation | Your Deliverable | Where to Find It |
|:--|:--|:--|
| **Defensible delegation boundaries** | Slide 3 + Spec § 3 | SLIDES.html + Comprehensive-Spec-GFM.md § 3 |
| **Spec is buildable (no ambiguity)** | Slide 4 + Spec § 4 | SPEAKER-NOTES.md + Comprehensive-Spec-GFM.md § 4 |
| **Honest unknowns (5+ genuine)** | Slide 7 + Spec § 1 | QUICK-REFERENCE-Guide.md + Comprehensive-Spec-GFM.md § 1 |
| **Problem framing** | Slide 1 + Spec § 2 | SPEAKER-NOTES.md + Comprehensive-Spec-GFM.md § 2 |
| **Success metrics** | Slide 8 + Spec § 2 | SPEAKER-NOTES.md + Comprehensive-Spec-GFM.md § 2 |
| **Edge cases + failure modes** | Slide 6 + Spec § 5 | SPEAKER-NOTES.md (V2–V5) + Comprehensive-Spec-GFM.md § 5 |
| **Boundary enforcement** | Slide 5 + Slide 6 | SPEAKER-NOTES.md (V4–V5) + Comprehensive-Spec-GFM.md § 5 (V4–V5) |
| **Response to challenges** | Quick-Ref guide | Scenario-5-QUICK-REFERENCE-Guide.md |

---

## Backup Answers for Tricky Questions

### **"How do you know this works?"**
- Reference Slide 6 (validation)
- Open `Scenario-5-Comprehensive-Spec-GFM.md` § 5 (Validation Design)
- Show V1 (happy path), V4 (boundary test), V9 (audit test)

### **"What if [assumption] fails?"**
- Open `Scenario-5-Comprehensive-Spec-GFM.md` § 1
- Show the specific assumption (A1–A10)
- Explain the validation method and what we do if it fails
- Show in QUICK-REFERENCE-Guide.md "Challenge 5: What if rules don't exist?"

### **"Why not just automate X?"**
- Reference Slide 5 (8 hard prohibitions)
- Or Slide 3 (delegation modes)
- Explain the boundary: "That's clinical judgment, so it stays human"

### **"What's your success metric?"**
- Reference Slide 8 (5 metrics, specific targets)
- Emphasize: "Not vague ('better'), but specific (95%, 50%, 40%)"

### **"What's your risk?"**
- Reference QUICK-REFERENCE-Guide.md "If Someone Asks 'What's Your Biggest Risk?'"
- Answer: "A3 (prior-auth rules) and A5 (operational queues). Both flagged for validation before build."

---

## Presentation Troubleshooting

| Problem | Solution |
|:--|:--|
| HTML slides won't load | Download HTML file locally, open in Chrome |
| Screen share lag | Use PDF printout instead; share that |
| Lost your place mid-presentation | Say "Let me make sure I'm making this point clearly" and back up |
| Coach interrupts early | Acknowledge, say "I'll address that in a moment," keep advancing |
| Coach asks very detailed Q&A | Open spec on second screen, find exact section, quote it |
| You stumble on a sentence | Keep going; fluency beats perfection. Recover on the next point. |
| Timer cuts you off at 10 min | Land on Slide 10 (closing principles) even if rushed. They remember the ending. |
| No second monitor available | Have speaker notes printed on paper on the table |

---

## The Narrative Arc (What Story You're Telling)

1. **Problem:** Intake is manual, mistakes happen, people are suffering.
2. **Insight:** Most intake is mechanical; some needs judgment. Separate them.
3. **Boundaries:** Three delegation modes, each justified.
4. **Spec:** This is buildable, tested, and safe.
5. **Principles:** Fail closed. Route and flag. Audit everything.
6. **Close:** We've thought through the hard parts. We're ready.

**Coaches will hear:** "This person understands delegation. They've thought about safety. They're honest about unknowns. They can build this."

---

## Things to AVOID Saying

- ❌ "The agent will handle everything"
- ❌ "This will solve all intake problems"
- ❌ "It's basically magic"
- ❌ "We'll figure it out as we go"
- ❌ "Hopefully the integrations work"
- ❌ "If X happens, we'll decide then"
- ❌ "The agent is pretty smart"

## Things to DO Say

- ✅ "The agent automates mechanical tasks"
- ✅ "Humans stay in control of clinical decisions"
- ✅ "We've validated these assumptions, and here's the fallback if they fail"
- ✅ "This is testable in code"
- ✅ "Safety is enforced by logic, not by hope"
- ✅ "We've thought through edge cases"
- ✅ "Here's where we need to confirm before build"

---

## After You Present: Next Steps

**If approved:**
1. Get stakeholder sign-off on assumptions (U1–U8)
2. Run closed build loop with Claude Code
3. Use `spec-ambiguity-vs-builder-mistakes.md` to classify any mismatches
4. Iterate until validation scenarios pass

**If asked for revisions:**
1. Note the feedback
2. Update `Scenario-5-Comprehensive-Spec-GFM.md` (primary source of truth)
3. Update slides and speaker notes accordingly
4. Re-present (use same 10-minute structure)

**If not approved:**
1. Ask for specific feedback
2. Determine if it's: delegation boundary unclear, assumptions not validated, spec ambiguous, or safety boundary loose
3. Address root cause, not just symptoms
4. Prepare for Friday peer review with updated spec

---

## File Manifest (Print This)

```
Scenario5_Delivery/
├── Scenario-5-Comprehensive-Spec-GFM.md ........... MAIN SPEC (production reference)
├── Scenario-5-10min-Defense-Presentation.md ....... TEXT SLIDES (markdown, 10 slides)
├── Scenario-5-10min-Defense-SLIDES.html ........... VISUAL SLIDES (interactive, live presentation)
├── Scenario-5-QUICK-REFERENCE-Guide.md ........... CHALLENGES + DEFENSES (backup Q&A)
├── Scenario-5-SPEAKER-NOTES.md ................... FULL SCRIPT (word-by-word, timed)
└── Scenario-5-Complete-Defense-Package-README.md (this file)
```

---

## Final Checklist (5 min before you go on)

- [ ] Slides loaded and visible
- [ ] Speaker notes in front of you (screen or printed)
- [ ] Spec open on second screen (or backup PDF on laptop)
- [ ] Timer ready (phone stopwatch or slide timer)
- [ ] Breathing: slow, calm
- [ ] Closing line memorized: "The agent does what's mechanical, escalates what's clinical, and proves everything happened."
- [ ] Confidence: HIGH (you've done the work)

---

## You're Ready

You have:
- ✅ A comprehensive, buildable spec
- ✅ A 10-minute presentation that hits all the right points
- ✅ Speaker notes (word-for-word script)
- ✅ Quick reference for likely challenges
- ✅ Backup answers for deep questions
- ✅ A narrative arc that lands

**Go present this. Show the thinking. Answer the questions. Land the defense.**

**You've got this.**

---

**Questions or last-minute edits?** Everything is in markdown or HTML. Edit and re-export as needed.

**Want to customize the slides?** Edit the HTML file directly (colors, wording, examples) or copy content into PowerPoint.

**Nervous?** Re-read the "Final Confidence Boost" section in QUICK-REFERENCE-Guide.md. Then present.
