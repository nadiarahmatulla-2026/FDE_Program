# Scenario 5: Patient Intake Coordination Agent — Comprehensive Specification

**Status:** Week 1 Practice Spec | **Scenario:** 5 — Small-Clinic Patient Intake | **Target:** AI-Buildable Capability Specification

---

## Executive Summary

This spec defines a **Patient Intake Coordination Agent** for a 6-physician family medicine practice (2 locations, ~180 patients/day) that automates non-clinical administrative intake work while preserving human control over all clinical judgment.

**The hard constraint:** the agent must never diagnose, triage medically, or replace human clinical judgment. It collects, routes, validates, and escalates administrative intake tasks; clinical decisions remain human.

**The opportunity:** reduce front-desk workload (4 people managing ~180 daily intakes) by automating structured administrative workflow steps — questionnaire dispatch, insurance verification, prior-auth checking, medication change detection, allergy flag retrieval, and visit-reason routing.

---

## 1. Assumptions & Unknowns

### Known Facts From Scenario

- 6 physicians across 2 locations
- ~180 patients per day
- 4-person front-desk team
- Intake steps: insurance verification, prior-auth checks, pre-visit questionnaire, medication reconciliation, allergy-flag review, reason-for-visit triage
- Most common failures: **expired prior auths** and **unreviewed medication changes**
- EHR: athenahealth
- Insurance verification: separate tool (integration method unknown)
- Compliance: HIPAA + state medical-record requirements (exact retention/audit policy unknown)
- No AI infrastructure today

### Critical Assumptions (with validation discipline)

#### Assumption A1: athenahealth Has Machine-Accessible APIs

**Assumption:** athenahealth exposes structured, machine-consumable APIs for appointment data, medication lists, and allergy flags.

**Hypothesis:** If athenahealth APIs exist and expose appointment_id, medication data, and allergy records, then the agent can retrieve required EHR data automatically to initialize intake workflows.

**How to test it:** Request integration documentation from clinic IT or athenahealth success team. Confirm available endpoints, authentication method, rate limits, and whether data is structured (XML, JSON) or unstructured (text, PDFs).

**Confidence:** **MEDIUM** — athenahealth is industry-standard for small practices and does offer APIs, but this specific clinic may not have API access provisioned, or may have contractual restrictions.

**Impact if false:** Major sections of the spec (appointment sync, EHR retrieval, medication comparison) become unfeasible. Design shifts to manual or RPA approaches, or remains limited to post-visit workflows.

---

#### Assumption A2: Insurance Eligibility Tool Supports Integration

**Assumption:** The separate insurance eligibility tool is accessible through an API, batch interface, or approved RPA connection (not manual-portal-only).

**Hypothesis:** If machine-accessible integration exists, the agent can submit eligibility checks programmatically and consume structured responses (active/inactive/error/timeout).

**How to test it:** Ask the clinic operations lead about the eligibility tool vendor, current usage, and whether API/batch/RPA options are available or approved. Confirm response structure and timeout behavior.

**Confidence:** **MEDIUM-HIGH** — many eligibility vendors do offer integrations, but if this clinic is currently running manual verification, integration may require procurement or IT approval.

**Impact if false:** Insurance verification remains manual or is handled by the clinic's existing backend system. The agent either integrates with an existing backend or is removed from this workflow.

---

#### Assumption A3: Prior-Auth Rules Are (Or Can Be) Structured

**Assumption:** The clinic maintains or can provide an explicit, structured source of truth for which procedures require prior authorization (keyed by procedure code, payer, or other deterministic attributes).

**Hypothesis:** If prior-auth rules are rule-based and documented, the agent can determine prior-auth requirement deterministically without inferring from incomplete knowledge.

**How to test it:** Ask the clinic (likely practice manager or compliance) whether prior-auth logic is documented, centralized, or distributed across staff knowledge. If documented, request the rule set or decision tree. If distributed, ask how it's currently managed.

**Confidence:** **MEDIUM** — many small practices manage prior-auth as tribal knowledge or a spreadsheet. Some may have it in their EHR or use a payer portal. The level of structure is unknown.

**Impact if false:** Prior-auth checking becomes less reliable. The agent would escalate more cases or avoid this automation step. Design must shift to human-first approach with agent support for data gathering, not autonomy.

---

#### Assumption A4: Patient Digital Intake Is Feasible

**Assumption:** Patients can receive and complete pre-visit questionnaires through an approved channel (patient portal, email, SMS, IVR) without creating operational friction.

**Hypothesis:** If patients have access to a digital intake channel and the clinic has existing notification infrastructure, the agent can deliver questionnaires and track completion automatically.

**How to test it:** Confirm whether athenahealth (or another EHR) includes a patient portal. Ask whether the clinic currently sends pre-visit communications (reminders, forms, instructions). Estimate what percentage of patients would realistically complete a digital intake.

**Confidence:** **MEDIUM-HIGH** — athenahealth includes a patient portal, and many practices do send pre-visit instructions. However, adoption rates and fallback processes for non-respondents are unknown.

**Impact if false:** Questionnaire collection remains entirely manual or paper-based. The agent cannot automate this step reliably.

---

#### Assumption A5: Distinct Queues Exist for Escalations

**Assumption:** The clinic has (or can implement) operational queues for:
- `FRONT_DESK` — non-clinical administrative blockers (missing questionnaire, insurance inactive, expired prior auth)
- `CLINICAL_STAFF` — safety-sensitive reviews (medication changes, allergy flags, ambiguous visit reasons)
- `PRACTICE_MANAGER` — operational escalations (system outages, compliance issues)

**Hypothesis:** If these queues are operationally owned and monitored, escalations will actually be reviewed and resolved in time for the visit.

**How to test it:** Ask the practice manager whether these operational queues exist, who monitors them, and what the expected response times are.

**Confidence:** **MEDIUM** — many practices do not have formal escalation queues. This may need to be implemented as part of the solution.

**Impact if false:** Escalations may be created but not resolved, causing them to pile up silently. The system could appear to work but fail operationally.

---

#### Assumption A6: Unverified Patient-Reported Medication Data Can Be Staged

**Assumption:** The clinic's compliance model allows storing patient-reported medication changes in a temporary staging area (separate from the official medical record) before human clinical review.

**Hypothesis:** If temporary staging is allowed, the agent can collect patient-reported medication updates without triggering premature chart updates or compliance violations.

**How to test it:** Ask the clinic's compliance officer or EHR administrator whether unverified patient inputs can be stored outside the official EHR, how they must be labeled, and what retention rules apply.

**Confidence:** **LOW-MEDIUM** — HIPAA allows this, but clinic policy may be more restrictive. Unknown.

**Impact if false:** Medication change collection workflow shifts. Patient input may need to go directly to a human reviewer rather than being staged for comparison.

---

#### Assumption A7: Allergy Flags Are Retrievable in Structured Form

**Assumption:** athenahealth stores allergy information in structured format (active/inactive, severity optional) that can be detected automatically without reading unstructured clinical notes.

**Hypothesis:** If allergy data is structured, the agent can retrieve the presence of any allergy flag deterministically.

**How to test it:** Request athenahealth data dictionary or documentation. Confirm whether allergy records include structured fields (name, category, severity, date recorded).

**Confidence:** **HIGH** — most EHR systems do structure allergy flags. This is less risky than some other assumptions.

**Impact if false:** Allergy flag retrieval may fail or require manual chart review, limiting this automation step.

---

#### Assumption A8: Clinic-Specific Visit-Reason Routing Rules Can Be Defined

**Assumption:** The clinic can articulate a narrow set of administrative visit-reason categories (e.g., "follow-up for hypertension," "annual physical," "medication refill") and provide a list of urgent-trigger phrases that always require human escalation.

**Hypothesis:** If narrow admin categories + urgent-phrase list are defined, the agent can route routine cases without clinical interpretation while safely escalating ambiguous cases.

**How to test it:** Ask the clinical staff what visit reasons they see most often (routine categories). Ask what phrases in visit reasons would always concern them (urgent-trigger list).

**Confidence:** **MEDIUM** — depends on whether the clinic has thought about this explicitly. Many practices have not.

**Impact if false:** Visit-reason routing becomes unsafe or all-or-nothing (fully automated or fully manual).

---

#### Assumption A9: Operational Roles and Permissions Are Distinct

**Assumption:** The clinic's staff roles (front-desk, clinical staff, manager) have distinct system permissions that support role-based escalation routing and override tracking.

**Hypothesis:** If roles are distinct in the EHR or a supporting system, the agent can route escalations to the correct role and log overrides by user identity.

**How to test it:** Confirm the clinic's user management model in athenahealth. Identify which roles exist and what permissions each has.

**Confidence:** **MEDIUM-HIGH** — most EHRs support role-based access. However, this clinic may not be using role separation today.

**Impact if false:** Escalation routing becomes ambiguous. Override tracking cannot be attributed to specific users.

---

#### Assumption A10: Compliance Logging Requirements Are Definable

**Assumption:** The clinic has (or can articulate) specific compliance and audit-logging requirements for PHI access logs, workflow state changes, escalations, and overrides — including retention period, required fields, and storage location.

**Hypothesis:** If logging requirements are defined explicitly, the agent can implement logging that satisfies audit and compliance needs.

**How to test it:** Ask the clinic's compliance officer or privacy officer about HIPAA audit-log requirements. Request any documentation on data retention, workflow audit trails, or override logging policies.

**Confidence:** **MEDIUM** — most practices understand HIPAA logging at a high level but may not have specific policies written down.

**Impact if false:** The system may be operationally functional but non-compliant. This is a blocker for production.

---

### Unknowns That Must Be Resolved

| ID | Unknown | Why Critical | Validation Method |
|:--|:--|:--|:--|
| U1 | athenahealth API availability and contract terms in this clinic environment | Determines whether core EHR integration is feasible | Request integration documentation from clinic IT / athenahealth team |
| U2 | Insurance eligibility tool vendor, API contract, response format, timeout behavior | Determines feasibility of automated insurance verification | Ask clinic operations lead about eligibility tool; request integration specs |
| U3 | Where prior-auth rules are stored (centralized vs. tribal knowledge) and how structured | Determines whether prior-auth automation is safe | Ask practice manager or compliance lead for prior-auth documentation |
| U4 | Percentage of patients who will complete digital questionnaires; fallback process | Determines questionnaire automation effectiveness | Run pilot with 10-20 patients; measure completion rate |
| U5 | Exact HIPAA compliance requirements: log retention, audit-trail scope, override documentation | Determines compliance-safe logging design | Request compliance officer's written policy or HIPAA risk assessment |
| U6 | Current baseline metrics: intake completeness %, time per patient, rate of visit disruption due to intake | Determines whether success targets are realistic | Request data from clinic for past 30 days: missed prior auths, medication issues, questionnaire failures |
| U7 | Whether front-desk, clinical staff, and manager roles have distinct permissions today | Determines whether escalation routing can be role-based and auditable | Request access controls from IT / EHR admin |
| U8 | Existing IT infrastructure and vendor relationships (hosting, identity, auth) | Determines deployment and security model | Ask clinic CTO/IT lead about current infrastructure |

---

## 2. Problem Statement & Success Metrics

### Problem From User Perspective

**Front-desk team (4 people, ~180 patients/day):**
- Currently managing intake manually for each visit, including insurance verification, prior-auth status, questionnaire collection, medication reconciliation, allergy review
- Administrative workload is high relative to team size and patient volume
- Mistakes happen: expired prior authorizations are not caught before the visit, medication changes are not surfaced to the provider
- No clear way to hand off incomplete intake to clinical staff in time for the physician to review before the visit starts

### Problem From Business Perspective

**Practice manager's challenges:**
- Visit efficiency is impacted when physicians start appointments with incomplete administrative information
- Front-desk team is at capacity; hiring more staff is expensive and difficult
- Current manual process scales poorly to 180 visits/day and creates bottlenecks
- Operational mistakes (missed prior auths, medication misses) lead to visit disruptions, rework, and potential compliance gaps
- No clear audit trail of who did what in intake workflow

### Core Tensions

1. **Volume vs. Safety:** The practice needs to process ~180 intakes per day reliably, but cannot sacrifice safety by removing human oversight of clinical-adjacent tasks (medication changes, ambiguous visit reasons).

2. **Automation vs. Clinical Control:** Temptation exists to have the system "decide" if a medication change is important or if a visit reason sounds urgent. This must be resisted. The system must route, not judge.

3. **Operational Speed vs. Completeness:** Intake must be completed fast enough to inform the physician before the visit. If escalations are too slow or there is no operational owner, escalations pile up and become useless.

### Success Metrics (Proposed, Pending Stakeholder Validation)

| Metric | Target | Baseline to Validate | Why It Matters |
|:--|:--|:--|:--:|
| **Intake Completeness Before Visit** | ≥95% of scheduled visits begin with all required admin steps completed or correctly escalated | Current completeness rate (unknown) | Measures whether the system is making a real operational difference |
| **Reduction in Missed Prior-Auth and Medication-Change Defects** | ≥50% reduction from baseline | Current rate of missed prior auths / medication issues at visit start (unknown) | Directly addresses the two most common intake failures named in the scenario |
| **Front-Desk Administrative Workload** | 30–40% reduction in average intake handling time per patient | Current intake time baseline (unknown) | Measures whether the system is actually reducing front-desk burden |
| **Same-Day Visit Disruption Due to Intake Failures** | ≥40% reduction | Current disruption rate (unknown) | Measures business impact: how often do visits get delayed or reworked due to intake issues? |
| **Urgent/Ambiguous Visit Reasons Routed to Human Review** | 100% of cases that match urgent-trigger phrases are escalated | N/A | Ensures the delegation boundary is working: no false "safe" classification of potentially urgent cases |
| **Clinical Boundary Violations** | **0** — The system never diagnoses, triages, or judges medication significance | N/A | Non-negotiable safety requirement |
| **Insurance/Prior-Auth Status Accuracy** | ≥98% | Current administrative accuracy (if tracked) | Measures data quality of automated checks |
| **Audit Logging Coverage** | 100% of PHI accesses, state changes, escalations, and overrides are logged | N/A | Compliance and accountability requirement |

---

### Why These Metrics Matter

**Intake Completeness & Defect Reduction:** The scenario explicitly states that physicians regularly discover missed information (expired prior auths, unreviewed medication changes). These two metrics directly measure whether the system is fixing the core problem.

**Workload Reduction:** If the 4-person front-desk team is handling 180 intakes/day without automation, they're managing ~45 intakes per person per day. Even a 30% reduction (to ~31 per person) is meaningful. This metric measures whether the investment in the agent is justified.

**Safe Visit Disruption:** If visits still get disrupted due to intake issues, the system hasn't solved the problem operationally.

**Clinical Boundary:** Zero violations is non-negotiable. Any false classification, any inferred diagnosis or medication judgment, is a failure.

**Audit Logging:** HIPAA audit-trail requirements are non-negotiable. 100% coverage or the system is non-compliant.

---

### Hard Constraints (Non-Negotiable)

1. **No Clinical Judgment:** The agent must never diagnose, triage medically, recommend treatment, judge medication importance, assess allergy severity, or infer clinical significance.
2. **No Suppressed Escalation:** If a visit reason, medication change, or allergy flag reaches an escalation trigger, the system must escalate. It must not silence escalations to appear "efficient."
3. **HIPAA Compliant:** All PHI access and workflow actions must be logged. Logs must be retained per clinic policy and be immutable.
4. **Complete Audit Trail:** Every state change, escalation, and override must be logged with actor identity and timestamp.
5. **Human Ownership of Safety-Sensitive Decisions:** Medication changes, allergy flags, urgent-looking visit reasons, and any exception state must require human review before the case is marked ready.

---

## 3. Delegation Analysis

### Delegation Modes

The spec uses exactly three delegation modes:

- **`AGENT_ALONE`** — The agent executes autonomously; no human review required unless an error occurs.
- **`AGENT_PLUS_HUMAN_REVIEW`** — The agent performs the mechanical portion (detection, routing, flagging); human must review and resolve before the case is marked ready.
- **`HUMAN_DECIDES`** — The agent provides data and context; the human makes the final decision.

---

### Delegation Matrix

| **Task** | **Mode** | **Rule** | **Justification** |
|:--|:--|:--|:--|
| **Appointment Sync** | AGENT_ALONE | Query athenahealth for scheduled appointments in next 2 calendar days every 15 minutes. | Fully deterministic; no clinical content; structured data only. Failure is handled through retry and escalation, not human review. |
| **Create Intake Work Item** | AGENT_ALONE | One work item per appointment; idempotent (no duplicates). | Operational bookkeeping. No judgment required. Duplicates are a bug, not a safety issue if handled via idempotency. |
| **Dispatch Questionnaire** | AGENT_ALONE | Send automatically when intake item is created, unless a valid completed questionnaire already exists. | Fully deterministic outreach. Patients can ignore or complete. Response tracking is administrative. |
| **Check Questionnaire Completion** | AGENT_ALONE | Evaluate required fields against received form. If all required fields present = complete; otherwise incomplete. | Form validation is mechanical. No clinical interpretation. |
| **Verify Insurance Eligibility** | AGENT_ALONE | Call eligibility integration; record returned status (active/inactive/error/timeout) exactly as received. | Mechanical integration call; response is structured. The agent is not deciding coverage; it's recording what the payer says. |
| **Record Insurance Outcome** | AGENT_ALONE | Persist status and timestamp. If status ≠ active, create escalation to FRONT_DESK. | Status recording is mechanical. Escalation trigger is deterministic (status ≠ active). |
| **Check Prior-Auth Requirement** | AGENT_PLUS_HUMAN_REVIEW | Query clinic-maintained explicit rule set. If procedure code matches rule → determine required/not required. If no rule match → set UNDETERMINED and escalate. Never infer. | Prior-auth rules are clinic-specific business logic. Safe only if rules are explicit. If no rule exists, human must decide. Inference would be unsafe. |
| **Record Prior-Auth Status** | AGENT_PLUS_HUMAN_REVIEW | Record NOT_REQUIRED / VALID / EXPIRED / MISSING / UNDETERMINED. If status ≠ VALID and ≠ NOT_REQUIRED, create escalation to FRONT_DESK. | Status recording is mechanical; escalation trigger is deterministic. But prior-auth errors directly impact visit readiness and must be reviewable. |
| **Collect Medication Changes** | AGENT_ALONE | Ask patient: "Have your medications changed since [last visit date]?" Store response. | Patient questionnaire is administrative data collection. No clinical interpretation. |
| **Compare Medications (Patient vs. EHR)** | AGENT_ALONE | Retrieve EHR med list. Compare patient-reported changes against it on fields: medication name, dose, frequency. Flag any difference. | Mechanical field-by-field comparison. The system detects difference; it does not interpret meaning. |
| **Escalate Medication Changes** | AGENT_PLUS_HUMAN_REVIEW | Any change reported OR any mismatch detected → create escalation to CLINICAL_STAFF with details. Do not decide whether change is significant. | Medication safety is clinical. The system detects change; a clinician determines meaning. |
| **Retrieve Allergy Flags** | AGENT_ALONE | Query EHR allergy records. Detect presence or absence of any active flag. | Structured EHR retrieval. Presence/absence is deterministic. |
| **Escalate Allergies** | AGENT_PLUS_HUMAN_REVIEW | If any allergy flag exists → create escalation to CLINICAL_STAFF. Do not interpret severity or significance. | Allergy significance is clinical. The system detects presence; a clinician assesses importance. |
| **Route Visit Reason (Routine)** | AGENT_PLUS_HUMAN_REVIEW | Match visit reason text against approved narrow admin categories (e.g., "follow-up for hypertension," "annual physical"). If match and no urgent-trigger phrases → mark ROUTINE_ROUTED (no escalation required). | Narrow routing based on explicit category list is safe. Free-form clinical interpretation is not. |
| **Escalate Visit Reason (Ambiguous/Urgent)** | AGENT_PLUS_HUMAN_REVIEW | If visit reason is blank, ambiguous, unparseable, or contains any phrase from clinic's urgent-trigger list → create escalation to CLINICAL_STAFF. Do not classify urgency. | Ambiguous or urgent-looking text must reach human. Automatic urgency scoring by the agent would be clinical judgment (prohibited). |
| **Decide Medical Urgency** | HUMAN_DECIDES | Never implement. Provide escalated case with data; clinician reviews. | Urgency determination is clinical diagnosis (prohibited by Scenario 5). |
| **Decide Medication Importance** | HUMAN_DECIDES | Never implement. Provide flagged changes to clinical staff; they assess. | Medication significance is clinical judgment (prohibited). |
| **Decide Allergy Significance** | HUMAN_DECIDES | Never implement. Provide flagged allergy to clinical staff; they assess. | Allergy assessment is clinical judgment (prohibited). |
| **Override Exception State** | HUMAN_DECIDES | Allow authorized staff to override hold/escalation states. Require reason and user ID logging. | Overrides are operational decisions with safety implications. Must remain human-owned and auditable. |
| **Mark Intake Ready for Visit** | AGENT_PLUS_HUMAN_REVIEW | Can only mark READY_FOR_VISIT if: insurance active, questionnaire complete, visit reason routed safely (routine or human-reviewed), no medication escalations open, no allergy escalations open, no integration failures. If any condition false → HOLD_FOR_REVIEW. | Readiness is the gate. An intake with unresolved escalations is not ready. This is deterministic logic but enforces the safety boundary. |

---

### Queue Assignment Rules

When escalations are created, assign them to operational queues:

| **Escalation Trigger** | **Queue** | **Why** |
|:--|:--|:--|
| Questionnaire incomplete | FRONT_DESK | Administrative data gathering; front-desk owns follow-up |
| Questionnaire not received by deadline | FRONT_DESK | Administrative tracking issue |
| Insurance inactive / error / timeout | FRONT_DESK | Administrative integration issue; front-desk has relationships with payers |
| Prior-auth missing / expired / undetermined | FRONT_DESK | Administrative prior-auth tracking; front-desk typically handles payer communications |
| Medication change reported | CLINICAL_STAFF | Medication safety decision; clinical staff must review |
| Medication mismatch detected | CLINICAL_STAFF | Medication reconciliation is clinical |
| Allergy flag present | CLINICAL_STAFF | Allergy assessment is clinical |
| Visit reason urgent-trigger phrase / blank / ambiguous | CLINICAL_STAFF | Urgent or safety-sensitive routing; clinical staff must triage |
| System outage affecting multiple work items | PRACTICE_MANAGER | Operational/infrastructure issue requiring escalation |

---

### Why This Delegation Boundary Is Correct

**AGENT_ALONE tasks share these properties:**
- Structured inputs
- Deterministic decision rules
- No clinical judgment required
- Mechanical or administrative
- Failures are handled through escalation or retry, not by suppression

**AGENT_PLUS_HUMAN_REVIEW tasks share these properties:**
- Require deterministic initial action (detection, routing, flagging)
- But require human judgment to resolve or confirm before readiness
- Escalation is built into the workflow
- Examples: flagging medication changes (agent detects, human assesses), routing ambiguous visit reasons (agent detects ambiguity, human routes)

**HUMAN_DECIDES tasks:**
- Require clinical judgment, subjective assessment, or safety-critical decision-making
- No autonomous logic is safe
- Examples: determining medical urgency, medication significance, allergy severity, override decisions

**The key principle:** The agent maximizes automation for **administrative repetitive work** and escalates **immediately and loudly** when ambiguity, safety risk, or clinical judgment is needed.

---

## 4. Agent Specification (Capability-Driven)

### Capability Name

**Patient Intake Coordination Agent**

---

### Purpose

Automate non-clinical administrative intake workflow for scheduled primary-care visits, reducing front-desk burden and improving intake completeness by processing structured administrative tasks (questionnaire dispatch, insurance verification, prior-auth checking, medication reconciliation, allergy review, visit routing) at scale (~180 patients/day).

---

### Scope

**In Scope:**
- Appointment sync and intake initialization
- Pre-visit questionnaire dispatch and completion tracking
- Insurance eligibility verification
- Prior-auth requirement determination (rule-based only)
- Patient-reported medication change collection and EHR comparison
- Allergy flag retrieval from EHR
- Visit-reason classification using narrow administrative rules
- Escalation creation and queue routing
- Intake readiness evaluation
- Audit logging

**Out of Scope:**
- Diagnosis
- Clinical triage or urgency determination
- Treatment recommendations
- Medication safety assessment
- Allergy severity assessment
- Autonomous visit rescheduling or cancellation
- Any decision requiring clinical judgment

---

### Inputs (Data Required)

**From athenahealth (EHR):**
- scheduled appointment records (appointment_id, patient_id, provider_id, location_id, scheduled_start_time, visit_type, procedure_code if present)
- patient current medication list
- patient allergy flag records

**From Insurance Eligibility Tool:**
- eligibility check response (active / inactive / error / timeout)

**From Prior-Auth Rule Source:**
- structured prior-auth requirement rules (keyed by procedure_code, payer, or other deterministic attributes)
- prior-auth records if they exist

**From Patient (Questionnaire / Intake Form):**
- pre-visit questionnaire responses (all required fields present/absent)
- patient-reported medication changes
- visit reason (free text or selected category)

---

### Outputs (Data Produced)

**IntakeWorkItem entity:**
- status: CREATED / PROCESSING / READY_FOR_VISIT / HOLD_FOR_REVIEW
- questionnaire_status: PENDING_SEND / SENT / COMPLETE / INCOMPLETE / DEADLINE_MISSED
- insurance_status: PENDING / VERIFIED_ACTIVE / VERIFIED_INACTIVE / ERROR / TIMEOUT
- prior_auth_status: NOT_REQUIRED / VALID / EXPIRED / MISSING / UNDETERMINED / N_A
- medication_status: NO_CHANGES_REPORTED / CHANGES_REPORTED / MISMATCH_DETECTED / N_A
- allergy_status: NO_FLAGS / FLAGS_PRESENT
- visit_reason_status: ROUTINE_ROUTED / ESCALATED_URGENT / ESCALATED_AMBIGUOUS / ESCALATED_BLANK
- completion_decision: READY_FOR_VISIT / HOLD_FOR_REVIEW
- assigned_queue: FRONT_DESK / CLINICAL_STAFF / PRACTICE_MANAGER (if escalated)

**EscalationEvent entity:**
- trigger_code: specific reason (e.g., INSURANCE_INACTIVE, MEDICATION_CHANGE_REPORTED, URGENT_PHRASE_DETECTED)
- queue: FRONT_DESK / CLINICAL_STAFF / PRACTICE_MANAGER
- status: CREATED / ACKNOWLEDGED / IN_PROGRESS / RESOLVED / OVERRIDDEN

**AuditLog entry (one per significant action):**
- entity_type: IntakeWorkItem / EscalationEvent / PHI_ACCESS
- entity_id: the work item or log ID
- action_type: APPOINTMENT_SYNCED / QUESTIONNAIRE_SENT / INSURANCE_CHECKED / ESCALATION_CREATED / STATE_CHANGED / PHI_READ / OVERRIDE_APPLIED
- actor_type: AGENT / FRONT_DESK_USER / CLINICAL_USER / MANAGER_USER
- actor_id: user ID or SYSTEM
- timestamp: ISO 8601, UTC
- details_json: structured context (e.g., `{integration: "athenahealth", field: "medication_list", patient_id: "...", status: "success"}`)

---

### Decision Logic (Core Workflow)

**FR1. Appointment Sync and Work-Item Creation**

```
Every 15 minutes:
1. Query athenahealth for appointments scheduled in next 2 calendar days with status=SCHEDULED
2. For each appointment:
   a. Check if intake work item already exists for this appointment_id
   b. If exists and status=READY_FOR_VISIT or COMPLETED: skip
   c. If does not exist: 
      - Create new IntakeWorkItem(appointment_id, patient_id, ...)
      - Set status = CREATED
      - Log: APPOINTMENT_SYNCED
   d. If exists and status ≠ READY_FOR_VISIT and ≠ COMPLETED and < 4 hours until visit:
      - Continue processing (see FR2–FR7)
   e. If exists and status ≠ READY_FOR_VISIT and visit time has passed:
      - Set status = ABANDONED (visit start reached without completion)
      - Do not create new work items for past appointments
3. If athenahealth call fails:
   - Log failure with error_code and timestamp
   - Retry with exponential backoff (2s, 4s, 8s) up to 3 times
   - If all retries fail, alert PRACTICE_MANAGER and do not create work items
```

**Acceptance Criteria:**
- Exactly one active work item exists per appointment
- Duplicate creation is forbidden
- No work items created for cancelled appointments
- Idempotent: re-running the sync with same data produces no duplicates

---

**FR2. Pre-Visit Questionnaire Dispatch and Tracking**

```
When: a new IntakeWorkItem is created, or every 2 hours if questionnaire_status = PENDING_SEND

1. Check if a completed valid questionnaire exists for this patient/appointment
   a. If yes: set questionnaire_status = COMPLETE, skip to FR3
   
2. If questionnaire_status = PENDING_SEND or not yet set:
   a. Send questionnaire to patient via approved channel (patient portal, email, SMS per clinic config)
   b. Log: QUESTIONNAIRE_SENT (include channel, patient_contact_method)
   c. Set questionnaire_status = SENT
   d. Record sent_timestamp

3. If questionnaire_status = SENT:
   a. Check if response received
   b. If response received:
      - Validate that all required fields are present (no blanks, null, or empty strings in required fields)
      - If all required fields present: set questionnaire_status = COMPLETE
      - If any required field missing: set questionnaire_status = INCOMPLETE and escalate (trigger: QUESTIONNAIRE_INCOMPLETE)
   c. If no response received AND scheduled_start_time - now < 12 hours:
      - Set questionnaire_status = DEADLINE_MISSED
      - Create escalation (trigger: QUESTIONNAIRE_NOT_RECEIVED) to FRONT_DESK
      - Note: front-desk can still manually collect or proceed with clinical staff review

4. Log: QUESTIONNAIRE_STATUS_CHANGED (from_status, to_status, reason)
```

**Acceptance Criteria:**
- Questionnaire sent to all patients for whom no valid completed form exists
- Missing required fields trigger escalation (not silent failure)
- Deadline escalation occurs exactly 12 hours before visit
- Completed questionnaires can be accepted from any approved channel

---

**FR3. Insurance Eligibility Verification**

```
When: IntakeWorkItem status = CREATED, and insurance_status not yet set

1. Call insurance eligibility integration:
   a. Input: appointment.patient_id, appointment.scheduled_start_date, appointment.procedure_code (if exists)
   b. Timeout: 10 seconds
   c. Retry logic: On timeout or HTTP 5xx, retry up to 2 times with backoff (2s, 4s)
   d. Call with auth credentials from clinic config (env var, secrets manager, per integration spec)

2. Handle response:
   a. If response = {status: "active"}: 
      - Set insurance_status = VERIFIED_ACTIVE
      - Log: INSURANCE_VERIFIED (status=ACTIVE, payer, timestamp)
      - Continue to FR4 (no escalation)
   
   b. If response = {status: "inactive"}:
      - Set insurance_status = VERIFIED_INACTIVE
      - Log: INSURANCE_VERIFIED (status=INACTIVE, payer, timestamp)
      - Create escalation (trigger: INSURANCE_INACTIVE) to FRONT_DESK
   
   c. If response = {status: "error"} or HTTP 4xx:
      - Set insurance_status = ERROR
      - Log: INSURANCE_FAILED (error_code, error_message, timestamp)
      - Create escalation (trigger: INSURANCE_ERROR) to FRONT_DESK
   
   d. If timeout or HTTP 5xx:
      - Set insurance_status = TIMEOUT
      - Log: INSURANCE_FAILED (error_code="TIMEOUT", timestamp)
      - Create escalation (trigger: INSURANCE_TIMEOUT) to FRONT_DESK

3. All outcomes are logged; no silent failures
4. Inactive/error/timeout outcomes block readiness (see FR8)
```

**Acceptance Criteria:**
- Integration calls succeed for active coverage; escalations are created for inactive/error/timeout
- No guessed insurance status
- Failures are properly logged and escalated
- Timeout does not exceed 10 seconds per call

---

**FR4. Prior-Auth Determination**

```
When: IntakeWorkItem status = CREATED and prior_auth_status not yet set

1. Check if this visit includes a procedure code
   a. If visit_type ≠ procedure or procedure_code not present:
      - Set prior_auth_status = NOT_REQUIRED
      - Skip FR4 (no escalation)
   
   b. If procedure_code present, query prior-auth rule source:

2. Look up prior-auth requirement rule:
   a. Query clinic's rule set with inputs: procedure_code, payer (if available), location, provider specialty
   b. If explicit rule found:
      - If rule says "REQUIRED": set prior_auth_status = NOT_REQUIRED (no prior auth needed)
      - If rule says "OPTIONAL": set prior_auth_status = NOT_REQUIRED
      - If rule says "REQUIRED": proceed to 3
   c. If no explicit rule found:
      - Set prior_auth_status = UNDETERMINED
      - Create escalation (trigger: PRIOR_AUTH_UNDETERMINED) to FRONT_DESK
      - Log: PRIOR_AUTH_RULE_NOT_FOUND (procedure_code, payer, timestamp)
      - Stop (do not infer)

3. If rule says "REQUIRED":
   a. Query prior-auth records for this procedure/payer:
      - Check if a valid prior-auth exists (auth_date <= today, expiration_date >= today)
   b. If valid prior-auth found:
      - Set prior_auth_status = VALID
      - Log: PRIOR_AUTH_VERIFIED (auth_id, expiration_date, timestamp)
      - No escalation
   c. If prior-auth expired (expiration_date < today):
      - Set prior_auth_status = EXPIRED
      - Create escalation (trigger: PRIOR_AUTH_EXPIRED) to FRONT_DESK
      - Log: PRIOR_AUTH_EXPIRED (auth_id, expiration_date, timestamp)
   d. If no prior-auth record exists:
      - Set prior_auth_status = MISSING
      - Create escalation (trigger: PRIOR_AUTH_MISSING) to FRONT_DESK
      - Log: PRIOR_AUTH_MISSING (procedure_code, timestamp)

4. All outcomes logged
```

**Acceptance Criteria:**
- Prior-auth requirement is always determined by explicit rule, never inferred
- Missing rule → UNDETERMINED + escalation (not guessed)
- Expired or missing prior-auth creates escalation
- Valid prior-auth allows workflow to proceed
- No clinical judgment; only rule-matching

---

**FR5. Medication Change Collection and Comparison**

```
When: IntakeWorkItem questionnaire_status = COMPLETE

1. Extract patient responses from questionnaire:
   a. Question: "Have your medications changed since [last visit]?"
   b. If patient answer = "No" or blank:
      - Set medication_status = NO_CHANGES_REPORTED
      - Skip FR5 escalation
   
   c. If patient answer = "Yes":
      - Set medication_status = CHANGES_REPORTED
      - Log: MEDICATION_CHANGE_REPORTED (patient_id, timestamp)
      - Create escalation (trigger: MEDICATION_CHANGE_REPORTED) to CLINICAL_STAFF
      - Include patient narrative in escalation context
      - Do NOT attempt to score significance; escalate immediately

2. If patient reported changes, retrieve EHR medication list:
   a. Query athenahealth for patient's current medication list (as of today)
   b. If query fails:
      - Set medication_status = ERROR
      - Log: MEDICATION_LIST_FAILED (error_code, timestamp)
      - Escalation remains (conservatively)
   
   c. If query succeeds:
      - Compare patient-reported changes to EHR list on fields: medication_name, dose, frequency
      - If any mismatch found:
        * Set medication_status = MISMATCH_DETECTED
        * Log: MEDICATION_MISMATCH (differences: [{field, ehr_value, patient_value}], timestamp)
        * Create/update escalation (trigger: MEDICATION_MISMATCH) to CLINICAL_STAFF if not already escalated
      - If no mismatch but patient reported changes:
        * Escalation remains (triggered by MEDICATION_CHANGE_REPORTED in step 1c)

3. Log: MEDICATION_STATUS_CHANGED (from_status, to_status, reason)
4. Do not attempt to assess whether change is "important" or "minor" — that's clinical judgment
```

**Acceptance Criteria:**
- Any reported medication change → escalation (no filtering)
- Mismatch detection is field-by-field comparison only
- No clinical significance assessment
- EHR retrieval failure does not suppress escalation
- Escalation includes full context for clinical review

---

**FR6. Allergy Flag Retrieval**

```
When: IntakeWorkItem status = CREATED

1. Query EHR for patient allergy records:
   a. Input: patient_id
   b. Timeout: 5 seconds
   c. Retrieve all active (non-deleted) allergy flag records

2. Evaluate presence:
   a. If any active allergy flag found:
      - Set allergy_status = FLAGS_PRESENT
      - Create escalation (trigger: ALLERGY_FLAG_PRESENT) to CLINICAL_STAFF
      - Log: ALLERGY_RETRIEVED (count, flag_names [without clinical interpretation], timestamp)
      - Include full allergy record context in escalation
   
   b. If no allergy flags found:
      - Set allergy_status = NO_FLAGS
      - No escalation
      - Log: ALLERGY_RETRIEVED (count=0, timestamp)

3. If query fails:
   a. Set allergy_status = ERROR
   b. Log: ALLERGY_FAILED (error_code, timestamp)
   c. Escalate conservatively (trigger: ALLERGY_RETRIEVAL_ERROR) to CLINICAL_STAFF
   d. Do not assume "no flags" on retrieval failure

4. Do NOT attempt to:
   - Interpret severity
   - Assess clinical impact
   - Decide whether allergy should change care plan
   - Filter or suppress allergy flags
```

**Acceptance Criteria:**
- All allergy flags are detected if retrieval succeeds
- Retrieval failure triggers escalation (not silent failure)
- No clinical interpretation of flags
- Full allergy record is included in escalation context

---

**FR7. Visit Reason Routing**

```
When: IntakeWorkItem questionnaire_status = COMPLETE

1. Extract visit reason from questionnaire:
   a. Input: patient's free-text or selected visit reason

2. Check against clinic-defined urgent-trigger phrase list:
   a. Load clinic's urgent-trigger phrases (configurable list, e.g., ["chest pain", "shortness of breath", "severe", ...])
   b. If any phrase from list is found in visit reason (case-insensitive):
      - Set visit_reason_status = ESCALATED_URGENT
      - Create escalation (trigger: URGENT_PHRASE_DETECTED) to CLINICAL_STAFF
      - Log: VISIT_REASON_URGENT_PHRASE (phrase_matched, full_reason, timestamp)
      - Stop (do not continue to step 3)

3. Check if visit reason is blank or ambiguous:
   a. If visit reason is blank or all whitespace:
      - Set visit_reason_status = ESCALATED_BLANK
      - Create escalation (trigger: VISIT_REASON_BLANK) to CLINICAL_STAFF
      - Log: VISIT_REASON_BLANK (timestamp)
      - Stop
   
   b. If visit reason cannot be parsed (e.g., unstructured text that doesn't match any category):
      - Set visit_reason_status = ESCALATED_AMBIGUOUS
      - Create escalation (trigger: VISIT_REASON_AMBIGUOUS) to CLINICAL_STAFF
      - Log: VISIT_REASON_AMBIGUOUS (reason_text, timestamp)
      - Stop

4. If visit reason is clear and matches a clinic-defined routine admin category:
   a. Load clinic's approved routine categories (e.g., ["follow-up for hypertension", "annual physical", "medication refill", ...])
   b. If visit reason matches a category and no urgent phrases present:
      - Set visit_reason_status = ROUTINE_ROUTED
      - No escalation from this check
      - Log: VISIT_REASON_ROUTED (category, timestamp)

5. Do NOT:
   - Attempt to score medical urgency
   - Classify symptoms
   - Decide whether the visit "really is" urgent
   - Suppress escalations to appear efficient
```

**Acceptance Criteria:**
- All urgent-trigger phrases trigger escalation
- Blank/ambiguous visit reasons trigger escalation
- Routine categories are matched only against clinic-provided list
- No clinical interpretation of reason text
- Escalation includes full context for clinical review

---

**FR8. Readiness Evaluation and Completion Decision**

```
When: All prior FRs (1–7) have completed OR deadline (4 hours before visit start) is reached

1. Evaluate all completion conditions:

   a. Questionnaire: questionnaire_status = COMPLETE
      - If false: HOLD_FOR_REVIEW (missing intake data)
   
   b. Insurance: insurance_status = VERIFIED_ACTIVE
      - If false: HOLD_FOR_REVIEW (insurance issue)
   
   c. Prior-Auth (if applicable): prior_auth_status = VALID or NOT_REQUIRED
      - If prior_auth_status = EXPIRED, MISSING, or UNDETERMINED: HOLD_FOR_REVIEW
      - If prior_auth_status = N_A (not applicable): pass this check
   
   d. Medication: medication_status ≠ ESCALATED (i.e., no open medication escalation)
      - Check: Is there an open escalation with trigger MEDICATION_CHANGE_REPORTED or MEDICATION_MISMATCH that has not been resolved?
      - If yes: HOLD_FOR_REVIEW
      - If no: pass this check
   
   e. Allergy: allergy_status ≠ ESCALATED (i.e., no open allergy escalation)
      - Check: Is there an open escalation with trigger ALLERGY_FLAG_PRESENT that has not been resolved?
      - If yes: HOLD_FOR_REVIEW
      - If no: pass this check
   
   f. Visit Reason: visit_reason_status = ROUTINE_ROUTED or (ESCALATED_* with resolved escalation)
      - If visit_reason_status = ESCALATED_URGENT, ESCALATED_BLANK, or ESCALATED_AMBIGUOUS and escalation ≠ RESOLVED:
        - HOLD_FOR_REVIEW
      - If visit_reason_status = ROUTINE_ROUTED: pass this check
      - If escalation has been resolved (status = RESOLVED or OVERRIDDEN): pass this check
   
   g. Integration Failures: no blocking integration failures since work item creation
      - Check: Is there a recent failed integration call that would prevent accurate intake?
      - If athenahealth call failed: HOLD_FOR_REVIEW
      - If insurance integration failed (and status not manually overridden): HOLD_FOR_REVIEW
      - Otherwise: pass this check

2. Final Decision:
   a. If ANY condition a–g is false:
      - Set completion_decision = HOLD_FOR_REVIEW
      - Log: READINESS_BLOCKED (reason, timestamp)
      - Set status = HOLD_FOR_REVIEW
      - The case is not ready; clinical staff must manually review before visit
   
   b. If ALL conditions a–g are true:
      - Set completion_decision = READY_FOR_VISIT
      - Log: READINESS_APPROVED (all_conditions_met, timestamp)
      - Set status = READY_FOR_VISIT
      - The case is ready; intake is complete

3. The readiness decision is made automatically based only on the above conditions.
   - No subjective assessment
   - No human override of readiness logic (but humans can override individual escalations and then readiness re-evaluates)
```

**Acceptance Criteria:**
- A case marked READY_FOR_VISIT has no open unresolved escalations
- A case with any open escalation is marked HOLD_FOR_REVIEW
- Readiness logic is deterministic and reproducible
- No silent exceptions (all failures are escalated)

---

**FR9. Escalation Creation and Queue Assignment**

```
When: Any escalation trigger is activated (FR2–FR7)

1. Create EscalationEvent entity:
   a. escalation_id: UUID
   b. intake_work_item_id: reference to work item
   c. trigger_code: specific trigger (QUESTIONNAIRE_INCOMPLETE, INSURANCE_INACTIVE, ..., URGENT_PHRASE_DETECTED, etc.)
   d. queue: FRONT_DESK, CLINICAL_STAFF, or PRACTICE_MANAGER (determined by queue assignment rule)
   e. status: CREATED
   f. created_at: timestamp
   g. created_by: AGENT
   h. context: full structured details relevant to escalation

2. Log:
   a. ESCALATION_CREATED (escalation_id, trigger_code, queue, context, timestamp)

3. Notify owning queue:
   a. Send notification to the appropriate queue (FRONT_DESK, CLINICAL_STAFF, or PRACTICE_MANAGER)
   b. Notification method: per clinic config (email, dashboard flag, in-app notification, etc.)
   c. Notification must include work item ID and escalation reason

4. Human review and resolution:
   a. Owning queue user receives escalation
   b. User reviews context and makes decision
   c. User can:
      - Resolve escalation (status = RESOLVED, add resolution_note)
      - Override escalation (status = OVERRIDDEN, add override_reason with user_id logging)
      - Escalate further (status = ESCALATED_FURTHER, assign to higher queue)
   d. When escalation is resolved or overridden, readiness logic (FR8) re-evaluates
   e. If all escalations are resolved, case may transition to READY_FOR_VISIT

5. Escalations are immutable once created:
   - Escalation record cannot be deleted
   - Resolution/override only changes status, not the escalation record itself
   - Full history is preserved in audit log
```

**Acceptance Criteria:**
- Every escalation is logged
- Escalation queue is assigned per rule
- Escalations are routable and retrievable by queue
- Resolution or override updates status without deleting record
- Audit trail of all escalation lifecycle events is complete

---

**FR10. Audit Logging (All Actions)**

```
For every action, create immutable AuditLog entry:

Log When:
1. Appointment sync completes (success or failure)
2. Intake work item is created
3. Questionnaire is sent
4. Questionnaire response is received
5. Questionnaire status changes
6. Insurance verification is called (including timeout/error)
7. Insurance status changes
8. Prior-auth rule is queried
9. Prior-auth status is determined
10. EHR medications are retrieved
11. EHR allergies are retrieved
12. Medication mismatch is detected
13. Escalation is created
14. Escalation status changes (resolved, overridden)
15. Readiness decision is made
16. Work item status transitions
17. Any human override (with user_id)

Log Schema:
```json
{
  "audit_log_id": "UUID",
  "timestamp": "ISO 8601 UTC",
  "entity_type": "IntakeWorkItem | EscalationEvent | PHI_ACCESS",
  "entity_id": "UUID of work item or escalation",
  "action_type": "one of the action types above",
  "actor_type": "AGENT | FRONT_DESK_USER | CLINICAL_USER | MANAGER_USER",
  "actor_id": "SYSTEM (if agent) or user_id (if human)",
  "details": {
    "patient_id": "UUID",
    "appointment_id": "UUID",
    "field_name": "if applicable",
    "old_value": "previous state",
    "new_value": "new state",
    "reason": "cause or justification",
    "error_code": "if applicable",
    "integration_name": "if integration-related"
  },
  "immutable": true
}
```

Logging Requirements:
1. All logs are immutable once written (append-only)
2. Logs are retained for [clinic-defined retention period, default 7 years for financial/compliance]
3. Logs can be queried by date range, patient_id, actor_id, action_type
4. Logs include all PHI accesses (athenahealth queries, insurance checks)
5. Logs must be accessible for audit purposes (separate audit repository preferred)
```

**Acceptance Criteria:**
- All significant actions create audit logs
- Logs are immutable
- PHI accesses are logged
- Logs include actor identity (SYSTEM or user_id)
- Logs retain full context for compliance investigation

---

### Required Entities (Data Model)

```
Entity: Appointment (read-only from athenahealth)
- id: UUID (primary key, athenahealth appointment_id)
- patient_id: UUID
- provider_id: UUID
- location_id: UUID
- scheduled_start_at: ISO 8601 timestamp
- scheduled_end_at: ISO 8601 timestamp
- visit_type: enum [ROUTINE_VISIT, PROCEDURE_VISIT, EMERGENCY, URGENT_CARE, ...]
- procedure_code: string optional (CPT code if procedure visit)
- status: enum [SCHEDULED, CANCELLED, COMPLETED, NO_SHOW]
- created_at: ISO 8601 timestamp
- updated_at: ISO 8601 timestamp

Entity: IntakeWorkItem (state machine)
- id: UUID (primary key, generated at creation)
- appointment_id: UUID (foreign key, unique per appointment)
- patient_id: UUID
- status: enum [CREATED, PROCESSING, READY_FOR_VISIT, HOLD_FOR_REVIEW, ABANDONED]
  - State transitions: CREATED → PROCESSING (implied by ongoing FR execution) → (READY_FOR_VISIT or HOLD_FOR_REVIEW) or → ABANDONED (if past visit time)
- questionnaire_status: enum [PENDING_SEND, SENT, COMPLETE, INCOMPLETE, DEADLINE_MISSED, ERROR]
- insurance_status: enum [PENDING, VERIFIED_ACTIVE, VERIFIED_INACTIVE, ERROR, TIMEOUT]
- prior_auth_status: enum [NOT_REQUIRED, VALID, EXPIRED, MISSING, UNDETERMINED, N_A, ERROR]
- medication_status: enum [NO_CHANGES_REPORTED, CHANGES_REPORTED, MISMATCH_DETECTED, ERROR, N_A]
- allergy_status: enum [NO_FLAGS, FLAGS_PRESENT, ERROR]
- visit_reason_status: enum [ROUTINE_ROUTED, ESCALATED_URGENT, ESCALATED_AMBIGUOUS, ESCALATED_BLANK, PENDING, ERROR]
- completion_decision: enum [READY_FOR_VISIT, HOLD_FOR_REVIEW, PENDING]
- assigned_queue: nullable enum [FRONT_DESK, CLINICAL_STAFF, PRACTICE_MANAGER] (set if escalation exists)
- created_at: ISO 8601 timestamp (immutable)
- updated_at: ISO 8601 timestamp (updated on any state change)
- escalations: array of EscalationEvent (one-to-many)

Entity: EscalationEvent
- id: UUID (primary key)
- intake_work_item_id: UUID (foreign key)
- trigger_code: string enum [QUESTIONNAIRE_INCOMPLETE, INSURANCE_INACTIVE, INSURANCE_ERROR, INSURANCE_TIMEOUT, PRIOR_AUTH_MISSING, PRIOR_AUTH_EXPIRED, PRIOR_AUTH_UNDETERMINED, MEDICATION_CHANGE_REPORTED, MEDICATION_MISMATCH, ALLERGY_FLAG_PRESENT, URGENT_PHRASE_DETECTED, VISIT_REASON_AMBIGUOUS, VISIT_REASON_BLANK, INTEGRATION_FAILURE, PRIOR_AUTH_RULE_NOT_FOUND, QUESTIONNAIRE_NOT_RECEIVED, ALLERGY_RETRIEVAL_ERROR]
- queue: enum [FRONT_DESK, CLINICAL_STAFF, PRACTICE_MANAGER]
- status: enum [CREATED, ACKNOWLEDGED, IN_PROGRESS, RESOLVED, OVERRIDDEN, ESCALATED_FURTHER]
- created_at: ISO 8601 timestamp (immutable)
- created_by: string = "SYSTEM"
- resolved_at: nullable ISO 8601 timestamp
- resolved_by: nullable string (user_id if resolved by human)
- resolution_note: nullable string (human explanation of resolution)
- override_reason: nullable string (required if status = OVERRIDDEN)
- context: object (full details needed for human review, e.g., {insurance_response, medication_mismatch_details, allergy_records, visit_reason_text})

Entity: AuditLog (immutable append-only)
- id: UUID (primary key)
- timestamp: ISO 8601 UTC (immutable)
- entity_type: enum [IntakeWorkItem, EscalationEvent, PHI_ACCESS, INTEGRATION_CALL]
- entity_id: UUID
- action_type: string enum [APPOINTMENT_SYNCED, WORK_ITEM_CREATED, QUESTIONNAIRE_SENT, QUESTIONNAIRE_RECEIVED, INSURANCE_CHECKED, INSURANCE_STATUS_CHANGED, PRIOR_AUTH_CHECKED, PRIOR_AUTH_STATUS_CHANGED, EHR_MEDICATIONS_RETRIEVED, EHR_ALLERGIES_RETRIEVED, MEDICATION_MISMATCH_DETECTED, ESCALATION_CREATED, ESCALATION_RESOLVED, ESCALATION_OVERRIDDEN, READINESS_APPROVED, READINESS_BLOCKED, STATUS_CHANGED, INTEGRATION_FAILED, PHI_ACCESSED]
- actor_type: enum [SYSTEM, FRONT_DESK_USER, CLINICAL_USER, MANAGER_USER]
- actor_id: string (SYSTEM if agent, user_id if human)
- patient_id: nullable UUID (included if PHI access)
- details_json: object (contextual information, error codes, values)
- immutable: true (cannot be updated or deleted after creation)
```

---

### Integration Points (Contracts)

**Integration A: athenahealth EHR**

```
Purpose: Retrieve scheduled appointments, medication lists, allergy records

Endpoint (pattern): 
- GET /appointments?scheduled_start_after={date}&scheduled_start_before={date}&status=SCHEDULED
- GET /patients/{patient_id}/medications
- GET /patients/{patient_id}/allergies

Authentication: Bearer token stored in env var ATHENA_API_KEY

Request Format (Appointment List):
{
  "scheduled_start_after": "2024-01-15T00:00:00Z",
  "scheduled_start_before": "2024-01-17T23:59:59Z",
  "status": "SCHEDULED"
}

Response Format (Success, HTTP 200):
{
  "data": [
    {
      "id": "apt_123456",
      "patient_id": "pat_987654",
      "provider_id": "prov_555",
      "location_id": "loc_222",
      "scheduled_start_at": "2024-01-16T09:00:00Z",
      "visit_type": "ROUTINE_VISIT",
      "procedure_code": null or "99213",
      "status": "SCHEDULED"
    }
  ],
  "next_page_token": null or "token_xyz"
}

Response Format (Medications):
{
  "patient_id": "pat_987654",
  "medications": [
    {
      "medication_id": "med_111",
      "name": "Lisinopril",
      "dose": "10",
      "dose_unit": "mg",
      "frequency": "daily",
      "status": "ACTIVE"
    }
  ]
}

Response Format (Allergies):
{
  "patient_id": "pat_987654",
  "allergies": [
    {
      "allergy_id": "allergy_001",
      "allergen_name": "Penicillin",
      "severity": "SEVERE" or null,
      "reaction": "anaphylaxis" or null,
      "status": "ACTIVE"
    }
  ]
}

Error Response (HTTP 4xx/5xx):
{
  "error_code": "string",
  "error_message": "string",
  "timestamp": "ISO 8601"
}

Timeout: 10 seconds per call
Retry: 2 retries on HTTP 5xx or timeout; exponential backoff (2s, 4s)
Rate Limit: [clinic-specific, confirm with athenahealth]
Fallback: If unavailable, do not create new work items; log failure and alert PRACTICE_MANAGER
```

**Integration B: Insurance Eligibility Tool**

```
Purpose: Verify patient insurance coverage for scheduled visit

Endpoint (assumed): POST /eligibility-check

Authentication: [clinic-specific; configure in secrets manager]

Request Format:
{
  "patient_id": "string or patient SSN or medical record number per tool contract",
  "member_id": "string if known",
  "payer_id": "string (health plan payer code)",
  "service_date": "2024-01-16",
  "procedure_code": "CPT code" or null
}

Response Format (Success):
{
  "patient_identifier": "...",
  "coverage_status": "ACTIVE" or "INACTIVE" or "UNKNOWN",
  "payer_name": "string",
  "member_name": "string",
  "effective_date": "2024-01-01",
  "termination_date": null or "2024-12-31",
  "copay_primary": 20 or null,
  "copay_specialist": 40 or null,
  "deductible": 1500 or null,
  "response_timestamp": "ISO 8601"
}

Response Format (Error or Timeout):
{
  "status": "ERROR" or "TIMEOUT",
  "error_message": "string",
  "error_code": "string"
}

Timeout: 10 seconds
Retry: 2 retries on timeout; exponential backoff (2s, 4s)
Rate Limit: [clinic-specific, confirm]
Fallback: If unavailable after retries, set insurance_status = TIMEOUT and escalate to FRONT_DESK (do not assume active coverage)
```

**Integration C: Prior-Auth Rule Source**

```
Purpose: Determine whether a procedure requires prior authorization

Contract (assumed): Rule set is clinic-maintained, e.g., a spreadsheet or database table

Query Interface:
- Load rule set at agent startup or every 1 hour (refresh for manual updates)
- Look up rule by: procedure_code + payer + other attributes as relevant

Rule Format Example:
{
  "procedure_code": "99213",
  "payer": "BlueCross",
  "location": "any",
  "prior_auth_required": false
}

{
  "procedure_code": "25645",
  "payer": "Anthem",
  "location": "any",
  "prior_auth_required": true,
  "rule_effective_date": "2024-01-01"
}

Query Logic:
- If rule found matching [procedure_code, payer]: use its prior_auth_required flag
- If multiple rules match (e.g., location-specific override): use most specific match
- If no rule found: return UNDETERMINED and escalate (never infer)

Prior-Auth Record Lookup (assumed):
- If prior_auth_required = true, query clinic's prior-auth records:
  - SELECT * FROM prior_auth_records WHERE patient_id = ? AND procedure_code = ? AND payer = ? AND expiration_date >= today
  - If record found: status = VALID
  - If no record found: status = MISSING
  - If record expired: status = EXPIRED

Response:
{
  "procedure_code": "CPT code",
  "prior_auth_required": true or false or null (if UNDETERMINED),
  "auth_status": "VALID" or "MISSING" or "EXPIRED" or "UNDETERMINED"
}
```

---

### Hard Prohibitions

The agent **must never** implement any of the following:

1. **Diagnosis** — inferring or suggesting a diagnosis based on symptoms
2. **Clinical Triage** — determining whether a visit is urgent, routine, or non-emergent based on clinical assessment
3. **Treatment Recommendations** — suggesting treatment, medication, or diagnostic procedures
4. **Medication Significance Assessment** — scoring or classifying a medication change as "important," "minor," "dangerous," etc.
5. **Allergy Significance Assessment** — deciding whether an allergy affects care or can be ignored
6. **Inference-Based Prior-Auth** — determining prior-auth requirement without an explicit rule
7. **Silent Suppression of Escalations** — marking a case ready when required human review is unresolved
8. **Fabricated Data** — guessing or inferring missing integration data instead of escalating

If a proposed implementation path would require any of the above, **do not implement it**. Escalate or remove the feature instead.

---

### Assumptions Embedded in Spec

1. athenahealth exposes machine-accessible APIs for appointments, medications, allergies
2. Insurance eligibility tool supports integration (API, batch, or RPA)
3. Clinic maintains or can provide explicit prior-auth rules
4. Patients can receive and complete digital pre-visit questionnaires
5. Queues (FRONT_DESK, CLINICAL_STAFF, PRACTICE_MANAGER) exist and are operationally owned
6. Unverified patient-reported medication data can be staged separately from official EHR
7. Allergy data is structured in EHR
8. Clinic can define narrow admin visit-reason categories and urgent-trigger phrases
9. Staff roles are distinct (front-desk, clinical, manager) with role-based permissions
10. Compliance logging requirements can be defined and are enforceable

---

## 5. Validation Design

### Validation Approach

The agent must pass the following scenarios to prove it is working correctly. Each scenario produces a testable outcome (PASS, ESCALATE, BLOCK, HOLD_FOR_REVIEW).

**Core Assertions (All Scenarios):**
1. Exactly one active work item exists per appointment (no duplicates)
2. All workflow state transitions are logged
3. All PHI accesses are logged
4. Any open escalation prevents READY_FOR_VISIT
5. No clinical interpretation is produced in safety-sensitive scenarios
6. No data is fabricated or guessed when integration fails

---

### Scenario V1: Happy Path — Routine Visit, Complete Intake

**Objective:** Verify a routine scheduled visit with complete administrative data moves to READY_FOR_VISIT without escalation.

**Setup:**
- Appointment: visit_type=ROUTINE_VISIT, no procedure_code
- Questionnaire: all required fields completed
- Insurance: returns VERIFIED_ACTIVE
- Medications: patient reports "No changes"; EHR confirms no recent changes
- Allergies: no flags in EHR
- Visit reason: "Follow-up for hypertension" (matches approved routine category)
- All integrations succeed

**Expected Behavior:**
- IntakeWorkItem: questionnaire_status=COMPLETE, insurance_status=VERIFIED_ACTIVE, prior_auth_status=NOT_REQUIRED, medication_status=NO_CHANGES_REPORTED, allergy_status=NO_FLAGS, visit_reason_status=ROUTINE_ROUTED
- Escalations: 0
- Completion decision: READY_FOR_VISIT
- Audit logs created: WORK_ITEM_CREATED, QUESTIONNAIRE_RECEIVED, INSURANCE_VERIFIED (ACTIVE), ALLERGIES_RETRIEVED (none), READINESS_APPROVED

**Outcome:** ✅ PASS

**Assertions to Code:**
```python
assert work_item.status == "READY_FOR_VISIT"
assert len(work_item.escalations) == 0
assert work_item.completion_decision == "READY_FOR_VISIT"
assert len(audit_logs) >= 4  # created, questionnaire, insurance, readiness
assert all(log.action_type in EXPECTED_ACTIONS for log in audit_logs)
```

---

### Scenario V2: Edge Case — Incomplete Questionnaire

**Objective:** Verify that missing required questionnaire fields trigger escalation without progressing to readiness.

**Setup:**
- Appointment: routine visit, no procedure_code
- Questionnaire: returns with one required field missing (e.g., visit reason blank)
- Insurance: would return VERIFIED_ACTIVE
- No medication/allergy processing happens yet

**Expected Behavior:**
- IntakeWorkItem: questionnaire_status=INCOMPLETE
- Escalations: 1 (trigger: QUESTIONNAIRE_INCOMPLETE, queue: FRONT_DESK)
- Completion decision: HOLD_FOR_REVIEW
- Status: HOLD_FOR_REVIEW (not READY_FOR_VISIT)
- Audit log: QUESTIONNAIRE_RECEIVED (status change to INCOMPLETE), ESCALATION_CREATED

**Outcome:** ✅ ESCALATE

**Assertions:**
```python
assert work_item.status == "HOLD_FOR_REVIEW"
assert work_item.questionnaire_status == "INCOMPLETE"
assert len(work_item.escalations) == 1
assert work_item.escalations[0].trigger_code == "QUESTIONNAIRE_INCOMPLETE"
assert work_item.escalations[0].queue == "FRONT_DESK"
```

---

### Scenario V3: Failure Mode — Insurance Integration Timeout

**Objective:** Verify the system fails closed when insurance verification times out.

**Setup:**
- Appointment: routine visit
- Questionnaire: complete
- Insurance integration: times out after 10 seconds
- Other data: would succeed

**Expected Behavior:**
- IntakeWorkItem: insurance_status=TIMEOUT
- Escalations: 1 (trigger: INSURANCE_TIMEOUT, queue: FRONT_DESK)
- Completion decision: HOLD_FOR_REVIEW
- Audit logs: INSURANCE_CHECKED (error_code=TIMEOUT, includes retry attempts), ESCALATION_CREATED
- No guess about coverage; no assumption of "active"

**Outcome:** ✅ BLOCK

**Assertions:**
```python
assert work_item.insurance_status == "TIMEOUT"
assert work_item.status == "HOLD_FOR_REVIEW"
assert any(log.trigger_code == "INSURANCE_TIMEOUT" for log in work_item.escalations)
assert len(audit_logs_with_action("INSURANCE_FAILED")) > 0
```

---

### Scenario V4: Boundary Test — Urgent-Phrase Visit Reason

**Objective:** Verify the system detects urgent-trigger phrases and escalates without performing clinical triage.

**Setup:**
- Appointment: routine visit
- Questionnaire: complete
- Insurance: VERIFIED_ACTIVE
- Visit reason text: "Chest pain since last night"
- Urgent-trigger phrase list includes: ["chest pain", "shortness of breath", "severe", ...]

**Expected Behavior:**
- IntakeWorkItem: visit_reason_status=ESCALATED_URGENT
- Escalations: 1 (trigger: URGENT_PHRASE_DETECTED, queue: CLINICAL_STAFF)
- Completion decision: HOLD_FOR_REVIEW
- Audit log: VISIT_REASON_URGENT_PHRASE (phrase_matched="chest pain", full_reason="Chest pain since last night")
- **Critical Assertion:** No output contains: diagnosis, urgency score, treatment advice, or clinical judgment

**Outcome:** ✅ ESCALATE (with clean boundary)

**Assertions:**
```python
assert work_item.visit_reason_status == "ESCALATED_URGENT"
assert any(e.trigger_code == "URGENT_PHRASE_DETECTED" for e in work_item.escalations)
assert work_item.escalations[0].queue == "CLINICAL_STAFF"
assert work_item.status == "HOLD_FOR_REVIEW"

# Verify no clinical output
escalation_context = work_item.escalations[0].context
assert "diagnosis" not in escalation_context  # or equivalent clinical keyword
assert "urgency_level" not in escalation_context
assert "recommend" not in escalation_context.lower()
```

---

### Scenario V5: Boundary Test — Medication Change Detected

**Objective:** Verify the system detects a medication change and escalates without assessing clinical significance.

**Setup:**
- Appointment: routine visit, no procedure
- Questionnaire: complete
- Insurance: VERIFIED_ACTIVE
- Patient reports: "I changed my blood pressure medication dose"
- EHR medications: show old dose
- Detected mismatch: medication_dose differs

**Expected Behavior:**
- IntakeWorkItem: medication_status=CHANGES_REPORTED or MISMATCH_DETECTED
- Escalations: 1 (trigger: MEDICATION_CHANGE_REPORTED or MEDICATION_MISMATCH, queue: CLINICAL_STAFF)
- Completion decision: HOLD_FOR_REVIEW
- Audit logs include full medication context for clinical review
- **Critical Assertion:** No output contains: "this is minor," "this is significant," "this is dangerous," or any clinical severity assessment

**Outcome:** ✅ ESCALATE (with clean boundary)

**Assertions:**
```python
assert work_item.medication_status in ["CHANGES_REPORTED", "MISMATCH_DETECTED"]
assert any(e.trigger_code in ["MEDICATION_CHANGE_REPORTED", "MEDICATION_MISMATCH"] for e in work_item.escalations)
assert work_item.escalations[0].queue == "CLINICAL_STAFF"
assert work_item.status == "HOLD_FOR_REVIEW"

# Verify no clinical judgment
escalation_context = work_item.escalations[0].context
for keyword in ["significant", "important", "minor", "dangerous", "safe", "unsafe"]:
    assert keyword not in escalation_context.lower()
```

---

### Scenario V6: Failure Mode — No Allergy Flag Rule Match

**Objective:** Verify that a missing prior-auth rule produces UNDETERMINED + escalation, not inference.

**Setup:**
- Appointment: procedure visit, procedure_code=25645 (orthopedic procedure)
- Questionnaire: complete
- Insurance: VERIFIED_ACTIVE
- Prior-auth rule lookup: no rule matches this procedure_code + payer combination
- No prior-auth record exists

**Expected Behavior:**
- IntakeWorkItem: prior_auth_status=UNDETERMINED
- Escalations: 1 (trigger: PRIOR_AUTH_UNDETERMINED, queue: FRONT_DESK)
- Completion decision: HOLD_FOR_REVIEW
- Audit log: PRIOR_AUTH_CHECKED (status=UNDETERMINED, rule_not_found, procedure_code, payer)
- **Critical:** System does not guess or infer prior-auth requirement

**Outcome:** ✅ ESCALATE

**Assertions:**
```python
assert work_item.prior_auth_status == "UNDETERMINED"
assert any(e.trigger_code == "PRIOR_AUTH_UNDETERMINED" for e in work_item.escalations)
assert work_item.escalations[0].queue == "FRONT_DESK"
assert work_item.status == "HOLD_FOR_REVIEW"
```

---

### Scenario V7: Edge Case — Allergy Flag Present

**Objective:** Verify that any allergy flag presence triggers escalation without clinical interpretation.

**Setup:**
- Appointment: routine visit, no procedure
- Questionnaire: complete
- Insurance: VERIFIED_ACTIVE
- Patient medications: no changes
- Visit reason: routine
- EHR allergies: "Penicillin (SEVERE)" or "Shellfish (unknown severity)"

**Expected Behavior:**
- IntakeWorkItem: allergy_status=FLAGS_PRESENT
- Escalations: 1 (trigger: ALLERGY_FLAG_PRESENT, queue: CLINICAL_STAFF)
- Completion decision: HOLD_FOR_REVIEW
- Audit logs include full allergy record for clinical review
- **Critical:** No severity assessment or clinical interpretation

**Outcome:** ✅ ESCALATE

**Assertions:**
```python
assert work_item.allergy_status == "FLAGS_PRESENT"
assert any(e.trigger_code == "ALLERGY_FLAG_PRESENT" for e in work_item.escalations)
assert work_item.escalations[0].queue == "CLINICAL_STAFF"

# Verify no clinical assessment
escalation_context = work_item.escalations[0].context
assert "severity_assessment" not in escalation_context
assert "risk_level" not in escalation_context
```

---

### Scenario V8: Concurrency Test — Duplicate Work Item Prevention

**Objective:** Verify idempotency: syncing the same appointment twice does not create duplicate work items.

**Setup:**
- Appointment: scheduled visit
- Run appointment sync once: creates work_item_1
- Run appointment sync again with identical data: should not create work_item_2

**Expected Behavior:**
- Only one active IntakeWorkItem exists for this appointment_id
- If work_item_1 is in progress, re-run does not create a second item
- Idempotency is preserved through work_item uniqueness constraint or by query-before-create logic

**Outcome:** ✅ PASS (Idempotency)

**Assertions:**
```python
work_items = database.query(IntakeWorkItem).filter(appointment_id=appointment_id)
assert len(work_items) == 1
assert work_items[0].id == work_item_1.id
```

---

### Scenario V9: Audit Trail Completeness

**Objective:** Verify that all significant actions produce audit logs with proper structure.

**Setup:**
- Run Scenario V1 (happy path) to completion
- Collect all audit logs created during workflow

**Expected Behavior:**
- Audit logs exist for: APPOINTMENT_SYNCED, WORK_ITEM_CREATED, QUESTIONNAIRE_SENT, QUESTIONNAIRE_RECEIVED, INSURANCE_CHECKED, INSURANCE_STATUS_CHANGED, EHR_ALLERGIES_RETRIEVED, READINESS_APPROVED
- Each log has: timestamp (ISO 8601 UTC), entity_id, action_type, actor_type (SYSTEM), actor_id (SYSTEM), details_json
- Logs are retrievable by patient_id, appointment_id, action_type
- No logs are modified after creation (immutability test)

**Outcome:** ✅ PASS (Audit Completeness)

**Assertions:**
```python
expected_actions = [
    "APPOINTMENT_SYNCED", "WORK_ITEM_CREATED", "QUESTIONNAIRE_SENT",
    "QUESTIONNAIRE_RECEIVED", "INSURANCE_CHECKED", "INSURANCE_STATUS_CHANGED",
    "EHR_ALLERGIES_RETRIEVED", "READINESS_APPROVED"
]
for action in expected_actions:
    logs = database.query(AuditLog).filter(action_type=action, entity_id=work_item.id)
    assert len(logs) >= 1, f"Missing audit log for {action}"
    
# Verify immutability
log = logs[0]
original_timestamp = log.timestamp
# (Attempt to modify log — should fail or be prevented)
try:
    log.details_json = {"modified": True}
    database.save(log)
    assert False, "Audit log should be immutable"
except Exception as e:
    assert "immutable" in str(e).lower() or "read-only" in str(e).lower()
```

---

### Scenario V10: Human Override + Re-Evaluation

**Objective:** Verify that when a human resolves an escalation, readiness logic re-evaluates and may transition to READY_FOR_VISIT.

**Setup:**
- Scenario V4 (urgent-phrase escalation) has left work_item in HOLD_FOR_REVIEW
- Clinical staff reviews escalation and determines it's not urgent
- Clinical staff resolves escalation (status = RESOLVED, resolution_note = "Patient clarified: took an antacid, chest pain resolved")
- Readiness logic is re-run

**Expected Behavior:**
- EscalationEvent status changes to RESOLVED
- Audit log: ESCALATION_RESOLVED (by clinical_user, resolution_note recorded)
- Work item re-evaluates readiness
- If all other conditions met (insurance active, questionnaire complete, etc.), work_item status transitions to READY_FOR_VISIT
- Audit log: READINESS_APPROVED (after escalation resolution)

**Outcome:** ✅ PASS (Override + Re-evaluation)

**Assertions:**
```python
escalation = work_item.escalations[0]
assert escalation.status == "CREATED"  # Before

# Human resolves
escalation.status = "RESOLVED"
escalation.resolved_by = "clinical_user_123"
escalation.resolution_note = "Patient clarified: chest pain resolved"
database.save(escalation)

# Readiness re-evaluated
re_evaluate_readiness(work_item)
assert work_item.status == "READY_FOR_VISIT"  # After
assert any(log.action_type == "ESCALATION_RESOLVED" for log in work_item.audit_logs)
assert any(log.action_type == "READINESS_APPROVED" for log in work_item.audit_logs)
```

---

### Validation Harness

Claude must implement a test runner that can:

```
1. Seed test data (appointments, patient profiles, insurance responses, EHR data)
2. Mock integration responses (success, timeout, error)
3. Execute each scenario
4. Assert outcomes
5. Report pass/fail and collect audit logs for inspection

Pseudo-code:
```python
class IntakeTestHarness:
    def setup_scenario(self, scenario_id):
        # Create test appointment, patient, mock integrations
        pass
    
    def execute_scenario(self):
        # Run appointment sync through to readiness decision
        pass
    
    def assert_outcome(self, expected_status, expected_escalations):
        # Verify work item state, escalations, audit logs
        pass
    
    def verify_no_clinical_output(self):
        # Scan audit logs and escalation context for prohibited keywords
        pass

# Run all scenarios
for scenario in [V1, V2, V3, V4, V5, V6, V7, V8, V9, V10]:
    harness = IntakeTestHarness()
    harness.setup_scenario(scenario)
    harness.execute_scenario()
    harness.assert_outcome(scenario.expected_status, scenario.expected_escalations)
    if "boundary" in scenario.name:
        harness.verify_no_clinical_output()
    result = "PASS" if harness.all_assertions_passed else "FAIL"
    print(f"{scenario.name}: {result}")
```

---

## 6. Build Guidance For Claude

### Core Principles

1. **Fail Closed, Not Open:** If data is missing, integration fails, or ambiguity exists, escalate. Do not guess, infer, or assume success.

2. **Rule-Based, Not ML-Inferred:** All decisions are based on explicit rules (prior-auth rules, urgent-phrase list, admin visit-reason categories). There is no model-based inference about clinical meaning.

3. **Deterministic and Reproducible:** The same input always produces the same output. There is no randomness or probabilistic logic in the core workflow.

4. **Explicit Over Implicit:** Every condition, state transition, and escalation trigger is explicitly listed in the spec. There are no "best practices" or assumptions.

5. **Audit Everything:** Every action is logged. Logs are immutable. If something is important enough to do, it's important enough to log.

6. **Clinical Boundary Enforcement:** No logic anywhere in the system performs clinical judgment. That boundary is hard and monitored.

### Build Checklist

Before marking the build complete:

- [ ] All FRs (1–10) are implemented and pass the validation scenarios
- [ ] No FR produces clinical output (diagnosis, urgency, medication judgment, allergy judgment)
- [ ] All escalation triggers are implemented and route to correct queue
- [ ] Readiness logic is deterministic: the same state always produces the same readiness decision
- [ ] Audit logging is comprehensive: every action is logged with full context
- [ ] Integration adapters are configurable (behind interfaces; credentials in config, not hardcoded)
- [ ] All error paths are handled: no silent failures, no guessed data
- [ ] Validation tests (V1–V10) all pass
- [ ] Boundary tests (V4, V5) explicitly verify no clinical output
- [ ] Concurrency test (V8) verifies idempotency
- [ ] Audit test (V9) verifies log completeness and immutability

### Implementation Priorities

**Phase 1 (Core Workflow):**
1. FR1 — Appointment sync
2. FR2 — Questionnaire dispatch and tracking
3. FR3 — Insurance verification
4. FR8 — Readiness evaluation

**Phase 2 (Safety-Critical):**
5. FR4 — Prior-auth determination
6. FR5 — Medication change detection
7. FR6 — Allergy flag retrieval
8. FR7 — Visit reason routing

**Phase 3 (Operations):**
9. FR9 — Escalation creation and routing
10. FR10 — Audit logging

**Phase 4 (Testing):**
11. Validation scenarios (V1–V10)
12. Boundary tests (V4, V5)

### Configuration

Make these configurable (do not hardcode):

- athenahealth API endpoint and credentials
- Insurance eligibility tool endpoint and credentials
- Prior-auth rule source (file path, API endpoint, database)
- Approved visit-reason admin categories
- Urgent-trigger phrase list
- Queue endpoints and notification methods
- Audit log retention period
- Integration retry counts and backoff strategy
- Questionnaire deadline (hours before visit start)
- Workflow execution schedule (appointment sync interval, etc.)

---

## 7. Summary: Fitness Criteria for Production Handoff

This spec is ready for AI coding agent handoff (Claude Code) when:

✅ **Buildability:**
- Every FR has testable acceptance criteria
- No vague language ("appropriate," "soon," "reasonable")
- All integrations have explicit request/response contracts
- All decision logic is rule-based and deterministic

✅ **Safety:**
- Clinical boundary is explicit and testable
- All escalation triggers are deterministic
- No clinical output is produced
- Audit trail is comprehensive

✅ **Completeness:**
- All assumptions are listed and flagged for validation
- All unknowns are enumerated
- Success metrics are defined
- Delegation analysis justifies every boundary

✅ **Testability:**
- Validation scenarios are concrete and reproducible
- Happy path, edge cases, and failure modes are specified
- Boundary tests verify delegation enforcement
- Audit trail completeness is testable

---

## 8. Sources & References

- `README-Participants-Week1-Scenarios.md` — Scenario 5 definition
- `README-Participants-Intro-Week1.md` — Week 1 programme structure and thinking discipline
- `production-spec-checklist.md` — Spec quality standards (buildability, entity precision, delegation, integration contracts, validation, assumptions)
- `spec-ambiguity-vs-builder-mistakes.md` — Build-loop diagnosis taxonomy
- `claude-md-examples-guide.md` — Implementation guidance for AI coding agents
- `Week1-Thinking-Discipline-Primer.md` — Thinking discipline and hypothesis testing framework

---

**END OF SPEC**

**Next Step:** Hand this spec to Claude Code and run closed build loop. Diagnose any mismatches using the taxonomy from `spec-ambiguity-vs-builder-mistakes.md`.
