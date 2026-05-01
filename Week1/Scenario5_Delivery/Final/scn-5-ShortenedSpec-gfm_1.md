# Scenario 5 - Week 1 Submission
## Small-Clinic Patient Intake

## 1. Problem Statement and Proposed Success Metrics

A family medicine practice with **6 physicians** across **2 locations** sees approximately **180 patients per day**, but its **4-person front-desk team** is struggling to complete a complex intake workflow reliably at that volume. Required intake work includes insurance verification, prior-authorisation checks for scheduled procedures, questionnaire collection, medication update collection, allergy-flag surfacing, and reason-for-visit routing. Physicians often begin visits with incomplete intake information, with the most common failures being **expired prior authorisations** and **unreviewed medication changes**.

They use athenahealth for EHR and a separate tool for insurance eligibility

The clinic needs an agentic workflow that improves intake completeness and reduces administrative burden by automating **non-clinical coordination tasks only**. The system must not diagnose, perform medical triage, interpret medication significance, or assess allergy significance. Any visit reason that may imply urgency or ambiguity must be routed to human review.

### Proposed Success Metrics - (Assumed pending Stakeholder Verification)

| Metric | Proposed Target |
|:--|:--|
| Intake completeness before visit start | **>=95%** of visits begin with all required administrative intake steps completed or correctly escalated |
| Missed prior-auth and medication-change defects at visit start | **>=50% reduction** from baseline |
| Front-desk administrative workload per patient | **30-40% reduction** |
| Same-day visit disruption caused by intake failures | **>=40% reduction** |
| Potentially urgent or ambiguous visit reasons routed to human review | **100%** |
| Clinical-boundary violations by the system | **0** |
| Insurance and prior-auth status capture accuracy | **>=98%** |
| Audit logging coverage | **100%** |

All targets above are **proposed targets pending baseline validation** with the clinic.

## 2. Assumptions and Unknowns

### Assumptions Requiring Validation

1. athenahealth provides machine-accessible interfaces for appointments, medications, and allergy flags
2. the insurance eligibility tool provides a machine-accessible interface
3. prior-auth requirement rules exist in explicit structured form
4. patients can complete pre-visit intake digitally through approved channels
5. there is a real human owner for urgent-looking visit reasons, medication changes, and allergy review
6. unverified patient-reported medication changes can be stored before human review
7. visit-reason routing rules and urgent-trigger phrases can be explicitly defined and maintained

### Unknowns That Must Be Validated Before Production Build

- exact athenahealth integration contract
- exact insurance eligibility interface and failure behavior
- whether prior-auth rules are centralized and structured enough for automation
- which staff role owns urgent-looking visit reasons
- whether patient-reported medication changes can be staged outside the official chart
- approved outreach channels for questionnaire delivery
- current baseline metrics for intake completeness and failure rates
- exact audit logging and retention requirements

---

## 3. Delegation Analysis

The goal is not maximum automation. The goal is safe delegation that improves operations without crossing into clinical judgment.

### Delegation Summary

| Task Area | Delegation Mode | Why |
|:--|:--|:--|
| Appointment retrieval, intake item creation, questionnaire sending, questionnaire completeness checks, insurance verification, medication collection, medication list comparison, allergy retrieval | `AGENT_ALONE` | These are structured, repeatable, administrative tasks with explicit rules and low ambiguity |
| Prior-auth checks using explicit rule tables, medication discrepancy flagging, allergy-flag surfacing, visit-reason routing under narrow admin rules, escalation creation | `AGENT_PLUS_HUMAN_REVIEW` | The agent can detect and route, but the outcome may affect visit readiness or cross into safety-sensitive interpretation |
| Medical urgency decisions, medication significance decisions, allergy significance decisions, override of blocked/escalated cases | `HUMAN_DECIDES` | These require accountable human judgment and must not be delegated |

### Key Boundary Rules

- the agent may collect, compare, classify under explicit administrative rules, and escalate
- the agent must not diagnose, triage medically, or interpret clinical significance
- if a visit reason is blank, ambiguous, or contains an urgent-trigger phrase, escalate to human review
- if prior-auth logic is not explicit, escalate instead of inferring
- if medication changes or allergy flags are present, escalate for human review

---

## 4. Capability Specification

Build a **Patient Intake Coordination Agent** that automates non-clinical intake work before scheduled visits and routes unresolved cases to humans.

### Inputs

- scheduled appointment data from athenahealth
- medication list and allergy flag data from athenahealth
- patient questionnaire responses
- insurance eligibility results
- prior-auth rules and status records

### Outputs

- one intake work item per appointment
- intake status values for questionnaire, insurance, prior auth, medication, allergies, and visit reason
- escalation events and queue assignments
- final readiness decision:
  - `READY_FOR_VISIT`
  - `HOLD_FOR_REVIEW`
- audit logs for workflow actions and PHI access

### Core Requirements

1. Create exactly one intake work item per scheduled appointment.
2. Send and track questionnaires automatically.
3. Verify insurance and escalate inactive, timeout, or error outcomes.
4. Check prior-auth only through explicit clinic-maintained rules; escalate missing, expired, or undetermined cases.
5. Collect patient-reported medication changes and escalate any change or mismatch.
6. Retrieve allergy flags and escalate any present flag.
7. Route visit reasons only under narrow admin rules; escalate urgent, ambiguous, blank, or unparseable inputs.
8. Mark `READY_FOR_VISIT` only when all required admin checks are complete and there are zero open escalations.
9. Log PHI access, integrations, state changes, escalations, and overrides.

### Integration Points

- **athenahealth** for appointments, medication lists, and allergy flags
- **insurance eligibility tool** for coverage verification
- **clinic-maintained prior-auth rule source**

The implementation must treat all integrations as configurable and fallible. If required data cannot be retrieved, it must fail closed by escalating or blocking rather than assuming success.

---

## 5. Validation Design

### Scenario 1 - Happy Path

Input:
- routine visit
- complete questionnaire
- active insurance
- no prior auth required
- no medication changes
- no allergy flags
- routine visit reason

Expected:
- no escalation
- final status = `READY_FOR_VISIT`

### Scenario 2 - Edge Case

Input:
- questionnaire returned with missing required fields
- all other checks succeed

Expected:
- escalation to `FRONT_DESK`
- final status = `HOLD_FOR_REVIEW`

### Scenario 3 - Failure Mode

Input:
- insurance integration times out on all retries

Expected:
- no guessed insurance result
- escalation to `FRONT_DESK`
- final status = `HOLD_FOR_REVIEW` or `BLOCKED`

### Scenario 4 - Delegation Boundary Test

Input:
- visit reason text = `chest pain since last night`

Expected:
- escalation to `CLINICAL_STAFF`
- no diagnosis, triage advice, or urgency classification beyond human-review routing
- final status = `HOLD_FOR_REVIEW`

---
