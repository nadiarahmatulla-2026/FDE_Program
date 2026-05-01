# Gate 1 — Assumptions & Unknowns: FNOL Claims Processing Agent
## Nadia Rahmatulla

**Scenario:** A mid-size insurance company processes 300 FNOL (First-Notice-of-Loss) reports per day with a current error rate of 18% on routing and 31% SLA breaches (2-hour SLA). This document surfaces all assumptions embedded in the broader build and identifies the 10+ genuine unknowns that must be validated before implementation.

---

## Executive Summary

This solution assumes moderate data structure in FNOL inputs, a deterministic policy coverage model, round-robin routing to specialists, and regulatory permission for automated acknowledgment. These are reasonable for a mid-size insurer but must be validated. Ten critical unknowns are detailed below, each with confidence assessment and a test plan.

**Confidence Summary:** 
- **High confidence** (5/10 unknowns): System integration patterns, acknowledgment SLA structure, pilot scope, error root causes
- **Medium confidence** (3/10 unknowns): Specialist workflow, success metrics, policy system latency
- **Low confidence** (2/10 unknowns): FNOL data quality/structure, "high-value" and "ambiguous" definitions

**Key assumption (load-bearing):** If FNOL data quality is poor (missing fields, unsanitized names/dates, no policy number in 20%+ of claims), the entire validation layer fails silently or escalates everything. This is the highest-risk assumption.

---

## Section 1: Core Business Assumptions (Stated in Scenario, Restated Here for Clarity)

| Assumption | Why It Matters | Confidence | Test |
|---|---|---|---|
| **A1: 2-hour SLA is absolute requirement** | Shapes agent latency budget (< 10 min per claim = ~5 min triage/validation, ~5 min routing/ack). If SLA can slip, design relaxes. | High | Ask client: "Is 2-hour SLA hard deadline or aspirational?" |
| **A2: Current 12-specialist team is baseline** | Shapes whether agent replaces, supplements, or augments specialist capacity. If team will be reduced 50%, routing must be even smarter. | Medium | Confirm with HR/ops: "What's the post-automation headcount plan?" |
| **A3: Triage is mandatory (not optional).** | Agent must classify severity even if it auto-approves low-severity claims. If severity is not actionable downstream, triage is waste. | Medium | Ask adjusters: "Do you use severity flag? How does it affect your work?" |
| **A4: Policy coverage validation is deterministic.** | Agent can apply rule engine (not fuzzy judgment). If coverage rules require specialist discretion (e.g., "judgment call on fraud indicators"), agent can't validate autonomously. | Medium-Low | Walk through 5 policy documents. Can client articulate coverage rules in decisioning logic? |
| **A5: Claimant acknowledgment is async (not blocking).** | Agent sends ack, proceeds with routing. If ack must be synchronous (await response, confirm delivery), processing stalls. | High | Check: does client expect immediate customer callback, or is email okay? |

---

## Section 2: Data & Input Assumptions

| Assumption | Why It Matters | Confidence | Test | Impact if Wrong |
|---|---|---|---|---|
| **A6: FNOL inputs contain >85% of critical fields: claimant name, policy number, incident date, incident type.** | Agent can triage and route with partial data. If critical fields missing in 30%+ of claims, agent escalates or errs. | Low | Analyze 50 real FNOL reports (last 2 weeks). Measure field presence, format consistency, completion rate. | If <70% complete: agent escalation rate climbs to 40%+, defeating automation gains. |
| **A7: Free-form text in FNOL is grammatically coherent (not voice-to-text garbage or truncated emails).** | NLP extraction relies on reasonable sentence structure. Garbled data = failed extraction = escalation or misclassification. | Low | Sample 10 phone transcripts, 10 emails, 10 web forms. Measure: % with clear incident description vs % that are ambiguous/incomplete. | If 20%+ are incoherent: consider human pre-processing step or manual-only path. |
| **A8: Policy numbers and claimant names are consistently formatted** (e.g., policy number is always 8 digits, claimant name is First Last). | Allows fuzzy-matching and validation without custom parsing. If formats vary wildly, extraction is fragile. | Medium | Check current data: what formats exist? Are there variations (PL-2024-001 vs 20240001 vs P_2024_0001)? | If high variation: add regex/parsing layer to agent; increases complexity and test burden. |
| **A9: Document attachments (if present) are already digitized and retrievable via document system API** (not need to be OCR'd from paper). | Shapes whether agent can extract claim details from docs or only from structured fields. | Medium | Check: what % of FNOL comes with attached documents? Are docs digital or scanned paper? Is there an API to fetch docs? | If documents are paper or no retrieval API: agent cannot access docs; may miss key info (e.g., photo of damage). |
| **A10: Date/currency formats are ISO 8601 and USD only** (not locale-dependent or multi-currency). | Simplifies parsing and comparison. If dates are MM/DD/YY vs DD/MM/YY, agent fails on many dates. | Medium | Inspect 10 claims: date formats, currency symbols. Are they consistent? | If mixed formats: add locale-aware parsing; test burden increases. |

---

## Section 3: System & Integration Assumptions

| Assumption | Why It Matters | Confidence | Test | Impact if Wrong |
|---|---|---|---|---|
| **A11: CRM API responds in <500ms (p95).** | If CRM is slow, triage lookup stalls, SLA pressure increases. Agent can't meet 2-hour SLA if each integration call is 2+ seconds. | Medium | Run load test: 100 concurrent CRM queries, measure response time distribution (p50, p95, p99). | If p95 > 1s: add caching layer or async polling; design becomes more complex. |
| **A12: Policy admin SOAP endpoint responds in <3s (p95) and has >99% uptime.** | Legacy SOAP is slow; agent design must tolerate slow policy lookups. But if uptime is poor (e.g., 95%), agent needs queue + retry, not fail-fast. | Medium-Low | Get SLA docs from policy team. Run 100 policy lookups; measure latency and failure rate. If no docs available, assume worst case (5s, 95% uptime) and build for queue. | If >3s or <99% uptime: add queue system for policy lookup; adds infrastructure complexity. |
| **A13: Authentication to all three systems (CRM, policy, document) is via API key or OAuth** (not session-based or VPN-dependent). | Shapes how agent is deployed. If VPN-only, agent must run on-prem. If API key, can be containerized/cloud. | High | Check: what auth does each system support? Is there a service account for the agent? | If session-based or VPN: deployment constraints shift; may need to reconsider architecture (on-prem vs cloud). |
| **A14: Document management system has a retrieve API** (not just a file share). | If agent must FTP into a share, integration is manual/unreliable. If there's a retrieve API, it can be automated. | Medium | Ask: "How does the agent retrieve claim documents?" File share? S3? REST API? | If file share only: agent can't retrieve docs automatically; must escalate or ignore docs. |
| **A15: No rate limits or very permissive rate limits** (>1000 req/min per API key, or >5000 req/day). | 300 claims/day with 3 integrations per claim = 900 integration calls/day ≈ 60/hour. If rate limit is 10/min, agent hits limit quickly on volume spikes. | Medium | Check API docs: what are the rate limits? Is there burst allowance? | If tight limits (e.g., 100 req/day): agent must batch queries or queue claims; adds latency. |
| **A16: All three systems have graceful failure modes** (5xx = temporary, retry later; 4xx = permanent, fail fast). | If systems don't distinguish, agent can't decide: retry or escalate? | Medium | Test: call each API with invalid auth, bad request, timeout. Observe: what HTTP status codes? What error messages? | If no clear distinction: assume all failures are temporary; queue everything; potential for infinite retry loops. |

---

## Section 4: Process & Specialist Workflow Assumptions

| Assumption | Why It Matters | Confidence | Test | Impact if Wrong |
|---|---|---|---|---|
| **A17: Adjuster assignment is round-robin or least-loaded** (not skill-based or geography-based with complex rules). | Shapes whether agent can do deterministic routing or needs to maintain state (adjuster skills, zone assignments). | Medium | Interview 2-3 adjusters: "How do claims get assigned to you? Is there a system, or is it ad-hoc?" | If skill-based: agent needs a specialist registry; routing becomes more complex. |
| **A18: Specialists can be overloaded; no queue depth limit** (claims can wait for available adjuster, not auto-rejected). | Agent can assign even if adjuster is busy; specialist will work through queue. If there's a queue depth limit (e.g., "max 10 pending per adjuster"), agent must escalate or defer. | Medium | Check: do specialists have queue depth limits? Is there a max pending? | If yes: agent must monitor queue depth and make routing decisions; adds state management. |
| **A19: Specialists review agent escalations within 4 hours** (not SLA-critical, just a working assumption). | Escalated claims still need human touch; if specialist review is slow, escalated claims miss 2-hour SLA anyway. | Low | Ask: "For claims flagged as ambiguous or high-value, what's your SLA to review?" | If SLA is <1 hour for escalations: might be tighter than agent processing; defeats purpose. |
| **A20: Specialists can override agent triage without push-back** (not audited or penalized). | If override is discouraged or logged punitively, specialists will re-triage manually instead of trusting agent; escalation becomes permanent. | Medium | Check: can adjusters override? Is override logged? Are they held accountable for overrides? | If overrides are penalized: specialists will avoid agent assignments; back to manual processing. |
| **A21: No feedback loop from specialist overrides to agent learning.** | Agent does not learn from specialist corrections; each error is independent. If we want agent to improve over time, we need feedback. | Medium | Ask: "Can we log specialist corrections and use them to retrain the agent?" Infrastructure for this? | If no: agent error rate is fixed; validation must be very tight from day-1. |

---

## Section 5: Delegation & Decision Boundary Assumptions

| Assumption | Why It Matters | Confidence | Test | Impact if Wrong |
|---|---|---|---|---|
| **A22: "High-value" claims are operationally defined** (e.g., claims > $25K). | If "high-value" is adjuster judgment, agent can't flag them autonomously; must escalate all. If dollar amount is clear, agent can pre-filter. | Low | Ask client: "What's your threshold for high-value? Is it always a dollar amount, or are there other factors?" | If judgment-based: agent can't autonomously identify high-value; must escalate or use proxy signal (e.g., claim type). |
| **A23: "Ambiguous" claims have discernible signals** (e.g., multiple injury claims, liability disputed, incident type unclear). | Agent can't detect true ambiguity; it's subjective. But agent can detect signals correlated with ambiguity. | Low | Ask: "Show me 5 claims you flagged as ambiguous. What patterns do you see?" | If no patterns: can't teach agent to detect ambiguity; either over-escalate or miss real ambiguity. |
| **A24: Low-value, low-severity, clear claims (80%+ of volume) can be fully automated** (triage, validation, routing, ack — no specialist touch). | Determines automation ceiling. If 60% can be automated, ROI is lower; if 90%, ROI is higher. | Low | Estimate from current data: what % of 300/day are routine? What % are ambiguous/high-value? | If <50% automatable: agent value is capped; redesign may be needed. |
| **A25: Routing to an adjuster is "soft" assignment** (specialist can re-route or escalate) not irreversible. | If routing is sticky (specialist stuck with wrong assignment), agent must be very accurate; no room for error. | Medium | Check: can adjusters bounce claims back? Are there reassignment fees or friction? | If sticky: routing error becomes specialist's problem; damage is real; agent must be >95% accurate. |

---

## Section 6: Compliance & Regulatory Assumptions

| Assumption | Why It Matters | Confidence | Test | Impact if Wrong |
|---|---|---|---|---|
| **A26: No state-level regulation forbids automated FNOL acknowledgment.** | Some states require human acknowledgment or may restrict AI use in claims. If regulated, agent can't send auto-ack. | Medium | Check state insurance commission regulations (client's home state + any other states where they operate) for FNOL ack requirements. | If forbidden: agent can acknowledge only via human-written email; defeats automation ROI. |
| **A27: Audit trail and logging are required but not real-time.** | Agent can batch-log decisions at end of processing, not intrude on critical path. If real-time logging is required, latency adds 100-200ms per claim. | Medium | Ask compliance: "What audit requirements apply? Real-time logs? Immutable logs? Retention period?" | If real-time immutable logging required: design adds complexity; may violate SLA. |
| **A28: HIPAA does not apply** (claims are generally not medical data). | If medical claims are in scope (e.g., health insurance), agent must handle PHI carefully; may need encryption, sanitization. | High | Confirm: "Is this health insurance? Workers comp? Auto? Home?" Health = HIPAA rules. | If health: add PHI sanitization, encryption, audit trail; significantly more complex. |
| **A29: Fair Lending / Anti-Discrimination laws do not require human review of every routing decision.** | Agent can route to specialist without specialist confirming; if fair lending rules require human review of all, agent gains no time. | Medium-Low | Check: "Are there fair lending constraints on routing? Must each decision be reviewed?" Unlikely for claims, but worth asking. | If required: every routed claim needs specialist validation; defeats purpose. |

---

## Section 7: Success Metrics & Validation Assumptions

| Assumption | Why It Matters | Confidence | Test | Impact if Wrong |
|---|---|---|---|---|
| **A30: Primary success metric is SLA compliance** (currently 31% breach; target <10% breach). | Shapes agent design: prioritize speed and routing accuracy over perfection. If quality is primary, design shifts to be more conservative/escalating. | Medium | Ask C-level: "What's your #1 goal? Faster turnaround? Fewer errors? Cost savings?" | If quality is primary: agent must be >95% accurate; escalation rate may climb; SLA improvement is secondary. |
| **A31: Acceptable error rate post-automation is <5% routing error** (currently 18%). | Defines success gate. If client wants <3%, agent design needs 99%+ accuracy; if <10%, more lenient. | Medium | Confirm: "What's your target error rate? What's acceptable?" | If unrealistic target (e.g., <1%): agent design becomes conservative; escalation rate climbs; volume automation stalls. |
| **A32: Pilot scope (if any) is 10-20% of volume** (30-60 claims/day) for 1-2 weeks. | Shapes deployment: can we test in shadow mode, canary deployment, or must go full rollout? | Medium | Ask: "Is this a pilot or full deployment?" | If full deployment immediately: zero margin for error; spec must be bulletproof. |

---

## Section 8: Candidate Unknowns Not Yet Elevated to Assumptions

Below are scenarios where we genuinely don't know the answer and can't reasonably guess:

### U1: FNOL Input Data Structure and Quality ⚠️ **CRITICAL UNKNOWN**

**What we don't know:**
- What does a real FNOL report look like? Email body + attachments? Structured web form? Phone transcript?
- How much variation? What % have policy number in first sentence? What % are buried in prose?
- What % of claims have all critical fields vs partially filled?
- Are phone transcripts AI-generated summaries or human-written?
- Data quality: typos in policy numbers? Dates in multiple formats? Claimant names with special characters?

**Why it matters:**
- Dictates whether NLP extraction is simple (structured) or complex (free-form).
- If data quality is poor, validation layer fails silently or escalates constantly.
- Directly affects success metrics (error rate depends on input quality).

**Hypothesis:**
If FNOL inputs are >80% complete and <10% have non-standard formats, agent can extract and validate with <5% error rate on extraction itself. If <70% complete or >20% non-standard, agent error rate on extraction climbs to 15%+ and escalation rate becomes prohibitive.

**How I'd test it:**
- Request 50 real FNOL reports from the last 2-4 weeks (anonymized).
- Measure: field presence, format consistency, free-form text coherence, attachment rate.
- Test NLP extraction accuracy on a sample with Claude or GPT-4.
- Confidence: **Very Low**. Without real data, this is the biggest risk.

**Remediation if wrong:**
- If data is poor: add manual pre-processing step or human-in-loop for data quality checks before agent triage. ROI may not justify automation.
- If data is clean: proceed as designed; agent extraction is viable.

---

### U2: Policy Coverage Validation Rules ⚠️ **CRITICAL UNKNOWN**

**What we don't know:**
- What types of policies exist? (auto, home, commercial, specialty?)
- For each type, what are coverage categories? How many?
- What does "validate coverage" mean? Is the claim type covered? Is amount within limits? Are exclusions triggered?
- Are policy rules encoded in SOAP endpoint response, or must agent apply business logic?
- Example: if claim is "accidental damage to laptop," does policy have "accidental coverage"? If yes, is it active? Is there a deductible?
- How are exclusions handled? (E.g., "coverage excluded if person was intoxicated" — can agent detect this from FNOL data?)

**Why it matters:**
- Determines whether validation is a simple policy lookup or a complex decision tree.
- If rules are simple (claim type in policy = approved), agent can validate autonomously.
- If rules require discretion (e.g., "fraud indicators suggest questionable claim"), agent should escalate.
- Affects routing: some complex validations may need specialist expertise.

**Hypothesis:**
If policy coverage rules are deterministic (policy has coverage type yes/no, amount limit is numeric, exclusions are Boolean), agent can apply them. If rules require judgment calls (e.g., "this looks fraudulent" or "this is a corner case"), validation needs specialist review.

**How I'd test it:**
- Request 3-5 anonymized policy documents from different types.
- Walk through with client: for each policy, what are coverage categories? How many? Numeric limits or qualitative ranges?
- Get a sample of 5 recent claims and their validation outcomes. Ask: "How did the specialist decide if coverage applied?"
- Confidence: **Low**. Without policy docs and validation examples, we're designing blind.

**Remediation if wrong:**
- If rules are simple: agent can validate; proceed as designed.
- If rules are complex/qualitative: agent does lookup, specialist does validation; agent is triage-only.

---

### U3: Adjuster Routing Logic and Workload Model ⚠️ **HIGH UNKNOWN**

**What we don't know:**
- How are 12 adjusters allocated across 300 claims/day? By specialty (auto vs home)? Geography? Rotation?
- What's their capacity model? Do they have targets (e.g., 25 claims/day per adjuster)?
- What's the current queue depth at day-end? Pending claims? Rework?
- Are adjusters overloaded? If volume goes +50%, can they absorb, or does quality suffer?
- How does agent pick an adjuster? Round-robin? Least-loaded? Skill match?
- If an adjuster is unavailable (sick, leave), what happens? Queue holds, or reassign?

**Why it matters:**
- Routing accuracy depends on understanding adjuster capacity.
- If specialist 1 is overloaded and gets routed 10 more claims, quality suffers; defeats automation goal.
- Affects design: agent may need to monitor queue depth and escalate or defer if capacity is full.
- Affects SLA: if adjuster is backed up, routed claim won't be processed within SLA anyway.

**Hypothesis:**
If adjusters average 25 claims/day with 20% rework, agent can route 250 of 300 claims to specialists; 50 need escalation or specialist-led review. If agent can identify the 50 before routing (via high-value/ambiguous flags), specialist handles only 50 complex + rework; improves overall throughput.

**How I'd test it:**
- Get adjuster workload data for 5 days: # claims per adjuster, # rework, pending at day-end.
- Ask: "Are there days when adjusters can't keep up? What happens?"
- Interview 2-3 adjusters: "What makes a claim hard to process? How many per day can you handle well?"
- Confidence: **Medium**. Can gather from current ops data, but future state is uncertain.

**Remediation if wrong:**
- If adjusters are already overloaded: automation alone won't improve SLA; need adjuster hiring or workflow redesign.
- If they have capacity: agent can route aggressively; volume automation is viable.

---

### U4: Definition of "High-Value" and "Ambiguous" Claims ⚠️ **HIGH UNKNOWN**

**What we don't know:**
- Is "high-value" defined by dollar amount (e.g., >$25K)? Or by claim type (e.g., commercial claims)?
- Is there a written policy, or is it adjuster judgment?
- What % of 300/day fall into high-value category?
- Is "ambiguous" defined? Or is it subjective?
- Examples of ambiguous claims: multiple injuries? Liability disputed? Incident unclear? All of the above?
- What % are ambiguous?
- Are high-value and ambiguous exclusive, or can a claim be both?

**Why it matters:**
- Determines delegation boundary: which claims can agent fully handle, which need specialist review, which need both?
- If high-value = >$25K and represents 10% of volume (30 claims/day), agent auto-routes 270 to specialists + 30 for high-value flag. Specialist reviews high-value only.
- If high-value is undefined, agent can't flag it; specialist must identify manually; no improvement.
- Directly affects success metrics: automation rate depends on percentage of routine vs complex claims.

**Hypothesis:**
If high-value and ambiguous claims represent 20-30% of volume, agent can fully automate 70-80% of routine claims. If they represent >50%, agent is mostly escalating; ROI is limited.

**How I'd test it:**
- Ask client: "What's your definition of high-value? Is it a dollar amount or a judgment call?"
- Request 5-10 high-value claims from the last month. Analyze: what makes them high-value? Common factors?
- Ask: "What % of daily volume is high-value? Ambiguous?"
- Confidence: **Very Low**. These are likely undefined or subjective; requires stakeholder workshop.

**Remediation if wrong:**
- If high-value is undefined: work with client to define (e.g., claims >$50K, or commercial policies, or disputed liability). Without definition, agent can't act.
- If ambiguous is undefined: define signals agent can detect (e.g., multiple injuries, liability unclear, incident type not listed). Start with conservative set; escalate more than necessary.

---

### U5: Claimant Acknowledgment Format and SLA ⚠️ **HIGH UNKNOWN**

**What we don't know:**
- What format? Email? SMS? Portal notification? Automated phone call?
- Can it be auto-generated, or must it be human-written?
- What must it contain? Claim number? Incident summary? Expected next steps? Timeline?
- Is there a regulatory requirement (e.g., "insurer must acknowledge within 24 hours")?
- Does SLA clock start at FNOL submission or at acknowledgment sent?
- Can we skip triage and route immediately (ack then process), or must we validate first?

**Why it matters:**
- Affects whether acknowledgment is part of critical path (adds latency) or parallel path (no latency impact).
- If auto-ack is forbidden or unacceptable to client, agent can't send one; must escalate to human.
- If ack is required within 30 min, agent has only 90 min for triage/validation/routing; tight budget.
- Affects design: if agent sends ack before validation, and validation fails, need follow-up communication to claimant.

**Hypothesis:**
If auto-email acknowledgment is acceptable and can be sent within 5 minutes of FNOL receipt, agent gains 115 minutes for triage/validation/routing; SLA becomes achievable. If ack must be human-written or have custom content, latency increases by 20+ minutes; SLA pressure rises.

**How I'd test it:**
- Show client a sample auto-generated ack: "Thank you for reporting your claim. Claim #ABC-123. We'll contact you within 24 hours with next steps."
- Ask: "Is this acceptable? Or must it be personalized / human-written?"
- Check state insurance regulator website: are there FNOL ack requirements?
- Confidence: **Medium**. Can research regulatory requirements; client answer needed for feasibility.

**Remediation if wrong:**
- If auto-ack is acceptable: proceed as designed; ack is fast, parallel to processing.
- If human-required: ack becomes bottleneck; SLA pressure increases; may need to hire additional ACK-only staff.

---

### U6: Current Error Root Cause Analysis ⚠️ **MEDIUM UNKNOWN**

**What we don't know:**
- The 18% routing error: what types? Wrong specialty assigned? Wrong adjuster overloaded? Claim type misclassified?
- The 31% SLA breach: where do claims stall? Triage? Validation? Routing? Or wait-time with specialist?
- Are errors due to specialist misinterpretation of FNOL data, system failures, adjuster overload, or process gaps?
- What is the cost of each error? (Claimant dissatisfaction? Rework time? Compliance penalties?)
- Is the error rate stable or trending?

**Why it matters:**
- If errors are root-caused in data quality (e.g., missing policy numbers), automation won't help; need data cleanup first.
- If errors are root-caused in adjuster overload, automation alone won't improve SLA; need staffing increase.
- If errors are root-caused in process (e.g., no standardized triage criteria), agent design can improve by making triage deterministic.
- Informs success metrics: what's realistic post-automation?

**Hypothesis:**
If 50%+ of routing errors are due to data misinterpretation (specialist misread ambiguous FNOL), agent NLP extraction + deterministic routing can reduce error rate. If 50%+ are due to adjuster overload (wrong specialist chosen because correct specialist is busy), agent routing won't help; need staffing change.

**How I'd test it:**
- Request 30 routing errors from the last month. For each, ask: "Why did this go to the wrong specialist?" Categorize: data issue, capacity issue, or process issue.
- Interview 2-3 specialists: "Tell me about your last misdirected claim. What went wrong?"
- Confidence: **Medium**. Can gather from current data; may need ops analysis.

**Remediation if wrong:**
- If data issues are primary: NLP extraction layer + validation can help; agent design is sound.
- If capacity issues are primary: automation is limited ROI; need staffing plan.

---

### U7: Integration Latency and Availability SLAs ⚠️ **MEDIUM UNKNOWN**

**What we don't know:**
- CRM API response time? 100ms? 500ms? 2s?
- Policy admin SOAP response time? 500ms? 3s? 5s?
- Document system response time? Available via API or manual file retrieval?
- What are the system uptime SLAs? 99%? 99.5%? 99.9%?
- Are there maintenance windows when systems are offline?
- Rate limits? Concurrent request limits?
- Retry behavior: if a system fails, what's the retry strategy?

**Why it matters:**
- If CRM is slow (>2s per query), agent processing time per claim climbs; SLA pressure increases.
- If policy system is slow (>5s per query), and agent calls it for every claim, processing time per claim is >5s; SLA becomes very tight.
- If systems are unreliable (95% uptime), agent needs queue + retry + fallback; adds infrastructure complexity.
- Affects design: do we batch queries, cache results, or accept latency?

**Hypothesis:**
If CRM <500ms, policy <3s, document <2s, all with >99% uptime, agent can process 300 claims/day within 2-hour SLA with 5-minute buffer. If any integration exceeds these targets, SLA becomes at-risk; need optimization (caching, batching) or relaxed SLA.

**How I'd test it:**
- Get API SLA docs from each system owner.
- Run load test: 100 concurrent requests to each API; measure response time distribution (p50, p95, p99) and error rate.
- Confidence: **Medium**. Requires system access; likely available in ops/IT documentation.

**Remediation if wrong:**
- If integrations are fast: proceed as designed; latency is not a bottleneck.
- If integrations are slow: add caching layer (Redis), batch queries, or async polling; design complexity increases.

---

### U8: Human-in-the-Loop Workflow Design ⚠️ **MEDIUM UNKNOWN**

**What we don't know:**
- When agent escalates (e.g., ambiguous claim flagged), how does specialist see it? In same queue as manual claims? Separate escalation queue?
- What information must agent provide? Summary? Full FNOL transcript? Triage reasoning? Flag reason?
- Can specialist override agent triage (e.g., agent said "not covered" but specialist says "yes, covered")? Is override easy or friction-heavy?
- Is there a feedback loop? Do specialist overrides retrain the agent, or are they logged but ignored?
- What's the SLA for specialist review of escalated claims?

**Why it matters:**
- If escalation workflow is clunky, specialists will ignore agent and revert to manual processing; defeats automation.
- If specialist can override with one click, they'll use agent as a starting point; adoption is high.
- If overrides are fed back to agent learning, agent improves over time; if isolated, error rate stays constant.
- Affects design: agent may need to provide extensive context (slower) to make escalation useful, or minimal context (faster) if specialist will re-analyze anyway.

**Hypothesis:**
If agent can escalate complex claims to specialists with full context (triage decision, confidence, flag reason), and specialist can accept/override in <1 minute, agent+specialist workflow improves throughput vs manual-only. If specialist must re-analyze from scratch, agent adds no value.

**How I'd test it:**
- Prototype escalation workflow: create a test claim, flag as ambiguous with context. Give to a specialist; ask: "Can you act on this? How long does review take?"
- Interview specialist: "If the agent pre-triated and flagged a claim as ambiguous, would that help you, or would you re-analyze from scratch?"
- Confidence: **Low**. Requires prototyping and specialist feedback; not available from current data.

**Remediation if wrong:**
- If specialist workflow is smooth and values agent context: proceed as designed; escalation is viable.
- If specialist ignores agent and re-analyzes: agent is purely a triage tool; value is reduced; may need design shift (e.g., agent fully automates low-value, only escalates truly ambiguous).

---

### U9: Success Metrics and Stakeholder Alignment ⚠️ **MEDIUM UNKNOWN**

**What we don't know:**
- Is primary goal speed (reduce SLA breach), quality (reduce errors), or cost (reduce headcount)?
- What's the trade-off tolerance? (E.g., "I'll accept 2% error increase if SLA breach drops from 31% to 5%?")
- What's the success threshold? 50% of claims fully automated? 70%? 90%?
- Will the 12 specialists stay, be retrained, or be reduced?
- Is there a phase 2 (e.g., "automate triage, then automate validation, then automate routing")?

**Why it matters:**
- Different goals lead to different agent designs. If optimizing for speed, agent routes aggressively; escalates conservatively. If optimizing for quality, agent escalates aggressively; routes conservatively.
- Determines acceptance criteria for the spec.
- Affects staffing and ROI model.

**Hypothesis:**
If primary goal is SLA improvement (31% → <10%), agent design prioritizes speed + deterministic routing. If primary goal is quality (18% errors → <5%), agent design prioritizes accuracy + escalation. If primary goal is cost (reduce headcount), agent needs to automate as much as possible, which may increase error rate.

**How I'd test it:**
- Ask C-level stakeholder: "If we can automate 60% of claims, reduce SLA breach to 5%, but maintain error rate at 15%, is that a success?"
- Ask: "What's the ideal future state? Same team but faster? Smaller team? Different team skills?"
- Confidence: **Low**. Requires executive alignment; likely not documented.

**Remediation if wrong:**
- If speed is primary: prioritize SLA; accept modest error rate increase; design accordingly.
- If quality is primary: prioritize accuracy; accept that many claims escalate; design conservatively.

---

### U10: Pilot Scope and Rollout Plan ⚠️ **LOW UNKNOWN**

**What we don't know:**
- Is this a full deployment or a pilot?
- If pilot: what's the scope (10% of claims? Specific types? Specific geography)?
- How long is the pilot (1 week? 1 month?)?
- What's the success gate for moving from pilot to full deployment?
- Is the client willing to accept automation risk (agent failures, escalation surge) for learnings?
- Will the pilot run in shadow mode (agent processes but doesn't route; specialist does), or canary mode (agent routes to subset), or full mode (agent routes all pilot claims)?

**Why it matters:**
- Pilot scope dictates spec scope. Spec for 30 claims/day (pilot) is different from 300 claims/day (full).
- Shadow mode means agent can learn without risk; spec can be less bulletproof. Full mode means spec must be production-ready day-1.
- Affects deployment and rollback strategy.

**Hypothesis:**
If this is a full deployment (300 claims/day immediately), spec must be very tight; error tolerance is low. If this is a 10% pilot (30 claims/day) in shadow mode, spec can have rough edges; we can learn and iterate.

**How I'd test it:**
- Ask: "Are we building for production day-1, or are you comfortable with a learning phase?"
- Confirm: "Is this a full deployment or a pilot? If pilot, what's the scope?"
- Confidence: **Medium**. Likely part of sales/scoping conversation, but often not formally documented.

**Remediation if wrong:**
- If pilot: design for learning; accept some failures; spec can be less final.
- If full deployment: design for production; spec must be bulletproof; no margin for error.

---

## Section 9: Assumption Impact Matrix

| Assumption | Impact if Wrong | Mitigation | Priority |
|---|---|---|---|
| A1: 2-hour SLA is absolute | If relaxed to 4-6 hours, latency budget becomes less tight; design simplifies | Confirm with stakeholder in week-1 alignment | HIGH |
| A4: Coverage validation is deterministic | If subjective/complex, agent can't validate; escalates everything; ROI zero | Walk through 5 policy docs with underwriter | HIGH |
| A6: FNOL data >85% complete | If <70% complete, extraction fails; escalation rate climbs to 40%+ | Request 50 real FNOL samples; analyze completion | CRITICAL |
| A22: High-value claims are defined | If undefined, agent can't flag; manual segregation continues; no improvement | Define with stakeholder (e.g., claims >$25K) | HIGH |
| A23: Ambiguous signals are detectable | If no patterns, can't teach agent; over-escalate or miss ambiguity | Review 5 ambiguous claims; extract patterns | HIGH |
| A24: 80%+ of claims are automatable | If <50%, ROI is limited; may need workflow redesign | Estimate from current data | MEDIUM |
| A30: SLA improvement is primary goal | If quality is primary, agent design is too conservative; escalation rate climbs | Confirm with C-level | HIGH |

---

## Section 10: Test Plan & Validation Roadmap

| Unknown | Test | Owner | Timeline | Confidence→Assumption |
|---|---|---|---|---|
| U1: FNOL data quality | Analyze 50 real FNOL samples (structure, completeness, format) | Client Ops | Week 1 (Day 1-2) | U1a: >85% complete AND <10% non-standard ⟹ proceeding assumption |
| U2: Policy coverage rules | Walk through 5 policy docs; ask for 10 validation examples | Client Underwriting | Week 1 (Day 2-3) | U2a: Rules are deterministic (Boolean/numeric, no judgment calls) ⟹ proceeding assumption |
| U3: Adjuster routing & capacity | Get 5-day workload data; interview 2-3 adjusters | Client Ops | Week 1 (Day 2-3) | U3a: Adjusters have 20% spare capacity; routing can be deterministic ⟹ proceeding assumption |
| U4: High-value & ambiguous definitions | Stakeholder workshop; review 5 examples of each | Client Leadership + Ops | Week 1 (Day 1) | U4a: High-value = >$25K; ambiguous = 3+ signals present ⟹ proceeding assumption |
| U5: Claimant ack format & SLA | Show sample auto-email; check state regulations | Client Customer Service + Legal | Week 1 (Day 1) | U5a: Auto-email ack acceptable; regulatory OK ⟹ proceeding assumption |
| U6: Error root cause | Analyze 30 misdirected claims; categorize | Client Ops + Quality | Week 1 (Day 2) | U6a: >50% errors are data-driven (not capacity) ⟹ NLP layer is valuable |
| U7: Integration latency/uptime | Get SLA docs; run load test (100 concurrent requests) | Client IT / System Owners | Week 1 (Day 2-3) | U7a: <500ms (CRM), <3s (policy), >99% uptime ⟹ proceeding assumption |
| U8: Specialist workflow | Prototype escalation; get specialist feedback | Client Ops | Week 1 (Day 3) | U8a: Specialist can act on escalated claim in <2 min ⟹ escalation viable |
| U9: Success metrics & stakeholder alignment | Executive alignment meeting (problem statement validation) | Client Leadership | Week 1 (Day 1) | U9a: Primary goal = SLA <10% breach; secondary = <5% error ⟹ proceeding assumption |
| U10: Pilot scope | Ask: full deployment or pilot? | Client Leadership | Week 1 (Day 1) | U10a: Full deployment (300/day immediately) ⟹ spec must be production-ready |

---

## Section 11: Confidence Assessment Summary

| Category | Confidence Level | Reason | Risk |
|---|---|---|---|
| **Business/SLA** | High | Stated in scenario; unlikely to change. | Low |
| **Specialist workflow** | Medium | Can infer from current ops; future state TBD. | Medium |
| **System integration** | Medium | Can get from IT; depends on current deployment. | Medium |
| **Data quality** | **Low** | No samples provided; highly variable. | **CRITICAL** |
| **Policy rules** | **Low** | No samples provided; depends on complexity. | **CRITICAL** |
| **Delegation boundaries** | Low | Depends on high-value/ambiguous definitions (TBD) and data quality. | **CRITICAL** |
| **Success metrics alignment** | Low | Requires stakeholder workshop; not documented. | High |

**Overall Confidence: MEDIUM-LOW**

This is a high-risk spec. The three CRITICAL unknowns (data quality, policy rules, delegation boundaries) are load-bearing. If any is wrong, the entire solution fails or requires major redesign.

**Proceeding Assumption:** We assume week-1 stakeholder workshops will resolve U1, U2, U4, U5, U9 (80% confidence gain). If any cannot be resolved, spec will be marked with [TODO] and will require customer decision before build.

---

## Section 12: Delegation Boundary Assumptions (Justifying Where Agent Touches vs Human)

### Fully Agentic (No Human Touch)

| Task | Assumption | Confidence | Vulnerability |
|---|---|---|---|
| **Triage by severity** | FNOL data contains sufficient incident information (type, claimant injury status, asset value) to classify into predefined severity buckets (critical/high/medium/low) without judgment. | Medium | If severity is subjective (e.g., "moderate damage requires judgment"), human review needed; agent can't triage autonomously. |
| **Acknowledge claimant (email)** | Auto-email acknowledgment is acceptable per regulation + client policy. Template can be standardized (claim number, next-steps boilerplate). | Medium | If regulatory requirement exists or client culture demands human touch, auto-ack is not permissible. |
| **Route to adjuster (round-robin)** | Adjuster assignment is purely mechanical (least-loaded, or rotation). No skill matching required. | Medium-Low | If routing requires specialty/skill match, agent can't decide; needs specialist registry + matching logic. Adds complexity. |
| **Extract structured data from FNOL** | Unstructured text (email/phone/form) contains all critical fields with reasonable consistency. NLP extraction can achieve >95% accuracy on 80% of claims. | Low | If data is garbage (missing policy #, incoherent text), NLP fails; escalation rate climbs. |

### Agent-Led with Human Oversight

| Task | Assumption | Confidence | Vulnerability |
|---|---|---|---|
| **Validate against policy coverage** | Policy coverage rules are deterministic (look up claim type → is it covered? Check amount → is it within limit? Check exclusions → applied?). Agent applies rules; specialist spot-checks or reviews escalations. | Medium | If rules require judgment (e.g., "does this injury fall under mental health coverage?"), agent can't apply rules autonomously; specialist always needed. |
| **Flag high-value claims** | High-value is defined as dollar amount (e.g., >$25K) or policy type (e.g., commercial). Agent flags; specialist reviews. | Low | If high-value is undefined or subjective, agent can't flag autonomously. |
| **Flag ambiguous claims** | Ambiguous claims have detectable signals (e.g., multiple injuries, liability disputed, incident unclear). Agent flags if signals present; specialist decides if actually ambiguous. | Low | If ambiguity is purely subjective, agent can only use weak proxies; false positives are common. |

### Human-Led with Agent Support

| Task | Assumption | Confidence | Vulnerability |
|---|---|---|---|
| **Complex validation** | If claim fails basic validation (coverage not found, amount exceeds limits, fraud flags), specialist reviews agent's reasoning and makes final call. Agent gathers data and flags; specialist decides. | High | If specialist review SLA is slow (>4 hours for escalations), escalated claims miss 2-hour SLA anyway; defeats purpose. |
| **Resolve coverage disputes** | Specialist has authority to override agent coverage decision (e.g., "agent said not covered, but we'll cover it as goodwill"). Agent provides reasoning; specialist decides. | High | None — specialist judgment is expected here. |

### Human-Only (No Agent)

| Task | Assumption | Confidence | Vulnerability |
|---|---|---|---|
| **Complex claims requiring judgment** | Claims that are truly ambiguous, high-value, or involve discretionary decisions remain specialist-only. Agent does not intervene. | High | If agent over-escalates, specialist becomes bottleneck; throughput doesn't improve. |

**Delegation Boundary Risk:** If data quality is poor (U1), or policy rules are complex (U2), or high-value/ambiguous definitions are undefined (U4), the boundaries shift left (more escalation, less automation).

---

## Section 13: Recommendation & Gating

### **Pre-Build Gate Checklist**

Before agent specification (Deliverable 3) is finalized, the following unknowns MUST be resolved:

- [ ] **U1 (Data Quality):** 50 real FNOL samples analyzed; completion rate measured; format variation assessed. ✓ **Decision:** Proceed / Redesign / Escalate
- [ ] **U2 (Policy Rules):** 5 policy docs reviewed; 10 validation examples walked through; rule complexity assessed. ✓ **Decision:** Proceed / Escalate coverage to specialist-review only
- [ ] **U4 (High-Value & Ambiguous Definitions):** Operationally defined with stakeholder sign-off. ✓ **Decision:** Proceed / Re-define
- [ ] **U9 (Success Metrics):** Stakeholder alignment on primary goal (SLA vs quality vs cost). ✓ **Decision:** Proceed / Reprioritize

**If all four gates pass:** Proceed to agent specification with HIGH confidence.
**If any gate fails:** Spec requires modification; flag as [PENDING] until resolved.

---

## Section 14: Closing Reflection

This spec rests on 30+ assumptions, 10 of which are genuine unknowns with low-to-medium confidence. The three highest-risk unknowns are **FNOL data quality, policy rule complexity, and ambiguity/high-value definitions.** These are load-bearing: if wrong, the delegation model collapses, and the agent design becomes either (a) over-conservative (escalates 70%+ of claims, defeating ROI), or (b) dangerously permissive (auto-approves poor-quality claims, creating compliance risk).

**Honest assessment:** Without real data samples and stakeholder alignment, this spec is **80% speculative.** The other 20% (basic SLA, system integration patterns) is solid.

**Path forward:** Dedicate Week 1 to resolving U1, U2, U4, U9 via structured stakeholder interviews and data analysis. Use the 10-minute coach walkthrough to pressure-test delegation boundaries against these unknowns. If coaches flag misalignment, escalate to customer for clarification before build.

**Final confidence:** Ready to build IF U1-U2-U4-U9 gates pass. If any fails, agent spec requires substantial revision. Recommend phased approach (shadow-mode pilot, 30 claims/day for 1 week) to validate assumptions under real load before full 300-claim deployment.

---

## Appendix: Assumption Confidence Scale

- **High (90%+):** Stated in scenario, confirmed by IT/ops, or obvious from domain. Risk of being wrong is low.
- **Medium (60-80%):** Inferred from scenario context, but requires stakeholder confirmation. Risk is real; worth testing.
- **Low (<60%):** Speculation or requires real data. High risk if wrong; may require major design change.

---

**Document Version:** 1.0  
**Author:** Nadia Rahmatulla  
**Date:** [Gate 1 Submission]  
**Status:** Ready for Coach Walkthrough

---

*This document is part of the Gate 1 Exam deliverable set. See Deliverables 1-5 for problem statement, delegation analysis, agent specification, and validation design.*
