# Delegation Analysis

## Purpose

This section tells Claude exactly which parts of the Scenario 5 workflow it may automate, which parts require human review, and which parts must remain fully human. The implementation must preserve a hard boundary: **the system may automate administrative intake work, but it must not perform clinical judgment**.

Use this section to drive workflow logic, escalation rules, queue assignment, and completion rules.

## Required Delegation Modes

Use exactly these delegation modes in the implementation:

- `AGENT_ALONE`
- `AGENT_PLUS_HUMAN_REVIEW`
- `HUMAN_DECIDES`

Do not create additional delegation categories unless explicitly requested.

## Global Delegation Rules

1. The system may automate structured, deterministic, administrative tasks.
2. The system may collect, compare, route, and flag information.
3. The system must not interpret symptoms, medications, or allergies clinically.
4. If a task involves ambiguity, patient safety, or possible clinical urgency, the system must escalate to a human.
5. If a task outcome depends on a missing rule, unclear data, or unstated business logic, the system must escalate instead of inferring.
6. A case must not be marked ready if a required human review is still open.

## Delegation Matrix

| Task | Delegation Mode | Build Rule | Why |
|:--|:--|:--|:--|
| Pull scheduled appointments from athenahealth | `AGENT_ALONE` | Fetch all `SCHEDULED` appointments for the next 2 calendar days on the sync interval. | Structured retrieval task with low ambiguity. No clinical judgment required. |
| Create intake work item for each scheduled appointment | `AGENT_ALONE` | Create exactly one active intake work item per appointment. Make creation idempotent. | Administrative coordination task that must scale reliably across approximately **180 patients per day**. |
| Send pre-visit questionnaire | `AGENT_ALONE` | Send questionnaire automatically when a work item is created unless already completed. | Standardized outreach with explicit trigger conditions. |
| Check questionnaire completion status | `AGENT_ALONE` | Evaluate whether all required fields are present by the configured deadline. | Deterministic completeness check, not clinical interpretation. |
| Record questionnaire as complete or incomplete | `AGENT_ALONE` | Set questionnaire status based only on required-field presence. | This is form validation, not medical reasoning. |
| Verify insurance eligibility | `AGENT_ALONE` | Call eligibility integration and record returned status. | Repetitive, administrative, structured task with explicit machine outcomes. |
| Record insurance verification outcome | `AGENT_ALONE` | Persist `VERIFIED_ACTIVE`, `VERIFIED_INACTIVE`, `ERROR`, or `TIMEOUT` exactly as returned or derived from integration outcome. | Administrative status recording only. |
| Check prior-auth requirement using clinic-maintained explicit rules | `AGENT_PLUS_HUMAN_REVIEW` | Run only against configured structured rules. If no rule matches, set `UNDETERMINED` and escalate. | Safe only when rule-based. Unsafe if inferred from incomplete payer or clinical logic. |
| Record prior-auth status | `AGENT_PLUS_HUMAN_REVIEW` | Record `NOT_REQUIRED`, `VALID`, `EXPIRED`, `MISSING`, or `UNDETERMINED`. Escalate non-ready outcomes. | Status capture is administrative, but errors directly affect visit readiness and require accountable review. |
| Collect patient-reported medication changes | `AGENT_ALONE` | Ask whether medications changed and store patient response in structured form. | Collection is administrative. Interpretation is not. |
| Compare patient-reported medication data to EHR medication list | `AGENT_ALONE` | Detect mismatch on medication name, dose, or frequency fields only. | Deterministic comparison is safe. The system is detecting difference, not deciding meaning. |
| Flag medication change or mismatch | `AGENT_PLUS_HUMAN_REVIEW` | Create escalation to `CLINICAL_STAFF` whenever any change or mismatch is detected. | Medication significance is clinical and must remain human. |
| Retrieve allergy flags from EHR | `AGENT_ALONE` | Pull allergy flag records and detect whether any flag exists. | Structured retrieval only. |
| Flag presence of allergy records | `AGENT_PLUS_HUMAN_REVIEW` | If any allergy flag exists, require human review before readiness. | Presence can be detected automatically; significance cannot. |
| Route visit reason using narrow administrative rules | `AGENT_PLUS_HUMAN_REVIEW` | Route only when the input matches an approved routine administrative category with no urgent-trigger phrases. Otherwise escalate. | Free-text reason data can quickly become safety-sensitive. Narrow automation is acceptable; open interpretation is not. |
| Escalate urgent-looking, ambiguous, blank, or unparseable visit reasons | `AGENT_PLUS_HUMAN_REVIEW` | Create escalation immediately and stop autonomous routing. | Escalation is safer than interpretation where clinical urgency may be involved. |
| Decide whether a visit reason is medically urgent | `HUMAN_DECIDES` | Never implement model logic that determines medical urgency. Always require human review. | This is a clinical judgment and is explicitly out of bounds for Scenario 5. |
| Decide whether a medication change is clinically important | `HUMAN_DECIDES` | Do not implement any logic that scores or classifies medication significance. | Requires clinical knowledge and patient safety judgment. |
| Decide whether an allergy flag changes care | `HUMAN_DECIDES` | Do not implement any allergy severity or care-path decision logic. | This is clinical reasoning, not admin processing. |
| Override an escalated or blocked work item | `HUMAN_DECIDES` | Allow only authorized human override with mandatory reason logging. | Overrides are accountable operational decisions with safety implications. |
| Final confirmation that intake can proceed despite exception conditions | `HUMAN_DECIDES` | If any exception remains open, require explicit human action before proceeding. | Exception handling in safety-sensitive workflows must remain accountable and human-owned. |

## Implementation Rules By Delegation Mode

### Rules For `AGENT_ALONE`

Claude may implement autonomous execution only when all of the following are true:

- the task uses structured inputs
- the decision rule is explicit
- no clinical judgment is required
- the output can be logged deterministically
- failure can be handled through escalation or blocking

For `AGENT_ALONE` tasks:
- execute automatically
- log all actions
- do not ask for human review unless a failure or exception occurs

### Rules For `AGENT_PLUS_HUMAN_REVIEW`

Claude may implement autonomous preparation, detection, or routing, but must require human review when the output affects safety-sensitive readiness or could cross into clinical interpretation.

For `AGENT_PLUS_HUMAN_REVIEW` tasks:
- perform the mechanical or rule-based portion automatically
- create an escalation when review is required
- assign the correct queue
- block readiness until the review is resolved
- do not silently downgrade or suppress the escalation

### Rules For `HUMAN_DECIDES`

Claude must not implement autonomous decision logic for these tasks.

For `HUMAN_DECIDES` tasks:
- provide data, flags, context, and audit trail to the user
- do not generate a final decision
- require explicit human action
- require user identity and reason logging where relevant

## Queue Assignment Rules

When the system escalates, assign queues as follows:

| Trigger Type | Queue |
|:--|:--|
| Insurance inactive, insurance error, insurance timeout, prior-auth expired, prior-auth missing, prior-auth undetermined, questionnaire missing, questionnaire incomplete | `FRONT_DESK` |
| Medication change reported, medication mismatch detected, allergy flag present, urgent-trigger phrase detected, ambiguous visit reason, blank visit reason | `CLINICAL_STAFF` |
| Integration outage longer than 30 minutes affecting multiple work items | `PRACTICE_MANAGER` |

Do not route safety-sensitive cases back into a fully autonomous path until a human resolves the escalation.

## Readiness Rule

The system may mark an intake work item as `READY_FOR_VISIT` only if:

- insurance is verified active
- questionnaire is complete
- visit reason is either routine and safely classified under narrow admin rules or has been resolved by human review
- no medication change escalation remains open
- no allergy escalation remains open
- if prior auth applies, prior-auth status is valid or not required
- there are zero open escalations
- there are zero blocking integration failures

If any of the above is false, set readiness to `HOLD_FOR_REVIEW`.

## Hard Prohibitions

Claude must not build any feature that does the following:

- determines medical urgency from symptoms
- gives patient-facing triage advice
- scores medication changes by clinical importance
- dismisses allergy flags as minor
- infers prior-auth rules when no explicit rule exists
- marks a case ready while required human review is still unresolved

If a proposed implementation path would require one of these actions, do not implement it. Escalate instead.

## Why This Delegation Boundary Is Correct

This boundary is correct because it maximizes automation only where the work is:

- structured
- repetitive
- administrative
- low ambiguity

and preserves human control where the work is:

- clinically meaningful
- safety-sensitive
- ambiguous
- exception-based

This is the key design requirement in Scenario 5. The system should help the **4-person front-desk team** manage intake at a scale of approximately **180 patients per day**, but it must do so without replacing human clinical judgment.

## Validation Requirements For Delegation Logic

Claude must include tests that prove the delegation boundary is working.

Required tests:

1. A routine visit with complete paperwork can move through agentic workflow without human review.
2. A blank visit reason creates human review and does not produce autonomous interpretation.
3. A visit reason such as `chest pain since last night` creates escalation and does not produce a medical urgency label.
4. A medication dose change creates escalation and does not produce a significance assessment.
5. Any allergy flag creates escalation and does not produce severity interpretation.
6. Missing prior-auth rule creates `UNDETERMINED` and escalation rather than inference.
7. A work item with any open escalation cannot become `READY_FOR_VISIT`.

## Sources

- `README-Participants-Week1-Scenarios.md`
- `README-Participants-Intro-Week1.md`
- `production-spec-checklist.md`
- `claude-md-examples-guide.md`