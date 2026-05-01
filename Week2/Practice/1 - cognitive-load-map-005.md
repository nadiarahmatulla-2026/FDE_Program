# Scenario 5 — Cognitive Load Map (CLM)

## Overview

This document decomposes **Westbridge Family Medicine's patient intake workflow** into cognitive work: the Jobs to be Done (JTBDs), Cognitive Zones within each job, and the Breakpoints where control handoffs occur.

The map covers **all four work streams** (insurance verification, prior authorization, medication reconciliation, visit-reason triage) with **detailed depth on the primary two streams** (insurance verification and prior authorization).

---

## Map Metadata

| Property | Value |
|---|---|
| **CLM Version** | 1.0 |
| **Practice** | Westbridge Family Medicine |
| **Scenario** | Scenario 5 — Small-Clinic Patient Intake |
| **Mapped Work Streams** | Insurance Verification (detailed), Prior Authorization (detailed), Medication Reconciliation (summary), Visit-Reason Triage (summary) |
| **Unmapped (Summary-Only)** | None (all 4 streams included; 2 detailed, 2 summary) |
| **Total JTBDs (Detailed)** | 6 detailed JTBDs (Insurance Verification = 3, Prior Authorization = 3) |
| **Total CognitiveZones** | 13 zones identified |
| **Total Breakpoints** | 11 breakpoints identified |
| **Grounding Source** | Enriched scenario brief + Artefacts 5.1 (PA chase list), 5.2 (physician complaint), 5.3 (billing error) + Domain model research |
| **Lived vs. Documented Notes** | Included in Section 2 below |
| **Primary Friction Points** | Identified in Section 3 below |

---

## Part 1: Lived vs. Documented — Where Work Actually Differs From SOP

### Insurance Verification

**SOP Says**:
> "Verify insurance at check-in using Availity or patient portal. Confirm coverage is active. Collect copay. Flag if verification fails and escalate to front-desk manager."

**Lived Practice Actually**:
1. Insurance verification happens at **multiple times**: scheduling (3-5 days before), day-before reminder, arrival/check-in
2. Availity is queried, but **30% of queries have issues** (timeout, data mismatch, no-match)
3. **For problem cases**, front desk makes judgment calls:
   - Patient swears they're covered; Availity says no → front desk checks the patient's card, asks patient when they switched plans, may call insurer
   - Availity shows plan A; patient card shows plan B → front desk assesses which is more recent (card date vs. data freshness)
   - Availity times out → front desk falls back to phone verification or marks patient as "verify after visit"
4. **Data discrepancies are common**:
   - Medicaid data is 24-48 hours behind (state batch updates)
   - Medicare Advantage plans change at calendar-year boundaries (January 1) but Availity data sometimes lags
   - Patient's insurance changed (employer to spouse's plan, or new job) but patient doesn't remember/didn't tell practice
5. **Re-verification happens if**:
   - Patient's card looks more recent than prior verification
   - Patient reports insurance change at check-in
   - Front desk notices last verification was >6 months ago

### Prior Authorization

**SOP Says**:
> "Physician will order imaging or referral in athenahealth. System will auto-trigger PA request. Front desk will monitor athenahealth ticker for PA status and escalate if still pending 48 hours before appointment."

**Lived Practice Actually**:
1. **PA submission is manual, not automatic**. When physician orders MRI:
   - Order appears in athenahealth as a task
   - Front desk or Dana sees it (if they check)
   - Front desk manually creates PA request in Availity or calls insurer
   - Request is submitted by email/fax/phone (varies by insurer)
   - **If front desk is busy**, it's easy to miss
2. **PA tracking is 100% manual** (Dana's Google Sheet):
   - Dana maintains a personal tracker with insurer name, submission date, target chase date, appointment date
   - Chase timing varies by insurer (Aetna 5 days, UHC Choice 6-7 days, Wellpath 7+ days but also "always deny first time, resubmit with prior visit doc")
   - If Dana is out sick, no one else has visibility into what needs to be chased
   - If a date is entered wrong in the sheet, it goes untraced
3. **Denials are handled ad-hoc**:
   - Insurer sends denial via email/fax to front desk
   - Front desk reads denial reason (or doesn't; mail gets buried)
   - If reason is "missing info," someone needs to decide whether to resubmit or escalate
   - There's no systematic playbook for Wellpath denials (which are frequent and expected)
4. **Discovery at visit time is a failure**:
   - Ideal path: PA is chased, approved, documented well before appointment
   - Lived failure: PA is never checked until appointment time; patient shows up; technician says "no authorization"; appointment is cancelled (Artefact 5.2)

### Medication Reconciliation

**SOP Says**:
> "Patient completes pre-visit questionnaire naming current medications. DoseSpot integration auto-checks for interactions and allergies. Alerts are flagged to physician."

**Lived Practice Actually**:
1. **Pre-visit questionnaire has low adoption**: Many patients don't complete it (older patients, paper-form users, portal-unfamiliar). Alternatively, patient fills out paper form at check-in.
2. **OTC and supplements are often not captured**: Questionnaire doesn't ask for ibuprofen, aspirin, vitamin D, etc. Front desk doesn't have standing order to ask. Patient doesn't volunteer.
3. **DoseSpot is pharmacy-network-only**: If patient fills prescriptions at an independent pharmacy or out-of-pocket, DoseSpot doesn't see it.
4. **Specialist prescriptions are missing**: Cardiologist prescribed beta-blocker 6 months ago; specialist notes were received as PDF (not structured data); EHR med list wasn't updated; DoseSpot doesn't see specialist meds.
5. **Allergy data is stale**: Patient's EHR shows "Penicillin ALLERGY" from 15 years ago (actually was rash, not anaphylaxis, patient recovered). No one re-verifies this during visits. Physician avoids penicillins unnecessarily, leading to costlier alternatives.

### Visit-Reason Triage

**SOP Says**:
> "Scheduler confirms patient's reason for visit. Front desk may adjust based on EHR context. Reason determines visit length and prep instructions."

**Lived Practice Actually**:
1. **Reason is often vague at scheduling**: Patient says "I need a checkup" or "follow-up" (follow-up for what?). Scheduler doesn't probe.
2. **Acute issues hide under routine scheduling**: Patient calls for "routine checkup." Scheduler books 20-minute slot. Patient arrives and mentions "oh, I've had chest pain since yesterday." Now physician has 20 minutes to assess a complex issue.
3. **Procedure prep isn't always confirmed**: Patient is scheduled for "wart removal." SOP says patient should avoid aspirin × 5 days. Does front desk call to confirm? Not always.
4. **Patient's primary complaint changes between scheduling and visit**: Patient scheduled for diabetes follow-up. Patient arrives and says "Actually, I came because of this rash." Triage has to re-route.

---

## Part 2: Primary Friction Points (Where Work Breaks Down)

### Friction Point 1: Insurance Verification Data Lag + Judgment Calls
**Symptom**: Front desk calls insurer 3 times because Availity shows one thing, patient's card shows another, and they're not sure who to trust.

**Why it happens**: Medicaid updates are batch-based (24-48 hour lag). Availity data freshness is inconsistent. Patient doesn't track their own coverage changes.

**Cognitive load**: Front desk must decide "is this data lag or real coverage change?" Judgment criteria: card date, patient's memory, insurer's batch schedule.

**Impact**: 5-15 minutes per problem case; sometimes billing dispute later when claim is processed against wrong coverage.

---

### Friction Point 2: Prior Authorization Fall-Through
**Symptom** (Artefact 5.2): Patient scheduled for MRI follow-up. PA is never submitted. Patient arrives for appointment. Technician checks PA status. PA is missing. Appointment is cancelled. Patient: "This is the second time this has happened to me."

**Why it happens**: PA submission is manual, not triggered by the system. If front desk is busy, it's easy to miss. No automatic alert when PA is missing at appointment time.

**Cognitive load**: Someone must remember to:
1. Identify the PA requirement (look at procedure code + patient's insurance plan rules)
2. Gather required information (diagnosis code, procedure code, etc.)
3. Submit to insurer (email/fax/phone varies by insurer)
4. Track submission (Dana's Google Sheet)
5. Chase on schedule (varies by insurer: 5-7 days)
6. Handle denial (if denied, re-assess + resubmit)

**Impact**: Visit cancelled, patient harm, reputation damage, revenue loss.

---

### Friction Point 3: Medication Reconciliation Gaps
**Symptom**: Patient is on ibuprofen (OTC, not in EHR). Patient is also on warfarin (blood thinner, in EHR). Physician doesn't know about ibuprofen. Prescribes something else that interacts with both. Drug interaction goes undetected until patient has side effect.

**Why it happens**: OTC is not captured systematically. DoseSpot only sees pharmacy data. Patient doesn't volunteer OTC.

**Cognitive load**: Front desk or nurse must ask, patient must remember, system must capture and re-check.

**Impact**: Patient harm, medication error, liability.

---

### Friction Point 4: Allergy Data Stale + Not Re-Verified
**Symptom**: EHR shows "Penicillin ALLERGY" from 2010 (actually was urticaria, patient recovered). Physician avoids penicillins in 2024, choosing more expensive alternatives despite penicillin being first-line for strep throat.

**Why it happens**: Allergy data is entered once and not systematically re-verified. Physician assumes anaphylaxis risk without investigation.

**Cognitive load**: Physician must ask "what was the actual reaction?" if they want to clarify. But most physicians don't ask; they just avoid.

**Impact**: Cost inefficiency, unnecessary medication avoidance, patient potential harm if allergy was real and ignored.

---

### Friction Point 5: Visit-Reason Triage Misclassification
**Symptom**: Patient scheduled for "routine visit." Allocated 20 minutes. Patient arrives with acute chest pain (symptom started yesterday). Physician now has 20 minutes to assess a potentially serious issue. Schedule cascades (other patients delayed).

**Why it happens**: Scheduler didn't probe vague reason. Patient didn't mention acute issue when scheduling. No system flag for "patient reported new acute symptoms."

**Cognitive load**: Front desk must listen at arrival, recognize acute signal, escalate to physician, reschedule time buffer.

**Impact**: Inadequate assessment time, other patients delayed, potential clinical miss.

---

### Friction Point 6: System Blindness to PA Status
**Symptom**: PA is submitted via phone to insurer. Front desk doesn't log it in athenahealth. Dana's Google Sheet is the only record. Dana doesn't see it on her daily check (busy, missed a line). PA is never chased. Appointment time arrives. PA status is unknown. Discovered at arrival.

**Why it happens**: PA can be submitted via email/fax/phone. Not all submissions are logged to athenahealth. Dana's Google Sheet is error-prone (manual entry, single point of failure).

**Cognitive load**: Someone must remember to log + chase + verify status. High error rate if this is manual.

**Impact**: PA status lost; visit delayed or cancelled at appointment time.

---

## Part 3: Work Stream Decomposition

---

# WORK STREAM 1: INSURANCE VERIFICATION

## Overview
- **Volume**: ~180 patients/day; ~54 complex cases/day requiring judgment
- **Handling time**: 3 min (automated cases) + 5-12 min (judgment cases)
- **Primary cognitive outcome**: Determination of whether patient's stated insurance is active, valid, accepted by practice, and what copay/deductible applies
- **Key systems**: athenahealth (EHR), Availity (insurance intermediary), patient phone
- **Key data**: patient demographics, insurance card, Availity API response, athenahealth history

---

## JTBD 1.1: Query Availity for Insurance Coverage Status

### Purpose
Determine in real-time whether a patient's stated insurance coverage is active and valid.

### Actor
Front-desk staff (supported by agent in candidate design)

### Cognitive Zones

#### Zone 1.1.1: Availity Query Execution
**Micro-tasks**:
1. Retrieve patient ID and demographics from athenahealth
2. Enter patient info into Availity query interface
3. Submit query to Availity API
4. Receive response (active/pending/expired/no-match/error)

**Error tolerance**: **LOW** (wrong copay → billing dispute or patient charged wrong amount)

**Latency sensitivity**: **MEDIUM** (not real-time critical, but should complete within 2 min at check-in)

**Data dependencies**: Patient ID (athenahealth), Availity API access

**Human expertise required**: None (routine query execution)

**Failure modes**:
- Availity times out (system down or overloaded)
- Query returns error code (malformed request or patient not found)
- Query returns outdated data (Availity's upstream payer data is stale)

---

#### Zone 1.1.2: Response Interpretation
**Micro-tasks**:
1. Parse Availity response (code: "active" / "pending" / "expired" / "no-match")
2. Extract copay, deductible, visit restrictions
3. Reconcile response against patient's stated coverage
4. Identify any flags (e.g., "requires referral for specialist")

**Error tolerance**: **LOW** (wrong interpretation → billing error or coverage misapplication)

**Latency sensitivity**: **MEDIUM** (2-3 min acceptable)

**Data dependencies**: Availity response, patient's stated plan from insurance card or portal

**Human expertise required**: **CONDITIONAL**
- If response is clear (matches patient's stated plan) → No expertise needed
- If response is ambiguous or mismatches → Judgment required (see Zone 1.1.3)

**Failure modes**:
- Misinterpreting response code (e.g., "pending" means coverage is coming; front desk treats as "no coverage")
- Missing or misreading a flag (e.g., "requires referral" for HMO but front desk doesn't notice)
- Comparing Availity response to outdated patient card

---

#### Zone 1.1.3: Data Mismatch Resolution
**Micro-tasks**:
1. Identify discrepancy (Availity shows Plan A, patient card shows Plan B; or Availity shows expired but patient swears they renewed)
2. Assess data freshness (which source is more current?)
3. Make judgment call (trust Availity or trust patient/card?)
4. Take action (accept Availity result, call insurer for verification, or mark patient as "verify after visit")

**Error tolerance**: **MEDIUM** (resolution doesn't affect visit flow; can be corrected at billing later, but patient experience suffers)

**Latency sensitivity**: **HIGH** (if unresolved, patient check-in is delayed)

**Data dependencies**: Patient card, Availity response, date of last Availity verification (in athenahealth)

**Human expertise required**: **YES**
- Expertise: Understanding insurance plan changes (employment, spouse, new calendar year), data freshness expectations, Medicaid batch cycles
- Decision criteria: "Is Availity more reliable than the card right now?" (varies by payer, time of month)

**Failure modes**:
- Trusting stale Availity data over a recently-dated card (patient has new coverage)
- Accepting patient's claim without verifying (patient mistaken or dishonest)
- Calling insurer too often (over-cautious) or not calling enough (over-trusting Availity)

---

### Breakpoints

#### Breakpoint 1.1.A: Coverage Active, Clear Match
**Trigger**: Availity response = "active"; matches patient's stated plan; no mismatch

**From → To**: System → Front-desk execution (proceed to copay collection)

**Consequence if handled poorly**: None (no escalation needed)

**Agentic opportunity**: Agent can flag as "proceed" automatically

**Risk if autonomous**: None; this is routine

---

#### Breakpoint 1.1.B: Coverage Mismatch Detected
**Trigger**: Availity shows Plan A; patient's card shows Plan B; or Availity shows expired coverage

**From → To**: System → Front-desk judgment

**Consequence if handled poorly**: Billing error (claim denied against wrong plan), patient frustration ("why am I being charged wrong copay?")

**Agentic opportunity**: Agent flags mismatch and escalates to front desk with data (which is more recent?); front desk makes judgment call

**Risk if autonomous**: Agent assumes Availity is always right (it's not; Medicaid data lags 48 hours)

---

#### Breakpoint 1.1.C: Availity Timeout or Error
**Trigger**: Query returns error code or times out

**From → To**: System failure → Front-desk fallback

**Consequence if handled poorly**: Patient check-in delayed, or front desk proceeds without verification (billing risk)

**Agentic opportunity**: Agent could trigger automatic fallback (phone verification, mark patient as "pending," escalate to Dana)

**Risk if autonomous**: Agent defaults to "mark patient as self-pay" and charges full cost; later billing issue when insurance was actually valid

---

## JTBD 1.2: Verify Coverage Validity and Assess Data Freshness

### Purpose
Confirm that the insurance verification data is current and reflects the patient's actual coverage status (not outdated information or data lag).

### Actor
Front-desk staff (for decision) + system (for data lookup)

### Cognitive Zones

#### Zone 1.2.1: Coverage Age Assessment
**Micro-tasks**:
1. Check the timestamp of the last successful Availity verification (stored in athenahealth)
2. Compare to current date
3. Assess whether re-verification is needed based on time elapsed and known data freshness issues

**Error tolerance**: **LOW** (stale verification can lead to billing error if coverage has lapsed)

**Latency sensitivity**: **LOW** (this is a background check, not time-critical at check-in)

**Data dependencies**: athenahealth verification timestamp, current date, knowledge of Medicaid/Marketplace batch cycles

**Human expertise required**: **CONDITIONAL**
- For routine cases (last verified <30 days ago) → No expertise; automated rule applies
- For edge cases (Medicaid, birthday month transitions) → Judgment required (Medicaid renewals happen on specific dates; should we re-verify?)

**Failure modes**:
- Assuming "verified 6 months ago" is still valid (coverage expires, patient loses job, enrollment ends)
- Over-verifying every visit (unnecessary API calls, delays)
- Not accounting for Medicaid annual renewal (patients must re-enroll; coverage lapses on specific month/day)

---

#### Zone 1.2.2: Known Data Freshness Issues
**Micro-tasks**:
1. Identify patient's insurance type (commercial, Medicare, Medicaid, etc.)
2. For Medicaid: assess if it's renewal month (coverage may have changed)
3. For commercial: check if calendar year changed (plans typically change January 1)
4. For Medicare Advantage: check if it's annual enrollment season (Oct-Dec)
5. Decide whether to re-verify or trust cached data

**Error tolerance**: **LOW** (missing a known issue = billing error)

**Latency sensitivity**: **LOW** (background check)

**Data dependencies**: Patient insurance type, current month/date, knowledge of plan renewal cycles

**Human expertise required**: **YES** (requires knowledge of insurance industry calendar)

**Failure modes**:
- Not knowing Medicare Advantage plans change Oct-Dec (front desk assumes December patient still has Oct plan)
- Not knowing Medicaid renewal is mid-month for some states and end-of-month for others
- Assuming commercial plans never change outside January 1 (employer plan changes can happen anytime)

---

### Breakpoints

#### Breakpoint 1.2.A: Re-Verification Triggered
**Trigger**: Last verification >30 days ago, OR patient reports coverage change, OR it's Medicaid renewal month

**From → To**: System rule → Front-desk action (run Availity query again)

**Consequence if handled poorly**: Stale data used for billing; claim denied; billing dispute

**Agentic opportunity**: Agent automatically triggers re-verification and alerts front desk if new data differs from cached data

**Risk if autonomous**: Agent assumes all plans need re-verification every 30 days (conservative, but inefficient use of API calls)

---

#### Breakpoint 1.2.B: Known Data Lag Period
**Trigger**: Patient is Medicaid; current date is within 2 days of state batch-update cycle

**From → To**: System identification → Human judgment

**Consequence if handled poorly**: Availity shows old plan; front desk trusts it; claim is adjudicated against wrong plan

**Agentic opportunity**: Agent flags "Medicaid data may be stale" and recommends phone verification or conservative approach

**Risk if autonomous**: Agent assumes Availity is always current (it's not for Medicaid)

---

## JTBD 1.3: Collect Copay and Document Coverage in EHR

### Purpose
Capture the patient's financial obligation (copay) and formally document the verified coverage in the EHR for billing and audit purposes.

### Actor
Front-desk staff (execution); agent (candidate: can automate copay collection reminder and documentation)

### Cognitive Zones

#### Zone 1.3.1: Copay Amount Verification
**Micro-tasks**:
1. Retrieve copay amount from Availity response (or patient card if Availity is unavailable)
2. Cross-check against patient's prior visit history (is this the same copay as last time?)
3. Confirm copay with patient ("Your copay is $30 today; does that match what you expect?")
4. Note any special cases (waived copay for preventive, copay applies for visit but not labs)

**Error tolerance**: **LOW** (wrong copay amount = patient pays wrong amount or billing dispute)

**Latency sensitivity**: **MEDIUM** (needs to be clear before patient sees physician; copay collected at arrival)

**Data dependencies**: Availity response, patient prior visit copay history (athenahealth)

**Human expertise required**: NONE (routine lookup + confirmation)

**Failure modes**:
- Availity shows $30; patient card says $50; front desk doesn't notice discrepancy; charges wrong amount
- Patient on high-deductible plan; deductible not met; full visit cost is due (not just copay); front desk doesn't realize
- Preventive visit (copay waived); front desk charges routine copay anyway

---

#### Zone 1.3.2: Financial Responsibility Communication
**Micro-tasks**:
1. Explain copay to patient in clear language
2. Clarify any special cases (e.g., "This is your annual preventive visit; copay is waived")
3. Confirm patient understands and agrees to payment
4. Collect payment or note if patient cannot pay (scholarship, financial hardship)

**Error tolerance**: **MEDIUM** (communication failure can lead to patient frustration but is recoverable)

**Latency sensitivity**: **HIGH** (needs to happen before visit to avoid delays)

**Data dependencies**: Copay amount, practice financial policies (copay collection rules, payment options)

**Human expertise required**: **YES** (soft skill: explaining financial terms to patient, assessing patient's ability to pay)

**Failure modes**:
- Not explaining copay; patient is surprised at checkout
- Not clarifying that full cost is due if deductible isn't met; patient expects copay only
- Not offering payment alternatives (patient cannot pay; practice has no accommodation)

---

#### Zone 1.3.3: EHR Documentation
**Micro-tasks**:
1. Log insurance verification in athenahealth (plan name, copay amount, verification date/time)
2. Note any flags (e.g., "requires referral for specialist," "coverage expires 06/30/2025")
3. Ensure documentation is visible to physician (alerts, flags)
4. Document any discrepancies or special handling (e.g., "verified by phone; Availity was down")

**Error tolerance**: **LOW** (missing documentation = audit trail lost, billing issues)

**Latency sensitivity**: **LOW** (can be done after patient is checked in)

**Data dependencies**: athenahealth, Availity verification result, coverage details

**Human expertise required**: NONE (routine data entry; templated)

**Failure modes**:
- Not documenting verification (no audit trail if billing is disputed later)
- Documenting wrong copay amount (billing uses wrong amount later)
- Failing to flag "requires referral" (physician doesn't know when seeing an HMO patient)

---

### Breakpoints

#### Breakpoint 1.3.A: Copay Collection Complete
**Trigger**: Patient confirms copay amount and payment is collected (or waived)

**From → To**: Front-desk execution → EHR documentation complete → Ready for physician

**Consequence if handled poorly**: None (routine completion)

**Agentic opportunity**: Agent could confirm copay collection and auto-flag "insurance complete" in EHR

**Risk if autonomous**: None; this is straightforward

---

#### Breakpoint 1.3.B: Special Case Flag
**Trigger**: Preventive visit (copay waived), OR high-deductible plan (full cost due), OR patient cannot pay

**From → To**: System rule → Front-desk judgment

**Consequence if handled poorly**: Patient charged wrong amount, or patient cannot pay and visit is delayed

**Agentic opportunity**: Agent flags special case and alerts front desk; front desk makes judgment call on payment plan

**Risk if autonomous**: Agent charges full copay when visit is actually preventive and copay should be waived

---

---

# WORK STREAM 2: PRIOR AUTHORIZATION

## Overview
- **Volume**: ~25 patients/day; ALL require human judgment on PA requirements, many require manual submission
- **Handling time**: 2-5 min (PA requirement check) + 10-15 min (submission) + 5-10 min (chase/follow-up per scheduled interval)
- **Primary cognitive outcome**: Determination of whether a scheduled service requires pre-authorization; systematic submission, tracking, chase, and escalation
- **Key systems**: athenahealth (EHR), Availity (insurer intermediary), insurer portals (email/fax/phone), Dana's Google Sheet (tribal knowledge)
- **Key data**: patient insurance plan, scheduled service (CPT code, diagnosis), insurer PA requirements, appointment date/time
- **Critical failure mode** (Artefact 5.2): PA never submitted or not tracked; discovered missing at appointment time; appointment cancelled

---

## JTBD 2.1: Determine If Scheduled Service Requires Prior Authorization

### Purpose
Assess whether a physician's scheduled service order (imaging, procedure, specialty referral) requires pre-authorization from the patient's insurance plan.

### Actor
Front-desk staff or Dana (requires knowledge of insurer PA requirements)

### Cognitive Zones

#### Zone 2.1.1: PA Requirement Rule Lookup
**Micro-tasks**:
1. Identify scheduled service type (e.g., "MRI brain," "colonoscopy," "orthopedic consultation")
2. Map service to procedure/CPT code (e.g., MRI brain = 70553)
3. Identify patient's insurance plan (e.g., Aetna PPO, UHC Choice, Wellpath Medicaid)
4. Retrieve PA requirement rule for this plan + service combination (e.g., "Aetna requires PA for MRI; UHC Choice typically doesn't for screening colonoscopy age >50; Wellpath requires PA for all imaging")
5. Determine: PA required YES / NO / CONDITIONAL (conditional on prior attempt, diagnosis, age, etc.)

**Error tolerance**: **LOW** (if PA is missed, service is blocked at appointment time)

**Latency sensitivity**: **MEDIUM** (rule lookup needed before appointment scheduling, but not real-time urgent)

**Data dependencies**: Procedure code (from athenahealth), patient insurance plan, PA rule database (insurer policy manual or coded rules)

**Human expertise required**: **YES**
- Expertise: Knowledge of insurance industry PA requirements, specific insurer policies, exception cases
- Decision criteria: "Does this plan typically require PA for this service?" (varies widely by insurer and service type)

**Failure modes**:
- Confusing "routine imaging" (e.g., chest X-ray, often no PA) with "advanced imaging" (e.g., MRI, usually PA required)
- Assuming all insurers have same PA requirements (they don't; UHC and Aetna differ significantly)
- Forgetting insurer-specific exceptions (e.g., Medicare Advantage plans have different PA thresholds than commercial)
- Not checking patient's plan year (plans change Jan 1; 2024 rules may not apply to 2025)

---

#### Zone 2.1.2: Service-Specific Nuance Assessment
**Micro-tasks**:
1. Assess if service is "standard" or "non-standard" for the diagnosis
   - E.g., "MRI for knee pain" is routine; "MRI for mild headache" may trigger additional PA scrutiny
   - Some insurers require "medical necessity" justification even if PA is technically required
2. Check if service is a first attempt or repeat attempt
   - Some insurers allow imaging once per year without PA; second imaging in same year needs PA
3. Assess if there's a prior-authorization requirement for "prior diagnostic attempt"
   - E.g., some Medicaid plans require "X-ray first before MRI" (prove you've tried cheaper option first)
4. Determine: does this service need PA, and if so, what additional documentation is required?

**Error tolerance**: **LOW** (missing a nuance = PA denial + resubmission + delay)

**Latency sensitivity**: **MEDIUM** (needed before submission, but not immediate)

**Data dependencies**: Service type, patient's prior imaging history (athenahealth), diagnosis, insurer PA rules

**Human expertise required**: **YES** (clinical knowledge of typical imaging workflows, understanding of insurer medical necessity criteria)

**Failure modes**:
- Submitting PA for "second MRI" without noting that first MRI ruled out the suspected condition (insurer may deny as not medically necessary)
- Not including diagnosis in PA request because "diagnosis isn't a PA requirement" — but it is for some insurers
- Submitting PA for advanced imaging without documenting that simpler imaging was attempted first (insurer may deny citing stepped-care rules)

---

### Breakpoints

#### Breakpoint 2.1.A: PA Clearly Required
**Trigger**: Service is routine PA-required (e.g., MRI for orthopedic patient, specialty referral for HMO)

**From → To**: Front-desk rule lookup → Immediate escalation to PA submission (JTBD 2.2)

**Consequence if handled poorly**: None (expected path)

**Agentic opportunity**: Agent auto-detects PA requirement and flags "PA needs to be submitted"

**Risk if autonomous**: Agent incorrectly flags "PA required" when service is actually routine; unnecessary PA submissions waste insurer time

---

#### Breakpoint 2.1.B: PA Not Required or Unclear
**Trigger**: Service is routine (e.g., preventive colonoscopy age >50, basic X-ray) OR PA requirement is unclear (depends on prior attempts or prior imaging history)

**From → To**: Front-desk rule lookup → Front-desk judgment

**Consequence if handled poorly**: If PA is actually required but front desk misses it, service is blocked at appointment time

**Agentic opportunity**: Agent flags uncertain cases with supporting context ("PA may not be required, but double-check"; "Wellpath requires PA for all imaging, even routine")

**Risk if autonomous**: Agent assumes "not required" and downstream discovers PA was actually needed

---

#### Breakpoint 2.1.C: Conditional PA Requirement
**Trigger**: PA is required IF patient meets certain criteria (e.g., only first imaging per year, only if prior attempt was negative, only if certain diagnosis codes)

**From → To**: Front-desk rule lookup → System rule + front-desk judgment

**Consequence if handled poorly**: PA is submitted when not needed (wasted effort) OR PA is not submitted when needed (service blocked)

**Agentic opportunity**: Agent flags conditional requirement and alerts front desk to check specific criterion ("Does patient have prior imaging in athenahealth? If yes, PA is needed; if no, PA is not needed")

**Risk if autonomous**: Agent lacks access to patient's prior imaging history and can't resolve the condition

---

## JTBD 2.2: Prepare and Submit PA Request to Insurer

### Purpose
Compile all required information and formally submit a PA request to the insurer in the correct format and via the correct channel.

### Actor
Front-desk staff or Dana (with human review before submission)

### Cognitive Zones

#### Zone 2.2.1: PA Information Gathering
**Micro-tasks**:
1. Retrieve from athenahealth: patient demographics, insurance plan, diagnosis code (ICD-10), service procedure code (CPT), appointment date
2. Retrieve from physician's order: clinical justification (why is imaging necessary?)
3. Cross-reference with patient history: prior imaging results, relevant lab/exam findings
4. Compile insurer-specific required fields (varies by insurer):
   - Some want ICD-10 diagnosis code only
   - Some want ICD-10 + prior authorization clinical justification (2-3 sentences)
   - Some want ICD-10 + prior imaging results to confirm medical necessity
   - Wellpath specifically wants prior visit documentation (patient has been seen for this condition, not just phone call)

**Error tolerance**: **LOW** (missing required field = PA denial + rework)

**Latency sensitivity**: **LOW** (gathering is asynchronous; takes 5-10 min)

**Data dependencies**: athenahealth (patient record, diagnosis, procedure code), physician order (justification), patient prior imaging (if applicable)

**Human expertise required**: **YES** (must understand insurer-specific requirements; knowing that Wellpath wants prior visit docs is critical tribal knowledge from Dana's experience)

**Failure modes**:
- Not including diagnosis code (insurer: "can't process without ICD-10")
- Including outdated diagnosis (patient's condition has changed, diagnosis code is no longer current)
- Not including prior imaging results for Wellpath (Wellpath automatic denial if not included)
- Including unnecessary information (makes request confusing)

---

#### Zone 2.2.2: Submission Format & Channel Selection
**Micro-tasks**:
1. Identify insurer's preferred submission method (email, fax, insurer portal, phone)
   - Some insurers have online portals (Availity, Aetna, UHC)
   - Some prefer fax or email
   - Some have phone-only (small regional plans)
2. Format request according to insurer's requirements (template, form, freeform letter)
3. Include insurer-specific fields (some want "date prior imaging," some don't; some want "requested by" provider name, some don't)
4. Double-check all fields are complete before sending
5. Submit via correct channel

**Error tolerance**: **MEDIUM** (format errors slow down insurer's processing but usually don't cause outright denial)

**Latency sensitivity**: **HIGH** (submission timing matters; submitting 1 day late means chase cycle starts 1 day late)

**Data dependencies**: Insurer submission requirements (documented in playbook or known from experience)

**Human expertise required**: **YES** (requires knowledge of insurer preferences; Wellpath prefers email, UHC prefers fax, etc.)

**Failure modes**:
- Submitting via wrong channel (insurer doesn't see it; never processed)
- Not including insurer-specific required fields (Wellpath without prior visit doc; denial)
- Submitting at wrong time relative to appointment (submit day-before-appointment is too late for 5-day SLA)

---

#### Zone 2.2.3: Submission Documentation
**Micro-tasks**:
1. Record in athenahealth: PA submitted on [date], insurer [name], submission method [email/fax/phone], status [pending]
2. Record in Dana's Google Sheet: appointment date, submission date, expected chase date, insurer-specific notes
3. Alert: mark athenahealth PA status as "PENDING" so ticker is visible to physician + front desk

**Error tolerance**: **LOW** (missing documentation = lost tracking; PA is forgotten)

**Latency sensitivity**: **LOW** (documentation is asynchronous)

**Data dependencies**: athenahealth, Google Sheet

**Human expertise required**: NONE (routine documentation)

**Failure modes**:
- Not recording submission (no audit trail; later undetectable if PA exists or not)
- Recording wrong submission date (chase timeline is off)
- Not updating athenahealth ticker (physician and front desk both think PA status is unknown)
- Recording in Google Sheet but not athenahealth (and vice versa; two systems diverge)

---

### Breakpoints

#### Breakpoint 2.2.A: PA Information Complete and Verified
**Trigger**: All required fields gathered; matches insurer's requirements; verified against current patient record

**From → To**: Front-desk preparation → Human review (before submission)

**Consequence if handled poorly**: None (expected path)

**Agentic opportunity**: Agent gathers info and flags "ready for review"; human approves before submission

**Risk if autonomous**: Agent submits without human review; missing field goes unnoticed; insurer denies

---

#### Breakpoint 2.2.B: PA Information Incomplete
**Trigger**: Missing diagnosis code, OR missing prior imaging results (if Wellpath), OR missing prior visit documentation

**From → To**: Front-desk preparation → Front-desk/Dana escalation (gather missing info)

**Consequence if handled poorly**: PA is submitted incomplete; insurer denies; rework required

**Agentic opportunity**: Agent flags missing field and alerts front desk; front desk gathers info before submission

**Risk if autonomous**: Agent decides "I'll submit now and resubmit if denied" (wastes time, delays chase cycle)

---

#### Breakpoint 2.2.C: Insurer Requires Clinical Justification
**Trigger**: Insurer PA rule specifies "requires clinical justification" or insurer is Medicaid (all Medicaid require justification)

**From → To**: Front-desk preparation → Physician or RN escalation (provide clinical justification)

**Consequence if handled poorly**: PA is submitted without justification; Medicaid insurer denies for "insufficient medical necessity documentation"

**Agentic opportunity**: Agent flags "clinical justification required" and routes to RN or physician

**Risk if autonomous**: Agent generates generic justification ("imaging is medically necessary") — insurer rejects as insufficient

---

## JTBD 2.3: Track PA Status and Chase Pending Authorizations

### Purpose
Monitor submitted PA requests for approval/denial and follow up with insurer per schedule. This is the most manual, error-prone part of the workflow (currently done by Dana via Google Sheet).

### Actor
Dana (managed workflow) + Front-desk staff (executes individual chases)

### Cognitive Zones

#### Zone 2.3.1: Insurer-Specific Chase Timeline Management
**Micro-tasks**:
1. For each submitted PA, calculate expected approval date based on insurer's SLA:
   - Aetna: 5 business days (typical; sometimes 3)
   - UHC Choice: 6-7 business days
   - UHC PPO: 3-4 business days
   - Wellpath (Medicaid): 7-10 business days
   - Medicare Advantage (varies): 5-7 business days
2. Calculate "chase date" = expected date - 1 (check status day before expected approval)
3. Calculate appointment date relative to chase date:
   - If appointment date < chase date, need expedited submission or early chase
   - If appointment date = expected approval date, high risk (approval might come after appointment)
4. Track multiple PAs in queue (Dana's Google Sheet currently does this)

**Error tolerance**: **VERY LOW** (missed chase date = PA falls through; discovered at appointment time)

**Latency sensitivity**: **CRITICAL** (chase dates are time-sensitive; if appointment is Tuesday and PA approval date is Monday, chase must happen Monday morning)

**Data dependencies**: Insurer SLA knowledge (from experience + Dana's heuristics), submission date, appointment date

**Human expertise required**: **YES** (requires knowledge of insurer-specific SLAs; Wellpath timeline is different from Aetna)

**Failure modes**:
- Using generic 5-day SLA for all insurers (Wellpath is 7+ days; miss the chase window)
- Not accounting for weekends (submit Friday, SLA clock starts Monday; chase should be next Friday, not next Thursday)
- Assuming all insurers process 24/7 (some stop processing at 5pm; submission at 4:58pm is processed next day)
- Not flagging when appointment is before expected PA approval date (high-risk cases where expedited handling is needed)

---

#### Zone 2.3.2: Chase Execution
**Micro-tasks**:
1. On chase date, query insurer (via phone, email, or Availity) to check PA status
2. Classify result: APPROVED / PENDING / DENIED
3. If APPROVED: note approval code in athenahealth + Google Sheet; mark patient's appointment as "cleared for service"
4. If PENDING: set next chase date (usually 2 days later if appointment is < 5 days away; don't re-chase if appointment is >5 days away)
5. If DENIED: retrieve denial reason; route to JTBD 2.4 (denial handling)

**Error tolerance**: **VERY LOW** (missed chase = PA status unknown at appointment)

**Latency sensitivity**: **CRITICAL** (chase timing is essential)

**Data dependencies**: Insurer contact info (phone number, email), submission date, appointment date, current date

**Human expertise required**: **YES** (soft skill: communicating with insurer, interpreting denial reasons)

**Failure modes**:
- Not chasing on schedule (Dana is busy; missed chase window; PA doesn't get approved)
- Calling insurer at wrong time (insurer's PA department not staffed; can't get answer)
- Misinterpreting PA status (insurer says "still processing"; front desk interprets as "approved")
- Not escalating when appointment is imminent and PA is still pending (should call insurer for expedited or flag patient for rescheduling)

---

#### Zone 2.3.3: Status Tracking & Visibility
**Micro-tasks**:
1. Maintain master list of all pending PAs (Dana's Google Sheet is the current system; fragile single point of failure)
2. Update list on each chase (status, next chase date, notes)
3. Ensure athenahealth ticker is updated to reflect current PA status (visible to physician + front desk)
4. Flag high-risk cases (appointment < 48 hours and PA still pending)

**Error tolerance**: **VERY LOW** (lost tracking = PA forgotten)

**Latency sensitivity**: **LOW** (tracking is asynchronous, but must be kept current)

**Data dependencies**: PA submission list, chase results, appointment date

**Human expertise required**: **CONDITIONAL**
- Routine tracking = no expertise (pure data management)
- High-risk escalation = judgment (when to escalate patient for rescheduling vs. when to chase harder)

**Failure modes**:
- Losing track of PAs (spreadsheet is fragile; if Dana is out sick, no one else knows which PAs are pending)
- Not updating athenahealth ticker (physician doesn't see PA status)
- Not flagging high-risk cases (appointment is tomorrow, PA is still pending; no one alerts front desk to reschedule)
- Duplicate entries or missed entries in spreadsheet (manual data entry is error-prone)

---

### Breakpoints

#### Breakpoint 2.3.A: PA Approved
**Trigger**: Chase query returns "APPROVED"; approval code is retrieved

**From → To**: Dana chase execution → athenahealth update complete → Appointment cleared for service

**Consequence if handled poorly**: None (expected completion)

**Agentic opportunity**: Agent auto-updates athenahealth when approval is detected; flags "PA complete, appointment ready"

**Risk if autonomous**: None; straightforward

---

#### Breakpoint 2.3.B: PA Pending at Chase Time
**Trigger**: Chase query returns "PENDING"

**From → To**: Dana chase execution → Dana decision (chase again in 2 days, or escalate for expedited)

**Consequence if handled poorly**: No re-chase scheduled; PA falls through; discovered at appointment time (Artefact 5.2 scenario)

**Agentic opportunity**: Agent auto-schedules next chase per insurer SLA; alerts Dana if appointment is imminent (<48 hours)

**Risk if autonomous**: Agent schedules next chase but doesn't account for appointment timing (chases after appointment date)

---

#### Breakpoint 2.3.C: Appointment Imminent & PA Still Pending
**Trigger**: Appointment < 48 hours AND PA status is PENDING (or unknown)

**From → To**: Automatic escalation → Dana emergency handling

**Consequence if handled poorly**: No one notices; appointment proceeds without PA or is cancelled at last minute

**Agentic opportunity**: Agent alerts Dana that appointment is in 24 hours and PA is still pending; recommend patient contact or expedited PA request

**Risk if autonomous**: Agent deletes the appointment without alerting patient (wrong action taken)

---

#### Breakpoint 2.3.D: PA Denied
**Trigger**: Chase query returns "DENIED"; denial reason is provided

**From → To**: Dana chase execution → Denial handling (JTBD 2.4)

**Consequence if handled poorly**: Denial is not processed; no action taken; appointment is not cancelled and PA-required service is not delivered (compliance risk)

**Agentic opportunity**: Agent routes denial to Dana with denial reason highlighted; suggests action (resubmit with corrected info, escalate to physician for medical necessity, or cancel appointment)

**Risk if autonomous**: Agent automatically resubmits without understanding denial reason (wastes time if denial is for medical non-coverage)

---

## JTBD 2.4: Handle PA Denials and Escalate

### Purpose
Assess PA denial reasons, determine whether the denial is recoverable (resubmit with corrected info) or non-recoverable (medical non-coverage, patient ineligible), and take appropriate action.

### Actor
Dana + Front-desk staff (execution) + Physician (for medical necessity questions)

### Cognitive Zones

#### Zone 2.4.1: Denial Reason Classification
**Micro-tasks**:
1. Retrieve full denial reason from insurer (email, fax, phone call transcript)
2. Classify denial into categories:
   - **Missing information** (e.g., "ICD-10 code not provided," "prior visit documentation required") → Recoverable
   - **Procedural issue** (e.g., "submitted to wrong department") → Recoverable
   - **Medical necessity** (e.g., "imaging not medically necessary per our guidelines") → May be recoverable (appeals process)
   - **Coverage issue** (e.g., "patient is not covered," "deductible not met") → Non-recoverable
   - **Prior authorization already approved** (e.g., "PA was already approved on [date]") → Likely misunderstanding; investigate
   - **Insurer-specific quirk** (e.g., Wellpath: "missing prior visit documentation") → Recoverable; predictable per insurer pattern

**Error tolerance**: **LOW** (misclassifying denial = wrong action taken)

**Latency sensitivity**: **HIGH** (if appointment is soon, need quick decision to resubmit vs. reschedule)

**Data dependencies**: Denial reason (from insurer), insurer playbook (known denial patterns), patient coverage + prior history

**Human expertise required**: **YES** (must understand insurer denial patterns; knowing Wellpath always denies first time on certain services is critical)

**Failure modes**:
- Assuming all denials are recoverable (some are final; no point resubmitting)
- Not recognizing Wellpath "missing prior visit doc" pattern (Dana knows this; newer staff don't; missed resubmit opportunity)
- Misinterpreting "medical necessity" denial as final (may be appealable with more documentation)
- Not escalating "coverage" denials (patient is actually not covered; appointment should be cancelled or patient informed of cost)

---

#### Zone 2.4.2: Resubmission Assessment
**Micro-tasks**:
1. For recoverable denials, determine what information is missing or incorrect
2. Gather corrected/missing information:
   - If "ICD-10 required": retrieve correct code from physician or athenahealth
   - If "prior visit documentation required" (Wellpath): retrieve prior visit note from athenahealth
   - If "prior imaging results required": retrieve imaging report from patient's prior appointment
3. Prepare corrected PA request with all required fields
4. Determine resubmission timing:
   - If appointment is >5 days away: resubmit immediately; expect approval within SLA; chase per normal timeline
   - If appointment is 2-5 days away: resubmit + request expedited review
   - If appointment is <2 days away: escalate (likely need to reschedule appointment)

**Error tolerance**: **LOW** (incorrect resubmission = second denial + further delay)

**Latency sensitivity**: **CRITICAL** (timing determines whether appointment can proceed)

**Data dependencies**: Denial reason, missing information (from various systems), appointment date

**Human expertise required**: **YES** (must know what Wellpath needs, what other insurers need; must prioritize resubmit timing vs. rescheduling)

**Failure modes**:
- Resubmitting generic information without addressing specific denial reason (insurer denies again for same reason)
- Not requesting expedited review when appointment is imminent (normal SLA will miss appointment)
- Not escalating when resubmit timing is too tight; proceeding with resubmit when reschedule is only option

---

#### Zone 2.4.3: Escalation & Patient Notification
**Micro-tasks**:
1. For non-recoverable denials (coverage lapsed, patient ineligible, service not covered):
   - Alert physician (appointment cannot proceed as planned)
   - Alert front desk to contact patient
   - Options: reschedule appointment, convert to self-pay (if patient wants to proceed), cancel appointment
2. For recoverable denials with tight timing (<2 days):
   - Alert patient that PA is pending (delayed approval; appointment might be rescheduled)
   - Give patient option: wait for approval or reschedule
3. For appealable denials (medical necessity):
   - Escalate to physician for decision (is this worth appealing?)
   - If yes, prepare appeal letter with clinical justification

**Error tolerance**: **LOW** (not notifying patient = patient shows up and appointment is cancelled)

**Latency sensitivity**: **CRITICAL** (notification must be same-day to give patient time to reschedule)

**Data dependencies**: Denial reason, patient contact info, appointment date

**Human expertise required**: **YES** (soft skill: delivering bad news to patient, assessing whether appeal is worth pursuing)

**Failure modes**:
- Not contacting patient until day-of-appointment (too late to reschedule)
- Not explaining denial reason to patient (patient feels blindsided)
- Pursuing appeals that are unlikely to succeed (wasting time when reschedule is the practical option)

---

### Breakpoints

#### Breakpoint 2.4.A: Denial Is Recoverable; Information Is Available
**Trigger**: Denial reason is "missing ICD-10" or "missing prior visit doc" AND Dana has the missing info immediately available

**From → To**: Dana classification → Immediate resubmission

**Consequence if handled poorly**: None (expected path)

**Agentic opportunity**: Agent auto-detects missing-info denials; suggests information to include; prepares resubmit

**Risk if autonomous**: Agent resubmits without ensuring information is correct (second denial)

---

#### Breakpoint 2.4.B: Denial Is Recoverable; Information Needs to be Gathered
**Trigger**: Denial reason is "missing prior imaging results" or "needs clinical justification from physician"

**From → To**: Dana classification → Escalation to physician or prior records team

**Consequence if handled poorly**: Information gathering takes days; appointment is missed; reschedule is only option

**Agentic opportunity**: Agent flags required information and routes to relevant person (physician for justification, medical records for prior imaging)

**Risk if autonomous**: Agent delays escalation thinking information can be found; appointment passes; too late

---

#### Breakpoint 2.4.C: Denial Is Non-Recoverable
**Trigger**: Denial reason is "coverage lapsed," "patient is not eligible," or "service is not covered"

**From → To**: Dana classification → Patient notification + Appointment decision

**Consequence if handled poorly**: Patient is not told; shows up for appointment; told at appointment that PA is denied; patient is angry

**Agentic opportunity**: Agent alerts Dana immediately; Dana contacts patient same-day with options (reschedule, self-pay, cancel)

**Risk if autonomous**: Agent cancels appointment without patient consent

---

#### Breakpoint 2.4.D: Appointment Is Imminent; Denial Cannot Be Resolved in Time
**Trigger**: Appointment is <2 days away AND denial is recoverable but gathering information will take >24 hours

**From → To**: Dana classification + timing assessment → Forced rescheduling

**Consequence if handled poorly**: Front desk proceeds with appointment anyway (PA is still not approved); appointment is blocked at technician

**Agentic opportunity**: Agent alerts Dana that appointment must be rescheduled (no time to fix PA); suggests new date to patient

**Risk if autonomous**: Agent reschedules without patient consent or physician awareness

---

---

# WORK STREAM 3: MEDICATION RECONCILIATION (Summary-Level Decomposition)

## Overview
- **Volume**: ~180 patients/day; most are routine; ~10-15% have discrepancies needing escalation
- **Handling time**: ~6 min/case (pre-visit questionnaire + DoseSpot check + allergy confirmation)
- **Primary cognitive outcome**: Confirmation that patient's medication list is complete and accurate, allergy data is current, and no dangerous drug interactions exist
- **Key systems**: athenahealth (EHR), DoseSpot (pharmacy integration), patient portal/paper forms
- **Key data**: patient's self-reported medications, DoseSpot pharmacy history, patient's documented allergies

---

## JTBD 3.1: Pre-Visit Medication Questionnaire and OTC Capture

**Purpose**: Elicit complete medication list from patient, including OTC and supplements, to ensure comprehensive reconciliation.

**Actor**: Front-desk staff (administration) + Patient (reporting)

**Cognitive Zones**:
- **Zone 3.1.1: Questionnaire Administration** — Ask patient for all medications, OTC, supplements, herbal products
  - Error tolerance: LOW (missed OTC = drug interaction not caught)
  - Latency sensitivity: LOW (can be done pre-visit)
  - Human expertise required: CONDITIONAL (standard questions vs. probing for patient's memory gaps)

- **Zone 3.1.2: OTC/Supplement-Specific Questioning** — Explicitly ask about ibuprofen, aspirin, acetaminophen, vitamins, herbal supplements
  - Error tolerance: LOW (OTC interactions are clinically significant)
  - Latency sensitivity: LOW
  - Human expertise required: YES (must know which OTC are most commonly taken and most likely to interact)

- **Zone 3.1.3: Patient Memory Validation** — Cross-check patient's stated meds against EHR history ("You said you take Metformin; I see you've been on it since 2022. Still taking it?")
  - Error tolerance: MEDIUM (patient forgets they stopped taking something; can be corrected at visit)
  - Latency sensitivity: LOW
  - Human expertise required: NO (routine cross-check)

**Breakpoints**:
- Breakpoint 3.1.A: Patient cannot remember all medications → Escalate to "bring med bottles to appointment"
- Breakpoint 3.1.B: Patient admits they stopped taking a med listed in EHR → Update EHR before visit

---

## JTBD 3.2: DoseSpot Reconciliation and Interaction Checking

**Purpose**: Compare patient's self-reported medications to pharmacy records (DoseSpot) and check for drug-drug interactions, drug-allergy conflicts.

**Actor**: System (DoseSpot) + Front-desk staff (escalation)

**Cognitive Zones**:
- **Zone 3.2.1: Pharmacy History Reconciliation** — Compare DoseSpot list to patient's stated list
  - Micro-tasks: (1) Query DoseSpot for patient's prescription history (last 90 days), (2) Compare to patient's stated list, (3) Identify discrepancies (patient on Metformin in DoseSpot but didn't mention it; patient mentioned statin but DoseSpot shows it filled 6 months ago)
  - Error tolerance: LOW (missed med = incomplete reconciliation)
  - Human expertise required: YES (must interpret discrepancies: is patient non-adherent, or did patient stop and forget to tell us?)

- **Zone 3.2.2: Drug-Drug Interaction Detection** — Review DoseSpot's automatic interaction alerts
  - Micro-tasks: (1) Review alerts generated by DoseSpot, (2) Assess clinical significance (is this a serious interaction or minor?), (3) Escalate if significant
  - Error tolerance: VERY LOW (missed serious interaction = patient harm)
  - Latency sensitivity: MEDIUM (should be checked before visit but not real-time urgent)
  - Human expertise required: YES (clinical knowledge of which interactions are serious)

- **Zone 3.2.3: Drug-Allergy Conflict Detection** — Cross-check patient's current medications against documented allergies
  - Micro-tasks: (1) Identify all documented allergies in EHR, (2) Cross-reference against patient's current meds (is patient on a penicillin despite penicillin allergy?), (3) Escalate if conflict
  - Error tolerance: VERY LOW (drug-allergy conflict = potential anaphylaxis)
  - Latency sensitivity: HIGH (this is safety-critical)
  - Human expertise required: YES (clinical knowledge: is this allergy real anaphylaxis or intolerance?)

**Breakpoints**:
- Breakpoint 3.2.A: Serious interaction detected (e.g., warfarin + ibuprofen) → Escalate to physician before visit
- Breakpoint 3.2.B: Drug-allergy conflict detected → Escalate to physician immediately; may require medication change
- Breakpoint 3.2.C: DoseSpot shows patient on med not mentioned in questionnaire → Escalate to front desk for patient confirmation

---

## JTBD 3.3: Allergy Data Verification and Refresh

**Purpose**: Confirm that documented allergies are current and accurate; remove outdated allergy flags that may be blocking appropriate medication use.

**Actor**: Front-desk staff (questioning) + Physician (judgment)

**Cognitive Zones**:
- **Zone 3.3.1: Allergy Severity Assessment** — Confirm severity of documented allergies
  - Micro-tasks: (1) Review allergy record (e.g., "Penicillin ALLERGY" documented 15 years ago), (2) Ask patient about actual reaction (rash, swelling, anaphylaxis?), (3) Update severity code if needed
  - Error tolerance: VERY LOW (misclassifying anaphylaxis as mild rash could lead to serious reaction)
  - Latency sensitivity: LOW (can be done pre-visit)
  - Human expertise required: YES (clinical knowledge of allergic reactions)

- **Zone 3.3.2: Allergy Currency Assessment** — Determine if old allergy is still relevant
  - Micro-tasks: (1) Check allergy entry date (e.g., 2010), (2) Ask patient if reaction still occurs or if it's resolved, (3) Decide: keep allergy flag, remove flag, or modify
  - Error tolerance: VERY LOW (removing valid allergy = patient risk; keeping obsolete allergy = unnecessary drug avoidance)
  - Latency sensitivity: LOW
  - Human expertise required: YES (must understand that allergies can resolve over time)

- **Zone 3.3.3: New Allergy Capture** — Ask patient if any new allergies developed since last visit
  - Micro-tasks: (1) Ask "Any new medication allergies or reactions since we last saw you?", (2) If yes, get details (drug name, reaction type, severity), (3) Enter into EHR
  - Error tolerance: VERY LOW (missed new allergy = patient could be prescribed that drug)
  - Latency sensitivity: MEDIUM (new allergy should be flagged immediately)
  - Human expertise required: CONDITIONAL (routine capture vs. probing for severity if reported)

**Breakpoints**:
- Breakpoint 3.3.A: Allergy is confirmed as still current → Keep flag; ensure physician is aware
- Breakpoint 3.3.B: Allergy is outdated (patient says "I haven't had a reaction in 20 years") → Escalate to physician for decision to remove flag
- Breakpoint 3.3.C: New allergy reported → Enter into EHR immediately; flag for physician review before any prescribing

---

---

# WORK STREAM 4: VISIT-REASON TRIAGE (Summary-Level Decomposition)

## Overview
- **Volume**: ~180 patients/day
- **Handling time**: ~4 min/case (triage classification + prep instructions confirmation)
- **Primary cognitive outcome**: Classification of visit into appointment type (routine, urgent, procedure, preventive) and routing to appropriate scheduling and prep
- **Key systems**: athenahealth (EHR), scheduling system, pre-visit questionnaire
- **Key data**: Patient's stated reason, prior visit history, physician standing orders

---

## JTBD 4.1: Classify Visit Reason and Assess Urgency

**Purpose**: Determine the correct appointment type and urgency level based on patient's stated reason and clinical context.

**Actor**: Front-desk staff (with physician escalation for urgent cases)

**Cognitive Zones**:
- **Zone 4.1.1: Chief Complaint Parsing** — Extract patient's stated reason for visit
  - Micro-tasks: (1) Patient states reason (via phone, portal, paper form), (2) Classify into category (routine follow-up, acute complaint, preventive, procedure, referral), (3) Assess clarity (is reason clear or vague?)
  - Error tolerance: MEDIUM (misclassification delays appointment; can be corrected at check-in)
  - Latency sensitivity: MEDIUM (classification affects scheduling)
  - Human expertise required: CONDITIONAL (routine classifications vs. acute screening)

- **Zone 4.1.2: Urgent/Acute Signal Detection** — Identify if patient's reason suggests urgent or emergent care
  - Micro-tasks: (1) Listen for red-flag keywords (chest pain, difficulty breathing, severe injury, uncontrolled bleeding, neurological symptoms), (2) Assess acuity (how long has symptom been present? worsening?), (3) Escalate to physician if emergency signal detected
  - Error tolerance: VERY LOW (missed emergency = patient harm)
  - Latency sensitivity: CRITICAL (immediate escalation required)
  - Human expertise required: YES (must recognize medical emergencies)

- **Zone 4.1.3: Context Assessment** — Cross-check stated reason against patient's EHR history
  - Micro-tasks: (1) Review patient's recent visit notes (what was patient seen for last time?), (2) Review chronic problems (is this visit for routine DM2 management or new issue?), (3) Assess consistency (does stated reason align with patient's known history?)
  - Error tolerance: MEDIUM (context mismatch can be clarified at appointment)
  - Latency sensitivity: MEDIUM
  - Human expertise required: CONDITIONAL (routine cross-check vs. detecting clinical inconsistencies)

**Breakpoints**:
- Breakpoint 4.1.A: Routine follow-up (e.g., diabetes 3-month follow-up) → Standard appointment slot (20 min)
- Breakpoint 4.1.B: Acute complaint (e.g., cough, ear pain) → Urgent slot (same-day or next available urgent slot)
- Breakpoint 4.1.C: Emergency signal (e.g., chest pain, difficulty breathing) → Immediate physician alert; possible ED referral
- Breakpoint 4.1.D: Vague reason ("checkup," "follow-up without context) → Front desk clarifies with patient before scheduling

---

## JTBD 4.2: Confirm Procedure Prep and Special Scheduling

**Purpose**: Ensure patient understands pre-procedure preparation requirements and appointment scheduling reflects procedure type.

**Actor**: Front-desk staff (execution) + Physician standing orders (rules)

**Cognitive Zones**:
- **Zone 4.2.1: Procedure Identification** — Determine if visit is procedure-related
  - Micro-tasks: (1) Check physician order or reason for visit, (2) Identify procedure type (if applicable), (3) Look up standard prep requirements
  - Error tolerance: MEDIUM (missed prep = visit rescheduled at appointment time)
  - Latency sensitivity: MEDIUM (prep needs to be communicated before appointment)
  - Human expertise required: CONDITIONAL (routine procedures vs. complex procedures with variable prep)

- **Zone 4.2.2: Prep Instruction Communication** — Inform patient of pre-procedure requirements
  - Micro-tasks: (1) Retrieve standard prep for procedure type (e.g., "NPO 4 hours before wart removal," "avoid aspirin x5 days"), (2) Communicate to patient verbally and in writing, (3) Confirm patient understands ("Do you have any questions?")
  - Error tolerance: LOW (patient arrives unprepared; procedure delayed/cancelled)
  - Latency sensitivity: HIGH (patient needs 24-48 hours notice for most prep)
  - Human expertise required: YES (soft skill: explaining prep in patient-friendly language)

- **Zone 4.2.3: Special Scheduling** — Allocate appropriate appointment time and staffing for procedure
  - Micro-tasks: (1) Check procedure type, (2) Allocate time slot per standing orders (e.g., wart removal = 30 min, not standard 20 min), (3) Alert physician and nursing staff
  - Error tolerance: MEDIUM (under-allocating time leads to schedule pressure but can be recovered)
  - Latency sensitivity: MEDIUM (scheduling needs to reflect time)
  - Human expertise required: CONDITIONAL (standard procedures vs. complex/variable procedures)

**Breakpoints**:
- Breakpoint 4.2.A: Procedure prep confirmed; patient understands → Appointment scheduled with appropriate time and staff alert
- Breakpoint 4.2.B: Patient reports they cannot comply with prep (e.g., "I can't stop aspirin; I have a clot risk") → Escalate to physician for decision
- Breakpoint 4.2.C: Procedure requires imaging prep (e.g., ultrasound needs full bladder) → Alert patient of specific requirements

---

---

## Summary Statistics

| Metric | Count |
|---|---|
| **Total JTBDs (Detailed)** | 6 (Insurance × 3, Prior Auth × 3) |
| **Total JTBDs (Summary)** | 4 (Meds × 3, Triage × 1) |
| **Total Cognitive Zones** | 18+ zones across all JTBDs |
| **Total Breakpoints** | 15+ breakpoints identified |
| **Work Streams Detailed** | 2 (Insurance, Prior Auth) |
| **Work Streams Summary** | 2 (Meds, Triage) |
| **Friction Points Named** | 6 primary friction points |
| **Lived-vs-Documented Gaps** | 4 major gaps identified |

---

## Key Patterns Across the Map

### Pattern 1: Manual Systems Create Single Points of Failure
- Insurance verification: Availity is single point of failure (timeout = manual fallback)
- Prior Authorization: Dana's Google Sheet is single point of failure (if Dana is out sick, no PA tracking)

### Pattern 2: Judgment Is Hidden in "Simple" Tasks
- Insurance verification looks like "query and return result"; actually includes judgment about data freshness and coverage changes
- PA requirement determination looks routine; actually requires knowledge of insurer-specific rules

### Pattern 3: Data Lag Creates Discrepancies
- Medicaid eligibility is 24-48 hours behind (state batch updates)
- Specialist prescriptions are missing from primary care EHR
- Allergy data is stale and not re-verified

### Pattern 4: Cascades Are Real and Costly
- PA missed → visit scheduled → appointment time arrives → PA discovered missing → appointment cancelled
- Insurance verification incomplete → cannot submit PA → visit cannot proceed

### Pattern 5: Communication Gaps Are High-Risk
- Patient is not told about PA denial → patient shows up expecting procedure → told at appointment "no PA"
- Procedure prep not confirmed → patient arrives unprepared → procedure rescheduled

---

## Map Validation Checklist

✅ **CLM includes lived-vs-documented notes** (Section 1)
✅ **CLM identifies primary friction points** (Section 2)
✅ **Each JTBD has ≥2 CognitiveZones with micro-tasks** (Sections 3-6)
✅ **Each JTBD has ≥1 Breakpoint** (Sections 3-6)
✅ **Error tolerance is justified for each zone** (stated explicitly in each zone)
✅ **Human expertise requirements are specified** (stated for each zone)
✅ **Failure modes are documented** (within zones and in breakpoints)
✅ **Agentic opportunities are annotated** (in breakpoints and in Domain Model)
✅ **All four work streams are included** (2 detailed, 2 summary)

---
