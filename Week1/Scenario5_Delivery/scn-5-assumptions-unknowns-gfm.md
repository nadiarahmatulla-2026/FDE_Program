# Assumptions and Unknowns

## Purpose

This section records the assumptions currently required to make the Scenario 5 design buildable. These are not confirmed facts unless explicitly stated in the scenario. Claude must not treat these assumptions as guaranteed production truth.

Use this section to:

- separate known facts from inferred design choices
- identify what must be validated before production build
- prevent hidden or invented certainty
- surface dependencies on client data, systems, workflow ownership, and policy

## Known Facts From The Scenario

The following are known from Scenario 5:

- the client is a family medicine practice
- the practice has **6 physicians**
- the practice operates across **2 locations**
- the practice sees approximately **180 patients per day**
- the front-desk team has **4 people**
- intake work includes insurance verification, prior-auth checks for scheduled procedures, questionnaires, medication reconciliation support, allergy-flag review support, and reason-for-visit routing
- physicians often start visits with incomplete intake information
- the most common failures are **expired prior authorisations** and **unreviewed medication changes**
- the clinic uses **athenahealth**
- the clinic also uses a separate **insurance eligibility tool**
- the solution must respect HIPAA and state medical-record compliance
- clinical judgment must remain human

Anything beyond the above must be treated as an assumption or unknown unless validated.

## Assumptions Requiring Validation

### A1. Machine-Accessible EHR Interfaces Exist

**Assumption**  
athenahealth is accessible through APIs or another machine-consumable integration method for:
- scheduled appointments
- patient medication lists
- patient allergy flags

**Why this matters**  
The proposed workflow depends on retrieving structured EHR data automatically. Without this access, major parts of the agentic design cannot function as specified.

**What must be validated**
- which athenahealth interfaces are available in this clinic environment
- whether appointment, medication, and allergy data are exposed
- environment limitations, auth method, rate limits, and write restrictions

**Risk if false**  
The design would need to shift to manual or partially manual workflows, and several automated checks would not be implementable.

---

### A2. The Insurance Eligibility Tool Has Machine-Accessible Integration

**Assumption**  
The separate insurance eligibility system supports API access or another machine-driven integration pattern.

**Why this matters**  
Insurance verification is one of the core intake steps. The proposed automation depends on a structured eligibility result.

**What must be validated**
- whether an API exists
- if not, whether there is an approved RPA or batch interface
- returned status values
- timeout behavior
- authentication and throughput limits

**Risk if false**  
Insurance verification may remain human-led, reducing the degree of safe automation and changing readiness logic.

---

### A3. Prior-Auth Requirement Rules Exist In Structured Form

**Assumption**  
The clinic has, or can provide, an explicit structured source of truth for determining whether a scheduled procedure requires prior authorization.

**Why this matters**  
The agent can safely check prior-auth requirement only if the decision is based on explicit configured rules. It must not infer payer or clinical logic.

**What must be validated**
- whether prior-auth rules already exist in structured form
- whether rules vary by payer, procedure, location, or provider
- where those rules are maintained
- who owns updates and exception handling

**Risk if false**  
The agent would be unable to safely determine prior-auth requirement and would need to escalate more cases or remove this automation step.

---

### A4. Patient Intake Can Be Collected Digitally Before The Visit

**Assumption**  
Patients can receive and complete pre-visit questionnaires through an approved digital channel such as portal, SMS, or email.

**Why this matters**  
The proposed design assumes questionnaire collection can happen before the visit and can be tracked automatically.

**What must be validated**
- which communication channels are approved
- whether the clinic already has patient consent and delivery infrastructure
- what percentage of patients can realistically complete digital intake
- fallback process for patients who do not use digital channels

**Risk if false**  
Questionnaire completion may remain partially manual, limiting intake completeness improvements and requiring different operational design.

---

### A5. There Is A Defined Human Owner For Safety-Sensitive Escalations

**Assumption**  
The clinic has a real operational owner for human review queues, especially for:
- urgent-looking or ambiguous visit reasons
- medication changes
- allergy flag review

**Why this matters**  
An escalation path is only safe if a responsible team actually receives, monitors, and resolves the work in time.

**What must be validated**
- whether `CLINICAL_STAFF` is a real queue or a placeholder label
- who reviews urgent-looking visit reasons first
- expected response times
- what happens outside business hours or near visit start time

**Risk if false**  
The system could escalate correctly in software but still fail operationally because nobody owns the review.

---

### A6. Patient-Reported Medication Changes Can Be Stored Before Human Review

**Assumption**  
The system is allowed to store unverified patient-reported medication changes in a staging area before human review.

**Why this matters**  
The design requires collecting and comparing patient-reported medication information without automatically treating it as confirmed medical record data.

**What must be validated**
- whether unverified patient-reported medication data can be stored
- whether it may be written into athenahealth or must remain external until reviewed
- required labeling of unverified data
- retention and audit requirements

**Risk if false**  
Medication-change collection may need a different storage and workflow design.

---

### A7. Allergy Flags Are Available As Structured EHR Data

**Assumption**  
Allergy information is stored in athenahealth in a structured format that can be detected reliably.

**Why this matters**  
The design assumes the system can determine whether an allergy flag exists without interpreting unstructured notes.

**What must be validated**
- whether allergy flags are structured
- whether severity is structured or absent
- whether the system can distinguish active vs inactive allergy records
- whether access restrictions apply

**Risk if false**  
The agent may not be able to detect allergy presence safely, requiring broader human review.

---

### A8. Visit Reason Routing Rules And Trigger Phrases Can Be Defined Explicitly

**Assumption**  
The clinic can provide a narrow administrative routing taxonomy and a maintained list of urgent-trigger phrases that require human review.

**Why this matters**  
The design deliberately avoids free-form clinical interpretation. It depends on explicit rules for what counts as safe administrative routing versus mandatory escalation.

**What must be validated**
- what the approved routine categories are
- which phrases always require human review
- who maintains the list
- how updates are approved and deployed

**Risk if false**  
Visit-reason routing may become either too risky or too weakly defined to implement safely.

---

### A9. Staff Roles And Permissions Are Distinct Enough To Support Controlled Overrides

**Assumption**  
The clinic distinguishes front-desk, clinical staff, and manager permissions in a way that supports queue-based review and audited overrides.

**Why this matters**  
The design depends on different users being allowed to see, resolve, or override different types of intake issues.

**What must be validated**
- current role and permission model
- who may resolve which escalation types
- who may override blocked or hold states
- whether override reason capture is already policy

**Risk if false**  
The system could route work correctly but still fail governance or operational control requirements.

---

### A10. Logging, Retention, And Compliance Requirements Are More Specific Than The Scenario States

**Assumption**  
The clinic has defined retention and audit requirements for PHI access logs, workflow logs, escalation records, and override records.

**Why this matters**  
The scenario mentions HIPAA and state medical-record compliance, but not exact retention periods or audit controls. A production-safe build requires more precision.

**What must be validated**
- log retention periods
- whether workflow logs are part of the medical record
- required access-log fields
- approved storage location for logs
- deletion and archival policy

**Risk if false**  
The system may be operationally useful but non-compliant.

## Unknowns That Must Be Resolved Before Production Build

These are not just assumptions; they are unresolved design dependencies.

| ID | Unknown | Why It Matters |
|:--|:--|:--|
| U1 | Exact athenahealth integration contract | Determines whether the proposed automated EHR reads are feasible |
| U2 | Exact insurance eligibility interface and failure behaviors | Determines verification reliability and fallback design |
| U3 | Whether prior-auth logic is centralized, payer-specific, or staff tribal knowledge | Determines whether prior-auth checks can be automated safely |
| U4 | Which staff role owns urgent-looking visit reasons | Determines whether escalation logic is operationally viable |
| U5 | Whether patient-reported medication changes can be staged separately from the official chart | Determines safe storage model |
| U6 | Approved channels for patient outreach | Determines questionnaire delivery design |
| U7 | Actual baseline metrics for intake completeness, handling time, and failure rates | Needed to validate proposed success targets |
| U8 | Exact compliance requirements for audit logs and retention | Needed before production deployment |

## Validation Questions For The Client

Before production build, validate at least the following:

1. Which athenahealth data domains and APIs are available in your environment?
2. Does the insurance eligibility tool expose an API or only a manual workflow?
3. Where are prior-auth rules currently defined, and are they structured enough for automation?
4. Which channels are approved for sending pre-visit questionnaires?
5. Who owns review of urgent-looking or ambiguous visit reasons?
6. Can unverified patient-reported medication data be stored outside or inside the EHR before review?
7. How are allergy records represented in athenahealth?
8. What role-based permissions exist for front-desk staff, clinical staff, and managers?
9. What audit logging and retention requirements apply to this workflow?
10. What are the current baseline metrics for intake completeness, delays, and defect rates?

## Build Guidance For Claude

When building from this section:

- do not convert assumptions into hard-coded facts
- keep integrations configurable
- treat unresolved unknowns as blockers for production readiness, not reasons to invent behavior
- expose assumptions clearly in config, docs, or admin settings where possible
- prefer escalation and human review when an unknown affects safety or correctness
- keep any client-specific rule source externalized rather than embedded in model inference

## Sources

- `README-Participants-Week1-Scenarios.md`
- `README-Participants-Intro-Week1.md`
- `production-spec-checklist.md`
- `claude-md-examples-guide.md`