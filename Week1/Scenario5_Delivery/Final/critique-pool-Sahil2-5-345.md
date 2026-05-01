# Week 1 — Agent Specification: Small-Clinic Patient Intake Automation
**Scenario 5 | Practice Submission **

---

## ⚠️ Non-Clinical Boundary Declaration

This specification assigns **zero clinical judgment to the agent**. The agent performs administrative coordination, data completeness checks, rules-based routing, and escalation triggering only. All decisions involving diagnosis, clinical urgency interpretation, medication safety, or treatment appropriateness remain **exclusively human-led**. This is not a design preference — it is a hard constraint stated by the practice manager and reinforced by HIPAA, state medical-records law, and standard-of-care accountability.

---

## 1. Problem Statement & Success Metrics

### Context

A 6-physician family medicine practice (2 locations, ~180 patients per day) runs patient intake through a 4-person front-desk team. The intake workflow per visit spans six tasks: insurance verification, prior-authorisation (prior auth) check, pre-visit questionnaire, reason-for-visit routing, medication reconciliation, and allergy-flag review.

Physicians regularly discover at the visit that intake was incomplete — most commonly an **expired prior auth** or an **unreviewed medication change**. This causes visit delays, rescheduling, and clinician frustration.

The practice manager has three hard constraints:
1. No clinical judgment by the agent
2. Any contact with the stated visit reason must preserve a clear human escalation path
3. HIPAA and state medical-records compliance is non-negotiable

Existing stack: **athenahealth** (EHR), a **separate insurance eligibility tool** (name unknown at this stage — see Assumptions), and **no existing AI infrastructure**.

### Sizing the Problem

| Metric | Current State | Source |
|---|---|---|
| Patients per day (both locations) | ~180 | Scenario |
| Front-desk headcount | 4 | Scenario |
| Patients per front-desk person per day | ~45 | Derived |
| Primary miss-type at physician visit | Expired prior auth or unreviewed medication change | Scenario |
| AI infrastructure in place today | None | Scenario |

### Success Metrics (Assumed Pending Stakeholder Agreement)

The following metrics define "working" for this engagement. They are grounded in the scenario numbers, not generic benchmarks.

| Metric | Baseline (Assumed) | Target | How Measured |
|---|---|---|---|
| Prior auth expired at time of visit | Unknown — **must be measured** | <2% of scheduled visits with prior auth requirement | athenahealth visit record audit |
| Unreviewed medication change at visit | Unknown — **must be measured** | 0 visits where agent-flagged medication change was not reviewed before physician entry | Agent audit log vs. EHR visit record |
| Front-desk intake task completion time per patient | Unknown — **must be measured** | ≥40% reduction in time spent on insurance verification + prior auth check | Time-stamp comparison before/after |
| Intake questionnaire completion rate before visit | Unknown — **must be measured** | ≥85% of patients submit pre-visit questionnaire ≥30 min before appointment | Questionnaire system log |
| Agent escalation false-positive rate (flagged but not genuinely incomplete) | Not applicable today | <15% of escalations are false positives, measured monthly | Human review outcome log |
| HIPAA audit events per quarter | Not applicable today | Zero unlogged PHI access events | Audit trail review |

> **Note:** Baseline figures for all metrics are unknown. The first two weeks of any build must include instrumentation to establish baselines before measuring improvement. All targets above are hypotheses requiring stakeholder validation — see Section 2.

---

## 2. Assumptions & Unknowns

Each assumption is load-bearing. If the assumption is wrong, the column "What Breaks" describes the failure mode.

| ID | Assumption | Status | What Breaks If Wrong | Validate With |
|---|---|---|---|---|
| A1 | athenahealth exposes a REST or FHIR API with sufficient read access to patient demographics, appointments, medication lists, allergy lists, and prior auth records | **[Flagged for Validation]** | The entire integration architecture collapses — athenahealth has tiered API access and some data types require additional licensing | Practice manager + athenahealth account rep; test with sandbox credentials before any build |
| A2 | The insurance eligibility tool has an API or webhook that returns eligibility status and prior auth validity by patient ID and procedure code | **[Flagged for Validation]** | Prior auth check capability cannot be automated; this is the most commonly missed item per the scenario | Identify tool name; review API docs; test a sample auth check in sandbox |
| A3 | The pre-visit questionnaire can be sent via SMS or email link and the practice already has patient contact data (mobile + email) stored in athenahealth | **[Assumed]** | Questionnaire delivery mechanism must be built from scratch or vendor-sourced; adds significant scope | Confirm with practice manager; test contact data completeness on a sample of 50 records |
| A4 | Medication reconciliation means comparing the patient's current medication list in athenahealth against any new medications reported in the pre-visit questionnaire — not against an external pharmacy system | **[Assumed]** | If pharmacy system integration is required, scope doubles and a new integration contract is needed | Confirm with practice manager: does "reconciliation" mean EHR comparison only, or does it include external pharmacy data? |
| A5 | "Reason for visit" is a structured field in athenahealth (or the pre-visit questionnaire), not free-text that requires NLP interpretation | **[Flagged for Validation]** | If free-text, the agent cannot reliably route without NLP, which adds model cost, latency, and clinical interpretation risk | Pull 20 sample visit records and inspect the visit-reason field format |
| A6 | The 4-person front-desk team will serve as the human-in-the-loop reviewers for all agent escalations; no physician or clinical staff involvement is expected in the escalation workflow | **[Assumed]** | If physicians expect to receive escalation notifications directly, the notification routing logic must change | Confirm with practice manager; map escalation recipients explicitly before build |
| A7 | State medical-records compliance requirements are consistent with HIPAA minimum-necessary access — no state-specific additional constraints exist (e.g., California CMIA stricter consent requirements) | **[Flagged for Validation]** | Additional consent or data-handling rules could require changes to audit trail and PHI access logging | Review applicable state law with practice's compliance officer or legal counsel before go-live |
| A8 | Both practice locations use the same athenahealth instance (single tenant, single API credential set) | **[Assumed]** | If separate instances, all integration work doubles and cross-location deduplication logic is needed | Confirm with practice manager during discovery |

---

## 3. Delegation Analysis

### Guiding Principle

The delegation boundary is set by three constraints: (1) the practice manager's hard rules, (2) reversibility of error, and (3) HIPAA accountability. Tasks are pushed to **Agent Alone** only when the consequence of an error is correctable before it reaches the physician. Tasks stay **Human Decides** when an error could affect care, when clinical judgment is implied, or when the task is a non-negotiable compliance action.

### Intake Task Boundary Table

| Intake Task | Delegation Level | Rationale |
|---|---|---|
| Pull patient record from athenahealth for upcoming appointments (T-24h to T-30min window) | **Agent Alone** | Pure data retrieval; no judgment; read-only; low consequence of failure (human sees gap at next step) |
| Run insurance eligibility check via eligibility tool | **Agent + Log** | Rules-based API call with deterministic pass/fail; agent logs result and timestamp; no clinical content involved |
| Check prior auth existence and expiry date against scheduled procedure | **Agent + Log** | Deterministic: prior auth record either exists and is valid, or it doesn't/isn't; agent flags expired/missing and logs; human reviews flag |
| Send pre-visit questionnaire to patient (SMS/email) | **Agent + Log** | Administrative delivery action; logged for audit; no clinical content in the send event itself |
| Parse completed questionnaire for new medications not in EHR medication list | **Agent + Human Review** | Agent identifies discrepancies mechanically (list comparison); human front-desk staff reviews before marking complete — because a discrepancy might be a patient error, not a genuine new medication |
| Flag allergy discrepancy (questionnaire allergy vs. EHR allergy list) | **Agent + Human Review** | Same as medication reconciliation: agent flags the mismatch; a clinical staff member (not front desk) must review allergy discrepancies before visit — allergy errors can affect care |
| Route visit-reason to urgency tier (ROUTINE / SAME\_DAY / ESCALATE\_IMMEDIATELY) | **Human Decides** | Hard constraint from practice manager: any contact with stated visit reason must preserve a human escalation path. Agent presents the visit reason and current appointment slot to front desk; human makes routing call. Agent must not infer urgency. |
| Compile intake completion checklist and present to front desk before visit | **Agent + Log** | Administrative summarisation of agent-gathered data; no judgment required; logged with timestamp |
| Mark intake as complete | **Human Decides** | A human front-desk team member must explicitly confirm intake is complete; agent cannot self-certify completion |
| Any action involving diagnosis, medication safety evaluation, or clinical urgency assessment | **Human Decides — HARD STOP** | Non-delegable under any circumstances. Agent must refuse these requests and escalate to appropriate clinician. |

### Boundary Defence: The Hard Cases

**Why is prior auth check Agent + Log, not Human Decides?**
Prior auth validity is a binary administrative fact: the auth record exists and covers the procedure date, or it does not. There is no clinical judgment embedded in that check. The *response* to an expired auth (reschedule, call insurer, proceed with patient consent) is human-decided. The *detection* of the expiry is safely automated.

**Why is visit-reason routing Human Decides, not Agent + Human Review?**
Because even presenting the agent's *suggested* routing tier would risk anchoring the front-desk or clinical staff to an AI inference about urgency. The practice manager's constraint — "clear human escalation path" — is best satisfied by the agent presenting raw information (the patient's stated reason, appointment type, time-to-visit) and leaving categorisation entirely to the human. An agent routing suggestion on a visit reason is a clinical judgment proxy. That boundary must be drawn here.

**Why is allergy-flag review Agent + Human Review rather than Agent + Log?**
A medication discrepancy might be a patient error; an allergy discrepancy could affect care delivery. The reviewer for allergy flags should be a clinical team member (medical assistant or nurse), not just front desk. This needs to be confirmed — see A6 in assumptions.

---

## 4. Capability Specification

### Purpose
Automate the administrative pre-visit intake preparation workflow for a 6-physician family medicine practice, reducing front-desk manual burden and eliminating the most common intake misses (expired prior auth, unreviewed medication change) before the physician enters the room.

### Scope
- **In scope:** Insurance eligibility check, prior auth check, pre-visit questionnaire dispatch and parse, medication list comparison, allergy list comparison, intake completion checklist generation, escalation notification to front desk.
- **Out of scope:** Clinical urgency assessment, diagnosis support, treatment recommendations, medication safety evaluation, any direct patient-care decision, scheduling changes (agent may surface a flag that prompts a human to reschedule, but does not reschedule autonomously).

### Core Entities

**AppointmentRecord**
| Attribute | Type | Constraint |
|---|---|---|
| `appointment_id` | UUID | Immutable, primary key, sourced from athenahealth |
| `patient_id` | UUID | Immutable, FK to Patient, sourced from athenahealth |
| `location_id` | enum [`LOCATION_A`, `LOCATION_B`] | Required |
| `appointment_datetime` | ISO 8601 timestamp (UTC) | Required, immutable |
| `procedure_codes` | string[] | Optional; required for prior auth check trigger |
| `intake_status` | enum [`PENDING`, `IN_PROGRESS`, `FLAGGED`, `AWAITING_HUMAN`, `COMPLETE`] | Default `PENDING` |
| `created_at` | ISO 8601 timestamp | Immutable, set on record creation |
| `updated_at` | ISO 8601 timestamp | Updated on any state change |

**State machine for `intake_status`:**
- `PENDING` → `IN_PROGRESS` (agent begins processing, T-24h trigger)
- `IN_PROGRESS` → `FLAGGED` (any check fails or discrepancy found)
- `IN_PROGRESS` → `AWAITING_HUMAN` (questionnaire sent, awaiting patient response)
- `AWAITING_HUMAN` → `IN_PROGRESS` (questionnaire received)
- `FLAGGED` → `IN_PROGRESS` (human acknowledges flag and marks reviewed)
- `IN_PROGRESS` → `COMPLETE` (human front-desk confirms all checks passed)
- `COMPLETE` is terminal for the appointment cycle

**IntakeFlag**
| Attribute | Type | Constraint |
|---|---|---|
| `flag_id` | UUID | Immutable |
| `appointment_id` | UUID | FK to AppointmentRecord |
| `flag_type` | enum [`PRIOR_AUTH_EXPIRED`, `PRIOR_AUTH_MISSING`, `INSURANCE_INACTIVE`, `MEDICATION_DISCREPANCY`, `ALLERGY_DISCREPANCY`, `QUESTIONNAIRE_INCOMPLETE`, `QUESTIONNAIRE_NOT_RETURNED`] | Required, exhaustive |
| `flag_detail` | string | Max 500 chars; plain-language description of the discrepancy |
| `status` | enum [`OPEN`, `REVIEWED`, `RESOLVED`, `DISMISSED`] | Default `OPEN` |
| `reviewed_by` | string (front-desk staff ID) | Nullable; set when status → `REVIEWED` |
| `reviewed_at` | ISO 8601 timestamp | Nullable; set when status → `REVIEWED` |
| `created_at` | ISO 8601 timestamp | Immutable |

### Requirements

| ID | Requirement | Acceptance Criterion |
|---|---|---|
| R1 | The agent must poll athenahealth for appointments scheduled within the next 24 hours, once per hour, starting at 06:00 local clinic time. | For each appointment returned, an `AppointmentRecord` with `intake_status = PENDING` is created (or updated if existing). Zero appointments scheduled in the 24h window = zero records created. Duplicate appointment_ids are ignored (idempotent). |
| R2 | For each `PENDING` appointment, the agent must call the insurance eligibility API with `patient_id` and today's date. If response is `INACTIVE` or API returns an error, the agent must create an `IntakeFlag` with `flag_type = INSURANCE_INACTIVE` and set `intake_status = FLAGGED`. | Happy path: eligibility = `ACTIVE` → no flag created, record proceeds. Fail path: eligibility = `INACTIVE` → flag created within 5 seconds of API response, front desk notified via dashboard alert. |
| R3 | If `procedure_codes` is non-empty on the appointment, the agent must check prior auth status via the eligibility tool. If no valid prior auth exists for the procedure code and appointment date, or if the prior auth expiry date is < appointment date, the agent must create an `IntakeFlag` of type `PRIOR_AUTH_EXPIRED` or `PRIOR_AUTH_MISSING` respectively. | Prior auth check must complete and flag (if applicable) must be created before T-4 hours prior to appointment. If appointment is within 4 hours of discovery, escalation notification must be sent immediately (not batched). |
| R4 | The agent must send a pre-visit questionnaire link to the patient via SMS (primary) or email (fallback) at T-24h. Questionnaire must include: current medications, any new medications since last visit, known allergies, and reason for visit (free-text, unconstrained — not interpreted by agent). | SMS sent and delivery receipt logged within 2 minutes of T-24h trigger. If SMS fails (invalid number or carrier error), email sent within 5 minutes. If both fail, `IntakeFlag` of type `QUESTIONNAIRE_INCOMPLETE` created and front desk notified. |
| R5 | The agent must parse the returned questionnaire and compare the patient-reported medication list against the medication list in athenahealth. Any medication present in questionnaire but absent from EHR (or vice versa, if patient reports stopping a medication) must generate an `IntakeFlag` of type `MEDICATION_DISCREPANCY`. The agent must not evaluate whether the discrepancy is clinically significant. | Comparison is string-normalised (lowercase, stripped whitespace). Each unmatched medication = one flag. Agent must not merge or suppress multiple flags. |
| R6 | The agent must compare patient-reported allergies in the questionnaire against the allergy list in athenahealth. Any discrepancy must generate an `IntakeFlag` of type `ALLERGY_DISCREPANCY`. The agent must not assess severity. | Same string-normalisation as R5. Flag created for each discrepancy. Flag detail must include both the EHR value and the patient-reported value verbatim. |
| R7 | If the questionnaire is not returned by T-2 hours before appointment, the agent must create an `IntakeFlag` of type `QUESTIONNAIRE_NOT_RETURNED` and notify front desk. The agent must not send a second questionnaire without front-desk instruction. | Flag created at exactly T-2h. Front-desk notification via dashboard. No automated reminder without human authorisation. |
| R8 | The agent must compile an intake completion checklist for each appointment and surface it to the front-desk dashboard. The checklist must show: eligibility status, prior auth status (with expiry date if applicable), questionnaire status, open flag count, and a list of all unresolved `IntakeFlag` records. The agent must not mark intake as complete. | Checklist visible in dashboard ≥30 minutes before appointment. All IntakeFlag records with `status = OPEN` must be listed. Checklist is read-only — the "Mark Complete" action is only available to front-desk users. |
| R9 | Every agent action that reads or writes PHI must be logged with: `timestamp` (ISO 8601 UTC), `action_type`, `patient_id`, `appointment_id`, `agent_version_id`, `data_fields_accessed` (list). Logs are immutable and retained for 6 years (HIPAA minimum). | Zero PHI access events may be unlogged. Logs must be written before the action completes, not after. Any log write failure must cause the action to abort and alert the ops/admin user. |
| R10 | The agent must refuse any request to interpret, assess, or categorise the clinical urgency of a visit reason. If any downstream process attempts to pass a visit reason to the agent for urgency classification, the agent must return a structured error: `{"error": "CLINICAL_JUDGMENT_REFUSED", "message": "Visit reason urgency assessment is not within agent scope. Route to front-desk or clinical staff."}` and log the attempt. | Test: send a prompt containing a visit reason and ask for urgency classification. Expected: error returned, no classification produced, attempt logged with `action_type = CLINICAL_BOUNDARY_VIOLATION_ATTEMPT`. |

### Integration Contracts

**Integration 1: athenahealth EHR API** (machine readable Placeholder as per Unknowns - Flagged for Validation)

| Contract Element | Value |
|---|---|
| Purpose | Read patient demographics, appointments, medication lists, allergy lists, prior auth records |
| Endpoint pattern | `GET https://api.athenahealth.com/v1/{practiceid}/appointments` (appointments); `GET /patients/{patientid}/medications`; `GET /patients/{patientid}/allergies`; `GET /patients/{patientid}/authorization` |
| Auth method | OAuth 2.0 client credentials; access token stored in secrets manager (key: `ATHENA_ACCESS_TOKEN`); token refresh on 401 response |
| Request format | JSON; `practiceid` and `patientid` are integer strings |
| Response format | JSON; array of appointment/medication/allergy/auth objects per athenahealth FHIR R4 schema |
| Timeout | 10 seconds per call |
| Retry logic | HTTP 5xx: retry 2× with exponential backoff (3s, 6s); HTTP 429: retry after `Retry-After` header value; HTTP 4xx (except 401): do not retry, log error, escalate to admin |
| Rate limits | [**Unknown — Flagged for Validation**] athenahealth rate limits vary by tier; assume 100 req/min until confirmed |
| Fallback | If athenahealth is unavailable for >10 minutes: create `IntakeFlag` of type `QUESTIONNAIRE_INCOMPLETE` for all in-progress appointments; notify admin; halt further processing until connectivity restored |
| PHI data mapping | `patient_id` maps to `Patient.id` in internal model; all PHI fields access-logged per R9 |

**Integration 2: Insurance Eligibility Tool** (machine readable Placeholder as per Unknowns - Flagged for Validation)

| Contract Element | Value |
|---|---|
| Purpose | Check insurance eligibility status and prior auth validity |
| Endpoint | **[Unknown — Flagged for Validation]** Tool name not specified in scenario; endpoint and auth method TBD |
| Auth method | **[Unknown — Flagged for Validation]** |
| Request format | **[Assumed]** `{ "patient_id": string, "insurance_member_id": string, "procedure_code": string (optional), "service_date": ISO 8601 date }` |
| Response format | **[Assumed]** `{ "eligibility_status": enum [ACTIVE, INACTIVE, UNKNOWN], "prior_auth": { "exists": boolean, "auth_number": string\|null, "expiry_date": ISO 8601 date\|null, "procedure_code": string\|null } }` |
| Timeout | 8 seconds |
| Retry logic | Same as athenahealth pattern above |
| Fallback | If eligibility tool unavailable: create `IntakeFlag` of type `INSURANCE_INACTIVE` for all appointments pending eligibility check, with flag_detail = "Eligibility tool unavailable — manual verification required"; notify front desk |

**Integration 3: Notification Service (SMS/Email)** (Placeholder until connctivity mechanism is Confirmed - Currently UnKnown)

| Contract Element | Value |
|---|---|
| Purpose | Deliver pre-visit questionnaire links; send escalation alerts to front desk |
| SMS provider | **[Unknown — Flagged for Validation]** Assumed Twilio-compatible; endpoint `POST https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json` |
| Email fallback | **[Unknown]** SMTP or SendGrid; TBD |
| Auth | Bearer token; stored in secrets manager (key: `NOTIFICATION_SERVICE_TOKEN`) |
| Timeout | 5 seconds |
| Fallback | If SMS and email both fail: create `QUESTIONNAIRE_INCOMPLETE` flag; log delivery failure with error code; front desk notified via dashboard only |

---

## 5. Validation Design

### Scenario V1 — Happy Path: Full Clean Intake

| Element | Detail |
|---|---|
| **Given** | Patient has appointment T-24h; insurance is ACTIVE; prior auth exists and is valid for the procedure; patient returns questionnaire T-3h with medications and allergies matching EHR; no discrepancies |
| **When** | Agent processes full intake cycle |
| **Then** | Zero `IntakeFlag` records created; `intake_status = IN_PROGRESS` transitions to front-desk dashboard showing green checklist; front desk can mark complete; all PHI access events logged |
| **Pass Criteria** | Checklist visible ≥30 min before appointment; zero open flags; intake completion available to human; all 4 API calls (athenahealth ×2, eligibility ×1, notification ×1) logged with timestamps |

### Scenario V2 — Edge Case: Appointment Within 4 Hours of Discovery (Expired Prior Auth)

| Element | Detail |
|---|---|
| **Given** | A same-day appointment is booked at 08:00; agent's T-24h run did not fire (e.g., appointment booked late); next poll at 10:00 discovers appointment at 13:00 (3 hours away); prior auth expired 2 days ago |
| **When** | Agent processes the late-discovered appointment |
| **Then** | `IntakeFlag` of type `PRIOR_AUTH_EXPIRED` created immediately; R3 states "if appointment is within 4 hours of discovery, escalation notification sent immediately (not batched)"; front desk receives immediate dashboard alert plus SMS/email notification; `intake_status = FLAGGED` |
| **Pass Criteria** | Immediate (not next-batch) escalation; flag created within 30 seconds of discovery; notification delivered and delivery logged; physician not notified by agent (front desk decides next action) |

### Scenario V3 — Failure Mode: Agent Receives Visit-Reason Urgency Request (Delegation Boundary Test)

| Element | Detail |
|---|---|
| **Given** | A downstream integration or misconfigured workflow step passes the patient's visit reason ("chest pain since yesterday") to the agent and requests urgency tier classification |
| **When** | Agent receives the classification request |
| **Then** | Agent returns `{"error": "CLINICAL_JUDGMENT_REFUSED", "message": "Visit reason urgency assessment is not within agent scope. Route to front-desk or clinical staff."}`; no classification produced; attempt logged with `action_type = CLINICAL_BOUNDARY_VIOLATION_ATTEMPT`, including the requesting system ID and timestamp |
| **Pass Criteria** | No urgency tier produced under any input; refusal response returned in <1 second; log entry exists with full context; no PHI from the visit reason is retained in agent working memory beyond the log entry |
| **Why This Test Matters** | This is the delegation boundary test. It verifies that the hard constraint ("no clinical judgment by agent") is enforced at the code level, not just in documentation. A passing spec that allows this request to produce output has failed the most important design requirement. |

### Scenario V4 — Failure Mode: Both athenahealth and Eligibility Tool Unavailable Simultaneously

| Element | Detail |
|---|---|
| **Given** | At T-24h processing window, athenahealth API returns 503 on all retries; eligibility tool also unavailable |
| **When** | Agent attempts to process 12 upcoming appointments |
| **Then** | For each appointment: `IntakeFlag` of type `INSURANCE_INACTIVE` created with `flag_detail = "Eligibility tool unavailable — manual verification required"`; admin alert sent; agent halts further processing for affected appointments; does not retry indefinitely; logs all failure events |
| **Pass Criteria** | No silent failures; every affected appointment has a flag; admin is notified; front desk dashboard shows all affected appointments as `FLAGGED`; no partial or phantom data written to EHR |

### Scenario V5 — Edge Case: Medication List Comparison With Spelling Variants

| Element | Detail |
|---|---|
| **Given** | EHR lists medication as "metformin HCl 500mg"; patient reports "metformin 500" on questionnaire |
| **When** | Agent runs medication reconciliation (R5) |
| **Then** | After string normalisation (lowercase, strip whitespace), "metformin hcl 500mg" ≠ "metformin 500" → flag created with `flag_type = MEDICATION_DISCREPANCY`; flag_detail includes both values verbatim |
| **Pass Criteria** | Flag is created (not suppressed); agent does not attempt to determine if they are the same drug; front-desk reviewer sees both values; human makes the determination |
| **Design Note** | This is intentionally conservative. False positives here are acceptable; false negatives (suppressing a genuine discrepancy) are not. The rate of false positives from string comparison should be tracked and used to decide if fuzzy matching warrants a future spec update. |

---

## Appendix: Governance & HIPAA Compliance Summary

| Requirement | Specification |
|---|---|
| PHI access logging | Every read or write of PHI logged: `timestamp`, `action_type`, `patient_id`, `appointment_id`, `agent_version_id`, `data_fields_accessed`. Immutable. |
| Log retention | 6 years minimum (HIPAA §164.530(j)); practice legal counsel to confirm state-specific requirement (see A7) |
| PHI in logs | Logs record field names accessed, not field values — except where the field value is non-identifying (e.g., appointment_id). Patient name, DOB, diagnosis never stored in agent logs. |
| HITL checkpoints | (1) Allergy flag review: clinical staff only. (2) Intake completion: front-desk confirmation required. (3) Visit-reason routing: front desk only. (4) Any prior auth response action: front desk decides. |
| Agent cannot do | Diagnose, triage clinically, assess medication safety, infer urgency, complete its own intake checklist, contact the patient more than once without human authorisation |
| Minimum necessary rule | Agent requests only the data fields required for the specific check being performed. No bulk patient record downloads. Each API call scoped to appointment_id or patient_id + field type. |
| BAA requirement | A Business Associate Agreement must be in place with athenahealth, the eligibility tool vendor, the SMS provider, and any notification service before PHI is transmitted. This is a go/no-go condition, not a nice-to-have. |

---

*Source references: `README-Participants-Week1-Scenarios.md` (Scenario 5 constraints and numbers), `production-spec-checklist.md` (delegation categories, integration contract template, validation design standard, assumptions register format), `claude-md-examples-guide.md` (entity definition and CLAUDE.md pattern), `spec-ambiguity-vs-builder-mistakes.md` (boundary test rationale).*