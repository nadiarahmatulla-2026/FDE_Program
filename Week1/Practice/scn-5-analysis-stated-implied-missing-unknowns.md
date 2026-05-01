# Scenario 5 Analysis: What's Stated, Implied, Missing, and Unknown

**Objective:** Before drafting the five specification deliverables, systematically extract what the scenario tells us, what we can infer, what is obviously missing, and what genuine unknowns exist that must be validated or assumed.

---

## Part 1: What Is Explicitly Stated

### Business Context
- **Organization:** Mid-size insurance company
- **Claims process:** FNOL (first-notice-of-loss) processing
- **Current volume:** 300 reports per day
- **Current team size:** 12 specialists
- **Current handling time:** 22 minutes per claim (average)
- **Current error rate (routing):** 18%
- **Current SLA breach rate:** 31%

### Requirements
- **Triage:** By severity
- **Validation:** Against policy coverage
- **Routing:** To appropriate adjuster
- **Acknowledgment:** To claimant
- **SLA:** All within 2 hours of receipt

### Input Format
- Unstructured text from three sources: email, phone transcript, web form

### Client Appetite
- "Open to full automation where appropriate"
- "Insist on human oversight for high-value or ambiguous claims"

### Systems Available
- Modern CRM with APIs
- Legacy policy administration system with SOAP endpoints
- Document management system
- **No AI infrastructure today**

---

## Part 2: What Is Implied

### From the Numbers

| Implication | Derivation |
|---|---|
| **Current capacity constraint** | 12 people × 22 min/claim = 264 minutes ≈ 4.4 person-hours consumed per day. With 300 claims, team is at or above full utilization. |
| **Current quality crisis** | 18% routing error rate = ~54 misdirected claims/day. 31% SLA breach = ~93 claims not handled in 2 hours. Quality >> capacity is the problem. |
| **Manual triage is inconsistent** | Routing errors are likely due to manual interpretation of unstructured data. Severity triage probably varies by person. |
| **2-hour SLA is tight** | With 300 claims spread across 12 people, if each takes 22 min, some will breach immediately. The SLA is partly a process bottleneck, not just an efficiency problem. |
| **Claimants experience poor service** | 31% of claimants don't get acknowledgment within 2 hours. First notification of loss is emotionally salient (car accident, home damage, theft). Delay is high friction. |

### From the Systems

| Implication | Derivation |
|---|---|
| **API-first CRM means modern data** | CRM with APIs suggests structured customer data, accessible programmatically. Less data wrangling in CRM layer. |
| **Legacy policy system is a bottleneck** | SOAP endpoints (not REST) suggests older, slower interface. Policy lookup will be integration friction. Likely slower than CRM. |
| **No modern data warehouse** | Scenario mentions three systems but no mention of a data lake, warehouse, or unified data API. Agent will need to orchestrate multiple calls. |
| **Document management exists** | Suggests claims documents (photos, police reports, medical records) are digitized or can be digitized. But how are they retrieved? No endpoint mentioned. |

### From the Client's Framing

| Implication | Derivation |
|---|---|
| **Humans will stay in the loop** | "Insist on human oversight" means this is not a full automation play. Some decision gates will require human judgment. |
| **High-value and ambiguous are human gates** | These are the two categories the client explicitly mentioned. Implies low-value and clear should be agent-led. |
| **No AI platform exists** | Client will need to stand up infrastructure, APIs, deployment, monitoring. This is not a vendor plugin scenario. |
| **Client is not anti-AI, but cautious** | "Open to" rather than "demanding" suggests exploratory appetite, not existential threat concern. They're testing. |

---

## Part 3: What Is Explicitly Missing

### Data and Systems
- **Sample FNOL data:** No example of what an unstructured FNOL report looks like. Email format? Phone transcript markup? Web form schema? Severity categories? Policy coverage rules?
- **CRM API contract:** No endpoint documentation. No auth scheme. No data schema.
- **Policy admin SOAP contract:** No WSDL reference. No example request/response. No policy structure.
- **Document management API:** Not even mentioned as having an API. How does agent retrieve claim documents?
- **Definition of "severity":** No triage categories (critical, high, medium, low?). No criteria for severity classification.
- **Definition of "policy coverage":** Are we validating that the claim type is covered? That it's within limits? Both?
- **Definition of "appropriate adjuster":** Routing logic is unknown. By specialty? By workload? By geography? Rotation?
- **Definition of "high-value" claim:** What is the dollar threshold, if any?
- **Definition of "ambiguous" claim:** What makes a claim ambiguous?

### Process and Business Rules
- **Current process flow:** Step-by-step, as executed by specialists. How do they triage? What data do they use for validation? What's the decision tree?
- **Escalation rules:** When does a specialist escalate? To whom? What happens then?
- **Claimant acknowledgment format:** Email? SMS? Phone call? Portal notification? What must it contain?
- **SLA clock:** Starts when FNOL is received? Or when it enters the system?
- **SLA penalties:** What happens if missed? Regulatory fine? Customer credit? Internal metric?
- **Integration with downstream systems:** Once routed to adjuster, what happens next? Does adjuster receive it in a queue? An email? Does agent create a task in CRM?
- **Working hours:** Is FNOL processing 24/7 or business hours only? How does that affect SLA?

### Compliance and Governance
- **Applicable regulations:** State insurance commission regulations? HIPAA (medical data)? No mention.
- **Audit requirements:** Are all decisions logged? Who can access logs?
- **Claimant communication SLAs:** Are there regulatory minimums for acknowledging a claim?
- **Error tolerance:** Is 18% routing error acceptable? What error rate would the client want after automation?

### Integration and Infrastructure
- **Authentication:** How does agent authenticate to CRM, policy system, document system?
- **Network security:** Is this on-prem, cloud, hybrid?
- **Failure handling:** What if policy system is down? Do we queue and retry, or escalate?
- **Rate limits:** How many concurrent FNOL reports can the systems handle?
- **Data residency:** Where must data live (US, state-specific, etc.)?

### Validation and Success Metrics
- **Target metrics:** What does "good" look like? 5% routing error? 95% SLA compliance? 50% of claims fully automated?
- **Rollout plan:** Is this a pilot (10% of volume) or full deployment?
- **Human adjuster capacity:** Will the 12 specialists stay? Or scale with automation?

---

## Part 4: Genuine Unknowns (At Least 5)

### U1: FNOL Input Data Structure and Quality
**What we don't know:**
- What does an unstructured FNOL report actually look like? How much variation is there between email, phone transcript, and web form?
- Are emails full paragraphs, bullet lists, or semi-structured forms?
- Are phone transcripts human-written summaries or ASR-generated?
- How much key information (claimant name, claim amount, policy number, incident date, incident type) is present in every FNOL?
- What is the data quality? Missing fields? Typos in policy numbers? Dates in different formats?
- Are there embedded documents (photos, police reports, medical records) already attached, or does the agent need to request them separately?

**Why it matters:**
- Dictates how much NLP/extraction the agent needs to do before validation/routing.
- If structured, agent can validate immediately. If highly unstructured, agent may need to ask clarifying questions (human handoff).
- Data quality determines false-positive rate (agent thinks validation passes, but data is garbage).

**How I'd test it:**
- Ask client for 20 real FNOL reports (anonymized). Analyze variation in structure, completeness, and quality.
- Test extraction accuracy on a sample with an LLM.
- Confidence: **low**. Without sample data, we're designing blind.

---

### U2: Policy Coverage Validation Rules
**What we don't know:**
- What are the policy types? (auto, home, life, commercial, umbrella, etc.)
- For each type, what are the coverage categories? (collision, comprehensive, liability, etc.)
- What does "validate against coverage" mean? Check claim type is covered? Check amount is within limits? Check exclusions don't apply?
- How are exclusions encoded? Business rules? Policyholder choices? Date-based (e.g., deductible tiers)?
- Are policies stored in the legacy SOAP system? If so, what is the response time for a policy lookup?
- Is the policy admin system read-only or can it be queried?
- Are there dependencies (e.g., "medical coverage is only active if mental health rider is added")?

**Why it matters:**
- Determines whether validation is a simple lookup or a complex rule engine.
- If rules are complex or ambiguous, validation may need human review.
- If policy system is slow (>5s latency), agent may need to batch queries or fall back to a local cache.

**How I'd test it:**
- Ask for a policy document (anonymized). Walk through with client: how many claim types? How many coverage categories? What exclusions matter?
- Run a policy lookup against SOAP endpoint. Measure latency for 10 queries across different policy types.
- Confidence: **low to medium**. We can infer complexity from policy documents, but implementation details (SOAP response schema, exclusion logic) are unknown.

---

### U3: Adjuster Routing Logic and Capacity Model
**What we don't know:**
- How are the 12 specialists allocated? By claim type (e.g., auto, home)? By geography? By skill level?
- What is the workload model? Do they have capacity targets? Can we see their queue depth?
- How does the agent route? Round-robin? Least-loaded? Specialty match?
- What if an adjuster is overloaded? Does the agent queue the claim or escalate?
- Are there on-call specialists for high-priority claims?
- Is routing dynamic (can we re-route if an adjuster says "I can't take this")? Or static assignment?
- Will automation reduce the adjuster team size, or expand their throughput?

**Why it matters:**
- Routing to an overloaded adjuster defeats SLA improvement.
- If we automate triage but still hand off to overloaded humans, we haven't solved the problem.
- Affects delegation boundary: if adjuster capacity is limited, some claims may need escalation to automation (full resolution) rather than adjuster assignment.

**How I'd test it:**
- Get adjuster schedule and workload data for 5 days. How many claims per adjuster? How many are pending at end of day?
- Interview a specialist: how do you decide which claims to take? What makes a claim unassignable?
- Confidence: **medium**. Can gather from current data, but future state (post-automation) is speculative.

---

### U4: Definition of "Ambiguous" and "High-Value" Claims
**What we don't know:**
- What makes a claim ambiguous? Multiple injuries? Liability dispute? Unclear incident description?
- Is there a defined list, or is it adjuster judgment?
- What is the current percentage of claims flagged as ambiguous?
- What is the dollar threshold for "high-value"? $5K? $50K? $500K?
- Are "high-value" and "ambiguous" exclusive, or can a claim be both?
- For ambiguous claims, who decides: the agent (escalate) or the adjuster (review on handoff)?
- For high-value claims, do we skip validation/routing and go straight to human?

**Why it matters:**
- Determines which claims stay human-led vs become agent-led.
- If definition is loose, agent will over-escalate (defeating automation gains) or over-automate (compliance risk).
- Affects success metrics: what % of claims should be fully automated vs agent-led?

**How I'd test it:**
- Ask client: "Show me 5 claims you flagged as ambiguous last month. What did each one have in common?"
- Ask: "Is there a dollar amount you'd flag as high-value? What's your tolerance for losing a high-value claim to automation?"
- Confidence: **low**. These are subjective categories. Likely to change post-automation.

---

### U5: Claimant Acknowledgment Expectations and SLA Definition
**What we don't know:**
- What format? Email? SMS? Automated phone call? Portal notification?
- What must the acknowledgment contain? Claim number? Estimated timeframe? Next steps?
- Can we auto-generate, or must it be human-written?
- Is there a regulatory requirement (e.g., "insurer must acknowledge within X hours")?
- Does the SLA clock start when FNOL is submitted, or when it's acknowledged?
- If we miss 2 hours but later process faster, does that count as a breach?
- Are there different SLAs for different severity levels? (Urgent claims < 30 min, routine < 2 hours?)

**Why it matters:**
- Affects whether acknowledgment is fully automated, agent-generated, or human-written.
- Determines SLA baseline: if acknowledgment takes 5 minutes, we have 115 minutes for triage/validation/routing.
- Regulatory compliance risk if we get this wrong.

**How I'd test it:**
- Ask for a recent acknowledgment email. Show it to 3 claimants: does it feel automated or human? What information is important to them?
- Check state insurance regulator for FNOL acknowledgment requirements.
- Confidence: **medium**. Can gather from existing documentation, but regulatory details may require research.

---

### U6: Current Error Attribution and Root Causes
**What we don't know:**
- The 18% routing error: what types of errors are they? Wrong claim type assigned? Wrong adjuster specialty? Claims routed to people on leave?
- The 31% SLA breach: is it time to acknowledgment, or end-to-end resolution?
- Are errors due to human misinterpretation of unstructured data, system failures, or adjuster overload?
- What is the cost of each error? Claimant dissatisfaction? Rework? Compliance penalties?
- Is the error rate stable or trending?

**Why it matters:**
- If errors are due to data quality, automation may not help — or may make it worse (LLM hallucination).
- If errors are due to adjuster overload, automation of triage won't help; we need capacity increase.
- If errors are due to process confusion, better structure (agent-driven) could improve.
- Success metrics depend on understanding root causes.

**How I'd test it:**
- Request 50 misdirected claims from the last month. Categorize: data issue, process issue, capacity issue, or unknown.
- Interview a specialist: "Tell me about the last claim you misrouted. What went wrong?"
- Confidence: **low**. Without root-cause analysis, we're guessing.

---

### U7: Integration Latency and Availability Expectations
**What we don't know:**
- What is the SLA for each system? CRM API uptime? Policy SOAP endpoint uptime? Document system availability?
- What are typical response times? CRM lookup: 100ms? 1s? Policy lookup: 500ms? 5s?
- Rate limits? Can the agent make 1000 policy queries per day, or are there strict limits?
- Retry behavior: if policy lookup fails, do we retry immediately, queue for later, or escalate?
- What happens if document system is down? Do we process without documents, or escalate?
- Are there maintenance windows when systems are unavailable?

**Why it matters:**
- If policy system is slow (>5s), processing time per claim suffers.
- If systems are unreliable, agent design must include fallbacks/queuing.
- Affects SLA achievability: 2-hour SLA means we need sub-10-minute processing per claim; if integrations take 30+ seconds, we're already tight.

**How I'd test it:**
- Get API SLAs and response time benchmarks from client's current integrations.
- Run a load test: simulate 100 concurrent FNOL submissions and measure end-to-end latency.
- Confidence: **low**. Requires actual system access; scenario doesn't provide this.

---

### U8: Human Oversight Workflow Design
**What we don't know:**
- When agent escalates a claim for human review, how does the specialist see it? Same queue as manual claims? Separate "escalation" queue?
- What information must the agent present to help the specialist decide? Summary? Full transcript? Flag reason?
- Can the specialist override the agent's triage/validation? If so, what happens to that data (feedback loop)?
- Is there an SLA for specialist review of escalated claims? (e.g., "review within 15 minutes")
- Can the agent learn from specialist overrides, or is each review isolated?

**Why it matters:**
- If escalation workflow is friction-heavy, specialists will reject it and revert to manual processing.
- If there's no feedback loop, agent doesn't learn from failures; error rate won't improve.
- Affects design: agent might need to pre-fetch more information (slower) to help specialists, or might minimize handoff effort (faster but less context).

**How I'd test it:**
- Walk through a prototype: agent escalates a claim; show specialist the UI. Does specialist understand why? Can they act quickly?
- Ask specialist: "If the agent gives you a summary, is that helpful? Or do you need the full transcript?"
- Confidence: **low**. Requires prototyping and specialist feedback.

---

### U9: Success Metrics and Stakeholder Alignment
**What we don't know:**
- What does the client want to optimize? Volume (process more claims)? Quality (fewer errors)? Cost (fewer specialists)? SLA (faster turnaround)?
- Is there a trade-off tolerance? (e.g., "I'll accept 2% error rate increase if we reduce SLA breach to 5%")
- What is the success threshold? 50% fully automated? 95% SLA met? <5% routing error?
- Is this a cost-cutting exercise, or a service improvement?
- Will the 12 specialists stay after automation? Retrain to more complex claims? Reduce headcount?

**Why it matters:**
- Different success definitions lead to different agent behaviors. (E.g., if optimizing for volume, agent routes aggressively; if optimizing for quality, agent escalates more.)
- Affects validation design: what do we measure?
- Affects delegation boundaries: if cost-cutting is primary, more claims automate; if service is primary, more specialist oversight.

**How I'd test it:**
- Ask client: "If we automate 60% of claims, reduce SLA breach to 5%, but maintain error rate at 15%, would that be a success?"
- Probe: "What happens to the specialist team?"
- Confidence: **low**. Requires executive alignment.

---

### U10: Pilot Scope and Rollout Plan
**What we don't know:**
- Is this a full deployment or a pilot? If pilot, what's the sample (10% of volume? Specific claim types? Specific geographies)?
- How long is the pilot? 1 week? 1 month?
- What's the success gate for moving from pilot to full deployment?
- Is the client willing to accept automation risk (agent failures, escalation surge) in exchange for learnings?

**Why it matters:**
- Pilot scope dictates spec scope. Full automation spec is different from a 10% pilot.
- Affects validation: if pilot, we can learn and iterate; if full deployment, spec must be more bulletproof.
- Determines how we handle bootstrap (e.g., don't need full specialist feedback loop if piloting to 10%).

**How I'd test it:**
- Ask: "Are we building for production day-1, or are you comfortable with a learning phase?"
- Confidence: **low**. Likely to be part of sales/scoping conversation, not documented.

---

## Summary: Unknowns vs Assumptions

| Unknown | Severity | Assumption Strategy |
|---------|----------|---------------------|
| U1: FNOL input data structure | Critical | Assume moderately structured inputs (80% have policy#, claimant name, incident type); unstructured free-form text for detailed description |
| U2: Policy coverage validation rules | Critical | Assume policy lookup is a read-only SOAP query returning policy details; agent does basic coverage-type matching, not complex exclusion evaluation |
| U3: Adjuster routing logic | High | Assume round-robin or least-loaded assignment; agent does not predict adjuster availability (that's current system's job) |
| U4: "Ambiguous" and "high-value" definitions | High | Assume: high-value = claims > $25K; ambiguous = no clear incident description or liability disputed. Agent flags both for specialist review. |
| U5: Acknowledgment SLA details | High | Assume: auto-email acknowledgment within 2 hours is acceptable; regulatory guidance doesn't forbid automation; can include claim number and "we'll contact you within 24 hours" |
| U6: Root cause analysis of errors | Medium | Assume: routing errors are mix of data quality (50%) and adjuster overload (50%); agent design improves data quality path, doesn't solve adjuster capacity |
| U7: Integration latency/availability | High | Assume: CRM <500ms, policy SOAP <3s, document system <2s; all have 99.5% uptime; retry on 5xx, fail-fast on 4xx |
| U8: Human oversight workflow | High | Assume: specialist sees flagged claims in a dedicated queue; can override agent decision; overrides are logged but don't auto-feed back to agent |
| U9: Success metrics / stakeholder priorities | Critical | Assume: primary goal is SLA improvement (31% → <10% breach); secondary is quality (18% errors → <5%); volume automation is nice-to-have, not primary |
| U10: Pilot scope and rollout | Medium | Assume: full deployment, not pilot; all 300/day claims in scope; 2-week turnaround for spec → build → initial testing |

---

## Next Steps

With these unknowns named, we can now build the five deliverables:

1. **Problem Statement & Success Metrics** — will rest on assumptions about SLA impact and quality baselines
2. **Delegation Analysis** — will rest on assumptions about "high-value," "ambiguous," and specialist workflow
3. **Agent Specification** — will rest on assumptions about data structure, policy rules, and integration latency
4. **Validation Design** — will rest on assumptions about acceptable error rates and failure modes
5. **Assumptions & Unknowns** — will surface all 10+ unknowns with tests and confidence levels

Each deliverable will flag dependencies on these unknowns, making it explicit where validation is needed before build.
