# Gate 1 Defense — 3-Minute Walkthrough
## FNOL (First Notice of Loss) Claims Processing Agent | Nadia Rahmatulla

---

## The Walkthrough (No Slides — Direct from Document)

---

### Opening (15 seconds)

"I'm walking you through my specification for First Notice of Loss (FNOL) claims processing agent. The client processes 300 claims a day with 12 specialists, 22 minutes per claim. They're hitting 18% routing errors and 31% SLA breaches. Zero capacity slack. The problem isn't the people — it's that specialists are doing structured, repeatable work manually from unstructured inputs under time pressure."

---

### Delegation Boundary — Why This Line, Not Another (60 seconds)

"Let me start with the delegation boundary, because this is where the thinking happens.

**The agent does six things fully autonomously:** ingest from three channels, extract structured fields using an LLM with confidence scoring, validate policy coverage via SOAP, triage severity using deterministic rules, route LOW and MEDIUM claims to the best-match adjuster, and acknowledge the claimant within 15 minutes.

**It escalates four things to specialists:** HIGH or CRITICAL severity claims, coverage declines that need approval before sending, any extraction where a required field falls below the confidence threshold, and system failures where it can't complete the workflow.

**Here's why that line is defensible.** A task is fully agentic when three conditions hold: the decision logic is codifiable, errors are detectable before they cause harm, and there's no regulatory requirement for named human accountability. 

LOW and MEDIUM claims meet all three. The routing rules are deterministic — specialisation match plus load balance. If the agent routes wrong, the adjuster sees it immediately and re-assigns, which we're measuring. And there's no legal requirement for a named human to approve routine claim intake.

HIGH and CRITICAL claims fail condition three. The client explicitly stated that high-value claims require specialist oversight. Even if the agent *could* route them accurately, the accountability requirement keeps them in human oversight. That's not a technical limitation — it's a business and legal boundary.

**The test:** I can explain why every task is at its delegation level. Not 'feels right,' but named criteria. If a coach asks 'why not auto-route HIGH claims,' the answer is: because accountability for high-stakes claims is a legal and client requirement, regardless of technical capability."

---

### Assumptions — Honest, Not Hidden (45 seconds)

"Section 1 comes before the problem statement for a reason. Every non-trivial claim in the rest of the spec rests on at least one assumption here. I've got ten assumptions, each structured the same way: what I'm assuming, why it matters, how I'd test it, and my confidence level.

**Three are marked LOW confidence, and they're load-bearing.** 

Assumption 3: I'm assuming SOAP responds in under 5 seconds at p95. If it's actually 30 seconds, my latency budget collapses and the 2-hour SLA is at risk. Confidence is LOW because 'legacy SOAP system' is a red flag and I have no performance data. The test is 50 calls in staging before committing to the 2-retry pattern.

Assumption 4: I'm assuming 'high-value' means £50,000. But that's a placeholder. If the client means £10,000, the entire delegation boundary shifts because more claims become HIGH severity and require specialist oversight. Confidence is LOW. The test is a threshold workshop in sprint zero with written sign-off.

Assumption 8: I'm assuming I'll have LLM API access within the build timeline. Insurance has strict data residency and PII restrictions. If procurement takes three months, I need to re-scope the extraction module to regex and rules for sprint one, which changes the accuracy assumption. This is a sprint zero blocker — no code until I have written confirmation of provider, data residency approval, and PII handling.

**I also have eight critical unknowns.** They're not filler. Every one has a blocker attached. U.1 is 'what does high-value mean numerically' — that blocks the severity classifier. U.3 is SOAP latency profile — that blocks the validation module design. U.7 is FCA constraints on automated declines — that might force a redesign of the NOT_COVERED flow.

**The principle here:** honest uncertainty beats a plausible-sounding guess. If I don't know, I say so, I say why it matters, and I say how I'll resolve it."

---

### Spec Precision — Could an AI Build This? (45 seconds)

"The test of precision is: could an AI coding agent start building from this spec without asking clarifying questions? I've tested that principle in closed build loops with Claude Code. Every time the builder produces something unintended, I classify it: spec ambiguity — I own the fix. Builder misread — the builder owns the fix. Unjustified addition — I ask for a rollback.

**Three things make the spec build-ready.**

One: the state machine is executable. A claim is in exactly one state at all times. Every transition is logged with timestamp, actor, and reason. Every state has a maximum time. If a claim sits in EXTRACTING for more than 90 seconds, the watchdog moves it to SPECIALIST_QUEUE with a note that says 'LLM call hung or low-confidence field.' No silent hangs.

Two: module contracts are unambiguous. M4, the severity classifier, has eight rules applied in priority order. First match wins. Rule 8 is the default, and it's MEDIUM, not LOW. That's a deliberate choice. If I default to LOW, I risk auto-routing a high-stakes claim without review. False positives on escalation are safer than false negatives.

Three: agent rules are non-negotiable. Never auto-route HIGH or CRITICAL. Never send a decline without specialist approval. Never retry SOAP more than twice. claim_id is set at ingestion and immutable. These aren't guidelines — they're constraints the agent cannot violate.

**The LLM prompt isn't 'extract relevant information.'** It's pinned in the spec with the exact JSON schema and re-prompt logic. If a required field comes back below threshold, run a structured clarification once. Still below? Move to EXTRACTION_FAILED and escalate with the full dossier. No ambiguity."

---

### Validation — How I Know It's Working (45 seconds)

"I've defined 'working' as five testable conditions measured over a rolling five-day window with at least 500 claims.

One: routing error rate is 5% or less. I'm measuring adjuster re-assignments divided by total claims.  
Two: SLA compliance is 90% or higher. routed_at minus received_at is under 120 minutes.  
Three: zero silent failures. Every terminal-state claim has a complete audit trail.  
Four: zero unacknowledged routed claims. Every claim that reaches ROUTED has a notification record.  
Five: escalation rate is between 15 and 35%. If it's outside that band, something's wrong and I investigate.

**The dangerous failures are the quiet ones** — the ones that don't throw errors. 

If the agent extracts the wrong policy number but assigns high confidence, I won't know unless I audit. So I'm running a weekly 5% random audit by specialists. Target is 97% accuracy or better.

If a HIGH damage amount is mentioned in the narrative but the structured damage field is empty, Rule 8 might default to MEDIUM when it should be HIGH. So I'm testing with 20 synthetic high-value narrative claims to verify the LLM flags them for escalation.

If a claim gets stuck in EXTRACTING because the LLM hangs, the watchdog test is: mock an LLM hang and verify the claim moves to the queue after 90 seconds.

**These failure modes are tied to decisions in the spec.** If the £50k threshold is too high, HIGH claims slip through as MEDIUM. If the SOAP 2x retry happens during an outage, that's 30 seconds per claim times 300 claims — queue backup. I've named the failure, the cause, and the test."

---

### Close — Why This Is Defensible (15 seconds)

"This spec is defensible because I can justify every delegation boundary with named criteria, I've marked three assumptions as LOW confidence and eight unknowns as blockers, the spec is precise enough that an AI builder could start without clarification, and validation is testable — five numeric conditions and the quiet failures are named. 

If a coach challenges a boundary, I have the reasoning. If they challenge an assumption, I have the test. If they challenge precision, I have the contracts. That's what I'm here to defend."

---

## How to Deliver This

### Tone
- **Confident but not defensive.** You're presenting reasoning, not justifying choices.
- **Concrete, not abstract.** "Rule 8 defaults MEDIUM" beats "we have fallback logic."
- **Own the unknowns.** "Confidence is LOW" is a strength, not a weakness.

### Pacing
- **Opening:** 15 seconds — set the problem context
- **Delegation:** 60 seconds — this is the core thinking
- **Assumptions:** 45 seconds — show honest uncertainty
- **Precision:** 45 seconds — prove it's build-ready
- **Validation:** 45 seconds — testable conditions
- **Close:** 15 seconds — summary stance

**Total: 3 minutes 45 seconds** (leaves 15 seconds buffer for breath/transitions)

### Physical Delivery
- **No slides.** You're walking through the document they have in front of them.
- **Reference section numbers:** "Section 1 comes before the problem statement for a reason..."
- **Point to specific examples:** "M4, the severity classifier, has eight rules..."
- **Pause after load-bearing statements:** "Confidence is LOW." [beat] "The test is..."

### What to Have in Front of You
- The full spec document (open, with sections bookmarked)
- One notecard with timing checkpoints:
  - 0:00 — Opening
  - 0:15 — Delegation boundary
  - 1:15 — Assumptions
  - 2:00 — Precision
  - 2:45 — Validation
  - 3:00 — Close

### Defending Under Pressure

#### If challenged on delegation boundary:
"Which task should move, and why? I'm using three criteria: codifiable logic, detectable errors, and legal accountability. If you're saying HIGH claims should be auto-routed, I'd need evidence that the accountability requirement doesn't apply. If you're saying MEDIUM should require oversight, I'd need to know what decision logic isn't codifiable."

#### If challenged on LOW-confidence assumptions:
"That's why it's marked LOW. I'm not pretending I know. Assumption 3 on SOAP latency — I have no performance data, so I've made it a sprint one test. If you're saying it should be sprint zero, I can accept that gate. But hiding it would be worse."

#### If challenged on spec ambiguity:
"Show me the ambiguity. If module M4 could be interpreted two ways, that's my bug and I'll fix it. But 'the severity classifier has eight rules in priority order, first match wins, default is MEDIUM' — where's the ambiguity?"

#### If challenged on validation:
"Which failure mode am I missing? The five conditions cover routing accuracy, SLA, silent failures, acknowledgment, and escalation rate. The quiet failure tests cover wrong extraction, wrong classification, and system hangs. If there's a sixth failure mode, I want to know it."

#### If a coach says "You're wrong about X":
"Walk me through it. If I've misunderstood the constraint, I'll update the spec cleanly. But if it's a judgment call — like where to draw the delegation line — I need to understand the criteria you're using that I'm not."

### Update Cleanly Under Pressure
If you realize mid-defense you made an error:

**Don't:** "Oh, yeah, I guess that doesn't work..."  
**Do:** "That's a valid catch. Assumption 4 should be sprint zero, not sprint one, because the threshold gates the entire severity classifier. I'm updating it to a sprint zero blocker now."

**Show the update:**
- Note it on the spec in front of you
- State the change clearly
- Explain why it doesn't cascade (or if it does, name the cascade)

### The Mindset
You're not defending a finished product. You're defending a *way of thinking.*

- **Delegation boundaries** → Can I explain why this line, not another?
- **Assumptions** → Am I honest about what I don't know?
- **Precision** → Could someone build from this without guessing?
- **Validation** → Do I know what failure looks like?

If a coach challenges you and they're right, updating cleanly *demonstrates* the thinking. If they're wrong, holding your ground *with reasoning* demonstrates it just as much.

### What Failure Looks Like
- **Failure:** "I think that task should be agentic" [no criteria given]
- **Success:** "That task is agentic because the decision logic is a lookup against the policy table, the error is visible to the adjuster immediately, and there's no legal accountability requirement."

- **Failure:** "I'm pretty sure SOAP will be fast enough" [confidence hidden]
- **Success:** "I'm assuming SOAP responds in under 5 seconds. Confidence is LOW because I have no data. Test is 50 staging calls. If p95 is over 10 seconds, I need a cache layer."

- **Failure:** "The agent will extract the policy number" [no failure case]
- **Success:** "The extractor runs the LLM with a JSON schema. If confidence is below 0.85, it re-prompts once. Still below? EXTRACTION_FAILED, escalate with dossier. Weekly 5% audit to catch high-confidence wrong extractions."

---

## Final Check Before You Walk In

Ask yourself:
1. Can I defend every delegation boundary with named criteria?
2. Can I name three LOW-confidence assumptions and explain why they're load-bearing?
3. Can I point to a module contract and explain why it's unambiguous?
4. Can I name a quiet failure and the test that catches it?

If yes to all four: **you're ready.**

---

*This is the test of your way of thinking. The document demonstrates it. Your walkthrough defends it.*
