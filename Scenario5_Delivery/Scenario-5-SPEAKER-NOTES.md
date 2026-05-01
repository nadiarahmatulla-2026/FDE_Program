# Scenario 5 Defense: Full Speaker Notes (What You Say)

**Read these for each slide. Time yourself. Aim for exactly 90 seconds per slide.**

---

## SLIDE 1: The Problem (0:00–1:30)
### "180 Intakes/Day, 4 People, Manual Process"

**What You Say:**

"Let me start with the problem. This is a 6-physician family medicine practice across 2 locations, seeing about 180 patients a day. They have 4 people on the front desk doing intake manually — for all 180 visits.

And here's what happens: Intake work includes insurance verification, prior-auth status checks, questionnaire collection, medication reconciliation, allergy review, and visit-reason triage. All of that, manually, 180 times a day.

The result? Two things fail consistently:

**First:** Expired prior authorizations are missed before the visit. The physician starts the appointment and discovers the procedure can't go forward because authorization lapsed. Rework, delay, frustration.

**Second:** Medication changes don't get reviewed by the physician before the visit starts. Patient says 'I changed my blood pressure med,' but that never surfaces to the doctor. Safety issue, compliance issue.

This is the problem we're solving — not just 'automate everything,' but specific, named operational failures that happen regularly."

**Tone:** Empathetic. You're not criticizing their current process; you're highlighting the real pressure they're under.

---

## SLIDE 2: The Opportunity (1:30–3:00)
### "Automate Mechanical, Keep Humans in Control"

**What You Say:**

"Here's the key insight: Most of intake is just moving data around. It's mechanical. Check the appointment time. Did the patient fill out the form? Is insurance active? Did medications change? These are questions we can ask and answer mechanically.

But here's what the agent does NOT do: The agent never looks at a medication change and decides 'Oh, this is minor, we can ignore it.' The agent never looks at a visit reason and decides 'This sounds urgent.' Those are clinical decisions. Those stay with the clinician.

So the division is clean: 

**The agent automates:**
- Appointment sync every 15 minutes
- Send questionnaires to patients
- Call the insurance eligibility integration
- Look up prior-auth rules
- Compare medication fields
- Pull allergy flags
- Route visit reasons to approved categories

**Humans decide:**
- Is this medication change significant? Clinician decides.
- Is this allergy important? Clinician decides.
- Is this visit really urgent? Clinician decides.
- Should I override this escalation? Manager decides.

The agent is a router and a flagging system. It moves data, detects mismatches, routes cases. Humans do the judgment. That's the boundary."

**Tone:** Confident, clear. This is the foundational principle. Hammer it home.

---

## SLIDE 3: The Delegation Boundary (3:00–4:30)
### "Three Defensible Modes"

**What You Say:**

"I've drawn three clear delegation lines, and each one is justified by the work itself.

**First: Agent Alone.** Seven tasks where the agent works autonomously. Examples: Sync appointments every 15 minutes, check if a questionnaire has all required fields filled in. Why is this safe? Because the logic is deterministic — there are clear rules, no clinical judgment needed, no ambiguity.

**Second: Agent Plus Human Review.** Eight tasks where the agent detects something mechanically, then a human confirms or resolves it before the case is marked ready. Example: Patient reports a medication change. The agent detects that change — mechanical field comparison. But a clinician has to review it and decide 'Yes, this is important' or 'No, we can address this at the visit.' The detection is mechanical; the judgment stays human.

**Third: Human Decides.** Five tasks that are entirely human-owned. Examples: Determining if a visit is urgent — that's clinical judgment. Deciding if a medication is significant — that's clinical judgment. Deciding whether to override an escalation — that's a management call. No agent logic here.

The principle: Each boundary is justified by the properties of the work, not by arbitrary 'let's automate this one.' Deterministic = automate. Deterministic detection + human judgment = agent with review. Clinical judgment = human only."

**Tone:** Structured, analytical. Coaches are looking for this thinking; show it.

---

## SLIDE 4: The Spec Is Buildable (4:30–6:00)
### "10 FRs + 4 Entities + 3 Integrations"

**What You Say:**

"The spec isn't prose. It's ready for a coding agent.

**Ten Functional Requirements** — each with pseudocode, decision logic, acceptance criteria, error handling. FR1 through FR10: Appointment sync, questionnaire dispatch and tracking, insurance verification, prior-auth determination, medication change collection and comparison, allergy flag retrieval, visit-reason routing, readiness evaluation, escalation creation and routing, audit logging.

Every requirement has:
- The algorithm (what happens step by step)
- Acceptance criteria (how we know it works)
- Error handling (what happens if integration fails or data is missing)
- Escalation triggers (when and where to escalate)

**Four Data Entities:** Appointment (read from EHR), IntakeWorkItem (the case being processed), EscalationEvent (when something needs human attention), and AuditLog (immutable record of every action).

All with explicit state machines, enums, audit fields. No ambiguity about what data goes where or what states are valid.

**Three Integration Contracts:** athenahealth EHR, insurance eligibility tool, prior-auth rules source. Each with request/response formats shown as JSON examples, timeout specifications (10 seconds), retry logic (exponential backoff), and fallback behavior (fail closed, escalate if unavailable).

**The result:** An AI coding agent can start building right now. No 'what should we do if this happens?' questions needed. The spec answers them all."

**Tone:** Technical but accessible. Show precision without jargon overload.

---

## SLIDE 5: Safety Guardrails (6:00–7:30)
### "8 Hard Prohibitions"

**What You Say:**

"This is the hard constraint from Scenario 5. Clinical judgment must remain human. So I've listed eight things the system will never do. Let me read them:

1. Diagnose.
2. Perform clinical triage.
3. Recommend treatment.
4. Assess medication significance.
5. Assess allergy severity.
6. Infer prior-auth requirement without an explicit rule.
7. Suppress escalations to appear efficient.
8. Fabricate data when an integration fails.

These aren't 'best practices.' They're hard lines. If you're building this and you feel the temptation to do any of those — 'Maybe the agent could score medication importance as a tie-breaker' — the answer is: Don't. Escalate instead. Remove the feature if you have to. But do not implement it.

**How do we enforce this?** Boundary tests. When the agent flags a medication change or an urgent-looking visit reason, we scan the escalation context for prohibited keywords. If we see 'significant,' 'important,' 'dangerous,' 'recommend,' the test fails. We catch violations in the build loop before they go to production.

That's how we keep the clinical boundary hard."

**Tone:** Firm. This is non-negotiable. Show it.

---

## SLIDE 6: Validation (7:30–9:00)
### "10 Concrete Test Scenarios"

**What You Say:**

"We're not just building and hoping. We have 10 test scenarios that prove the system works.

**V1 (Happy Path):** Routine visit, all data complete, integrations succeed. Expected outcome: case moves to READY_FOR_VISIT, zero escalations. Everything green.

**V2 and V3 (Failure Modes):** Incomplete questionnaire, insurance timeout. Expected outcome: Escalate. Don't suppress the problem; make it visible.

**V4 and V5 (Boundary Tests):** Patient says 'chest pain' — that's an urgent-trigger phrase. Patient reports medication change. Expected outcome: Escalate to clinical staff. But here's the critical part: We verify that the escalation context contains ZERO clinical judgment. No output like 'This sounds moderately urgent' or 'We recommend escalation.' Just the facts for a human to assess.

**V6 and V7 (Edge Cases):** No prior-auth rule matches this procedure. Allergy flag is present in EHR. Expected outcome: Escalate deterministically, no inference.

**V8 (Concurrency):** We sync the same appointment twice. Expected outcome: Idempotent. One work item created, not two. The system is reliable.

**V9 (Audit):** Run the happy path, collect all audit logs. Expected outcome: Complete coverage, all significant actions logged, logs are immutable.

**V10 (Override + Re-Evaluation):** Clinical staff reviews an escalation, resolves it, marks it resolved. Expected outcome: Readiness logic re-evaluates. If conditions are now met, case moves to READY_FOR_VISIT. The system is responsive to human decisions.

These aren't hypotheticals. They're concrete, reproducible, testable in code."

**Tone:** Methodical. Show that you've thought through not just the happy path but also where things break.

---

## SLIDE 7: Assumptions & Unknowns (9:00–10:30)
### "Honest Risk Register"

**What You Say:**

"I've been asked to list genuine unknowns, not filler. Here's what I don't know for certain:

**Top three assumptions:**

**Assumption 1:** athenahealth exposes machine-accessible APIs for appointment data, medications, allergies. I'm medium-confident this is true — athenahealth is industry-standard and does offer APIs — but this specific clinic may not have API access provisioned or may have contractual restrictions. Impact if wrong: Major sections of the spec become unfeasible. Design shifts to manual or RPA approaches.

**Assumption 3:** Prior-auth rules exist in structured form at the clinic. Could be a spreadsheet, could be in the EHR, could be tribal knowledge. I'm medium-confident they exist, but I don't know if they're documented clearly. Impact if wrong: Prior-auth automation becomes less reliable; we escalate more cases.

**Assumption 5:** Operational queues exist and are operationally owned. FRONT_DESK for administrative blockers, CLINICAL_STAFF for safety-sensitive items, PRACTICE_MANAGER for operational issues. If these queues exist and are monitored, escalations get resolved in time. If not, escalations pile up and the system fails operationally.

**Top three unknowns to validate before build:**

**Unknown 1:** Exact athenahealth API availability in this clinic's environment. Validation: Request integration documentation from their IT team.

**Unknown 5:** HIPAA logging requirements. This is a compliance blocker. Validation: Meet with their privacy officer, get written policy on audit-log retention and scope.

**Unknown 6:** Current baseline metrics. What's their current intake completeness rate? How often do prior-auth and medication issues happen? Validation: Request data from clinic for past 30 days.

This isn't weakness. This is discipline. I've separated what I'm confident about from what I need to validate. If any of these assumptions fail, we adapt the design. We don't just hope and proceed."

**Tone:** Thoughtful, honest. Show that you've done the thinking to distinguish confidence from assumption.

---

## CLOSING / PRINCIPLES (Last 30 seconds if needed)

**If you have time, or if the coach asks "What's the core idea?":**

"At the heart of this spec are three principles:

**First: Fail Closed, Not Open.** If data is missing, integration times out, or we're ambiguous about something, we escalate. We don't guess. We don't assume. We escalate loudly and clearly.

**Second: Automate What's Mechanical, Escalate What's Clinical.** The agent is relentless about moving data, comparing fields, routing cases. But the moment clinical judgment is needed, it escalates immediately. The agent is not a clinician. It's a router and a flag system.

**Third: Audit Everything.** Every action is logged with actor, timestamp, and context. If there's a question later — 'Did we check this?' 'Who approved that?' — the audit trail answers it. Accountability is built in.

This isn't just a spec. It's a safety boundary written into code. The agent does what's mechanical, escalates what's clinical, and proves everything happened.

That's how we solve the problem safely."

---

## Timing Breakdowns (Use These to Self-Check)

| Task | Time | Cumulative |
|:--|:--|:--|
| Slide 1 intro | 1:30 | 1:30 |
| Slide 2 | 1:30 | 3:00 |
| Slide 3 | 1:30 | 4:30 |
| Slide 4 | 1:30 | 6:00 |
| Slide 5 | 1:30 | 7:30 |
| Slide 6 | 1:30 | 9:00 |
| Slides 7–10 (summary) | 1:00 | 10:00 |

**If you're running behind:**
- Skip detailed examples; go straight to bullet points
- Compress Slide 7 (assumptions) to "10 assumptions flagged, 8 unknowns identified, all will be validated before build"
- Compress Slides 8–9 (metrics, readiness) to "5 measurable metrics, deterministic readiness gate"
- Land on Slide 10 (Principles) strong, even if rushed

**If you're running ahead:**
- Linger on Slide 5 (Safety Guardrails) and expand on boundary tests
- Go into detail on Slide 6 (Validation) — name specific test assertions
- Add backup examples for challenges

---

## Q&A Strategies (After You Present)

**If asked "What if the clinic says no to one of your assumptions?"**
- "We've validated the assumption. If the clinic says no, we adapt. If athenahealth APIs don't exist, we shift to manual + RPA. If prior-auth rules aren't structured, we escalate more. The core principle — fail closed, route and flag, audit — remains the same."

**If asked "When could this go live?"**
- "Build: 4–6 weeks. Pilot: 2 weeks with 10–20 patients to validate assumptions and collect baseline metrics. Production prep: 2–3 weeks for compliance review and staff training. Go-live could be 8–10 weeks from start if assumptions validate."

**If asked "How much does this reduce workload?"**
- "Proposed target: 30–40% reduction in average intake handling time per patient. That's based on automating appointment sync, questionnaire dispatch, insurance checks, and form validation. For a 4-person team managing 180/day, even 30% is meaningful."

**If challenged on any detail:**
- "That's a great question. Let me reference the spec." (Open the spec, show the relevant section.)
- Or: "That's flagged as an unknown (UA/U#). We validate it before build."
- Or: "That's a boundary test scenario (V#). We explicitly test for that behavior."

---

## Final Words Before You Go On

- **You've done serious work.** The spec is comprehensive, the thinking is disciplined, the safety is baked in. You belong in this presentation.
- **Coaches will see:** Problem understanding, delegation clarity, technical buildability, honest unknowns, and safety-first thinking. That's what they're looking for.
- **Stay calm.** If you fumble a sentence, keep going. If you get a tough question, say "Great question, let me find that in the spec" and take 5 seconds to locate it. Confidence and clarity beat perfection.
- **The closing line matters.** End strong: "The agent does what's mechanical, escalates what's clinical, and proves everything happened." They'll remember that.

**You're ready. Go present this.**
