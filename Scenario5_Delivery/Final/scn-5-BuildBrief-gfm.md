# CLAUDE.md

## Project Goal

Build a **Patient Intake Coordination Agent** for a family medicine practice that automates non-clinical pre-visit intake work for scheduled appointments.

The clinic context is:

- 6 physicians
- 2 locations
- approximately 180 patients per day
- 4-person front-desk team

The system must reduce front-desk administrative workload and improve intake completeness before visits.

## Hard Safety Boundary

This system is **non-clinical**.

Never implement any feature that:

- diagnoses
- recommends treatment
- performs medical triage
- determines medical urgency
- judges whether a medication change is clinically significant
- judges whether an allergy flag is clinically significant
- suppresses required human review for urgent-looking or ambiguous visit reasons

If a workflow condition may require clinical judgment, escalate to a human.

## Core Workflow

For each scheduled appointment:

1. retrieve appointment data
2. create one intake work item if one does not already exist
3. send and track a pre-visit questionnaire
4. verify insurance eligibility
5. if procedure-related, check prior-auth status using explicit clinic-maintained rules
6. collect patient-reported medication changes
7. compare patient-reported medications with EHR medication data
8. retrieve allergy flags
9. evaluate visit reason only under narrow administrative routing rules
10. create escalations where required
11. determine whether the appointment is administratively ready for visit

## Delegation Model

Use exactly these modes conceptually when building workflow logic:

- `AGENT_ALONE`
- `AGENT_PLUS_HUMAN_REVIEW`
- `HUMAN_DECIDES`

### Agent Alone

Allowed for:
- appointment retrieval
- intake work item creation
- questionnaire sending
- questionnaire completeness checks
- insurance verification
- medication collection
- medication list comparison
- allergy retrieval

### Agent Plus Human Review

Allowed for:
- prior-auth checks using explicit rules
- medication discrepancy flagging
- allergy-flag surfacing
- visit-reason routing under narrow admin rules
- escalation creation
- readiness evaluation where review remains open

### Human Decides

Required for:
- medical urgency decisions
- medication significance decisions
- allergy significance decisions
- override of blocked or escalated cases
- final exception handling where required review remains unresolved

## Required Entities

### Appointment

Fields:
- `id`
- `patient_id`
- `provider_id`
- `location_id`
- `scheduled_start_at`
- `visit_type`
- `scheduled_procedure_code`
- `status`

### IntakeWorkItem

Fields:
- `id`
- `appointment_id`
- `patient_id`
- `status`
- `questionnaire_status`
- `insurance_status`
- `prior_auth_status`
- `medication_status`
- `allergy_status`
- `visit_reason_status`
- `completion_decision`
- `assigned_queue`
- `created_at`
- `updated_at`

Rules:
- exactly one active work item per appointment
- all state transitions must be logged
- creation must be idempotent

### EscalationEvent

Fields:
- `id`
- `intake_work_item_id`
- `trigger_code`
- `queue`
- `status`
- `created_at`
- `resolved_at`
- `resolution_note`

Rules:
- every escalation must have exactly one queue
- open escalations block readiness

### AuditLog

Fields:
- `id`
- `entity_type`
- `entity_id`
- `action_type`
- `actor_type`
- `actor_id`
- `timestamp`
- `details_json`

Rules:
- log all PHI reads
- log all integration calls and failures
- log all state transitions
- log all escalations
- log all overrides with user ID and reason

## Status Expectations

Use explicit enums rather than free text.

Minimum statuses to support:

- questionnaire: `NOT_SENT`, `SENT`, `RECEIVED_COMPLETE`, `RECEIVED_INCOMPLETE`, `NOT_RECEIVED`
- insurance: `NOT_STARTED`, `VERIFIED_ACTIVE`, `VERIFIED_INACTIVE`, `ERROR`, `TIMEOUT`
- prior auth: `NOT_STARTED`, `NOT_REQUIRED`, `VALID`, `EXPIRED`, `MISSING`, `UNDETERMINED`
- medication: `NOT_REQUESTED`, `NO_CHANGES_REPORTED`, `CHANGES_REPORTED`, `MISMATCH_DETECTED`
- allergy: `NOT_CHECKED`, `NO_FLAGS`, `FLAGS_PRESENT`
- visit reason: `NOT_REVIEWED`, `ROUTINE_ROUTE`, `HUMAN_REVIEW_REQUIRED`
- completion decision: `NOT_EVALUATED`, `READY_FOR_VISIT`, `HOLD_FOR_REVIEW`

## Functional Requirements

### FR1. Appointment Sync
Poll scheduled appointments for the next 2 calendar days every 15 minutes.

Requirements:
- create exactly one active intake work item per scheduled appointment
- do not create work items for cancelled appointments
- make sync idempotent

### FR2. Questionnaire Workflow
Send a questionnaire automatically when a work item is created unless already completed.

Requirements:
- mark complete only when all required fields are present
- if incomplete, escalate to `FRONT_DESK`
- if not received by 12 hours before visit start, escalate to `FRONT_DESK`

### FR3. Insurance Verification
Verify insurance within 4 hours of work item creation.

Requirements:
- active -> mark verified active
- inactive -> escalate to `FRONT_DESK`
- timeout or error -> escalate to `FRONT_DESK`
- never guess insurance outcomes

### FR4. Prior-Auth Check
Run only when visit type is procedure-related or a procedure code is present.

Requirements:
- use only explicit clinic-maintained rules
- if no rule exists, set `UNDETERMINED` and escalate
- if missing or expired, escalate
- never infer prior-auth requirement from model reasoning

### FR5. Medication Change Handling
Collect patient-reported medication changes and compare against EHR medication data.

Requirements:
- any reported change -> escalate to `CLINICAL_STAFF`
- any mismatch on name, dose, or frequency -> escalate to `CLINICAL_STAFF`
- never classify significance

### FR6. Allergy Flag Handling
Retrieve allergy flags from EHR.

Requirements:
- any allergy flag present -> escalate to `CLINICAL_STAFF`
- never interpret severity or care impact

### FR7. Visit Reason Routing
Evaluate visit reason only against explicit administrative rules and an urgent-trigger phrase list.

Requirements:
- routine admin match with no urgent phrase -> may remain un-escalated
- blank, ambiguous, unparseable, or urgent-looking -> escalate to `CLINICAL_STAFF`
- never output clinical urgency or triage advice

### FR8. Readiness Logic
Mark `READY_FOR_VISIT` only when:
- insurance is verified active
- questionnaire is complete
- visit reason is safe for routine admin routing or resolved by human review
- no medication-change escalation remains open
- no allergy escalation remains open
- if prior auth applies, prior-auth status is valid or not required
- there are zero open escalations
- there are zero blocking integration failures

Otherwise mark `HOLD_FOR_REVIEW`.

### FR9. Audit Logging
Create immutable logs for:
- PHI access
- questionnaire sends and receives
- insurance checks
- EHR reads
- state changes
- escalations
- overrides
- integration failures

## Escalation Triggers

Create an escalation when any of the following occurs:

- questionnaire incomplete
- questionnaire missing by deadline
- insurance inactive
- insurance timeout
- insurance error
- prior auth missing
- prior auth expired
- prior auth undetermined
- medication change reported
- medication mismatch detected
- allergy flag present
- urgent-trigger phrase in visit reason
- blank visit reason
- ambiguous or unparseable visit reason
- required integration failure

## Queue Assignment

Use these queue rules:

- `FRONT_DESK`
  - questionnaire incomplete or missing
  - insurance inactive, error, timeout
  - prior auth missing, expired, undetermined

- `CLINICAL_STAFF`
  - medication change reported
  - medication mismatch
  - allergy flag present
  - urgent-trigger phrase
  - blank, ambiguous, or unparseable visit reason

- `PRACTICE_MANAGER`
  - system-wide integration outage longer than 30 minutes

## Integration Rules

### athenahealth Adapter

Must support:
- appointment retrieval
- medication retrieval
- allergy retrieval

Rules:
- do not fabricate missing EHR data
- log PHI reads
- log integration failures
- keep contract configurable behind an adapter

### Insurance Eligibility Adapter

Must support:
- eligibility check for scheduled visit

Rules:
- return active, inactive, error, or timeout
- retry if configured
- final unresolved failure must escalate
- never assume active coverage on failure

### Prior-Auth Rule Source

Must support:
- determine whether prior auth is required based on explicit clinic-maintained rules
- retrieve prior-auth record status if available

Rules:
- no matching rule -> `UNDETERMINED`
- never infer rule outcomes from model reasoning

## Override Rules

Only authorized humans may override blocked or escalated states.

When override occurs:
- require user ID
- require free-text reason
- log prior state
- log new state
- do not auto-resolve open escalations unless explicitly resolved by human action

## Validation Requirements

Implement at least these workflow tests:

1. happy path routine visit -> `READY_FOR_VISIT`
2. incomplete questionnaire -> escalation to `FRONT_DESK`
3. insurance timeout -> no guessed result, escalation, hold or block
4. visit reason `chest pain since last night` -> escalation to `CLINICAL_STAFF`, no triage output
5. medication dose change -> escalation to `CLINICAL_STAFF`, no significance interpretation

For all validation cases:
- assert one active work item per appointment
- assert all state transitions are logged
- assert all escalations have exactly one queue
- assert open escalations block readiness
- assert no clinical interpretation is produced in boundary scenarios

## Assumptions To Keep Configurable

Do not hard-code the following as fixed truths:

- exact athenahealth contract
- exact insurance tool contract
- prior-auth rule source details
- urgent-trigger phrase list
- admin routing taxonomy
- queue ownership details
- approved questionnaire delivery channels
- audit retention policy

## Build Priorities

Optimize for:
- correctness
- explicit rules
- auditability
- idempotency
- safe escalation
- fail-closed behavior

Do not optimize for:
- maximum automation at the expense of safety
- hidden inference
- silent defaults on missing data
- clinical interpretation

## Source Context

- `README-Participants-Week1-Scenarios.md`
- `README-Participants-Intro-Week1.md`
- `production-spec-checklist.md`
- `claude-md-examples-guide.md`