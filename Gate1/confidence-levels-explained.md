# Confidence Levels in Assumptions — Explained

## The Core Idea

**Confidence is not "how sure am I?"**  
**Confidence is "how much does this assumption carry the rest of the spec?"**

A LOW-confidence assumption is one where:
- If it turns out to be wrong, the entire spec breaks
- OR it gates a decision that affects multiple downstream modules
- OR you have no data to test it before building

A HIGH-confidence assumption is one where:
- If it turns out to be wrong, you can fix one small piece
- OR you have evidence it's true (industry standard, client confirmed)
- OR it's a minor point that doesn't cascade

---

## Three Levels Defined

### 🔴 LOW Confidence = Load-Bearing, Must Validate Before Build

> "I don't have data. If I'm wrong, the whole design cascades down."

**Characteristics:**
- No data yet (you're guessing based on context)
- If wrong → multiple modules must be redesigned
- OR it's a business/client decision you haven't confirmed
- OR it's a legacy system with known risk

**Examples from your spec:**

**A.3 — SOAP responds in ≤ 5 seconds**
- **Why LOW:** "Legacy SOAP system" is a red flag. You have no performance data.
- **If wrong:** If SOAP is actually 30s typical → entire validation latency budget collapses → SLA cannot be met → maybe don't build this at all yet
- **Blocks:** M3 (validator), M7 (escalation handling)
- **Test:** 50 staging calls before sprint 1 ends
- **What you do:** Make it a sprint 1 blocker. Don't design around 5-second assumption. Get the data first.

**A.4 — "High-value" means £50,000**
- **Why LOW:** Placeholder. You made it up. Client has not confirmed.
- **If wrong:** If client means £10k → Rules 3, 4 in M4 all change → routing boundary shifts → HIGH/CRITICAL volume spikes → specialist queue overwhelmed
- **Blocks:** M4 (severity classifier), M5 (routing decisions), delegation boundary
- **Test:** Client workshop, threshold table sign-off in sprint 0
- **What you do:** This is a sprint 0 gate. No code written on M4 until client says "yes, £50k is correct"

**A.8 — LLM API access within build timeline**
- **Why LOW:** Insurance = PII + data residency hell. Three-month procurement delay is realistic.
- **If wrong:** If takes 3+ months → can't use third-party LLM → must re-scope extraction to regex/rules → 15% escalation rate instead of 5% → ROI timeline changes
- **Blocks:** M2 (entire extraction module)
- **Test:** Sprint 0 blocker. Written confirmation of provider, data residency, PII handling before first line of extraction code
- **What you do:** Do not build extraction assuming LLM is available. Assume it isn't. Plan for plan-B.

---

### 🟡 MEDIUM Confidence = Probably True, But Verify Early

> "Industry standard or logical guess, but I should confirm with the client/data before scaling."

**Characteristics:**
- Based on industry practice or logical reasoning, not client confirmation
- If wrong → one module needs adjustment, not full redesign
- You can test it with a small data sample

**Examples from your spec:**

**A.1 — FNOL inputs contain policy number in ≥ 80% of cases**
- **Why MEDIUM:** Web forms usually require policy number (industry standard). Phone calls and emails often omit it (realistic). Net: probably 70–90%.
- **If wrong:** If only 40% have policy number → extraction module can't gate on policy lookup → most claims escalate → throughput collapses
- **Test:** 100-claim sample from client. Count how many have policy number.
- **What you do:** Request data in sprint 0, confirm by week 1. If below 50%, redesign extraction gating (don't fail, accept policy_number=null and match by other fields).

**A.2 — Policy coverage rules are fully deterministic**
- **Why MEDIUM:** Personal lines (motor, home) usually are codifiable. Commercial/specialty often aren't. Your scenario doesn't specify which.
- **If wrong:** If rules are fuzzy → coverage validation becomes judgment call → rule engine breaks down → must escalate most claims
- **Test:** Ask client for 3 examples of coverage decisions. Were they rule-based or judgment-based? Review policy admin schema.
- **What you do:** Sprint 0 blocker. If coverage isn't deterministic, you must redesign M3 (validation) completely or pivot to different scope.

**A.5 — Claimants accept automated ack**
- **Why MEDIUM:** Industry standard. But if this company has had complaints about automation, you're wrong.
- **If wrong:** If claimants reject automated acks → must route HIGH/CRITICAL acks through specialists → scalability problem
- **Test:** Review complaint logs. Ask client: "Have claimants complained about automated responses?"
- **What you do:** Low risk. If wrong, you can fix easily (move ack from agentic to agent-led). Not a cascading failure.

**A.6 — CRM data is near-real-time**
- **Why MEDIUM:** REST API suggests integration, but adjuster availability is often manual. Realistic uncertainty.
- **If wrong:** If CRM data is 12-hour-old → routing assigns to people at capacity → recreates current problem under new system → invisible failure
- **Test:** Ask client: "Is adjuster availability updated automatically or manually? How often?"
- **What you do:** If manual daily update → add data-freshness flag to routing. If adjuster data > 4 hours old, bypass auto-route. Not a breaking change.

**A.7 — 18% routing error is extraction inconsistency, not structural gap**
- **Why MEDIUM:** Scenario implies this (specialists misreading FNOLs). But you don't know for sure.
- **If wrong:** If problem is "no orthopedic adjuster for X type of injury" → you can't fix with better extraction → agent can't improve on baseline
- **Test:** Ask specialists: "When you route wrong, is it misreading or 'I don't know who handles this'?" Review 20 historical errors.
- **What you do:** Medium risk. If wrong, you've built a fast system that solves the wrong problem. Not a crash, but a wasted sprint.

**A.9 — Phone transcripts are pre-transcribed text**
- **Why MEDIUM:** Scenario says "phone transcript" (implies text). But not explicit. Some systems deliver audio.
- **If wrong:** If you receive audio files → M1 (normaliser) must do speech-to-text conversion → different architecture
- **Test:** Ask: "Do you have transcription already or do we handle audio?"
- **What you do:** Low risk. Not a cascading failure if wrong. Just changes M1.

**A.10 — DMS supports high-volume programmatic write**
- **Why MEDIUM:** DMS systems vary widely. REST API suggests yes, but many aren't tuned for 300 writes/day.
- **If wrong:** If DMS rate-limits → raw storage becomes bottleneck → ingestion stalls
- **Test:** Request DMS API docs, run load simulation.
- **What you do:** Medium risk. If wrong, you add caching layer. Not a design break, but a performance issue.

---

### 🟢 HIGH Confidence = Confirmed, or Minimal Impact if Wrong

> "Industry standard, or I have data, or wrong assumption doesn't break things."

**Characteristics:**
- Client confirmed
- OR widely true across the industry
- OR if wrong, you can fix with a small code change

**Examples from your spec:**

**A.5 — Claimants accept automated ack**
- Actually marked HIGH (I said it was MEDIUM above — that's a judgment call on your part)
- **Why HIGH:** Automated FNOL ack is industry-standard insurance practice worldwide. Very unlikely to be wrong.
- **If wrong:** Easy to fix (add specialist review to ack). Not a cascading failure.

---

## Why This Matters for Your Defense

### In Your 3-Minute Walkthrough

When you say: **"Confidence is LOW"** — you're demonstrating:

✅ **Honest thinking** — you see where the gaps are  
✅ **Load-bearing awareness** — you named what breaks if this is wrong  
✅ **Risk management** — you have a test and a sprint assignment  

When a coach asks: **"Why do you have LOW confidence on SOAP?"**

Don't say: "Because I don't know..."  
**Do say:** "Because 'legacy SOAP' is a known indicator of unpredictable latency. I have no performance data. If p95 latency is 30 seconds instead of 5, the entire validation module design is wrong — the latency budget collapses and SLA is unreachable. That's why it's a sprint 1 gate: 50 staging calls before we commit to the 2-retry pattern."

---

## The Three Levels in a Table

| Level | Meaning | If Wrong | Action | Example from Spec |
|---|---|---|---|---|
| 🔴 **LOW** | No data; cascading failure if wrong; gates multiple modules | Entire design breaks or changes radically | Sprint 0 gate + test before any code | A.3 (SOAP latency), A.4 (£50k threshold), A.8 (LLM access) |
| 🟡 **MEDIUM** | Industry standard or logical guess; one module needs rework | One module redesigned, not cascading | Sprint 0/1 early test, plan workaround | A.1 (80% have policy #), A.2 (rules deterministic), A.6 (CRM real-time) |
| 🟢 **HIGH** | Client confirmed or widely true; wrong assumption is minor | Small code change, no design impact | Plan to handle but not blocking | A.5 (claimants accept ack) |

---

## How to Use This in Your Defense

### Pacing (from your 45-second section on assumptions):

**"Three assumptions are marked LOW confidence because they're load-bearing.**

**A.3 — SOAP latency.** 'Legacy SOAP' is a red flag with no performance data. If it's 30 seconds typical instead of 5, the validation latency budget collapses and SLA is unreachable. **Confidence is LOW.** Test is 50 staging calls. If p95 exceeds 10 seconds, I need a cache layer.

**A.4 — £50k threshold.** Placeholder, not client-confirmed. If client means £10k, Rules 3 and 4 in the severity classifier change, routing boundary shifts, specialist queue volume spikes. **Confidence is LOW.** Test is client workshop. **Sprint 0 blocker** — no code on M4 until written sign-off.

**A.8 — LLM procurement.** Insurance = PII + data residency. Three-month delay is realistic. If provisioning takes longer than build timeline, extraction module is re-scoped to regex/rules, 15% escalation rate instead of 5%. **Confidence is LOW.** Test is written provider approval in sprint 0.

**Each LOW-confidence assumption has a test and a sprint gate. I'm not hiding the risk — I'm naming it."**

---

## Common Coach Challenge & How to Respond

### Challenge: "Why do you have LOW confidence if you included it in the spec?"

**Don't say:** "I don't know..."

**Do say:** "That's exactly why it's in the assumptions section marked LOW confidence. Hiding it would be worse. The spec shows where my reasoning is load-bearing and where it's not. A.3 is a sprint 1 gate — if SOAP latency data shows I was wrong, I redesign the validation module *before* writing extraction or routing code. That's the whole point of naming it upfront."

### Challenge: "Why not test all assumptions in sprint 0?"

**Do say:** "Some could be. But A.3 requires a staging environment with live SOAP access, which I may not have until IT provisioning in sprint 0. A.4 is a client decision, not a technical test — it's a workshop conversation. A.8 is procurement, which I can't force faster. So I've assigned each unknown to the sprint where it's resolvable. By end of sprint 0, A.4 and A.8 are confirmed or I know we have a blocker. A.3 moves to sprint 1 because IT access isn't guaranteed earlier."

---

## Summary

**Confidence levels are not "how sure am I?"**  
**They're "how much does this assumption carry the entire spec?"**

- **🔴 LOW** = load-bearing, gates multiple modules, must validate before build
- **🟡 MEDIUM** = one module affected, confirm early but not blocking
- **🟢 HIGH** = client confirmed or minor impact

This shows the coach three things:
1. You see where the risk is
2. You're not hiding dependencies
3. You have a plan to validate each unknown at the right time

That's the entire point of Section 1.
