# Scenario 5 Defense: Quick Reference Guide (10 min)

**Use this during your presentation to stay on track and handle questions confidently.**

---

## Before You Present: Setup (5 min before)

✅ **Have open:**
1. Scenario-5-10min-Defense-SLIDES.html (in browser, full screen)
2. Scenario-5-Comprehensive-Spec-GFM.md (on second monitor or printed)
3. Your notes (this file)

✅ **Test your tech:**
- Screen sharing works
- Slides visible at size
- Have backup: printed copy of slides

✅ **Mental setup:**
- You're not selling; you're defending thinking
- Coaches want to see: problem understanding, delegation logic, buildability, honesty about unknowns
- Take a breath; you've done the work

---

## The 10-Minute Flow (Strict Timing)

| Minute | Slide | Topic | Key Message |
|:--|:--|:--|:--|
| 0:00–1:30 | 1 | Problem | 180/day, 4 staff, manual, missing things |
| 1:30–3:00 | 2 | Opportunity | Automate mechanical, escalate clinical |
| 3:00–4:30 | 3 | Delegation | 3 modes, each justified |
| 4:30–6:00 | 4 | Buildable | 10 FRs, 4 entities, 3 integrations = ready to code |
| 6:00–7:30 | 5 | Safety | 8 prohibitions, boundary enforced |
| 7:30–9:00 | 6 | Validation | 10 test scenarios, boundary tests included |
| 9:00–10:30 | 7–10 | Close | Assumptions, metrics, readiness, principles |

**If interrupted by questions:**
- Briefly answer (< 1 min)
- Say "Great question, let me note that for Q&A after"
- Keep advancing slides
- Catch up by skipping backup detail on later slides

---

## What Coaches Will Likely Challenge (And Your Defenses)

### Challenge 1: "Why can the agent automate medication changes?"

**Your Defense:**
- "The agent doesn't assess medication changes. It detects them. If a patient reports a change OR if the EHR shows a mismatch, the agent flags it and escalates to clinical staff.
- The agent **never** decides if the change is significant. A clinician does.
- This is AGENT_PLUS_HUMAN_REVIEW, not AGENT_ALONE."

**If they push:** "Look at FR5 in the spec. The agent compares fields deterministically. It doesn't score, interpret, or suppress."

---

### Challenge 2: "What if integrations fail?"

**Your Defense:**
- "Fail closed. Never guess.
- If athenahealth times out, set status=TIMEOUT, escalate to FRONT_DESK, log it. Don't assume 'no data' means 'safe to proceed.'
- If insurance verification fails, escalate. Don't assume coverage is active.
- The case waits for human review or manual override. Readiness is blocked until resolved."

**If they push:** "See FR3 (Insurance) error handling. Every failure is logged and escalated, never silent."

---

### Challenge 3: "Isn't this just 'if X then escalate'? That's not really an agent."

**Your Defense:**
- "It's exactly right that it's deterministic routing. That's the point. An AI agent doesn't need to be a 'smart decider.' It needs to be a reliable router and flagging system.
- The agent frees the front desk from manual repetitive work (sync, dispatch, call integrations, compare fields). That 30–40% workload reduction is real.
- Humans keep the judgment. The agent keeps the process moving. That's a good division."

**If they push:** "The value isn't in the agent 'thinking.' It's in consistency, speed, and auditability. Humans do the same checks; the agent does them 180 times/day without errors."

---

### Challenge 4: "How do you know no clinical judgment leaked in?"

**Your Defense:**
- "Boundary tests V4 and V5 explicitly check for it.
- We scan escalation contexts for prohibited keywords: 'diagnosis,' 'urgency_level,' 'significant,' 'recommend,' 'safe,' 'unsafe.'
- If any keyword found, test fails. We catch it in the build loop before production."

**If they push:** "Look at Scenario V4 (Urgent Phrase). If the system outputs anything like 'this looks moderately urgent' or 'recommend escalation due to severity,' that's a boundary violation we'd catch immediately."

---

### Challenge 5: "What if the clinic doesn't have structured prior-auth rules?"

**Your Defense:**
- "That's Assumption A3, and I've flagged it as MEDIUM confidence, not HIGH.
- If rules don't exist or are tribal knowledge, the design shifts. We escalate all prior-auth questions to FRONT_DESK. The system asks 'rule exists?' If no, escalate.
- We never infer. That's a hard prohibition."

**If they push:** "See Assumptions section of the spec. This is listed as an unknown to validate before build (U3). If we discover post-build that rules aren't structured, we adapt: escalate more, automate less. Safety first."

---

### Challenge 6: "Why 7 years for audit log retention?"

**Your Defense:**
- "That's a placeholder. The actual retention period is clinic-specific and depends on their HIPAA risk assessment and state law. That's Assumption U5 — a compliance question we validate with their privacy officer before build.
- It might be 3 years, 5 years, 7 years, or longer. But it's configurable, and we'll log it based on their policy, not our guess."

**If they push:** "This is exactly why I flagged it as unknown. We don't just assume compliance; we confirm before implementation."

---

### Challenge 7: "What's the happiest-case timeline?"

**Your Defense:**
- "Build: 4–6 weeks for core FRs 1–10 + validation.
- Test: 2 weeks of pilot (10–20 patients) to validate assumptions and baseline metrics.
- Production prep: 2–3 weeks (security review, compliance sign-off, workflow training).
- Go-live: Day 1 might be 30–50 intakes, scale up to 180 over a week."

**If they push:** "It depends on how quickly we validate Assumptions A1–A5 (athenahealth APIs, integration availability, queue setup, prior-auth rules, patient adoption). If those are slow, timeline extends."

---

## Key Phrases to Use (Coaches Like These)

- ✅ **"Fail closed, not open."** (Safety principle)
- ✅ **"Mechanical vs. judgment."** (Delegation clarity)
- ✅ **"Deterministic, not probabilistic."** (Technical rigor)
- ✅ **"Escalate immediately and loudly."** (Safety enforcement)
- ✅ **"The spec is buildable."** (Readiness for coding)
- ✅ **"Honest unknowns."** (Disciplined thinking)
- ✅ **"Audit everything."** (Accountability)
- ✅ **"Route and flag, don't decide."** (Delegation simplicity)

---

## If You Get Off Track (Scenario)

**You:** (realizing you're on wrong slide)
**You say:** "Let me back up and make sure I'm making the point clearly."
**Then:** Click back one slide, re-explain, advance.
**Impact:** Looks deliberate and thoughtful, not panicked.

**If out of order:** "I want to make sure I'm hitting the key defenses in the right sequence. Let me jump to [slide X]."

---

## If Someone Asks "What's Your Biggest Risk?"

**Your Answer:**
"Assumption A3: if prior-auth rules don't exist in structured form at the clinic, that entire automation piece shifts to escalation-based. We'd escalate more, automate less.

Second risk: Assumption A5: if operational queues aren't owned and monitored, escalations pile up unresolved and become useless. That would break the whole system.

Both are flagged and validated before build. If they fail, we adapt the design. We don't just hope they're true."

---

## If Someone Asks "Why Should We Trust This Spec?"

**Your Answer:**
"Three reasons:

1. **It's defensive, not wishful.** I've listed 10 assumptions and said exactly what I'm uncertain about. I'm not hiding doubts.

2. **It's testable.** 10 validation scenarios with concrete pass/fail criteria. Boundary tests verify safety explicitly.

3. **It constrains the agent, not empowers it.** 8 things the agent is explicitly forbidden to do. That's not limitation; that's clarification."

---

## Final Confidence Boost

You have:
- ✅ A spec that's buildable (10 FRs with pseudocode)
- ✅ A delegation boundary that's defensible (3 modes, each justified)
- ✅ Safety guardrails that are testable (8 prohibitions, boundary tests)
- ✅ Honest unknowns (10 assumptions + 8 unknowns, validation plans)
- ✅ Validation that's concrete (10 test scenarios, V4–V5 boundary enforcement)
- ✅ A presentation that's clear (problem → opportunity → build → test → close)

**You're not overselling. You're showing work.**

Coaches will see:
- Serious thinking about delegation
- Deep understanding of where AI is and isn't appropriate
- Clinical safety baked into design, not bolted on
- Honest about unknowns
- Ready for build loop

**You're good. Present with confidence.**

---

## Last-Minute Checklist (1 min before)

- [ ] Slides loaded and visible
- [ ] Spec open on second screen (if available)
- [ ] Voice clear, hydrated
- [ ] Notes in front of you (or memorized key talking points)
- [ ] Time: Start at 0:00, don't talk past 10:00
- [ ] Q&A: Take notes, say "Great question, I'll address that in closing" if tight on time
- [ ] Closing line memorized: "The agent does what's mechanical, escalates what's clinical, and proves everything happened."

---

**You're ready. Go defend this.**
