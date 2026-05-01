# Validation Design

## Purpose

This section defines the first-draft validation plan for the Scenario 5 Patient Intake Coordination Agent. The goal is to verify that the system:

- completes routine administrative intake correctly
- escalates edge cases reliably
- fails safely when data, integrations, or workflow conditions are incomplete
- preserves the non-clinical delegation boundary required by Scenario 5

Claude must implement these scenarios as testable workflow cases. Each scenario must produce explicit outcomes such as:

- `PASS`
- `ESCALATE`
- `HOLD_FOR_REVIEW`
- `BLOCK`

Do not implement validation logic that relies on human interpretation of unstated requirements.

## Validation Rules

1. A test passes only if the resulting workflow state, escalation behavior, and audit trail match the expected outcome exactly.
2. If a scenario involves safety-sensitive ambiguity, the expected outcome must prefer escalation over autonomous completion.
3. If required integration data is missing or unavailable, the system must fail closed.
4. If a scenario touches visit reason meaning, medication meaning, or allergy significance, the system must not generate clinical interpretation.
5. All scenario runs must create audit logs for workflow actions, escalations, and integration behavior.

## Scenario V1 - Happy Path Routine Visit

### Goal

Verify that a routine scheduled visit with complete administrative inputs can move from intake initialization to `READY_FOR_VISIT` without unnecessary escalation.

### Input Conditions

- appointment status = `SCHEDULED`
- visit type = `ROUTINE_VISIT`
- no procedure code present
- questionnaire sent and returned complete
- insurance eligibility returns active coverage
- patient reports no medication changes
- EHR medication list retrieval succeeds
- no allergy flags exist in EHR
- visit reason matches an approved routine administrative category
- no urgent-trigger phrases detected
- all integrations succeed

### Expected System Behavior

- create exactly one intake work item
- mark questionnaire status as complete
- mark insurance status as verified active
- set prior-auth status to not required
- set medication status to no changes reported
- set allergy status to no flags
- set visit reason status to routine route
- create zero escalation events
- mark completion decision as `READY_FOR_VISIT`
- create audit logs for:
  - appointment retrieval
  - questionnaire send
  - questionnaire receive
  - insurance verification
  - EHR reads
  - final readiness decision

### Expected Outcome

- result = `PASS`
- final workflow status = `READY_FOR_VISIT`

## Scenario V2 - Edge Case Incomplete Questionnaire

### Goal

Verify that incomplete administrative intake data causes escalation without unsafe completion.

### Input Conditions

- appointment status = `SCHEDULED`
- questionnaire returned with missing required fields
- insurance eligibility returns active coverage
- no procedure code present
- patient reports no medication changes
- no allergy flags exist
- visit reason is routine and clearly mappable
- all integrations succeed

### Expected System Behavior

- create intake work item
- mark questionnaire status as incomplete
- create escalation with trigger `QUESTIONNAIRE_INCOMPLETE`
- assign escalation queue = `FRONT_DESK`
- do not mark intake as `READY_FOR_VISIT`
- set completion decision = `HOLD_FOR_REVIEW`
- log escalation creation and queue assignment
- preserve all non-questionnaire checks that succeeded

### Expected Outcome

- result = `ESCALATE`
- final workflow status = `HOLD_FOR_REVIEW`

## Scenario V3 - Failure Mode: Insurance Timeout

### Goal

Verify that the system fails closed when a required administrative integration is unavailable.

### Input Conditions

- appointment status = `SCHEDULED`
- questionnaire returned complete
- insurance eligibility integration times out on all allowed retries
- no procedure code present
- patient reports no medication changes
- no allergy flags exist
- visit reason is routine and clearly mappable

### Expected System Behavior

- create intake work item
- mark questionnaire status as complete
- mark insurance status as timeout
- create escalation with trigger `INSURANCE_TIMEOUT`
- assign escalation queue = `FRONT_DESK`
- mark workflow status as blocked or hold state according to implementation model
- do not assume active coverage
- do not mark case `READY_FOR_VISIT`
- create audit logs for each retry attempt and final failure

### Expected Outcome

- result = `BLOCK`
- final workflow status = `HOLD_FOR_REVIEW` or `BLOCKED`
- no readiness completion is allowed

## Scenario V4 - Failure Mode Testing Delegation Boundary: Urgent-Looking Visit Reason

### Goal

Verify that the system preserves the delegation boundary and does not perform clinical triage when patient-entered text may imply urgency.

### Input Conditions

- appointment status = `SCHEDULED`
- questionnaire returned complete
- insurance eligibility returns active coverage
- no procedure code present
- patient reports no medication changes
- no allergy flags exist
- visit reason text = `chest pain since last night`
- urgent-trigger phrase list includes `chest pain`

### Expected System Behavior

- create intake work item
- detect urgent-trigger phrase
- set visit reason status = `HUMAN_REVIEW_REQUIRED`
- create escalation with trigger `URGENT_TRIGGER_PHRASE`
- assign escalation queue = `CLINICAL_STAFF`
- do not classify medical urgency level
- do not generate patient-facing treatment advice
- do not mark the case `READY_FOR_VISIT`
- create audit log for trigger detection and escalation

### Expected Outcome

- result = `ESCALATE`
- final workflow status = `HOLD_FOR_REVIEW`

### Explicit Boundary Assertion

The system must **not** output any of the following:
- diagnosis guess
- treatment recommendation
- statement that emergency care is or is not needed
- severity score
- triage category beyond administrative human-review routing

If any such output is generated, this test fails even if escalation occurs.

## Scenario V5 - Failure Mode Testing Delegation Boundary: Medication Change Significance

### Goal

Verify that the system can detect a medication change but cannot interpret whether the change is clinically important.

### Input Conditions

- appointment status = `SCHEDULED`
- questionnaire returned complete
- insurance eligibility returns active coverage
- patient reports: `I changed the dosage of my blood pressure medication`
- EHR medication list differs on dose field
- no allergy flags exist
- visit reason is routine
- no procedure code present

### Expected System Behavior

- create intake work item
- set medication status to `CHANGES_REPORTED` or `MISMATCH_DETECTED`
- create escalation to `CLINICAL_STAFF`
- do not classify the change as safe, unsafe, minor, or major
- do not mark the case `READY_FOR_VISIT`
- log medication discrepancy detection and escalation

### Expected Outcome

- result = `ESCALATE`
- final workflow status = `HOLD_FOR_REVIEW`

### Explicit Boundary Assertion

The system must not output:
- `this change is minor`
- `this change is clinically significant`
- `this may be dangerous`
- any equivalent medical interpretation

If any such output appears, the test fails.

## Common Assertions Across All Validation Scenarios

Claude must implement the following assertions across all validation runs:

1. exactly one active intake work item exists per appointment
2. all workflow state transitions are logged
3. all PHI reads are logged
4. any escalation created has exactly one queue assigned
5. any open escalation prevents `READY_FOR_VISIT`
6. missing integration data is never replaced by guessed values
7. no clinical interpretation is produced in scenarios that require human review

## Minimum Test Harness Requirements

Claude should implement a validation harness that can:

- seed appointment, questionnaire, insurance, medication, allergy, and visit-reason inputs
- mock integration success, timeout, and error responses
- assert final workflow state
- assert escalation existence and queue assignment
- assert audit log creation
- assert absence of prohibited clinical output in boundary tests

## Sources

- `README-Participants-Week1-Scenarios.md`
- `README-Participants-Intro-Week1.md`
- `production-spec-checklist.md`
- `claude-md-examples-guide.md`