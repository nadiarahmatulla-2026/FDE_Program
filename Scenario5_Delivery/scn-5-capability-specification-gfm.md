# Capability Specification

## Purpose

Build a **Patient Intake Coordination Agent** for Scenario 5 that automates non-clinical patient intake work for a family medicine practice with **6 physicians**, **2 locations**, and approximately **180 patients per day**.

The agent must reduce front-desk administrative burden and improve pre-visit intake completeness by handling structured administrative workflow steps before the visit starts.

The agent must **not** perform clinical judgment. It may collect, compare, classify under explicit administrative rules, and escalate. It must not diagnose, triage medically, assess medication significance, or assess allergy significance.

## Scope

### In Scope

The agent must support:

- scheduled appointment intake initialization
- questionnaire sending and tracking
- insurance eligibility verification
- prior-auth status checking using explicit clinic-maintained rules
- patient-reported medication change collection
- medication mismatch detection against EHR data
- allergy flag retrieval from EHR
- visit-reason routing using narrow administrative rules
- human escalation creation and queue assignment
- intake completion evaluation
- audit logging for workflow actions and PHI access

### Out of Scope

The agent must not support:

- diagnosis
- treatment advice
- clinical triage
- medical urgency determination
- medication safety analysis
- allergy severity analysis
- autonomous visit rescheduling or cancellation
- any decision that substitutes for human clinical judgment

## Inputs

The agent must accept or retrieve the following inputs:

### From Scheduling / EHR
- appointment ID
- patient ID
- provider ID
- location ID
- scheduled start time
- visit type
- scheduled procedure code if present
- current medication list
- current allergy flags

### From Patient Questionnaire / Intake Form
- visit reason text
- questionnaire required fields
- patient-reported medication changes
- confirmation of no medication changes if applicable

### From Insurance Eligibility Tool
- coverage status
- payer response timestamp
- error or timeout state if applicable

### From Prior-Auth Rules / Records
- procedure code
- prior-auth required flag from explicit clinic-maintained rule set
- prior-auth record status if present

## Outputs

The agent must produce:

- one intake work item per appointment
- status values for questionnaire, insurance, prior auth, medication changes, allergy flags, and visit reason routing
- escalation events where required
- queue assignment for each escalation
- final readiness decision:
  - `READY_FOR_VISIT`
  - `HOLD_FOR_REVIEW`
- immutable audit log events for all workflow actions, PHI reads, state transitions, escalations, and overrides

## Required Data Model

Claude should implement at least these entities.

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

## Core Decision Logic

The agent must implement the following logic:

1. If an appointment is scheduled, create exactly one intake work item.
2. If a questionnaire is incomplete or missing by deadline, escalate to `FRONT_DESK`.
3. If insurance verification fails, times out, or returns inactive coverage, escalate to `FRONT_DESK`.
4. If prior auth is required and missing, expired, or undetermined, escalate to `FRONT_DESK`.
5. If the patient reports any medication change, escalate to `CLINICAL_STAFF`.
6. If the patient-reported medication list differs from EHR medication data by name, dose, or frequency, escalate to `CLINICAL_STAFF`.
7. If any allergy flag exists in EHR, escalate to `CLINICAL_STAFF`.
8. If the visit reason is blank, ambiguous, unparseable, or contains an urgent-trigger phrase, escalate to `CLINICAL_STAFF`.
9. The system may mark a case `READY_FOR_VISIT` only if all required administrative checks are complete and there are zero open escalations.
10. If required input data is unavailable because of an integration failure, the system must fail closed by escalating or blocking, not by assuming success.

## Escalation Triggers

The agent must create an escalation event when any of the following occurs:

- questionnaire incomplete
- questionnaire not received by deadline
- insurance inactive
- insurance integration error
- insurance timeout
- prior auth missing
- prior auth expired
- prior auth undetermined
- medication change reported
- medication mismatch detected
- allergy flag present
- urgent-trigger phrase detected in visit reason
- visit reason blank
- visit reason ambiguous or unparseable
- required integration failure

## Queue Assignment Rules

The agent must assign escalation queues using these rules:

| Trigger | Queue |
|:--|:--|
| Questionnaire incomplete or missing | `FRONT_DESK` |
| Insurance inactive, error, or timeout | `FRONT_DESK` |
| Prior auth missing, expired, or undetermined | `FRONT_DESK` |
| Medication change reported | `CLINICAL_STAFF` |
| Medication mismatch detected | `CLINICAL_STAFF` |
| Allergy flag present | `CLINICAL_STAFF` |
| Urgent-trigger phrase, blank visit reason, ambiguous visit reason | `CLINICAL_STAFF` |
| System-wide integration outage longer than 30 minutes | `PRACTICE_MANAGER` |

## Integration Points

The scenario explicitly identifies:

- **athenahealth** for EHR access
- a separate **insurance eligibility tool**

Claude must implement integration adapters behind interfaces so contracts can be configured later.

### Integration A: athenahealth Adapter

Required capabilities:
- list scheduled appointments
- retrieve patient medication list
- retrieve patient allergy flags

Behavior rules:
- if appointment retrieval fails, do not create new work items from guessed data
- if medication or allergy retrieval fails, do not assume no issues exist
- log all PHI reads and integration failures

### Integration B: Insurance Eligibility Adapter

Required capabilities:
- submit eligibility check for a scheduled visit
- return one of:
  - active
  - inactive
  - error
  - timeout

Behavior rules:
- do not assume active coverage if the integration fails
- retries are allowed, but final unresolved failure must escalate
- log all calls and failures

### Integration C: Prior-Auth Rule Source

Required capabilities:
- determine whether prior auth is required for a procedure code using explicit clinic-maintained rules
- retrieve prior-auth record status if available

Behavior rules:
- if no explicit rule exists, return `UNDETERMINED`
- never infer prior-auth requirement from model reasoning

## Functional Requirements

Claude must implement at least the following requirements.

### FR1. Appointment Sync and Work Item Creation
The system must poll scheduled appointments for the next 2 calendar days every 15 minutes and create exactly one active intake work item per scheduled appointment.

Acceptance criteria:
- duplicate work item creation is forbidden
- processing must be idempotent
- cancelled appointments must not create new work items

### FR2. Questionnaire Dispatch and Tracking
The system must send a questionnaire automatically when a new intake work item is created unless a valid completed questionnaire already exists.

Acceptance criteria:
- if all required fields are present, mark questionnaire complete
- if required fields are missing, mark questionnaire incomplete and escalate
- if no response is received by 12 hours before the visit, escalate to `FRONT_DESK`

### FR3. Insurance Verification
The system must verify insurance for every scheduled appointment within 4 hours of intake work item creation.

Acceptance criteria:
- active coverage -> mark verified active
- inactive coverage -> escalate to `FRONT_DESK`
- timeout or error -> escalate to `FRONT_DESK`
- no guessed insurance result is allowed

### FR4. Prior-Auth Determination
If the visit is a procedure visit or includes a procedure code, the system must determine prior-auth status using explicit clinic-maintained rules and available prior-auth records.

Acceptance criteria:
- no rule match -> `UNDETERMINED` and escalation
- required but missing -> escalation
- required but expired -> escalation
- not required or valid -> no escalation from this check

### FR5. Medication Change Detection
The system must collect medication update responses from the patient and compare them with EHR medication data.

Acceptance criteria:
- any reported change -> escalate to `CLINICAL_STAFF`
- any mismatch on name, dose, or frequency -> escalate to `CLINICAL_STAFF`
- no clinical significance scoring is allowed

### FR6. Allergy Flag Handling
The system must retrieve allergy flags from EHR for every intake work item.

Acceptance criteria:
- any allergy flag present -> escalate to `CLINICAL_STAFF`
- no allergy interpretation is allowed

### FR7. Visit Reason Routing
The system must evaluate visit reason text only against a narrow administrative ruleset and urgent-trigger phrase list.

Acceptance criteria:
- routine admin match with no urgent phrase -> may remain un-escalated
- blank, ambiguous, unparseable, or urgent-looking -> escalate to `CLINICAL_STAFF`
- no medical urgency classification is allowed

### FR8. Readiness Evaluation
The system must mark an intake work item `READY_FOR_VISIT` only if all required admin checks are complete and no open escalation exists.

Acceptance criteria:
- any open escalation -> `HOLD_FOR_REVIEW`
- any blocking integration failure -> `HOLD_FOR_REVIEW`
- readiness logic must be explicit and deterministic

### FR9. Audit Logging
The system must create immutable audit records for PHI access, state changes, outreach events, integration calls, escalations, and overrides.

Acceptance criteria:
- each audit record must include actor, timestamp, action, entity, and details
- human overrides must include user ID and reason

## Hard Constraints

Claude must enforce these constraints in implementation:

- never diagnose
- never triage medically
- never recommend treatment
- never judge medication importance
- never judge allergy significance
- never infer prior-auth rules without explicit source data
- never mark a work item ready while required human review is unresolved
- never fabricate missing integration data

## Assumptions

These are implementation assumptions and must remain configurable or explicitly validated later.

1. athenahealth exposes machine-accessible interfaces for appointments, medications, and allergy flags.
2. the insurance eligibility tool exposes a machine-accessible interface
3. the clinic maintains explicit structured prior-auth rules
4. the clinic has an approved questionnaire delivery channel
5. the clinic has distinct `FRONT_DESK`, `CLINICAL_STAFF`, and `PRACTICE_MANAGER` queues
6. unverified patient-reported medication data can be stored in a staging area
7. urgent-trigger phrases for visit reason escalation can be configured by the clinic

## Build Notes For Claude

When building this capability:

- use explicit enums for all statuses
- make workflow execution idempotent
- make integrations configurable behind adapters
- treat missing or failed data as unresolved, not successful
- prefer escalation over silent inference
- separate data detection from human meaning-making
- keep all decision logic rule-based and traceable
- fail closed on safety-sensitive ambiguity

## Sources

- `README-Participants-Week1-Scenarios.md`
- `README-Participants-Intro-Week1.md`
- `production-spec-checklist.md`
- `claude-md-examples-guide.md`