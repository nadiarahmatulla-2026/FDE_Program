# CLAUDE.md — Week 2 ATX Workflow: Scenario 5 — Small-Clinic Patient Intake

## Project Purpose

This project assists FDE participants in executing **Agentic Transformation (ATX)** on Scenario 5 — a 6-physician family medicine practice (Westbridge Family Medicine, US mid-Atlantic suburb) with patient intake work distributed across a 4-person front-desk team.

The participant is applying ATX methodology to assess **how patient intake work actually happens, where it breaks down, what cognitive decisions drive the process, and where delegation to an agentic system creates value without introducing unacceptable risk**.

You are not building the agent directly. You are building the **cognitive and governance artefacts** that will enable the practice to design agents safely: a cognitive load map grounded in lived intake practices, honest assessment of which tasks can be delegated and to what degree, discovery of hidden assumptions and integration challenges, and transparent escalation / autonomy boundaries.

This is one participant's deliverable for one week. The workflow runs:
1. **Monday–Thursday**: Apply ATX to this scenario; produce cognitive load map, delegation suitability matrix, volume × value analysis, and agent purpose document
2. **Thursday 14:15 CET**: Submit artefacts for peer cross-review
3. **Friday**: Timed gate exercise under seal (same ATX methodology, new previously-unseen scenario)

---

## Scenario Context

### The Practice: Westbridge Family Medicine

- **Location**: US mid-Atlantic suburb, two locations 12 miles apart
- **Staffing**: 6 physicians, ~180 patients/day, mix of routine visits, chronic-disease management, urgent same-day, pre-procedure screenings
- **Front-desk intake team**: 4 people total (typically 2 per location; cross-site rotation when one is short-staffed)
- **Practice Manager**: Dana Velazquez, RN-trained, 11 years at Westbridge, oversees the function

### The Four Work Streams

1. **Insurance verification** (~180/day; ~3 min/case if auto-verified; ~5 min/case for ~30% that fail auto-verify)
2. **Prior-authorisation check** (~25/day; ~12 min/case for scheduled procedures, imaging, specialty referrals)
3. **Pre-visit questionnaire & visit-reason triage** (~180/day; ~4 min/case for routine vs urgent vs same-day classification)
4. **Medication reconciliation & allergy-flag review** (~180/day; ~6 min/case for pharmacy history, allergy alerts, change flagging)

### Tooling in Place

- **athenahealth** (EHR, modern SaaS, REST APIs available)
- **Availity** (insurance eligibility checks, separate REST-API tool)
- **DoseSpot** (pharmacy/medication reconciliation, integrated with athenahealth)
- **Phone + paper intake forms** (for patients without portal accounts)
- **Google Sheets** (Dana's PA chase list and flagged-patient tracker — not in any formal system)

### Hard Constraints — Non-Negotiable

1. **No clinical judgment by the agent.** The agent does not interpret symptoms, advise on visit urgency, or make medical decisions.
2. **Clear human escalation for stated visit reason.** Any contact with the patient's stated reason for visit must preserve a clear path back to a human.
3. **HIPAA and state medical-records compliance.** Any agent design must meet federal privacy requirements and state medical-record rules.

### The Stakeholder

**Dana Velazquez**, Practice Manager, RN background, 11 years at Westbridge. Recently, the senior physician discovered three intake misses in the last quarter (expired prior auths) and asked Dana to "look at this AI thing the medical society keeps emailing about." Dana's role is to translate front-desk intake challenges into a requirements statement.

---

## Core Entities: Scenario-Specific

### Work Streams Decomposed

Each of the four work streams has distinct cognitive zones and failure modes, not all of which are documented in the SOP.

#### WorkStream 1: Insurance Verification
- **Volume**: ~180/day
- **Typical handling**: 3 min automated; 5 min for the ~30% that fail auto-verify
- **Cognitive outcome**: Determination of whether the patient's stated insurance is active, valid, and accepted by the practice; capture of co-pay/deductible if needed
- **Systems touched**: athenahealth, Availity, patient phone/portal
- **is_decomposed**: true (detail in CLM)

#### WorkStream 2: Prior-Authorisation Check
- **Volume**: ~25/day (scheduled procedures, imaging, specialty referrals)
- **Typical handling**: ~12 min/case, often split across multiple sessions (initial check + chase if pending)
- **Cognitive outcome**: Determination of whether a named procedure/referral needs pre-auth; confirmation that auth exists and is valid; identification of missing/pending auths that need escalation
- **Systems touched**: athenahealth, Availity, insurer portals (manual), Dana's PA chase list
- **Failure mode**: Expired or missing PAs discovered at visit time; patient arrives expecting procedure but can't proceed
- **is_decomposed**: true (detail in CLM)

#### WorkStream 3: Pre-Visit Questionnaire & Visit-Reason Triage
- **Volume**: ~180/day
- **Typical handling**: ~4 min/case
- **Cognitive outcome**: Routing of patient to correct scheduling bucket (routine = scheduled slot; urgent = same-day; emergency = immediate physician alert)
- **Systems touched**: athenahealth portal, paper forms, phone
- **Failure mode**: Misrouting (urgent case treated as routine, or vice versa); missed emergency flags
- **is_decomposed**: true (detail in CLM)

#### WorkStream 4: Medication Reconciliation & Allergy-Flag Review
- **Volume**: ~180/day
- **Typical handling**: ~6 min/case; triggered on every visit
- **Cognitive outcome**: Confirmation that pharmacy history in DoseSpot matches patient's stated medication list; identification of new/missing/discontinued medications; flag for allergy changes
- **Systems touched**: DoseSpot, athenahealth, patient self-report
- **Failure mode**: Reconciliation mismatch not caught; allergy update missed; duplicate/contraindicated meds not flagged
- **is_decomposed**: true (detail in CLM)

### Jobs to be Done (JTBD) — Scenario-Specific Hypothesis

These are provisional; will be refined through coach interaction and artefact review.

#### JTBD 1: Determine Insurance Active Status & Eligibility
- **Actor**: Front-desk staff (currently) or agent (candidate)
- **Cognitive zones**:
  - **Insurance lookup**: query Availity for active coverage; interpret response
  - **Co-pay/deductible capture**: extract and record patient's out-of-pocket responsibility
  - **Coverage gap handling**: recognize when patient has no insurance or coverage has lapsed
  - **Follow-up triage**: route to Dana if coverage unclear or patient disputes
- **Breakpoints**:
  - Coverage absent or lapsed → escalate to Dana for patient conversation
  - Multiple insurance policies listed → human judgment on primary/secondary
  - Coverage active but patient disputes eligibility → physician or Dana decision

#### JTBD 2: Confirm Prior-Authorization Status for Scheduled Procedures
- **Actor**: Front-desk staff (currently, for initial check); Dana (for chase/escalation)
- **Cognitive zones**:
  - **PA requirement determination**: identify whether the planned procedure requires pre-auth per insurer rules
  - **PA lookup**: query Availity or insurer portal for existing auth
  - **PA validity check**: confirm auth is active (not expired, not for wrong procedure/provider)
  - **PA missing/pending triage**: if missing/pending, determine chase timing and escalation
- **Breakpoints**:
  - PA required but missing → escalate to Dana for insurer contact
  - PA expired but visit imminent → escalate to Dana for emergency resubmission
  - PA missing and visit date > 5 days → route to Dana chase queue

#### JTBD 3: Classify Visit Urgency & Route to Scheduling
- **Actor**: Front-desk staff (currently, with discretion)
- **Cognitive zones**:
  - **Chief complaint parsing**: extract patient's stated reason for visit
  - **Urgency signal detection**: identify keywords or patterns that signal urgent/emergency (chest pain, difficulty breathing, severe injury, etc.)
  - **Triage routing decision**: routine vs urgent vs immediate-physician-alert
  - **Patient communication**: inform patient of routing decision and expected timeframe
- **Breakpoints**:
  - Ambiguous chief complaint → escalate to RN triage line for phone assessment
  - Potential emergency signal → physician alert (not agent decision)
  - Patient describes existing chronic condition check-up → routine routing

#### JTBD 4: Reconcile Medication List & Check for Allergy Updates
- **Actor**: Front-desk staff or nurse (currently)
- **Cognitive zones**:
  - **Pharmacy history query**: pull patient's current medication list from DoseSpot
  - **Patient-stated medications review**: compare DoseSpot list to what patient says they're taking
  - **New/missing/discontinued identification**: flag discrepancies
  - **Allergy change detection**: prompt patient about allergy changes; check for new contraindications
  - **Escalation triage**: route to pharmacist or nurse if discrepancy is significant
- **Breakpoints**:
  - Patient on two competing medications → pharmacist review before visit
  - New allergy reported → escalate to physician before visit
  - Reconciliation discrepancy that affects treatment → route to nurse

### Cognitive Zones — High-Confidence List

These zones emerge from the artefacts and scenario brief:

| Zone | Stream | Name | Micro-tasks | Error tolerance | Latency sensitivity | Human expertise |
|---|---|---|---|---|---|---|
| Z1.1 | Insurance | Query & interpret Availity response | Call Availity API; parse JSON response; map response to "active", "pending", "lapsed", "coverage gap" | Zero (wrong eligibility determination → wrong billing + patient frustration) | Medium (not time-critical but same-day ideally) | None (rule-based) |
| Z1.2 | Insurance | Co-pay extraction & recording | Capture copay amount, deductible amount, record in athenahealth | Low (wrong amount → rework at billing stage) | Low | None |
| Z1.3 | Insurance | Coverage gap triage | Recognize when Availity response is "no coverage" or "lapsed"; escalate to Dana for patient conversation | Zero (wrong escalation → uninsured patient treated without pre-conversation) | High (patient on phone) | RN or practice manager expertise (conversation judgment) |
| Z2.1 | PA check | PA requirement lookup | Query insurer rules for this procedure/CPT code; determine if pre-auth is required | Low (if missed, agent recommends unnecessary auth → wasted phone call) | Low | None (rule-based per insurer) |
| Z2.2 | PA check | PA existence check | Query Availity or insurer portal for existing auth for this patient/procedure | Low | Medium (time-sensitive if PA is expired/pending) | None (lookup) |
| Z2.3 | PA check | PA validity assessment | Compare auth date, procedure code, provider ID to scheduled visit; determine if still valid | Zero (wrong validity → visit blocked at last minute) | High (visit is imminent) | RN or physician (medical judgment on procedure match) |
| Z2.4 | PA check | Missing/pending PA escalation | If PA missing or pending, determine urgency and escalation path (Dana resubmit vs manual insurer call vs patient notification) | Low | Critical (if missed, visit doesn't happen) | RN or practice manager (business judgment on timing) |
| Z3.1 | Visit reason | Chief complaint parsing | Extract patient's stated reason from questionnaire or phone call; standardize to recognizable category | Medium (minor misclassification → suboptimal scheduling; if emergency signals missed → high error tolerance) | Medium | Nurse or physician (triage expertise) |
| Z3.2 | Visit reason | Urgency signal detection | Identify red-flag keywords/patterns (chest pain, difficulty breathing, severe injury, new neurological symptoms, active bleeding, etc.) | Zero (missed urgent signal → patient harm) | Critical (real-time detection) | Nurse or physician (medical judgment) |
| Z3.3 | Visit reason | Routine vs urgent routing | Classify visit into scheduling bucket; assign slot in appropriate time window | Medium (misrouting → patient dissatisfaction or scheduling mismatch) | Medium | Nurse or physician |
| Z4.1 | Meds | Pharmacy history retrieval | Query DoseSpot for patient's current medication list | Low (incomplete query → missing meds at reconciliation time) | Low | None |
| Z4.2 | Meds | Patient-stated vs DoseSpot reconciliation | Compare patient's self-reported meds to DoseSpot list; identify discrepancies | Medium (minor discrepancy missed → noted at visit; major discrepancy missed → potential harm) | Low | Pharmacist or nurse (clinical judgment) |
| Z4.3 | Meds | Allergy update capture | Ask patient about allergy changes; check for contraindications with current meds | Zero (missed new allergy → potential adverse event) | Medium | Pharmacist or nurse (clinical judgment) |

---

## Breakpoints — Scenario-Specific Hypothesis

| Name | Trigger | From → To | Consequence if handled poorly | Agentic opportunity | Risk if autonomous |
|---|---|---|---|---|---|---|
| Insurance not active | Availity query returns "lapsed" or "no coverage" | System → Human (Dana) | Patient on hook for bill; practice loses revenue; patient frustrated | Medium (agent can flag, but Dana must do the conversation) | Agent misclassifies coverage status → billing disaster |
| PA missing at visit | Visit date is < 5 days and PA not found | System → Human (Dana) | Visit blocked at last minute; patient frustration; rescheduling overhead | High (agent can detect and escalate; Dana can decide urgency of resubmission) | Agent flags as "missing" but PA was actually auto-approved and agent didn't see it |
| Emergency urgency signal | Patient reports chest pain, difficulty breathing, active bleeding | System → Physician | Delay → patient harm or death | High (agent should alert immediately) | Agent misses signal or over-alerts (boy who cried wolf) |
| Medication contraindication detected | DoseSpot shows patient on two competing meds; new allergy reported | System → Pharmacist/Nurse | Drug interaction or allergy reaction; patient harm | High (agent detects, escalates; nurse reviews and advises) | Agent misidentifies contraindication; patient takes harmful combo |
| Coverage dispute | Patient says insurance is active, Availity says lapsed | System → Dana | Billing confusion; patient frustration | Medium (agent flags discrepancy; Dana calls insurer or patient) | Agent accepts Availity as truth → patient charged despite having valid coverage elsewhere |
| PA validity unclear | Scheduled procedure doesn't exactly match PA procedure code | System → Nurse/Physician | Visit blocked or wrong procedure coded; compliance risk | Medium (agent flags mismatch; nurse or physician decides if it's material) | Agent auto-accepts mismatched PA → wrong procedure done |

---

## Volume & Value Analysis Hypothesis

This is provisional and will be refined through coach interaction.

### Streams Plotted

| Stream | Volume estimate | Value estimate | Rationale |
|---|---|---|---|
| Insurance verification | High (~180/day = ~46K/yr) | High | Every patient encounter; cost of failure = billing rework + patient churn |
| Prior-auth check | Low–Medium (~25/day = ~6.5K/yr) | Very High | Lower volume but critical path: visit can't happen without valid PA; cost of failure = lost visit revenue + patient frustration |
| Visit-reason triage | High (~180/day) | High | Every patient; cost of failure = misrouted visit + staff overtime or patient harm |
| Medication reconciliation | High (~180/day) | High | Every patient; cost of failure = drug interaction or allergy miss = patient harm |

### Primary Target Hypothesis

**Prior-Authorisation Check** is likely the primary target because:
- **High value, low volume**: 25 cases/day × 12 min/case = 5 hours/day of Dana's time on chase and escalation
- **High friction**: Artefact 5.1 shows Dana maintaining a manual chase list with insurer-specific SLAs (Aetna 5 days, UHC Choice 6+ days, Wellpath always denies first time)
- **High risk if missed**: Artefact 5.2 shows physician complaint about visit cancelled because PA was pending; patient frustrated ("second time this has happened")
- **Discrete workflow**: PA checking is rule-based and insurer-specific, not medical judgment
- **Integration opportunity**: Availity API + DoseSpot + athenahealth means agent can query, track, and escalate systematically

**Secondary targets**:
- Insurance verification (high volume, moderate friction)
- Medication reconciliation (high volume, high risk, but more clinical judgment required)

**Deprioritised**:
- Visit-reason triage (high volume but requires clinical judgment; urgent/emergency signals must stay human; low agentic value unless agent is purely administrative)

---

## Delegation Archetypes — Scenario-Specific

### Five Archetypes

1. **HUMAN_ONLY**: A human always decides and acts.
2. **HUMAN_LED_AUTOMATION_SUPPORT**: Human decides; automation assists (e.g., form-filling, data lookup).
3. **HUMAN_LED_AGENT_SUPPORT**: Human leads; agent proposes decisions and handles routine cases; human escalates exceptions.
4. **AGENT_LED_OVERSIGHT**: Agent decides and acts; human reviews flagged cases post-hoc.
5. **FULLY_AGENTIC**: Agent decides and acts; no human oversight.

### Hypothesis: Which Zones Can Be Delegated, and to What Extent?

| Zone | Current actor | Candidate archetype | Rationale |
|---|---|---|---|
| Z1.1 Query & interpret Availity | Front-desk staff + Dana (on exception) | HUMAN_LED_AGENT_SUPPORT | Agent queries Availity, interprets standard responses (active, pending, lapsed). Human (Dana) escalates coverage gaps for patient conversation. |
| Z1.2 Co-pay extraction | Front-desk staff | HUMAN_LED_AUTOMATION_SUPPORT or FULLY_AGENTIC | Extraction is rule-based; no judgment. Agent can extract and record in athenahealth. Human spot-checks periodically. |
| Z1.3 Coverage gap triage | Dana | HUMAN_ONLY (for now) | Requires conversation with patient; decision-making on next steps (alternative coverage, payment plan, etc.). Not suitable for agent. |
| Z2.1 PA requirement lookup | Front-desk staff | FULLY_AGENTIC | Rule-based per insurer/procedure. Agent queries insurer database; no judgment. |
| Z2.2 PA existence check | Front-desk staff | FULLY_AGENTIC | Lookup; no judgment. |
| Z2.3 PA validity assessment | Dana (on exception) | HUMAN_LED_AGENT_SUPPORT | Agent checks procedure code, auth date, provider ID. If all match → routine acceptance. If mismatch → escalate to nurse/physician for judgment. |
| Z2.4 Missing/pending PA escalation | Dana | AGENT_LED_OVERSIGHT | Agent determines urgency (visit date - today; PA status) and routes to Dana's chase queue with priority. Dana reviews and decides resubmit timing. Agent acts on Dana's decision (resubmit, patient notification, reschedule). |
| Z3.1 Chief complaint parsing | Front-desk staff | HUMAN_ONLY (or HUMAN_LED_AGENT_SUPPORT for parsing, HUMAN_ONLY for decision) | Parsing can be AI-assisted; routing decision requires clinical judgment. Keep human in loop. |
| Z3.2 Urgency signal detection | Front-desk staff | HUMAN_ONLY | Missed emergency signal = patient harm. Must stay human. |
| Z3.3 Routing decision | Front-desk staff | HUMAN_ONLY | Requires clinical judgment + knowledge of physician availability and patient preference. |
| Z4.1 Pharmacy history retrieval | Front-desk staff / Nurse | FULLY_AGENTIC | Lookup; no judgment. |
| Z4.2 Reconciliation comparison | Nurse | HUMAN_LED_AGENT_SUPPORT or HUMAN_ONLY | Agent flags discrepancies. Nurse decides if discrepancy is material and what action to take. |
| Z4.3 Allergy update capture | Nurse | HUMAN_ONLY | Requires clinical judgment and patient conversation. Not suitable for agent. |

---

## System and Data Inventory — Scenario-Specific

| System | Data needed | Access type | Availability | Gap/risk | Shared with other agents | Assumptions |
|---|---|---|---|---|---|---|
| **athenahealth** | Patient demographics, insurance, visit history, current meds, allergies, visit reason, scheduling | Read + write (scheduling, recording reconciliation results, recording PA status) | API available; REST | None | Yes (EHR is shared foundation for all clinic agents) | [ASSUME] athenahealth APIs are accessible from agent environment; no rate limits that prevent real-time queries |
| **Availity** | Insurance eligibility, active coverage, co-pay/deductible, PA status (some insurers) | Read + trigger (submit PA resubmission requests) | API available; REST; some insurers allow API PA submission | Medium risk: Availity coverage is not 100% — some regional insurers not in Availity; Dana has to call those manually | Yes (shared insurance infrastructure) | [ASSUME] Availity API is fast enough for real-time eligibility check (< 2 sec); [ASSUME] Availity maintains current-quarter insurance data (not stale) |
| **DoseSpot** | Current medication list, pharmacy history | Read | API available; integrated with athenahealth | Low; DoseSpot is integrated so queries are reliable | Yes (shared pharmacy infrastructure) | [ASSUME] DoseSpot reconciles with all pharmacies Westbridge's patients use; [ASSUME] patient self-report of meds is available via athenahealth pre-visit questionnaire |
| **Phone/patient portal** | Patient-stated chief complaint, stated allergies, stated medications, pre-visit questionnaire responses | Read (agent cannot call patients; Dana/front-desk only) | Portal available; patient-provided | Medium: paper forms and phone calls still happen; agent can only read portal submissions | No (patient communication layer) | [ASSUME] patients use portal to pre-populate questionnaire before visit (not 100% adoption); [ASSUME] phone-call transcription to athenahealth is manual (not automated) |
| **Dana's PA chase list** (Google Sheets) | Insurer-specific PA SLAs, prior PA resubmit history, patient-specific PA notes | Read + write | Sheets API available? Unclear | High risk: This is not in any formal system. Agent would need write access to Sheets or a replacement system. Currently manual. | No (practice-specific workflow) | [ASSUME] Dana's chase list is canonical source for PA tracking; [ASSUME] Sheets API can be accessed from agent environment; [ASSUME] Dana will replace this with a system-based workflow if automation is built |

---

## Discovery Questions — Scenario-Specific Seed

These are provisional. Coach-elicited material and artefact review will sharpen them.

### [SCOPE] Questions — Clarifying the boundary of what the agent does

1. **Insurance verification scope**: Should the agent attempt to resolve coverage gaps (e.g., offer self-pay or payment plan), or escalate all gaps to Dana? Current assumption: escalate all gaps.
   - Design impact: If agent can offer self-pay, SDI must include patient notification system; if escalate-only, agent just flags.

2. **PA resubmission authority**: Can the agent automatically resubmit an expired/missing PA, or must Dana approve each resubmission? Current assumption: agent proposes; Dana approves.
   - Design impact: If auto-resubmit, autonomy matrix shows full authority; if propose-approve, need human review step.

3. **Visit-reason triage**: Is triage in or out of scope for the agent? Current assumption: out of scope (requires clinical judgment; emergency signals must stay human).
   - Design impact: If out of scope, scope document explicitly excludes it; focus shifts to insurance + PA + meds.

### [BOUNDARY] Questions — Testing the edges of delegation

4. **Clinical vs. administrative**: What counts as "clinical judgment" that the agent cannot do? Current assumption: anything involving interpretation of symptoms, urgency assessment, drug interactions, allergy contraindications = clinical = human.
   - Design impact: Clarifies which zones in Z4 (medication reconciliation) are agent-capable vs. require nurse review.

5. **Insurer API coverage**: How many of Westbridge's key insurers have API access via Availity? Artefacts mention Aetna, UnitedHealthcare Choice, Wellpath, BCBS, Medicare Advantage (Humana), but Availity coverage may not be 100%.
   - Design impact: If 10% of insurers are API-unavailable, agent must escalate those to Dana; if 100% available, agent can handle all.

6. **Patient communication**: Can the agent send SMS/email to patients about PA status, visit confirmations, medication reconciliation issues? Or is all patient comms human-only?
   - Design impact: If patient comms are allowed, SDI includes SMS/email integration; if human-only, agent can only flag in the EHR.

### [HIDDEN_REQUIREMENT] Questions — Uncovering what's not obvious from the brief

7. **Malpractice and liability**: What is Westbridge's malpractice insurance position on AI-driven intake decisions? Is there a specific failure mode the practice is most anxious about (e.g., missed urgent signal, wrong PA, allergy miss)?
   - Design impact: Informs autonomy matrix and escalation triggers; may constrain what the agent can autonomously decide.

8. **Staff stakeholder concerns**: Beyond Dana, who else at the practice needs to trust the agent? What is their biggest fear about AI intake? (Physicians? Nurses? Front-desk staff?)
   - Design impact: Identifies hidden escalation requirements; informs governance model.

9. **Regulatory or compliance constraints**: Are there state-specific medical-record or privacy rules that affect how the agent can access/store patient data?
   - Design impact: May require data isolation or audit-trail requirements not obvious from HIPAA alone.

---

## Assumptions Log — Scenario-Specific Seed

These are provisional; coach interaction and artefact review will refine or test them.

| Assumption | Confidence | Basis | Test method | Impact if wrong |
|---|---|---|---|---|---|
| [HIGH] Patient insurance verification via Availity API is fast enough (< 2 sec latency) for real-time front-desk lookup. | High | Availity is a commercial service widely used; REST APIs are standard. | Latency test with Availity sandbox; check SLA in Availity terms. | If latency is 10+ sec, agent can't do real-time lookup; must use batch or cache; design changes to caching layer. |
| [HIGH] DoseSpot provides current pharmacy-reconciliation data for all Westbridge patients. | High | DoseSpot is integrated with athenahealth; it's a standard integration. | Confirm with Dana that DoseSpot is used for all patient meds; test coverage on sample patient. | If DoseSpot is partial or stale, agent misses medication reconciliation issues; escalation becomes human-only. |
| [MEDIUM] Dana's PA chase list (Google Sheets) can be replaced with a system-based workflow in athenahealth or Availity. | Medium | Sheets is a workaround; professional systems support PA tracking. But migration requires practice buy-in. | Discuss with Dana; check if athenahealth has PA-tracking module. | If Sheets is irreplaceable (political/workflow reason), agent can't reliably track PA chase state; must stay in Sheets. |
| [MEDIUM] Availity covers all major insurers Westbridge uses (Aetna, UnitedHealthcare, Wellpath, Medicare Advantage, BCBS). | Medium | Availity is comprehensive, but some regional/niche insurers may not be included. | Check Availity provider list; ask Dana which insurers are NOT in Availity. | If 20%+ of insurers are not in Availity, agent can only handle 80% of cases; remaining 20% route to Dana. Design must account for this. |
| [MEDIUM] Physicians + Dana will accept agent-proposed PA resubmission, with human approval step. | Medium | Automation of repetitive tasks is generally welcomed; but practice may have risk aversion. | Discuss with Dana; check with senior physician about trust/liability. | If physicians require full pre-authorization of every PA resubmission, autonomy is severely curtailed; cost-benefit of automation may flip. |
| [LOW] Visit-reason triage can be fully automated without clinical judgment. | Low | Artefacts and brief suggest triage requires urgency assessment, which is clinical. But scope of agent may exclude triage. | Clarify with Dana whether triage is in or out of scope; if in scope, what triggers require physician escalation. | If triage must be automated (practice demand) but requires clinical judgment, agent will over-escalate (low precision) or under-escalate (high risk). |
| [MEDIUM] Malpractice insurance and state medical-record regulations permit AI-driven intake decisions, as long as human oversight is in place. | Medium | HIPAA is clear; but practice-specific policy may be more conservative. | Review Westbridge's malpractice coverage terms; consult with practice legal/compliance. | If insurance or regulations prohibit certain agent decisions, those decisions must revert to human; autonomy matrix must reflect this. |
| [HIGH] Patient data is accessible to the agent in real-time (no batch-only restrictions). | High | athenahealth and other systems are cloud-based and real-time. | Confirm with Dana that front-desk systems are real-time; no batch windows. | If any core system is batch-only (unlikely but possible), agent can't do real-time lookup; design must use cache or queue. |

---

## Validation Rules and Anti-Patterns — Scenario-Specific

### Cognitive Load Map (CLM)

✅ **Must**:
- Decompose **all 4 work streams** to JTBD level (minimum 2–3 JTBDs per stream)
- Ground each JTBD in **artefacts** or **coach-elicited material** (not just brief)
- Include a "lived vs. documented" section naming how **actual intake practice differs from the SOP**
  - E.g., "The SOP says PA check happens at appointment scheduling; in practice, Dana maintains a manual chase list because Availity doesn't flag pending PAs reliably"
- Document **primary friction points** explicitly (artefact 5.1 shows PA chase SLA variance; artefact 5.2 shows visit cancelled because PA was pending)
- Assign error_tolerance to every cognitive zone; justify zones marked "zero"
  - E.g., Zone Z3.2 (urgency signal detection) = "zero" because missed emergency signal = patient harm

### Delegation Suitability Matrix (DSM)

✅ **Must**:
- Every task cluster must have all four scores populated (structurability, volume, expertise, reversibility)
- No task can default to "fully_agentic" without explaining why human involvement is minimal
  - E.g., "Insurance verification of active status is fully_agentic because it is 100% rule-based: Availity returns a code; agent maps code to decision. Reversibility is high: if wrong, Dana catches at payment time and refiles. Volume is high (180/day). Structurability is 10 (full rules). No human expertise needed."
- If structurability < 3 OR human_expertise_required > 7, archetype must NOT be "fully_agentic"
  - This prevents landing visit-reason triage as "fully_agentic" when it actually requires nurse/physician judgment

⚠️ **Anti-pattern check**:
- If >60% of task clusters are "fully_agentic", flag in review feedback (but don't prevent submission)
- Likely anti-pattern here: if visit-reason triage is in scope and marked "fully_agentic", that is suspicious (clinical judgment required)

### Volume × Value Analysis (VVA)

✅ **Must**:
- Plot all 4 streams: insurance (high vol, high val), PA check (low-med vol, very high val), visit-reason triage (high vol, high val), meds (high vol, high val)
- Justify primary target: "PA check is primary target because (a) volume of 25/day × 12 min/case = 5 hrs/day of Dana's time, (b) failure mode is visit cancelled at last minute, (c) Dana maintains manual chase list (friction indicator), (d) integration is discrete (Availity API + dates + insurer rules), (e) agentic value is clear (systematic SLA tracking + automatic resubmit proposal)"
- Economic thesis: "At 25 PAs/day × 12 min/case, agent saves 5 hrs/day of Dana's time. At Dana's loaded cost of $75/hr, that's $375/day or ~$97K/yr. Agent development + maintenance cost is [estimate]. Break-even is [X months]."
- Secondary target: "Insurance verification is secondary because (a) high volume (180/day × 3-5 min), (b) high friction (30% fail auto-verify), (c) but lower risk (failures are rework + billing, not patient harm)"

### Agent Purpose Document (APD)

✅ **Must**:
- Job to be Done is a **cognitive contract**, not a task name
  - ❌ **Bad**: "Manage prior authorizations"
  - ✅ **Good**: "Systematically check whether each scheduled procedure has a valid, current prior authorization; identify missing/expired/mismatched authorizations; propose resubmission and route to Dana for approval; notify patient of scheduling changes if PA resubmit delays the visit"
- KPIs are **specific and measurable**:
  - ❌ **Bad**: "High accuracy", "Low error rate"
  - ✅ **Good**: "Accuracy: ≥99% of PA validity checks match nurse's manual review (test on 100-case validation set). Coverage: 85%+ of scheduled procedures checked within 48 hours of visit. Throughput: 25 PAs/day processed (current load). HITL rate: ≤10% require Dana escalation (currently 30%)."
- Failure modes include **consequence** and **recovery**:
  - ❌ **Bad**: "Agent marks PA as valid when it's actually expired."
  - ✅ **Good**: "Agent marks PA as valid when it's actually expired. Consequence: visit proceeds without valid authorization; claim denied; patient may be charged full procedure cost; practice risks compliance audit. Recovery: Dana manually voids the visit and refiles with updated PA; patient is not billed."
- Autonomy matrix has **explicit cells** (not empty):
  - ✅ **Example**:
    - **Agent decides alone**: PA exists, is current, procedure code matches → agent flags as "ready" in athenahealth
    - **Agent proposes, human approves**: PA expired or missing → agent flags with "resubmit?" recommendation; Dana approves resubmission
    - **Human decides**: PA matches is ambiguous (e.g., procedure code is close but not exact) → agent escalates to nurse for judgment
    - **Human takes over**: Patient disputes PA requirement; mismatch is material → Dana or physician decides next steps
- Escalation triggers are **specific conditions** → **named roles**:
  - ❌ **Bad**: "Escalate if unsure"
  - ✅ **Good**:
    - "If PA required but not found in Availity, escalate to Dana (Practice Manager) for manual insurer call"
    - "If PA procedure code does not exactly match scheduled procedure, escalate to Nurse for clinical judgment"
    - "If patient disputes PA requirement, escalate to Dr. [senior physician] for medical decision"

### System and Data Inventory (SDI)

✅ **Must**:
- Every system mentioned in APD must appear in SDI
- Availability is marked **honestly**:
  - ❌ **Bad**: "athenahealth: available" (vague)
  - ✅ **Good**: "athenahealth: API available, REST, SLA 99.9%, rate limits 100 req/sec"
  - ✅ **Good**: "Dana's PA chase list: Google Sheets, no formal API, manual tracking, high operational risk, not scalable"
- Gap/risk field is **non-empty** for any system with availability != "api_available":
  - E.g., "Google Sheets (PA chase list): Gap — not a system of record; no audit trail; no integration with athenahealth; must be migrated or replaced if agent is built."
- Assumptions about system behaviour are **logged** (not assumed as facts):
  - E.g., "[ASSUME] Availity API latency < 2 sec for eligibility check; [ASSUME] Availity covers all insurers Westbridge uses"

### Discovery Questions

✅ **Must have at least 6, no more than 10**. Each must:
- Have **design_impact** naming a specific change to APD/SDI if answered differently
- Be **scenario-specific** (not generic)
- Name **target_stakeholder** explicitly
- Be organized by maturity level: ≥2 "clarifies_scope", ≥1 "uncovers_hidden_requirement"

❌ **Generic anti-patterns to avoid**:
- "Tell me about your PA process" → instead, "Can the agent automatically resubmit expired PAs, or must Dana approve each one?"
- "What are your top pain points?" → instead, "In the last quarter, how often has a visit been cancelled due to missing/expired PA, and what is the cost per cancellation?"

### Assumptions Log

✅ **Must have ≥5 documented**. Each must:
- Have **confidence_level**, **basis**, **test_method**, **impact_if_wrong**
- Low-confidence assumptions must have a named test method (not "TBD")
- No duplicates

---

## Handling Ambiguity & Escalation — Scenario-Specific

### When to Ask the Participant Before Proceeding

1. **CLM missing detail on a stream** (e.g., only insurance verification decomposed, not PA check or meds)
   - Escalate: "The CLM shows insurance verification in detail, but PA check is summarized. Week 2 requires ≥2 streams decomposed to JTBD level. Is PA check your second stream, or do you plan to add another?"

2. **Unmarked inference** (e.g., CLM claims "Availity covers all Westbridge insurers" but artefacts don't state this)
   - Escalate: "The CLM claims Availity covers all insurers, but the brief doesn't name all insurers. Should I log 'Availity coverage for all major insurers' as a [MEDIUM] confidence assumption, or do you know this is true?"

3. **Delegation archetype mismatch** (e.g., visit-reason triage marked "fully_agentic" when it requires urgency assessment)
   - Escalate: "This task is marked 'fully_agentic' but structurability is 2 (requires clinical judgment for urgency). Is this intentional, or should the archetype be 'human_led_agent_support' (agent flags for human decision)?"

4. **Empty autonomy matrix cell** (e.g., APD names "insurance reconciliation" but autonomy matrix doesn't specify agent authority)
   - Escalate: "The APD names insurance reconciliation, but the autonomy matrix doesn't specify: does the agent decide alone if coverage is clear, or does Dana always review? What's the authority boundary?"

5. **KPI vagueness** (e.g., APD says "high PA coverage" but doesn't specify a %)
   - Escalate: "The KPI says 'high PA coverage.' What does that mean numerically? 80%? 90%? 95%? The requirement must be specific so you know if the agent succeeds."

6. **HIPAA / regulatory constraint unclear** (e.g., participant doesn't address whether agent can store patient data)
   - Escalate: "The agent will access patient PII (demographics, medications, allergies). Does Westbridge's malpractice policy or HIPAA compliance team have constraints on where agent data can be stored or processed?"

### When to Decide Alone (Do Not Ask)

- **Correcting enum values**: if participant writes "high-agentic" instead of "agent_led_oversight", normalize silently
- **Marking zones with zero error_tolerance**: if a zone touches patient safety (urgency detection, allergy contraindication), mark "zero" without asking
- **Flagging missing artefact grounding**: if a CLM claim has no citation to artefacts, note it and ask participant to cite or convert to assumption
- **Honest gap naming in SDI**: if Google Sheets is the system of record for PA tracking, surface as a gap/risk (lack of audit trail, no API, scalability risk) regardless of participant's preferences

---

## Success Criteria — Scenario-Specific

### Strong Week 2 submission demonstrates:

✅ **Lived-work analysis** — CLM reflects **actual intake practice** with friction points explicitly named
   - E.g., "SOP says PA check is done at scheduling; actually, Dana runs a manual chase list because Availity doesn't auto-flag pending PAs. This is where the value is."

✅ **Disciplined delegation thinking** — each task archetype is justified; not everything defaults to "fully_agentic"
   - E.g., "Insurance verification is fully_agentic because it's 100% rule-based. Visit-reason triage stays human because urgency assessment requires clinical judgment."

✅ **Honest integration planning** — SDI surfaces real challenges (Google Sheets as PA tracker, Availity coverage gaps, DoseSpot reconciliation limitations)
   - E.g., "Gap: PA chase list is maintained in Google Sheets, not in any system. If agent is built, this must be migrated to athenahealth or Availity, or replaced with a new system. Migration effort: TBD."

✅ **Specific discovery** — questions would materially change the design if answered differently
   - E.g., "Can agent auto-resubmit PAs, or must Dana approve?" → answer changes autonomy matrix

✅ **Buildable agent design** — APD has clear purpose, specific KPIs, explicit autonomy boundaries, named escalation triggers
   - E.g., "Agent decides alone: PA exists and is valid. Agent proposes: PA missing or expired. Human decides: PA mismatch is ambiguous."

✅ **Transparent assumptions** — inferences logged with confidence level and test methods
   - E.g., "[MEDIUM] Availity covers all Westbridge insurers. Test: pull insurer list from athenahealth; check against Availity provider list."

✅ **Compounding mindset** — identifies shared integrations (athenahealth, DoseSpot, Availity are foundations for other agents)

### Weak submission shows:

❌ **Documented work as lived work** — CLM matches the SOP but misses the manual chase list, the cross-site staffing rotation, the insurer-specific SLA variance
❌ **Archetype drift** — 60%+ of tasks are "fully_agentic" without justified reasoning
❌ **Speculative design** — APD assumes Availity covers all insurers, or that DoseSpot reconciliation is error-free, without marking these as assumptions
❌ **Generic discovery** — "What is your PA process?" instead of "Can the agent auto-resubmit?"
❌ **Vague governance** — autonomy matrix is empty or says "agent decides" without specifying what "decides" means
❌ **Hidden assumptions** — treats "Availity is fast enough" or "DoseSpot covers all meds" as facts when they are assumptions

---

## Your Role: Assisting the Participant

You are helping the participant apply ATX methodology to Westbridge Family Medicine's intake function. You are **not building the agent**. You are:

1. **Clarifying methodology**: "What should go in a CLM?" → explain structure, point to reference docs, show examples
2. **Grounding claims in artefacts**: "Where does the brief say this?" → participant cites artefact or logs as assumption
3. **Tracing delegation reasoning**: "Why is PA check 'fully_agentic'?" → participant explains structurability, volume, expertise, reversibility
4. **Validating KPI specificity**: "What does 'high coverage' mean?" → participant defines target %
5. **Stress-testing discovery questions**: "Would this answer change the design?" → participant refines question
6. **Spotting anti-patterns**: >60% fully_agentic, generic questions, unmarked inferences → flag them
7. **Supporting peer review**: Brief on the anti-patterns to watch (delegation drift, speculative assumptions, vague governance)

---

## What This CLAUDE.md Specifies

This document governs:
- **Scenario context**: 6-physician family medicine practice, 4-person intake team, 4 work streams, 3 artefacts
- **Core entities**: WorkStreams, JTBDs, CognitiveZones, Breakpoints, Delegation Archetypes specific to Westbridge
- **Validation rules**: Acceptance criteria for CLM, DSM, VVA, APD, SDI, Discovery Questions, Assumptions
- **Anti-patterns**: What to watch for (archetype drift, generic questions, hidden assumptions)
- **Participant role**: Apply ATX to assess intake work; produce 7 deliverables; submit Thursday 14:15 CET
- **Your role**: Clarify methodology, ground claims in artefacts, stress-test assumptions, flag anti-patterns

This document does NOT govern:
- The live coach interaction (that's between participant and Dana role-play)
- Peer review scoring (that's peer reviewer discretion)
- Gate 2 results and feedback (that's coach/faculty responsibility)
