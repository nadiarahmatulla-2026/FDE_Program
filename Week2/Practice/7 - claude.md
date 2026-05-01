# CLAUDE.md — Week 2: Agentic Transformation (ATX) Workflow

## Project Purpose

This project assists FDE participants in executing **Agentic Transformation (ATX)** — the methodology for decomposing cognitive work, assessing delegation suitability, and designing agents that fit real business processes.

You are not building the agent directly. You are building the **cognitive and governance artefacts** that make agent-building possible: load maps, delegation matrices, purpose documents, and discovery strategies. These artefacts are the bridge between business problems and buildable agent specifications.

The Week 2 workflow runs once per participant per week:
1. **Monday–Thursday**: Apply ATX to a practice scenario; produce cognitive load map, delegation suitability matrix, and agent purpose document
2. **Thursday 14:15 CET**: Submit artefacts for peer cross-review
3. **Friday**: Timed gate exercise under seal (same ATX methodology, previously unseen scenario)

Your job is to help participants produce **precise, honest, non-speculative cognitive work analysis** under time pressure.

---

## Core Entities

### Participant ATX Submission
A complete artefact produced by one participant for one practice scenario.

Attributes:
- `scenario_domain`: string (e.g., "B2B SaaS billing reconciliation", "healthcare insurance claim assessment")
- `work_streams`: array of WorkStream objects (see below)
- `primary_target`: string (identifier of highest-priority agent opportunity)
- `cognitive_load_map`: CLM object (see below)
- `delegation_suitability_matrix`: DSM object (see below)
- `volume_value_analysis`: VVA object (see below)
- `agent_purpose_document`: APD object (see below)
- `system_data_inventory`: SDI object (see below)
- `discovery_questions`: array of DiscoveryQuestion objects (see below)
- `assumptions_log`: array of Assumption objects (see below)
- `submission_date`: ISO 8601 timestamp
- `submission_status`: enum [DRAFT, PEER_REVIEW_READY, PEER_REVIEWED, GATE_SUBMISSION]

#### WorkStream
Represents one coherent group of cognitive activities within the scenario.

Attributes:
- `name`: string, identifier for the work stream (e.g., "intake", "validation", "resolution", "reporting")
- `description`: string, what cognitive outcomes this stream produces
- `volume_estimate`: "high" | "medium" | "low" (relative to other streams in scenario)
- `jobs_to_be_done`: array of JTBD objects (see below)
- `is_decomposed`: boolean (true if this stream is mapped in detail in cognitive_load_map)

#### JTBD (Job to be Done)
A cognitive contract: the actor, the intent, the decision points, the outcome.

Attributes:
- `name`: string (e.g., "Establish customer intent from incoming request")
- `actor`: enum ["human", "system", "agent", "hybrid"]
- `cognitive_zones`: array of CognitiveZone objects (see below)
- `breakpoints`: array of Breakpoint objects (see below)
- `systems_and_data`: array of strings (list of systems/data sources this JTBD touches)
- `decision_points`: array of strings (what must be decided, not merely executed)
- `failure_modes`: array of strings (what does poor execution look like)
- `is_structurable`: boolean (can this job be expressed as a rule, or does it require judgment?)
- `tacit_knowledge_required`: boolean (does this require expert interpretation?)

#### CognitiveZone
A cluster of similar cognitive activity within a JTBD.

Attributes:
- `name`: string (e.g., "intent recognition", "data validation", "policy selection", "exception diagnosis")
- `micro_tasks`: array of strings (enumerate the specific tasks in this zone)
- `error_tolerance`: enum ["zero", "low", "medium", "high"] (consequence of getting it wrong in this zone)
- `latency_sensitivity`: enum ["critical", "high", "medium", "low"] (does speed matter?)
- `data_dependencies`: array of strings (what data is read to perform this zone)
- `human_expertise_required`: string | null (e.g., "domain expert", "compliance officer", null if not required)

#### Breakpoint
A control handoff: where a decision or action crosses a boundary (customer→agent, system→human, rule→judgment).

Attributes:
- `name`: string (e.g., "customer dispute escalation request")
- `trigger`: string (when does this breakpoint occur?)
- `from`: enum ["customer", "system", "rule", "agent"] (what initiates)
- `to`: enum ["agent", "human", "system", "customer"]
- `consequence_if_handled_poorly`: string (why is this point critical?)
- `agentic_opportunity`: boolean (can the agent improve handling at this point?)
- `risk_if_autonomous`: string | null (what goes wrong if agent decides alone?)

#### Delegation Archetype
One of five stable operating modes for this JTBD.

Enum values: "human_only", "human_led_automation_support", "human_led_agent_support", "agent_led_oversight", "fully_agentic"

Attributes (per task in delegation suitability matrix):
- `archetype`: Delegation Archetype enum
- `rationale`: string (why this delegation level, not another?)
- `evidence`: array of strings (citation to cognitive zones, breakpoints, or participant reasoning)
- `autonomy_ceiling`: string (what the agent may decide; what requires escalation)
- `failure_consequence`: string (what is at stake if the agent fails?)

### Cognitive Load Map (CLM)
Detailed decomposition of at least 2 of the 4 work streams into JTBDs, zones, breakpoints, and micro-tasks.

Attributes:
- `version`: string (e.g., "1.0")
- `mapped_streams`: array of WorkStream identifiers (which streams are in detail here)
- `unmapped_streams`: array of WorkStream identifiers (which are summary-only)
- `jtbd_count`: integer (count of JTBDs in detail)
- `zone_count`: integer (count of CognitiveZones identified)
- `breakpoint_count`: integer (count of Breakpoints identified)
- `lived_vs_documented_notes`: string (how does actual work differ from the SOP?)
- `primary_friction_points`: array of strings (where does work actually break down?)

### Delegation Suitability Matrix (DSM)
Scores each task cluster on four dimensions; assigns archetype with rationale.

Attributes:
- `matrix_version`: string
- `task_clusters`: array of TaskCluster objects

#### TaskCluster
A group of related micro-tasks scored together.

Attributes:
- `name`: string
- `related_jobs_to_be_done`: array of JTBD identifiers
- `structurability_score`: number 1–10 (can this be expressed as rules? 10=fully rule-bound, 1=pure judgment)
- `volume_score`: number 1–10 (how many times does this occur? 1=rare, 10=continuous)
- `human_expertise_required_score`: number 1–10 (how much expert knowledge? 1=none, 10=highly specialist)
- `reversibility_score`: number 1–10 (if the agent gets it wrong, how easy is it to fix? 10=trivial reversal, 1=irreversible)
- `assigned_archetype`: Delegation Archetype
- `archetype_justification`: string (why does this task cluster get this archetype, not another?)
- `evidence_from_clm`: array of strings (citations to specific zones/breakpoints/micro-tasks)
- `anti_pattern_check`: boolean (is this defaulting to "fully agentic" without justified reasoning?)

### Volume × Value Analysis (VVA)
Plots all 4 work streams on two axes: volume (throughput opportunity) and value (margin opportunity or cost reduction potential).

Attributes:
- `version`: string
- `streams_plotted`: array of objects {stream_name, volume_estimate, value_estimate, rationale}
- `primary_target`: string (identifier of highest-priority agent opportunity: usually high-volume + high-value or high-volume + high-risk)
- `target_justification`: string (why this stream wins relative to others)
- `secondary_opportunities`: array of strings (which streams are secondary priority)
- `deprioritised_streams`: array of {stream_name, reason_for_deferral}
- `economic_thesis`: string (at what volume and cost does agentic intervention pay for itself?)

### Agent Purpose Document (APD)
Specification of the single agent to be built for the highest-value opportunity.

Attributes:
- `agent_name`: string (descriptive, not generic)
- `job_to_be_done`: string (the cognitive contract: what outcome does this agent produce?)
- `business_context`: string (which department, process, customer journey step?)
- `primary_objectives`: array of strings (1–3 specific objectives; what success looks like)
- `kpi`: object with {accuracy_target: %, coverage_target: %, throughput_target: number, cost_per_case_target: string, hitl_rate_target: %}
- `failure_modes`: array of {mode: string, consequence: string, recovery_path: string}
- `delegation_archetype`: string (one of the five archetypes)
- `escalation_triggers`: array of {condition: string, escalate_to: string}
- `autonomy_matrix`: object with {agent_decides_alone: array, agent_acts_notified_after: array, agent_proposes_human_approves: array, human_takes_over: array}
- `scope_boundaries`: string (what the agent may and may not do)

### System and Data Inventory (SDI)
Enumeration of every system and data source the agent needs to access.

Attributes:
- `version`: string
- `inventory_items`: array of SystemAccess objects

#### SystemAccess
One system or data source the agent interacts with.

Attributes:
- `system_name`: string
- `data_needed`: array of strings (what specific data elements)
- `access_type`: enum ["read", "read_write", "trigger_only"]
- `availability`: enum ["api_available", "legacy_batch_export", "missing", "unreliable"]
- `gap_or_risk`: string | null (e.g., "rate limits", "missing real-time data", "integration effort: 3 weeks")
- `shared_with_other_agents`: boolean (can a future agent reuse this integration?)
- `assumptions`: array of Assumption objects (what is the participant assuming about this system's behaviour)

### Discovery Question (DiscoveryQuestion)
A question whose answer would materially change the agent design, not generic process questions.

Attributes:
- `question`: string (the question itself)
- `context`: string (why is this question material? what tension or gap does it resolve?)
- `design_impact`: string (how would a "yes" vs "no" answer change the agent design?)
- `target_stakeholder`: string (who should answer this? e.g., "operations lead", "compliance officer", "customer success")
- `maturity_level`: enum ["clarifies_scope", "refines_boundaries", "uncovers_hidden_requirement"] (how foundational is this question?)
- `is_generic`: boolean (false if specific to this domain/scenario; true if a template question — anti-pattern check)

### Assumption
A claim about the client's systems, priorities, or constraints that the participant is inferring, not stating as fact from the scenario brief.

Attributes:
- `assumption_text`: string (what is being assumed?)
- `confidence_level`: enum ["high", "medium", "low"] (how sure is the participant?)
- `basis`: string (where did this assumption come from? scenario excerpt, participant inference, ATX best practice?)
- `test_method`: string (how could this be verified during discovery?)
- `impact_if_wrong`: string (what changes if the assumption is false?)

---

## Naming Conventions

- **File paths**: snake_case, descriptive (e.g., `participant_atx_cognitive_load_map.md`, `gate_2_agent_purpose_document.md`)
- **Entity names in artefacts**: Title Case in headings, SCREAMING_SNAKE_CASE for enums (ACTIVE, REJECTED, FULLY_AGENTIC)
- **Timestamps in submissions**: always ISO 8601 with timezone (UTC)
- **Matrix cell values**: structured (not free text where possible); use enums, citations, and cross-references
- **Assumptions log**: prefix with confidence level in brackets: `[HIGH]`, `[MEDIUM]`, `[LOW]`
- **Discovery questions**: prefix with target maturity: `[SCOPE]`, `[BOUNDARY]`, `[HIDDEN_REQUIREMENT]`

---

## Validation Rules and Acceptance Criteria

### Cognitive Load Map (CLM)
- At least 2 of 4 work streams must be decomposed to JTBD level (minimum 3 JTBDs per stream)
- Every JTBD must have ≥2 CognitiveZones with named micro-tasks
- Every JTBD must have ≥1 Breakpoint identifying a control handoff or risk point
- CLM must include a "lived vs. documented" section explicitly naming differences between SOP and actual work
- No task can be marked "is_structurable: false" (pure judgment) without an explicit "human_expertise_required" field
- Zone error_tolerance must be justified: why is this zone "zero"?
- All CLM claims must be traceable to scenario artefacts (emails, calls, SOPs, system fragments) — inferences must be marked as assumptions

### Delegation Suitability Matrix (DSM)
- Every task cluster must have all four scores populated (structurability, volume, expertise, reversibility)
- Every task cluster must have archetype_justification that explains why this archetype was chosen over alternatives
- Anti-pattern check: if > 60% of task clusters are "fully_agentic", the matrix must include explicit reasoning for why human involvement is minimal, not just a statement that high volume justifies autonomy
- If structurability < 3 OR human_expertise_required > 7, archetype must not be "fully_agentic"
- All evidence citations must point to specific zones, breakpoints, or micro-tasks in the CLM

### Volume × Value Analysis (VVA)
- All 4 work streams must be plotted with explicit rationale for their volume and value positions
- Primary target must be justified against at least one alternative stream (e.g., "Chosen over X because Y")
- Economic thesis must name a specific volume threshold and cost target, not remain abstract
- Deprioritised streams must have explicit reason (e.g., "defer due to system integration complexity" not "lower priority")

### Agent Purpose Document (APD)
- Job to be Done must be phrased as a cognitive contract (outcome + decision + execution), not a task name
- All KPI targets must be specific and measurable (no "high accuracy" — what %?)
- Failure modes must include consequence (not just failure description) and recovery path (not just "escalate")
- Escalation triggers must be specific conditions (not "if unsure") and must name the target role
- Autonomy matrix cells must not be empty (if a decision type is not listed, why does the agent not have authority?)
- Scope boundaries must explicitly state what the agent may NOT do, not just what it may do

### System and Data Inventory (SDI)
- Every system mentioned in the APD must appear in the SDI
- Availability must be marked honestly ("missing", "unreliable", "batch export only") — do not hide integration challenges
- Gap/risk field must be non-empty for any system with availability != "api_available"
- Assumptions about system behaviour must be logged in the SystemAccess.assumptions array
- If 2+ agents share the same system integration, mark "shared_with_other_agents: true"

### Discovery Questions
- At least 6 discovery questions, and no more than 10
- Every question must have design_impact that names a specific change to the APD or SDI if answered differently
- No generic questions (e.g., "tell me about your process") — all questions must be scenario-specific or boundary-testing
- Questions must be organized by maturity level: at least 2 "clarifies_scope", at least 1 "uncovers_hidden_requirement"
- Each question must name target_stakeholder, not remain abstract

### Assumptions Log
- At least 5 assumptions documented
- Every assumption must have confidence_level, basis, test_method, and impact_if_wrong
- Low-confidence assumptions must have a named test method (not "TBD")
- Assumptions must not duplicate — if X is already an assumption, don't re-list it

---

## Workflow: ATX Execution Under Time Pressure

The Week 2 submission workflow is:

1. **Scenario orientation (first ~25 minutes)**
   - Read scenario brief
   - Identify the 4 work streams from the brief
   - If domain is unfamiliar: use AI to build a quick mental model of industry/process/failure modes (budgeted activity, not procrastination)
   - List initial assumptions in the Assumptions Log

2. **Cognitive Load Mapping (60–90 minutes)**
   - Pick 2 of the 4 work streams to decompose
   - For each stream: enumerate JTBDs, then for each JTBD, name CognitiveZones and Breakpoints
   - Use scenario artefacts (emails, SOPs, calls, system fragments) to ground each zone and breakpoint
   - Document "lived vs. documented" differences — this is where agentic value lives
   - Identify primary friction points where work breaks down

3. **Delegation Suitability Scoring (30–45 minutes)**
   - Create task clusters (group related micro-tasks)
   - For each cluster: score on four dimensions; assign archetype; justify
   - Run anti-pattern check: if defaulting to "fully agentic", why is human involvement minimal?
   - All citations must trace to CLM elements

4. **Volume × Value Analysis (20–30 minutes)**
   - Plot all 4 streams: relative volume (x-axis), relative value (y-axis)
   - Value = margin opportunity + cost reduction + compliance/risk reduction
   - Name primary target (usually high-volume + high-value, but may be high-risk with medium volume)
   - Defer low-volume or lower-value streams explicitly with reasoning

5. **Agent Purpose Document (45–60 minutes)**
   - Write for the primary target identified in VVA
   - Job to be Done: specific cognitive contract (what gets decided, what gets executed, what is the outcome?)
   - KPIs: all specific and measurable
   - Failure modes: consequence + recovery
   - Autonomy matrix: explicit cells (not empty), not just default "agent decides"
   - Escalation triggers: specific conditions → named roles

6. **System and Data Inventory (20–30 minutes)**
   - List every system the agent touches
   - Mark availability honestly (missing, batch export, API, unreliable)
   - Flag gaps and risks explicitly — integration effort, rate limits, legacy systems, missing real-time data
   - Mark shared integrations (opportunity for compounding)
   - Document assumptions about system behaviour

7. **Discovery Questions (15–20 minutes)**
   - Draft 6–10 questions
   - Ensure each has design_impact (what changes if answered differently?)
   - Ensure each names target_stakeholder
   - Organize by maturity level: at least 1 "uncovers_hidden_requirement"
   - Eliminate generic questions

8. **Assumptions Log Review (10 minutes)**
   - Ensure ≥5 documented
   - Check that low-confidence assumptions have test methods
   - Review for duplicates

---

## What the Agent Assistant Should NOT Do

- **Do not speculate beyond the scenario brief.** If the brief does not name a system, do not invent it. Log as an assumption instead.
- **Do not default every task to "fully agentic".** Force explicit reasoning for delegation levels. Query the participant if the matrix shows >60% fully agentic tasks.
- **Do not produce vague KPIs.** "High accuracy" is not a KPI. "≥95% correct policy application on standard cases" is.
- **Do not write generic discovery questions.** "Tell me about your process" doesn't signal FDE judgment. Questions must be scenario-specific or boundary-testing.
- **Do not hide integration challenges in the SDI.** If a system is batch-export-only, name it. If credentialing requires a vendor integration that doesn't exist yet, surface it.
- **Do not allow empty cells in the Autonomy Matrix.** If a decision type is not listed, the assistant should query why the agent isn't authorized for it, or confirm that it's explicitly out of scope.
- **Do not timestamp assumptions as facts.** Every claim about system behaviour, customer expectations, or compliance constraints that isn't in the scenario brief must be logged as an assumption with confidence level.
- **Do not merge work streams if their cognitive structures are different.** If intake, validation, resolution, and reporting have different error tolerances or expertise requirements, they are separate JTBDs, not sub-tasks of one stream.

---

## Handling Ambiguity and Escalation

### When to Ask the Participant Before Proceeding

1. **Missing CLM decomposition** (only 1 stream detailed, or fewer than 3 JTBDs per stream)
   - Escalate: "The CLM shows only 1 stream decomposed. Week 2 requires detail on at least 2 streams. Which is the second?"

2. **Unmarked inference in CLM** (CLM claims system behaviour not stated in brief)
   - Escalate: "The CLM claims [system behaviour] but the scenario brief doesn't mention this. Should I log this as an assumption, or is it in the artefacts?"

3. **Delegation archetype mismatch with scores** (structurability < 3 but archetype = "fully agentic")
   - Escalate: "This task has structurability 2 (mostly judgment) and human_expertise_required 8 (specialist), but the archetype is 'fully agentic'. Is this intentional, or should the archetype be 'human_led_agent_support'?"

4. **Empty Autonomy Matrix cell** (decision type exists in APD but not in autonomy_matrix)
   - Escalate: "The APD names 'exception handling' as a responsibility, but the autonomy matrix doesn't specify whether the agent decides alone, proposes for approval, or escalates. What's the intended authority?"

5. **Generic Discovery Question** (question does not include design_impact)
   - Escalate: "This question ('Tell me about compliance requirements') is generic to many processes. What specific design element would change if answered? Or is this a candidate for refinement?"

6. **Assumptions Log underspecified** (assumption has no test_method or impact_if_wrong)
   - Escalate: "This assumption ('System X has a REST API') has no test method. How would the participant verify this before development?"

### When to Decide Alone (Do Not Ask)

- **Scoring CLM zones on error_tolerance**: if a zone touches compliance or customer data, assign "zero" or "low" — this is ATX methodology, not participant choice
- **Validating SDI: honest gap naming**: if scenario says system is batch-export only, surface it as a gap regardless of participant's wishes
- **Correcting enum values**: if participant writes "high-agentic" instead of "fully_agentic", normalize silently
- **Timestamping submissions**: use ISO 8601 UTC, no negotiation
- **Checking anti-pattern**: if >60% of DSM is fully_agentic without justified reasoning, flag it in review feedback but do not prevent submission

---

## Integration Constraints

### Scenario Artefacts
- **Format**: PDFs, markdown, email transcripts, call transcripts, system screenshots, batch export samples
- **Access**: embedded in scenario brief document
- **Requirement**: every CLM claim must trace to one of these artefacts; inferences are logged as assumptions
- **Rate limit**: none (artefacts are static)

### AI-Accelerated Domain Orientation (First 25 Minutes)
- **When to use**: participant unfamiliar with the domain (healthcare, insurance, B2B SaaS, etc.)
- **Scope**: quick model of workflows, decision points, failure modes typical to the domain
- **Constraint**: do not use domain research to fill gaps in the scenario brief — only to provide context so the participant can interpret the brief correctly
- **Logging**: any domain assumption derived from AI orientation, not from the scenario brief, must be logged in Assumptions Log with confidence level and source

---

## Submission Evaluation: Gate 2 Rubric Elements

When reviewing a Week 2 submission, check against these dimensions (not exhaustive, but primary):

1. **Cognitive Load Map Fidelity**
   - Does it reflect lived work, not just SOP?
   - Are zones and breakpoints specific and defensible?
   - Are friction points named explicitly?
   - Cite to scenario artefacts, not inference

2. **Delegation Archetypes Are Justified**
   - Is every task cluster archetype explained, not defaulted?
   - Does the matrix show diversity (not 60%+ fully agentic)?
   - Do the scores (structurability, expertise, etc.) support the assigned archetype?

3. **Discovery Questions Signal FDE Judgment**
   - Are they scenario-specific, not generic?
   - Would different answers materially change the design?
   - Do they surface hidden assumptions?

4. **Assumptions Are Honest**
   - Are inferences logged as assumptions, not facts?
   - Do low-confidence assumptions have test methods?
   - Is the confidence level justified?

5. **Agent Purpose Document Is Buildable**
   - Is the Job to be Done phrased as a cognitive contract?
   - Are KPIs specific and measurable?
   - Does the Autonomy Matrix have explicit cells (not empty)?
   - Are escalation triggers specific conditions → named roles?

6. **System and Data Inventory Surfaces Real Challenges**
   - Are integration gaps named honestly?
   - Is legacy/batch-export systems flagged as gaps?
   - Are shared integrations identified for compounding?

---

## Participant Workflow Support: Your Role

You are assisting participants in applying ATX methodology to their scenario, not building the agent. Your role:

1. **Clarify methodology**: if participant asks "what should go in a CLM?", explain the structure and point to reference docs
2. **Flag incomplete work**: if a submission is missing CLM detail on 2 streams, escalate before peer review
3. **Surface methodological anti-patterns**: if >60% of DSM is fully agentic with minimal reasoning, flag it
4. **Help trace claims to artefacts**: if a CLM zone has no basis in scenario artefacts, ask participant to cite or convert to assumption
5. **Validate KPI specificity**: if APD has vague KPIs, send back for refinement (no "high accuracy" — what %)
6. **Stress-test discovery questions**: ensure each has design_impact and target_stakeholder
7. **Spot unmarked assumptions**: if a claim about system behaviour, customer priorities, or compliance constraints isn't in the brief, log it
8. **Support peer review**: brief on the anti-patterns to watch for (delegation archetype drift, generic discovery questions, speculative claims)

---

## Success Criteria

A strong Week 2 submission demonstrates:

✅ **Lived-work analysis** — CLM reflects actual work with friction points named explicitly, not just process documentation
✅ **Disciplined delegation thinking** — each task archetype is justified; not everything defaults to fully agentic
✅ **Honest integration planning** — SDI surfaces real challenges (legacy systems, missing APIs, rate limits); no hand-waving
✅ **Specific discovery** — questions would materially change the design if answered differently; no generic process questions
✅ **Buildable agent design** — APD has clear purpose, specific KPIs, explicit autonomy boundaries, named escalation triggers
✅ **Transparent assumptions** — inferences logged with confidence level and test methods; facts traced to scenario artefacts
✅ **Compounding mindset** — identifies shared integrations and platform assets that future agents can reuse

A weak submission shows:

❌ **Documented work as lived work** — CLM matches the SOP but misses actual friction, workarounds, and exceptions
❌ **Archetype drift** — 60%+ of delegation matrix is fully agentic; reasoning defaults to "high volume" without examining judgment demands
❌ **Speculative design** — APD assumes systems, integrations, or data availability not mentioned in brief
❌ **Generic discovery** — questions don't tie to specific tensions or boundaries in the scenario
❌ **Vague governance** — autonomy matrix is incomplete; KPIs lack specificity; escalation triggers are broad conditions, not precise rules
❌ **Hidden assumptions** — claims treated as facts (system availability, customer behavior, compliance rules) that aren't in the brief

---

## Token Discipline & Context Engineering

When engaging with a participant's submission:

1. **Reference artefacts precisely**: cite specific sections of CLM, DSM, APD, not entire documents
2. **Use structured feedback**: break feedback into methodological gaps vs. content gaps vs. buildability risks
3. **Escalate, don't invent**: if something is missing, ask for it; do not fabricate it to complete the submission
4. **Reuse examples from Week 2 reference materials**: ATX Concepts, Agent Mapping, Assessment references are your source of truth; do not improvise methodology

---

## What This Document Specifies

This CLAUDE.md governs:
- **What entities and artefacts exist** in the Week 2 ATX workflow (CLM, DSM, VVA, APD, SDI, Discovery Questions, Assumptions)
- **What validates each artefact** (acceptance criteria, anti-patterns to watch)
- **What the assistant should escalate vs. decide** (when to ask the participant, when to flag issues, when to normalize)
- **What the workflow looks like under time pressure** (8-step sequence, typical time budgets)
- **What success looks like** (strong vs. weak submissions against the Gate 2 rubric)

This CLAUDE.md does NOT govern:
- The live clarification round itself (that is between participant and coach)
- Peer review scoring (that is peer reviewer discretion)
- Gate 2 results and feedback (that is coach/faculty responsibility)
- Methodology evolution (ATX references are authoritative; changes come from program updates)