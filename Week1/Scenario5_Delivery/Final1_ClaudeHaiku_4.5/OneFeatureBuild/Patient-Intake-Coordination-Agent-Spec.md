# Patient Intake Coordination Agent — Week 1 Specification

**Scenario:** Scenario 5 — Small-Clinic Patient Intake  
**Feature Set:** FR1 (Appointment Sync) + FR3 (Insurance Verification)  
**Scope:** Administrative coordination only; no clinical judgment delegated to agent  
**Status:** Ready for agent build  

---

## EXECUTIVE SUMMARY

This specification defines a **deterministic, fail-closed** patient intake coordination system. The agent:

- **Syncs appointments** from athenahealth every 15 minutes (idempotent, fault-tolerant)
- **Verifies insurance eligibility** against insurance provider (deterministic: verified or escalated, never guessed)
- **Logs every action** for audit compliance (90-day minimum retention)
- **Escalates all ambiguity** to FRONT_DESK human team

**Key constraints:**
- No clinical judgment anywhere (diagnosis, treatment decisions, medical triage remain human-only)
- Fail-closed: timeout always escalates after 3 retries
- Deterministic: same inputs produce same outputs
- Immutable audit trail for all decisions

---

## TABLE OF CONTENTS

1. Data Model (3 entities: Appointment, IntakeWorkItem, Escalation)
2. Feature Requirement 1: Appointment Sync
3. Feature Requirement 3: Insurance Verification
4. Audit Logging
5. Validation Scenarios (V1 Happy Path, V3 Timeout)

---

# SECTION 1: DATA MODEL

Three entities define the intake workflow state machine. All entities are immutable once created except for specific state transition fields.

## Entity 1: Appointment

**Purpose:** Canonical record of scheduled visits; synced from athenahealth; immutable once created; triggers intake workflow.

```
Entity: Appointment
Primary Key: id (UUID, immutable)
Unique Index: athena_appointment_id (no duplicates across syncs)

Attributes:

id: UUID
  - Primary key, immutable, generated on first sync
  - Example: 550e8400-e29b-41d4-a716-446655440000

athena_appointment_id: string
  - Immutable, external reference to athenahealth system
  - Globally unique; prevents duplicate syncs
  - Example: "ATH-2026-04-25-14000"

patient_id: UUID
  - Foreign key to Patient
  - Immutable, required
  - Set at creation; must exist before Appointment created

scheduled_datetime: ISO 8601 timestamp UTC
  - Immutable, required
  - Appointment start time in UTC (not local clinic time)
  - Example: "2026-04-25T14:00:00Z"
  - Constraint: must be in future relative to created_at

provider_id: string
  - athenahealth provider identifier
  - Immutable, required
  - Example: "PROV-001"

location_id: string
  - athenahealth location/department identifier
  - Immutable, required
  - Example: "LOC-MAIN"

appointment_type: enum [ROUTINE, URGENT, FOLLOWUP, PREVENTIVE]
  - Immutable, required
  - Mapped from athena.appointmentType: "Routine"→ROUTINE, "Urgent"→URGENT, etc.

status: enum [SCHEDULED, CONFIRMED, COMPLETED, CANCELLED, NO_SHOW]
  - Required, default = SCHEDULED
  - Mutable (can change on sync or clinical action)
  
  State Machine:
  ┌─────────────────────────────────────────────────────────────┐
  │ SCHEDULED ──→ CONFIRMED (intake workflow ready)             │
  │           ──→ CANCELLED (athena sync detects cancellation)   │
  │           ──→ NO_SHOW (appointment time passed, no COMPLETED)│
  │           ──→ COMPLETED (clinical staff marks at visit end)  │
  │                                                               │
  │ CANCELLED ──→ TERMINAL (no further transitions)              │
  │ NO_SHOW ──→ TERMINAL (no further transitions)                │
  │ COMPLETED ──→ TERMINAL (no further transitions)              │
  └─────────────────────────────────────────────────────────────┘
  
  Note: Agent can only set SCHEDULED or CANCELLED; COMPLETED and CONFIRMED
        driven by clinical staff or intake workflow success.

intake_workitem_id: UUID, nullable
  - Foreign key to IntakeWorkItem
  - Set when intake workflow created for this appointment
  - Immutable once set
  - Unique per Appointment (one intake per appointment)

sync_status: enum [SYNCED, SYNC_PENDING, SYNC_ERROR]
  - Required, default = SYNC_PENDING
  - Mutable (changes on each sync attempt)
  
  SYNCED: last athenahealth sync succeeded; data current as of sync_timestamp
  SYNC_PENDING: first sync or retry pending
  SYNC_ERROR: last sync attempt failed; last_sync_error contains reason

created_at: ISO 8601 timestamp UTC
  - Immutable, set on first sync
  - Example: "2026-04-20T10:30:00Z"

updated_at: ISO 8601 timestamp UTC
  - Updated only when status or sync_status changes
  - Example: "2026-04-20T10:35:00Z"

last_sync_at: ISO 8601 timestamp UTC
  - Updated each time appointment is synced from athena
  - Tracks recency of data
  - Example: "2026-04-20T10:35:00Z"

synced_by: string
  - Agent ID or service name that performed sync
  - Immutable
  - Example: "ATHENA_SYNC_AGENT"
  - Null if not yet synced

last_sync_error: string, max 500 chars
  - Populated if sync_status = SYNC_ERROR
  - Cleared on successful sync
  - Example: "athena.appointmentId field missing in response"

Constraints:

1. athena_appointment_id is globally unique (no duplicate syncs)
2. Cannot modify: patient_id, athena_appointment_id, scheduled_datetime, appointment_type after creation
3. Cannot transition to COMPLETED or CONFIRMED by agent alone (human/clinical staff only)
4. If Appointment transitions to CANCELLED, any associated IntakeWorkItem must be archived
   and Escalation created (escalation_category = APPOINTMENT_SYNC_FAILED)
5. scheduled_datetime must be in future relative to created_at
```

---

## Entity 2: IntakeWorkItem

**Purpose:** Tracks administrative intake workflow for a single appointment. Contains all pre-visit verification tasks. State machine drives escalation logic.

```
Entity: IntakeWorkItem
Primary Key: id (UUID, immutable)
Unique Index: appointment_id (one intake per appointment)

Attributes:

id: UUID
  - Primary key, immutable, generated when intake created
  - Example: "660e8400-e29b-41d4-a716-446655440001"

appointment_id: UUID
  - Foreign key to Appointment
  - Immutable, required
  - Unique per appointment (no duplicate intakes)

patient_id: UUID
  - Foreign key to Patient
  - Immutable, required
  - Example: "770e8400-e29b-41d4-a716-446655440002"

status: enum [PENDING, READY, ESCALATED, HOLD, COMPLETED, ARCHIVED]
  - Required, default = PENDING
  - Mutable (changes as verifications complete or escalate)

  State Machine:
  ┌──────────────────────────────────────────────────────────────────┐
  │ PENDING ──→ READY (all required verifications succeed)            │
  │         ──→ ESCALATED (verification fails requiring human judgment)
  │         ──→ HOLD (verification timeout; awaiting retry)           │
  │         ──→ COMPLETED (clinic marks as finished; outside agent)   │
  │         ──→ ARCHIVED (associated appointment cancelled)           │
  │                                                                    │
  │ READY ──→ COMPLETED (clinic completes intake; outside agent)      │
  │       ──→ ARCHIVED (appointment cancelled)                        │
  │                                                                    │
  │ ESCALATED ──→ COMPLETED (human resolves escalation; outside agent)│
  │            ──→ ARCHIVED (escalation cancelled; outside agent)     │
  │                                                                    │
  │ HOLD ──→ PENDING (retry after escalation resolved; outside agent) │
  │      ──→ ESCALATED (retry exhausted)                              │
  │      ──→ ARCHIVED (appointment cancelled)                         │
  │                                                                    │
  │ COMPLETED ──→ TERMINAL (no further transitions)                   │
  │ ARCHIVED ──→ TERMINAL (no further transitions)                    │
  └──────────────────────────────────────────────────────────────────┘

appointment_sync_error: string, nullable, max 500 chars
  - Set if appointment sync from athenahealth fails at moment IntakeWorkItem created
  - If not null, status must be ESCALATED
  - Cleared on next successful sync+retry
  - Example: "athenahealth API returned HTTP 500; appointment data incomplete"

insurance_verification_status: enum [PENDING, VERIFIED, FAILED, TIMEOUT, ESCALATED]
  - Required, default = PENDING
  - Tracks progression of insurance verification (FR3)

  PENDING: not yet attempted or awaiting retry
  VERIFIED: insurance integration returned success; eligibility confirmed
  FAILED: insurance integration returned explicit failure (policy not found, etc.)
  TIMEOUT: insurance integration did not respond within timeout window
  ESCALATED: human intervention required (usually TIMEOUT after retries exhausted or FAILED with ambiguity)

insurance_verification_timestamp: ISO 8601 timestamp UTC, nullable
  - Set when verification attempt completes (success or failure)
  - Updated on each retry
  - Example: "2026-04-20T10:35:15Z"

insurance_provider_id: string, nullable
  - Reference to insurance provider returned by integration
  - Logged for audit purposes
  - Example: "BLUE_CROSS_01"

insurance_reference_id: string, nullable, max 200 chars
  - Set if insurance verification succeeds
  - Used for subsequent authorization checks and billing
  - Example: "ELIGIBILITY-2026-04-20-12345"

insurance_verification_error: string, nullable, max 500 chars
  - Populated if status = FAILED or TIMEOUT
  - Contains reason from integration or "TIMEOUT_AFTER_15s"
  - Cleared on successful verification
  - Example: "PATIENT_NOT_FOUND" or "TIMEOUT_AFTER_15s"

insurance_verification_retry_count: integer
  - Default 0
  - Incremented on each retry
  - Capped at 3; no more retries after 3 failures
  - Example: 1

created_at: ISO 8601 timestamp UTC
  - Immutable, set when IntakeWorkItem created
  - Example: "2026-04-20T10:30:30Z"

updated_at: ISO 8601 timestamp UTC
  - Updated whenever any field changes
  - Example: "2026-04-20T10:35:20Z"

created_by: string
  - Agent ID or service that created this intake
  - Immutable
  - Example: "ATHENA_SYNC_AGENT"

updated_by: string
  - Identifies what last modified the entity
  - Example: "INSURANCE_VERIFY_AGENT" or "FRONT_DESK_OVERRIDE"

escalated_at: ISO 8601 timestamp UTC, nullable
  - Set when status transitions to ESCALATED
  - Immutable once set (cannot "unescalate")
  - Example: "2026-04-20T10:35:45Z"

escalated_to: enum [FRONT_DESK], nullable
  - Required if escalated_at is not null
  - Identifies escalation target (human team responsible for this workitem)

escalation_reason: string, max 500 chars, nullable
  - Required if escalated_at is not null
  - Human-readable explanation of why escalation needed
  - Example: "Insurance verification timeout after 3 retries; requires manual verification"

Constraints:

1. Once escalated_at is set, it is immutable (cannot "unescalate")
2. Cannot transition from ESCALATED back to PENDING without external action (human must resolve)
3. insurance_verification_retry_count cannot exceed 3
4. If insurance_verification_retry_count >= 3 AND status still PENDING:
   - Transition to ESCALATED
   - Set escalation_reason = "Insurance verification failed after 3 retries; escalated to FRONT_DESK"
   - Set escalated_to = FRONT_DESK
5. If status = HOLD, must have insurance_verification_error populated (reason for hold)
6. Cannot transition to READY if insurance_verification_status != VERIFIED
7. If associated Appointment becomes CANCELLED, this IntakeWorkItem must transition to ARCHIVED
   (with Escalation created if status was not already COMPLETED or ARCHIVED)

Agent Triggers:

1. On Appointment sync success: create IntakeWorkItem with status=PENDING (if not exists)
2. On IntakeWorkItem.status=PENDING: initiate insurance verification (FR3)
3. If insurance verification succeeds: set insurance_verification_status=VERIFIED, attempt transition to READY
4. If insurance verification fails: set insurance_verification_status=FAILED or TIMEOUT
5. If retries exhausted: transition to ESCALATED
```

---

## Entity 3: Escalation

**Purpose:** Immutable audit trail of all escalations to human teams. Enables SLA enforcement and batch processing.

```
Entity: Escalation
Primary Key: id (UUID, immutable)
Unique Index: workitem_id (one escalation per workitem)
Index: assigned_to, status (for human team queries)

Attributes:

id: UUID
  - Primary key, immutable, generated on escalation creation
  - Example: "880e8400-e29b-41d4-a716-446655440003"

workitem_id: UUID
  - Foreign key to IntakeWorkItem
  - Immutable, required
  - Unique per workitem (one escalation per workitem)

patient_id: UUID
  - Foreign key to Patient
  - Immutable, required
  - Enables rapid indexing by patient (useful for human review)
  - Example: "770e8400-e29b-41d4-a716-446655440002"

appointment_id: UUID
  - Foreign key to Appointment
  - Immutable, required
  - Example: "550e8400-e29b-41d4-a716-446655440000"

escalation_reason: string, max 500 chars
  - Immutable, set at creation
  - Matches IntakeWorkItem.escalation_reason
  - Example: "Insurance verification timeout after 3 retries"

escalation_category: enum [
    INSURANCE_TIMEOUT,
    INSURANCE_FAILED,
    APPOINTMENT_SYNC_FAILED,
    VERIFICATION_RETRY_EXHAUSTED,
    MANUAL_OVERRIDE
  ]
  - Immutable, required
  - Enables triage and batch processing by reason
  - Examples:
    INSURANCE_TIMEOUT: insurance API did not respond within 15s after 3 retries
    INSURANCE_FAILED: insurance returned explicit failure (patient not found, policy inactive, etc.)
    APPOINTMENT_SYNC_FAILED: appointment cancelled in athena; intake halted
    VERIFICATION_RETRY_EXHAUSTED: other verification failed after 3 retries
    MANUAL_OVERRIDE: human forced escalation from FRONT_DESK

assigned_to: enum [FRONT_DESK]
  - Immutable, required
  - Identifies human team responsible for this escalation
  - Currently only FRONT_DESK; extensible for future teams

status: enum [OPEN, IN_PROGRESS, RESOLVED, CANCELLED]
  - Required, default = OPEN
  - Mutable (changes as human acts)

  State Machine:
  ┌──────────────────────────────────────────────────────────────┐
  │ OPEN ──→ IN_PROGRESS (human begins working)                  │
  │      ──→ CANCELLED (escalation no longer needed; e.g. appt   │
  │                     cancelled before human acts)              │
  │                                                                │
  │ IN_PROGRESS ──→ RESOLVED (human completes; decision logged)   │
  │             ──→ CANCELLED (no longer relevant)                │
  │                                                                │
  │ RESOLVED ──→ TERMINAL (no further transitions)                │
  │ CANCELLED ──→ TERMINAL (no further transitions)               │
  └──────────────────────────────────────────────────────────────┘

human_decision: string, nullable, max 500 chars
  - Populated when status transitions to RESOLVED
  - Human-readable summary of decision (e.g., "Verified insurance via phone call")
  - Null until resolved
  - Example: "Contacted patient; confirmed coverage via benefits card"

human_decision_timestamp: ISO 8601 timestamp UTC, nullable
  - Set when status transitions to RESOLVED
  - Immutable once set
  - Example: "2026-04-20T11:00:00Z"

resolved_by: string, nullable
  - User ID of FRONT_DESK staff member who resolved
  - Immutable once set
  - Example: "FRONT_DESK_USER_01"

created_at: ISO 8601 timestamp UTC
  - Immutable, set on escalation creation
  - Example: "2026-04-20T10:35:45Z"

updated_at: ISO 8601 timestamp UTC
  - Updated on status changes
  - Example: "2026-04-20T11:00:00Z"

sla_deadline: ISO 8601 timestamp UTC
  - Immutable, calculated as created_at + 4 hours
  - SLA: escalations must be acknowledged within 4h
  - Example: "2026-04-20T14:35:45Z"

sla_breached: boolean
  - Default false
  - Set to true if status=OPEN after sla_deadline passes
  - Set by nightly SLA audit job (outside agent scope)
  - Example: false

Constraints:

1. workitem_id is globally unique (one escalation per workitem)
2. Cannot set human_decision or resolved_by unless status = RESOLVED
3. If sla_deadline has passed and status=OPEN, sla_breached is set true by nightly audit
4. Escalation is immutable after creation except: status, human_decision, resolved_by, updated_at, sla_breached
5. Cannot modify: id, workitem_id, patient_id, appointment_id, escalation_reason, escalation_category, assigned_to, created_at
```

---

# SECTION 2: FEATURE REQUIREMENT 1 — APPOINTMENT SYNC

**Objective:** Sync scheduled appointments from athenahealth every 15 minutes. Idempotent, deterministic, fail-closed.

## FR1.1: Sync Interval and Trigger

```
Trigger: Agent executes appointment sync job on fixed schedule

Schedule:
  - Every 15 minutes on UTC boundary: 00:00, 00:15, 00:30, 00:45, 01:00, etc.
  - Timezone: UTC (not local clinic time)
  - If sync takes > 5 minutes and next run is due, queue next run immediately after current completes
    (prevents queueing multiple concurrent syncs; protects against API backpressure)

Timeout:
  - Each sync job must complete within 10 minutes
  - If job still running after 10 minutes, kill and log as TIMEOUT
  - Next sync scheduled normally (backpressure relief)
```

## FR1.2: Idempotent Fetch from athenahealth

```
[Agent Alone + Log]

Responsibility: agent executes without human intervention; all actions logged

Input: (no parameters; agent determines sync scope from configuration)
  - Clinic location_ids: [provided in configuration; fixed list of clinic locations]
  - Date range: today (UTC) through today + 30 days
    Why 30 days? Clinic wants advance notice of intake issues; 30-day window is operational standard
  - Filter: only appointments with status IN [Scheduled, Confirmed] (exclude completed/cancelled)

Integration Contract: athenahealth Appointment Finder

Endpoint: GET /v1/appointments

Authentication:
  - Method: Bearer token (OAuth or API key)
  - Token storage: secrets manager, key: ATHENA_API_KEY
  - Token rotation: per athenahealth contract (external to this spec)

Query Parameters (exact names and format):
  - departmentId: string (comma-separated list)
    Example: "LOC-MAIN,LOC-URGENT,LOC-PEDIATRICS"
  
  - startDate: string YYYY-MM-DD format (UTC date of sync, not local)
    Example: "2026-04-20"
  
  - endDate: string YYYY-MM-DD format (UTC date, 30 days out)
    Example: "2026-05-20"
  
  - appointmentStatuses: string (comma-separated)
    Fixed value: "Scheduled,Confirmed"
    (agent must not vary this filter; determinism requirement)

Request Headers:
  - Authorization: Bearer {ATHENA_API_KEY}
  - Content-Type: application/json
  - User-Agent: PatientIntakeAgent/1.0

Success Response: HTTP 200 OK

Body (JSON):
```json
{
  "appointments": [
    {
      "id": "ATH-2026-04-25-001",
      "patientId": "PAT-ATH-00123",
      "providerId": "PROV-001",
      "departmentId": "LOC-MAIN",
      "appointmentType": "Routine",
      "scheduledDateTime": "2026-04-25T14:00:00Z",
      "status": "Scheduled"
    },
    {
      "id": "ATH-2026-04-26-002",
      "patientId": "PAT-ATH-00456",
      "providerId": "PROV-002",
      "departmentId": "LOC-URGENT",
      "appointmentType": "Urgent",
      "scheduledDateTime": "2026-04-26T09:30:00Z",
      "status": "Confirmed"
    }
  ],
  "totalCount": 2,
  "pageNumber": 1,
  "pageSize": 100
}
```

Error Response: HTTP 4xx or 5xx

Body (JSON):
```json
{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "athenahealth API temporarily down for maintenance"
  }
}
```

Timeout: 30 seconds per API call (clock starts at request, ends at first byte of response or timeout)

Retry Logic:

HTTP 5xx (server error: 500, 502, 503, 504):
  - Retry up to 2 times with exponential backoff: 5s, then 10s
  - Log each retry attempt with timestamp and error
  - After 2 retries exhausted: do not retry further; log as ERROR; escalate to operations (not clinic)

HTTP 429 (rate limit):
  - Do not retry immediately
  - Log rate-limit event with headers (Retry-After if provided)
  - Pause sync for 1 hour (halt further syncs)
  - Escalate to operations (signal quota exceeded)

HTTP 4xx excluding 429 (client error: 400, 401, 403, 404, 422):
  - Do not retry
  - Log as CRITICAL (request format error or auth failure)
  - Escalate to operations; do not retry until human reviews

Timeout (no response after 30s):
  - Do not retry
  - Log as TIMEOUT
  - Mark all pending appointments in this sync as SYNC_PENDING (no change)
  - Escalate to operations only if timeout persists for > 1 hour (4 missed sync cycles)

Fallback Behavior:
  - If athenahealth unavailable for 1 sync cycle (15 min): queue next sync; no escalation to clinic
  - If unavailable for > 1 hour (4 missed cycles): escalate to operations with alert
  - Do not force error state or halt workflow; permit normal retry on next scheduled sync

Rate Limit: 100 requests/minute per API key (documented in athena contract; agent must not exceed)
  - Track requests per minute; if approaching limit, log warning but do not back off (athena owns throttling via 429)

Data Mapping (athena → Appointment):
  - athena.id → Appointment.athena_appointment_id
  - athena.patientId → Appointment.patient_id (lookup Patient record; if Patient doesn't exist, log ERROR and skip appointment; do not create Patient)
  - athena.providerId → Appointment.provider_id
  - athena.departmentId → Appointment.location_id
  - athena.appointmentType → Appointment.appointment_type
    Mapping: "Routine"→ROUTINE, "Urgent"→URGENT, "FollowUp"→FOLLOWUP, "Preventive"→PREVENTIVE
    If unexpected value: log ERROR and skip appointment (do not invent type)
  - athena.scheduledDateTime → Appointment.scheduled_datetime (already ISO 8601 UTC)
  - athena.status → mapped to Appointment.status (logic in FR1.3)
```

## FR1.3: Upsert Logic (Deterministic Idempotence)

```
[Agent Alone + Log]

For each appointment in athenahealth response, execute in order:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 1: Lookup Existing Appointment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Query database:
  SELECT * FROM Appointment WHERE athena_appointment_id = ?
  
Result: found (existing record) or not found (new appointment)
Log: Query executed; result (found/not found)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 2A: IF NOT FOUND — Create New Appointment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pre-flight checks:
  1. Lookup Patient by athena.patientId
     - If Patient not found: log WARNING and skip this appointment (do not create)
       Log format: "Appointment {athena_id} skipped; Patient {patient_id} not in system"
       Outcome: skip to next appointment; continue sync
  
  2. Validate athena.scheduledDateTime
     - Parse as ISO 8601 UTC
     - If parse fails: log ERROR and skip (do not create appointment with invalid date)

Create Appointment record:
  - id: generate new UUID
  - athena_appointment_id: from athena response (immutable)
  - patient_id: from athena response (immutable; already validated in pre-flight)
  - scheduled_datetime: from athena response (immutable)
  - provider_id: from athena response (immutable)
  - location_id: from athena response (immutable)
  - appointment_type: mapped from athena.appointmentType (immutable)
  - status: SCHEDULED (default for new appointments)
  - intake_workitem_id: null (will be set when intake created)
  - sync_status: SYNCED (API call succeeded)
  - created_at: current UTC timestamp
  - updated_at: current UTC timestamp
  - last_sync_at: current UTC timestamp
  - synced_by: "ATHENA_SYNC_AGENT"
  - last_sync_error: null

Create IntakeWorkItem record (automatically):
  - id: generate new UUID
  - appointment_id: from Appointment just created
  - patient_id: from Appointment
  - status: PENDING (ready for verification agents)
  - appointment_sync_error: null (sync succeeded)
  - insurance_verification_status: PENDING (not yet attempted)
  - all other fields: null or default
  - created_at: current UTC timestamp
  - created_by: "ATHENA_SYNC_AGENT"
  - updated_by: "ATHENA_SYNC_AGENT"

Update Appointment.intake_workitem_id to reference IntakeWorkItem just created

Log:
  - Level: INFO
  - Message: "Created Appointment {appointment_id} from athena sync"
  - Details: athena_appointment_id, patient_id, scheduled_datetime, appointment_type, status
  - Log: "Created IntakeWorkItem {workitem_id} for Appointment {appointment_id}"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 2B: IF FOUND AND athena.status IN [CANCELLED, COMPLETED]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Appointment is no longer in future; must be archived.

Update Appointment:
  - status: CANCELLED (if athena says cancelled) or COMPLETED (if athena says completed)
    Note: agent sets CANCELLED; clinical staff sets COMPLETED
    If athena already says COMPLETED, agent still marks CANCELLED to signal intake halt
  - updated_at: current UTC timestamp
  - last_sync_at: current UTC timestamp
  - sync_status: SYNCED

Log:
  - Level: INFO
  - Message: "Updated Appointment {appointment_id}; status → CANCELLED (from athena)"

Check for associated IntakeWorkItem:
  - Query: SELECT IntakeWorkItem WHERE appointment_id = ?
  - If found and status != COMPLETED and status != ARCHIVED:
    
    Create Escalation record:
      - id: generate new UUID
      - workitem_id: IntakeWorkItem.id
      - patient_id: IntakeWorkItem.patient_id
      - appointment_id: Appointment.id
      - escalation_reason: "Associated appointment cancelled; intake workflow halted"
      - escalation_category: APPOINTMENT_SYNC_FAILED
      - assigned_to: FRONT_DESK
      - status: OPEN
      - human_decision: null
      - created_at: current UTC timestamp
      - updated_at: current UTC timestamp
      - sla_deadline: current UTC timestamp + 4 hours
      - sla_breached: false
    
    Update IntakeWorkItem:
      - status: ARCHIVED
      - updated_at: current UTC timestamp
      - updated_by: "ATHENA_SYNC_AGENT"
    
    Log:
      - Level: INFO
      - Message: "Escalated IntakeWorkItem {workitem_id}; appointment was cancelled; created Escalation {escalation_id}"
  
  - If not found or already COMPLETED/ARCHIVED: no escalation; just log sync update

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 2C: IF FOUND AND athena.status IN [SCHEDULED, CONFIRMED]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Appointment is still in future; verify sync is idempotent.

Compare Appointment.status with athena.status:

Case 1: Status unchanged (idempotent sync)
  - No update needed
  - Log: "Appointment {appointment_id} idempotent sync; status unchanged"
  - Outcome: continue to next appointment

Case 2: Status changed (e.g., athena: SCHEDULED → CONFIRMED)
  - Update Appointment:
    - status: map from athena (CONFIRMED)
    - updated_at: current UTC timestamp
    - last_sync_at: current UTC timestamp
  - Log: "Updated Appointment {appointment_id}; status → CONFIRMED (from athena)"
  - Outcome: continue to next appointment

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 3: Mark Sync Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All appointments processed successfully:
  - Set sync_status = SYNCED for all affected records
  - Set last_sync_error = null for all affected records
```

## FR1.4: Sync Job Result Aggregation

```
[Agent Alone + Log]

After all appointments processed, compile result object:

Result Object (JSON):
{
  "sync_cycle_id": "UUID generated at start of sync",
  "sync_started_at": "ISO 8601 UTC timestamp",
  "sync_completed_at": "ISO 8601 UTC timestamp",
  "sync_duration_seconds": 45,
  "appointments_fetched": 12,
  "appointments_created": 3,
  "appointments_updated": 2,
  "appointments_skipped": 0,
  "appointments_cancelled_with_escalation": 1,
  "new_intakeworkitems_created": 3,
  "new_escalations_created": 1,
  "sync_status": "SUCCESS",
  "errors": []
}

Result Status Values:

SUCCESS:
  - All appointments synced without errors
  - Condition: athena API call succeeded AND all appointments processed without skip/error
  - Outcome: log at INFO level

PARTIAL_SUCCESS:
  - Some appointments synced; some skipped or errored
  - Condition: athena API call succeeded BUT some appointments skipped (missing patient, invalid data, etc.)
  - Outcome: log at WARN level; details: which appointments skipped and why

FAILED:
  - athena API call failed; no appointments synced
  - Condition: HTTP 5xx after retries, timeout, or auth error
  - Outcome: log at ERROR level; escalate to operations; no clinic escalation (infrastructure issue)

Errors Array:
  - Each entry: { timestamp, error_code, error_message, appointment_id (if applicable) }
  - Example:
    [
      {
        "timestamp": "2026-04-20T10:35:15Z",
        "error_code": "PATIENT_NOT_FOUND",
        "error_message": "Appointment ATH-2026-04-25-001 skipped; Patient PAT-ATH-00123 not in system",
        "appointment_id": "ATH-2026-04-25-001"
      }
    ]

Logging:
  - Log level: INFO for SUCCESS, WARN for PARTIAL_SUCCESS, ERROR for FAILED
  - Log sync_cycle_id in every log entry (enables tracing all logs from single sync run)
  - Logs retained: 90 days minimum (audit requirement)
  - Log format: structured JSON or key-value pairs for indexing and analysis
```

---

# SECTION 3: FEATURE REQUIREMENT 3 — INSURANCE VERIFICATION

**Objective:** Verify patient insurance eligibility. Deterministic decision: verified or escalated (never guessed). Timeout → escalate after 3 retries.

## FR3.1: When to Invoke

```
[Agent Alone]

Trigger: IntakeWorkItem with status = PENDING and insurance_verification_status = PENDING

For each IntakeWorkItem in this state:
  - Load appointment data (scheduled_datetime, etc.)
  - Invoke FR3 (insurance verification flow)

Determinism guarantee:
  - If FR3 is called twice for same IntakeWorkItem (e.g., first timeout, then retry):
    Same patient data (patient_id, demographics) used in both calls
    Same insurance API endpoint
    No divergence based on external state
    Result depends only on insurance API response, not agent mood or randomness
```

## FR3.2: Insurance Integration Contract

```
[Agent + Log]

Integration: Insurance Eligibility Service (provider TBD; assume generic RESTful API)

Endpoint: POST /api/v1/eligibility/verify

Authentication:
  - Method: Bearer token (OAuth or API key)
  - Token storage: secrets manager, key: INSURANCE_API_KEY
  - Token rotation: per insurance provider contract

Rate Limit: 50 requests/minute per API key
  - If approaching limit, log warning but do not back off (provider owns throttling via 429)

Request Format (JSON):
```json
{
  "patientId": "string (required)",
  "firstName": "string (required)",
  "lastName": "string (required)",
  "dateOfBirth": "string YYYY-MM-DD (required)",
  "memberId": "string (optional; speeds up lookup)",
  "groupNumber": "string (optional)",
  "planName": "string (optional; disambiguates if patient has multiple policies)"
}
```

Example Request:
```json
{
  "patientId": "MRN-00123",
  "firstName": "John",
  "lastName": "Doe",
  "dateOfBirth": "1985-06-15",
  "memberId": "BC-987654321"
}
```

Request Headers:
  - Authorization: Bearer {INSURANCE_API_KEY}
  - Content-Type: application/json
  - User-Agent: PatientIntakeAgent/1.0

Success Response: HTTP 200 OK

Body (JSON):
```json
{
  "status": "ACTIVE",
  "providerId": "BLUE_CROSS_01",
  "referenceId": "ELIGIBILITY-2026-04-20-12345",
  "effectiveDate": "2025-01-01",
  "terminationDate": null,
  "copay": 2500,
  "deductible": 50000,
  "deductibleMet": 15000,
  "coinsurance": 20,
  "outOfPocketMax": 700000,
  "outOfPocketMet": 8000,
  "planType": "PPO",
  "preventiveCareCovered": true
}
```

Status field values:
  - ACTIVE: patient has active coverage; eligible for services
  - INACTIVE: coverage has ended or been terminated
  - PENDING: eligibility is under review; not yet active
  - UNKNOWN: service cannot determine status (ambiguous; requires escalation)

Error Response: HTTP 4xx (client error)

Body (JSON):
```json
{
  "error": {
    "code": "PATIENT_NOT_FOUND",
    "message": "Patient record not found in insurance database"
  }
}
```

Defined Error Codes (4xx):
  - PATIENT_NOT_FOUND: patient not in insurance provider's system (may have expired record)
  - INVALID_MEMBER_ID: member ID format wrong or expired
  - NO_ACTIVE_POLICY: patient has no active insurance (may need new policy added)
  - INVALID_REQUEST: request format wrong or required field missing (should not happen if contract correct)
  - AUTHORIZATION_FAILED: API key invalid or insufficient permissions

Error Response: HTTP 5xx (server error)

Body (JSON):
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Insurance provider service encountered an error"
  }
}
```

Defined Error Codes (5xx):
  - INTERNAL_SERVER_ERROR: server-side error; may be transient
  - SERVICE_UNAVAILABLE: service temporarily down or overloaded

Timeout: 15 seconds (clock starts at request, ends at first byte of response or timeout)

Retry Logic:

HTTP 5xx:
  - Retry up to 2 times with exponential backoff: 3s, then 6s
  - Log each retry: timestamp, attempt number, error details
  - After 2 retries exhausted: mark as FAILED; do not retry further (fail-closed)

HTTP 4xx (excluding 429):
  - Do not retry (error is terminal; e.g., patient not found)
  - Treat as FAILED; see FR3.3 for handling

HTTP 429 (rate limit):
  - Not expected in normal operation (50 req/min is generous for clinic)
  - If occurs: do not retry; log error; escalate to operations

Timeout (no response after 15s):
  - Do not retry immediately; managed by retry_count in FR3.3
  - Mark as TIMEOUT; managed by IntakeWorkItem.insurance_verification_retry_count

Fallback Behavior:
  - If service unavailable: mark as TIMEOUT; do not queue for async retry
  - Clinic needs answer before visit; escalate after 3 retries if still down
```

## FR3.3: Verification Decision Logic

```
[Agent Alone + Log]

Input:
  - IntakeWorkItem.patient_id
  - Associated Appointment data
  - Current IntakeWorkItem.insurance_verification_retry_count

Output:
  - Updated IntakeWorkItem with new insurance_verification_status
  - Escalation created (if needed)
  - AuditLog entries

Execution Flow:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 1: Validate Inputs (Pre-flight)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Load Patient record by IntakeWorkItem.patient_id

Validation checks:

1. Patient record exists
   - If not found: 
     * Set insurance_verification_status = ESCALATED
     * Set insurance_verification_error = "Patient not found in system"
     * Create Escalation:
       - escalation_reason = "Patient not found in system"
       - escalation_category = VERIFICATION_RETRY_EXHAUSTED
       - assigned_to = FRONT_DESK
       - status = OPEN
     * Log: ERROR "Patient {patient_id} not found; escalating"
     * Return (do not call external API)

2. Patient demographics complete
   - Required: firstName, lastName, dateOfBirth
   - If any missing:
     * Set insurance_verification_status = ESCALATED
     * Set insurance_verification_error = "Patient missing required demographics: {list missing fields}"
     * Create Escalation:
       - escalation_reason = "Patient missing required demographics"
       - escalation_category = VERIFICATION_RETRY_EXHAUSTED
       - assigned_to = FRONT_DESK
       - status = OPEN
     * Log: ERROR "Patient {patient_id} missing demographics; escalating"
     * Return (do not call external API)

All checks passed: proceed to Step 2

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 2: Call Insurance API (FR3.2)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Build request:
  - patientId: Patient.medical_record_number
  - firstName: Patient.firstName
  - lastName: Patient.lastName
  - dateOfBirth: Patient.dateOfBirth (YYYY-MM-DD format)
  - memberId: Patient.insuranceMemberId (if available; optional)
  - groupNumber: Patient.insuranceGroupNumber (if available; optional)
  - planName: Patient.insurancePlanName (if available; optional)

Execute POST /api/v1/eligibility/verify
  - Timeout: 15 seconds
  - Retry logic: per FR3.2

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 3A: API SUCCESS (HTTP 200)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Insurance API returned eligibility response.

Parse response.status field:

[Case A] status = ACTIVE
─────────────────────────────────────
  Patient is eligible for services.

  Update IntakeWorkItem:
    - insurance_verification_status = VERIFIED
    - insurance_provider_id = response.providerId
    - insurance_reference_id = response.referenceId
    - insurance_verification_error = null (clear any prior error)
    - insurance_verification_timestamp = current UTC timestamp
    - insurance_verification_retry_count = unchanged (no retry needed)

  Log all response fields for audit trail:
    - copay, deductible, deductibleMet, coinsurance, outOfPocketMax, outOfPocketMet
    - planType, preventiveCareCovered, effectiveDate, terminationDate
    - log level: INFO
    - message: "Insurance verification succeeded for patient {patient_id}; status ACTIVE"

  No escalation needed.

  Attempt transition: IntakeWorkItem.status → READY
    - Caller can now transition intake to READY (if all other verifications passed)
    - Log: "IntakeWorkItem {workitem_id} ready for intake"

  Return: success


[Case B] status = INACTIVE
─────────────────────────────────────
  Patient's insurance is no longer active; ambiguous whether patient has alternative coverage.

  Update IntakeWorkItem:
    - insurance_verification_status = FAILED
    - insurance_verification_error = "Insurance status is INACTIVE; not ACTIVE"
    - insurance_verification_timestamp = current UTC timestamp
    - Log: WARN "Insurance verification returned INACTIVE for patient {patient_id}; escalating"

  Create Escalation:
    - escalation_reason = "Insurance eligibility returned INACTIVE; requires human review (patient may have new policy or uninsured)"
    - escalation_category = INSURANCE_FAILED
    - assigned_to = FRONT_DESK
    - status = OPEN
    - sla_deadline = current UTC + 4 hours
    - Log: "Escalation created for workitem {workitem_id}; reason INSURANCE_FAILED"

  Update IntakeWorkItem.status:
    - status = ESCALATED
    - escalated_at = current UTC timestamp
    - escalated_to = FRONT_DESK
    - escalation_reason = (from Escalation record)

  Return: escalated


[Case C] status = PENDING
─────────────────────────────────────
  Insurance eligibility is under review; not yet active.

  Same handling as INACTIVE:
    - insurance_verification_status = FAILED
    - insurance_verification_error = "Insurance status is PENDING; not yet ACTIVE"
    - Create Escalation with category = INSURANCE_FAILED
    - status = ESCALATED
    - Log: "Insurance verification returned PENDING; escalating"

  Return: escalated


[Case D] status = UNKNOWN
─────────────────────────────────────
  Insurance service cannot determine eligibility status (ambiguous).

  Update IntakeWorkItem:
    - insurance_verification_status = FAILED
    - insurance_verification_error = "Insurance service returned UNKNOWN status; ambiguous eligibility"
    - Log: WARN "Insurance verification returned UNKNOWN; escalating"

  Create Escalation:
    - escalation_reason = "Insurance service returned UNKNOWN status; manual verification required"
    - escalation_category = INSURANCE_FAILED
    - assigned_to = FRONT_DESK
    - status = OPEN

  Update IntakeWorkItem.status = ESCALATED

  Return: escalated

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 3B: API ERROR (HTTP 4xx)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Insurance API returned explicit error.

Parse response.error.code:

[Case A] error.code = PATIENT_NOT_FOUND
──────────────────────────────────────────
  Patient not in insurance provider's system (may have expired record, data mismatch).

  Update IntakeWorkItem:
    - insurance_verification_status = FAILED
    - insurance_verification_error = "PATIENT_NOT_FOUND from insurance provider"
    - insurance_verification_timestamp = current UTC timestamp
    - Log: WARN "Insurance verification returned PATIENT_NOT_FOUND; escalating"

  Create Escalation:
    - escalation_reason = "Patient not found in insurance provider system; may have expired record or data mismatch; verify patient identity"
    - escalation_category = INSURANCE_FAILED
    - assigned_to = FRONT_DESK
    - status = OPEN

  Update IntakeWorkItem.status = ESCALATED

  Return: escalated


[Case B] error.code = INVALID_MEMBER_ID
─────────────────────────────────────────
  Member ID format wrong, expired, or not recognized.

  Update IntakeWorkItem:
    - insurance_verification_status = FAILED
    - insurance_verification_error = "INVALID_MEMBER_ID from insurance provider"
    - Log: WARN "Insurance verification returned INVALID_MEMBER_ID; escalating"

  Create Escalation:
    - escalation_reason = "Insurance member ID invalid or expired; patient must verify ID or update policy"
    - escalation_category = INSURANCE_FAILED
    - assigned_to = FRONT_DESK
    - status = OPEN

  Update IntakeWorkItem.status = ESCALATED

  Return: escalated


[Case C] error.code = NO_ACTIVE_POLICY
──────────────────────────────────────────
  Patient has no active insurance policy with this provider (may need new policy or different provider).

  Update IntakeWorkItem:
    - insurance_verification_status = FAILED
    - insurance_verification_error = "NO_ACTIVE_POLICY from insurance provider"
    - Log: WARN "Insurance verification returned NO_ACTIVE_POLICY; escalating"

  Create Escalation:
    - escalation_reason = "Patient has no active insurance policy; verify patient or add new policy"
    - escalation_category = INSURANCE_FAILED
    - assigned_to = FRONT_DESK
    - status = OPEN

  Update IntakeWorkItem.status = ESCALATED

  Return: escalated


[Case D] error.code = AUTHORIZATION_FAILED
─────────────────────────────────────────────
  API key invalid or credentials insufficient (system error, not patient-related).

  Update IntakeWorkItem:
    - insurance_verification_status = ESCALATED (not FAILED; this is system error)
    - insurance_verification_error = "AUTHORIZATION_FAILED; check INSURANCE_API_KEY in secrets"
    - Log: CRITICAL "Insurance API authorization failed; check credentials"

  Do NOT create Escalation to FRONT_DESK (this is operations issue).
  Escalate to operations team with alert "Insurance API credentials expired or invalid"

  Return: system error (escalate to operations, not clinic)


[Case E] error.code = INVALID_REQUEST
───────────────────────────────────────
  Request format wrong or required field missing (should not happen if contract correct).

  Update IntakeWorkItem:
    - insurance_verification_status = ESCALATED
    - insurance_verification_error = "INVALID_REQUEST from insurance provider; check request format against contract"
    - Log: ERROR "Insurance API returned INVALID_REQUEST; likely spec mismatch"

  Escalate to operations (development team must review API contract).

  Return: system error

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 3C: API TIMEOUT (No Response After 15s)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Insurance API did not respond within 15-second timeout window.

Update IntakeWorkItem:
  - insurance_verification_status = TIMEOUT
  - insurance_verification_error = "TIMEOUT_AFTER_15s"
  - insurance_verification_timestamp = current UTC timestamp
  - Log: WARN "Insurance verification timeout; patient_id {patient_id}"

Check retry count:
  - Current: IntakeWorkItem.insurance_verification_retry_count

[Retry available: count < 3]
──────────────────────────────
  - Increment insurance_verification_retry_count (e.g., 0 → 1)
  - Update IntakeWorkItem.status = HOLD
    (await next retry; no escalation yet)
  - Log: "Insurance timeout attempt {retry_count}; retry available; status → HOLD"
  - Return: hold (will retry on next FR3 invocation)


[Retries exhausted: count >= 3]
────────────────────────────────
  - Increment insurance_verification_retry_count to 3 (if not already)
  - Update IntakeWorkItem:
    - status = ESCALATED (no more retries)
    - escalated_at = current UTC timestamp
    - escalated_to = FRONT_DESK
    - escalation_reason = "Insurance verification timeout after 3 retries; clinic may proceed without verification or contact provider"
  
  - Create Escalation:
    - escalation_reason = "Insurance verification timeout after 3 retries (15s each)"
    - escalation_category = INSURANCE_TIMEOUT
    - assigned_to = FRONT_DESK
    - status = OPEN
    - sla_deadline = current UTC + 4 hours
  
  - Log: "Insurance timeout exhausted retries; status → ESCALATED; escalation_id {escalation_id}"
  - Return: escalated

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 3D: API ERROR (HTTP 5xx)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Insurance API returned server error (500, 502, 503, 504).

Already retried up to 2 times (per FR3.2 retry logic).
Now decide whether to hold or escalate based on attempt count.

Update IntakeWorkItem:
  - insurance_verification_status = FAILED (treated as non-retryable; fail-closed)
  - insurance_verification_error = "Service unavailable (HTTP 5xx); insurance provider down"
  - insurance_verification_timestamp = current UTC timestamp
  - Log: ERROR "Insurance API returned 5xx; service unavailable"

Check retry count:

[Retry available: count < 3]
──────────────────────────────
  - Increment insurance_verification_retry_count
  - Update IntakeWorkItem.status = HOLD
  - Log: "Insurance service unavailable attempt {retry_count}; retry available; status → HOLD"
  - Return: hold (retry on next FR3 invocation)


[Retries exhausted: count >= 3]
────────────────────────────────
  - Update IntakeWorkItem:
    - status = ESCALATED
    - escalated_at = current UTC timestamp
    - escalated_to = FRONT_DESK
    - escalation_reason = "Insurance service unavailable after 3 retries; manual verification required"
  
  - Create Escalation:
    - escalation_reason = "Insurance provider service unavailable after 3 attempts; contact provider support"
    - escalation_category = INSURANCE_TIMEOUT (service failure, not data failure)
    - assigned_to = FRONT_DESK
    - status = OPEN
  
  - Log: "Insurance service unavailable; retries exhausted; escalating"
  - Return: escalated

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 4: Log Decision
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All outcomes logged immutably to AuditLog:

Log entry structure:
  - id: new UUID
  - timestamp: current UTC timestamp
  - agent_id: "INSURANCE_VERIFY_AGENT"
  - action: one of [INSURANCE_VERIFY_START, INSURANCE_VERIFY_SUCCESS, INSURANCE_VERIFY_TIMEOUT, INSURANCE_VERIFY_FAILURE]
  - entity_type: "INTAKEWORKITEM"
  - entity_id: IntakeWorkItem.id
  - outcome: one of [SUCCESS, HOLD, ESCALATED, FAILURE]
  - details: {
      workitem_id: IntakeWorkItem.id,
      patient_id: Patient.id,
      verification_status: IntakeWorkItem.insurance_verification_status,
      retry_count: IntakeWorkItem.insurance_verification_retry_count,
      error: IntakeWorkItem.insurance_verification_error,
      action_taken: outcome (VERIFIED / HOLD / ESCALATED)
    }

Retention: 90 days minimum; 1 year preferred

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 5: Update IntakeWorkItem
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All updates made in prior steps (Steps 3A–3D).

Summary of possible final states:

State: READY
  - insurance_verification_status = VERIFIED
  - No escalation
  - Patient ready for intake
  - Caller can proceed with appointment workflow

State: HOLD
  - insurance_verification_status = TIMEOUT or FAILED
  - insurance_verification_retry_count < 3
  - No escalation (retries available)
  - Workflow awaits next FR3 invocation

State: ESCALATED
  - insurance_verification_status = TIMEOUT or FAILED
  - insurance_verification_retry_count >= 3 OR data validation failed
  - Escalation created and assigned to FRONT_DESK
  - Human intervention required before intake can proceed

Update timestamp fields:
  - updated_at = current UTC timestamp
  - updated_by = "INSURANCE_VERIFY_AGENT"
```

---

# SECTION 4: AUDIT LOGGING

All agent actions logged immutably for compliance and traceability.

## AuditLog Entity

```
Entity: AuditLog (immutable)
Primary Key: id (UUID, immutable)
Index: timestamp, agent_id, entity_type, action (for efficient queries)

Attributes:

id: UUID
  - Primary key, immutable
  - Example: "990e8400-e29b-41d4-a716-446655440004"

timestamp: ISO 8601 timestamp UTC
  - Immutable, set at log creation
  - Example: "2026-04-20T10:35:15Z"

agent_id: string
  - Identifies which agent performed the action
  - Example: "ATHENA_SYNC_AGENT" or "INSURANCE_VERIFY_AGENT"

action: enum (below)
  - Type of action performed
  - Examples: APPOINTMENT_CREATED, INSURANCE_VERIFY_TIMEOUT, ESCALATION_CREATED

entity_type: enum
  - Type of entity affected
  - Values: APPOINTMENT, INTAKEWORKITEM, ESCALATION, SYNC_CYCLE

entity_id: UUID
  - Primary key of affected entity
  - Example: "550e8400-e29b-41d4-a716-446655440000"

details: JSON object (action-specific)
  - Free-form data; structure varies by action
  - Examples below

outcome: enum [SUCCESS, FAILURE, HOLD, ESCALATED]
  - Terminal result of action
  - Used for filtering and analytics

Retention: 90 days minimum; 1 year preferred (medical records compliance)
Immutability: once created, never modified or deleted
Index strategy: timestamp, agent_id, entity_type for efficient queries

Defined Actions:

[FR1: Appointment Sync]

action: APPOINTMENT_SYNC_START
  details: {
    "sync_cycle_id": "uuid",
    "started_at": "ISO 8601",
    "clinic_locations": ["LOC-MAIN", "LOC-URGENT"]
  }

action: APPOINTMENT_SYNC_COMPLETE
  details: {
    "sync_cycle_id": "uuid",
    "started_at": "ISO 8601",
    "completed_at": "ISO 8601",
    "duration_seconds": 45,
    "appointments_fetched": 12,
    "appointments_created": 3,
    "appointments_updated": 2,
    "appointments_skipped": 0,
    "new_intakeworkitems_created": 3,
    "new_escalations_created": 0,
    "sync_status": "SUCCESS"
  }

action: APPOINTMENT_SYNC_ERROR
  details: {
    "sync_cycle_id": "uuid",
    "error_code": "HTTP_500",
    "error_message": "athenahealth API temporarily unavailable",
    "retry_count": 1,
    "next_retry_at": "ISO 8601"
  }

action: APPOINTMENT_CREATED
  details: {
    "appointment_id": "uuid",
    "athena_appointment_id": "ATH-2026-04-25-001",
    "patient_id": "uuid",
    "scheduled_datetime": "2026-04-25T14:00:00Z",
    "appointment_type": "ROUTINE",
    "status": "SCHEDULED",
    "intakeworkitem_id": "uuid"
  }

action: APPOINTMENT_UPDATED
  details: {
    "appointment_id": "uuid",
    "athena_appointment_id": "ATH-2026-04-25-001",
    "status_before": "SCHEDULED",
    "status_after": "CONFIRMED",
    "reason": "athena sync"
  }

action: APPOINTMENT_SKIPPED
  details: {
    "athena_appointment_id": "ATH-2026-04-25-001",
    "patient_id": "uuid (from athena)",
    "reason": "Patient not found in system"
  }

[FR3: Insurance Verification]

action: INSURANCE_VERIFY_START
  details: {
    "workitem_id": "uuid",
    "patient_id": "uuid",
    "attempt": 1,
    "started_at": "ISO 8601"
  }

action: INSURANCE_VERIFY_SUCCESS
  details: {
    "workitem_id": "uuid",
    "patient_id": "uuid",
    "reference_id": "ELIGIBILITY-2026-04-20-12345",
    "provider_id": "BLUE_CROSS_01",
    "copay": 2500,
    "deductible": 50000,
    "plan_type": "PPO"
  }

action: INSURANCE_VERIFY_TIMEOUT
  details: {
    "workitem_id": "uuid",
    "patient_id": "uuid",
    "attempt": 1,
    "timeout_seconds": 15,
    "action_taken": "HOLD"
  }

action: INSURANCE_VERIFY_FAILURE
  details: {
    "workitem_id": "uuid",
    "patient_id": "uuid",
    "error_code": "PATIENT_NOT_FOUND",
    "error_message": "Patient record not found in insurance database",
    "attempt": 1,
    "action_taken": "ESCALATED"
  }

action: RETRY_ATTEMPT
  details: {
    "workitem_id": "uuid",
    "entity_type": "INTAKEWORKITEM",
    "reason": "INSURANCE_TIMEOUT",
    "retry_count": 2,
    "next_retry_at": "ISO 8601"
  }

[Escalations]

action: ESCALATION_CREATED
  details: {
    "escalation_id": "uuid",
    "workitem_id": "uuid",
    "patient_id": "uuid",
    "appointment_id": "uuid",
    "escalation_category": "INSURANCE_TIMEOUT",
    "assigned_to": "FRONT_DESK",
    "reason": "Insurance verification timeout after 3 retries",
    "sla_deadline": "ISO 8601"
  }

action: INTAKEWORKITEM_CREATED
  details: {
    "workitem_id": "uuid",
    "appointment_id": "uuid",
    "patient_id": "uuid",
    "status": "PENDING",
    "trigger": "APPOINTMENT_SYNC"
  }

action: INTAKEWORKITEM_UPDATED
  details: {
    "workitem_id": "uuid",
    "status_before": "PENDING",
    "status_after": "ESCALATED",
    "reason": "Insurance verification exhausted retries",
    "updated_by": "INSURANCE_VERIFY_AGENT"
  }
```

---

# SECTION 5: VALIDATION SCENARIOS

Two test scenarios drive implementation validation. Both must pass before handoff to Claude.

## Scenario V1: Happy Path (Active Insurance → Ready)

**Setup:**

```
Preconditions:
- Appointment synced from athena: 
  - id: ATH-2026-04-25-001
  - patient: Jane Smith (PAT-ATH-00123)
  - scheduled: 2026-04-25 14:00 UTC, ROUTINE
  - status: Scheduled
- Patient exists in system:
  - firstName: Jane
  - lastName: Smith
  - dateOfBirth: 1985-06-15
  - insuranceMemberId: BC-987654321
- Insurance provider returns HTTP 200 with status=ACTIVE:
  - copay: $25 (2500 cents)
  - deductible: $500 (50000 cents)
  - planType: PPO
```

**Expected Execution:**

```
Step 1: Appointment Sync (FR1)
  - FR1.2 fetches appointment from athena
  - FR1.3 creates Appointment record
    - appointment_id: (new UUID)
    - athena_appointment_id: ATH-2026-04-25-001
    - patient_id: (Jane Smith's UUID)
    - scheduled_datetime: 2026-04-25T14:00:00Z
    - status: SCHEDULED
    - sync_status: SYNCED
    - AuditLog: action=APPOINTMENT_CREATED
  
  - FR1.3 creates IntakeWorkItem
    - workitem_id: (new UUID)
    - appointment_id: (from above)
    - status: PENDING
    - insurance_verification_status: PENDING
    - AuditLog: action=INTAKEWORKITEM_CREATED

Step 2: Insurance Verification (FR3)
  - FR3.1 detects IntakeWorkItem.status=PENDING
  - FR3.3 STEP 1: Validate inputs
    - Load Patient → found, demographics complete ✓
  
  - FR3.3 STEP 2: Call insurance API
    - POST /api/v1/eligibility/verify
    - Request: patientId=MRN-123, firstName=Jane, lastName=Smith, dateOfBirth=1985-06-15
    - Response: HTTP 200, status=ACTIVE
    - AuditLog: action=INSURANCE_VERIFY_START
  
  - FR3.3 STEP 3A: Process success response
    - response.status = ACTIVE
    - Update IntakeWorkItem:
      - insurance_verification_status = VERIFIED
      - insurance_reference_id = ELIGIBILITY-2026-04-20-12345
      - insurance_provider_id = BLUE_CROSS_01
      - insurance_verification_timestamp = now
      - insurance_verification_error = null
    
    - Log all response fields for audit
    - AuditLog: action=INSURANCE_VERIFY_SUCCESS, outcome=SUCCESS

Step 3: Intake Ready
  - IntakeWorkItem transitions: status = READY
  - AuditLog: action=INTAKEWORKITEM_UPDATED
  - Escalations: [] (empty; no escalation needed)

Final State:
  - Appointment.status = SCHEDULED
  - IntakeWorkItem.status = READY
  - insurance_verification_status = VERIFIED
  - Escalations: 0
  - Patient is ready for check-in; no human escalation needed
  
Audit Trail:
  - APPOINTMENT_CREATED (appointment synced)
  - INTAKEWORKITEM_CREATED (intake workflow started)
  - INSURANCE_VERIFY_START (verification attempt)
  - INSURANCE_VERIFY_SUCCESS (eligibility confirmed)
  - INTAKEWORKITEM_UPDATED (status → READY)

Pass Criteria:
  ✓ Appointment created with correct athena reference
  ✓ IntakeWorkItem created and set to PENDING
  ✓ Insurance API called successfully
  ✓ Response parsed and fields populated
  ✓ IntakeWorkItem transitioned to READY
  ✓ No Escalations created
  ✓ All AuditLog entries present and correct
```

---

## Scenario V3: Timeout (Insurance Timeout → Hold → Escalate on Retry Exhaustion)

**Setup:**

```
Preconditions:
- Same as V1 (Appointment synced, Patient exists)
- Insurance provider API: times out (no response after 15s, all 3 retries timeout)
```

**Expected Execution:**

```
Attempt 1: Timeout
────────────────────

FR3.3 STEP 2: Call insurance API
  - POST /api/v1/eligibility/verify
  - No response; timeout after 15s
  - AuditLog: action=INSURANCE_VERIFY_START

FR3.3 STEP 3C: Process timeout
  - insurance_verification_status = TIMEOUT
  - insurance_verification_error = "TIMEOUT_AFTER_15s"
  - Check retry_count: 0 < 3 → retries available
  - Increment retry_count: 0 → 1
  - Update IntakeWorkItem:
    - status = HOLD
    - insurance_verification_retry_count = 1
    - insurance_verification_timestamp = now

  - AuditLog: action=INSURANCE_VERIFY_TIMEOUT, outcome=HOLD
  - Escalations: [] (no escalation yet; retries available)

Final State (After Attempt 1):
  - IntakeWorkItem.status = HOLD
  - insurance_verification_status = TIMEOUT
  - retry_count = 1
  - No Escalations


Attempt 2: Timeout Again
────────────────────────

(Time passes; next FR3 invocation triggered)

FR3.3 STEP 2: Call insurance API again
  - POST /api/v1/eligibility/verify
  - No response; timeout after 15s
  - AuditLog: action=INSURANCE_VERIFY_START

FR3.3 STEP 3C: Process timeout
  - Check retry_count: 1 < 3 → retries available
  - Increment retry_count: 1 → 2
  - Update IntakeWorkItem:
    - status = HOLD (still)
    - insurance_verification_retry_count = 2

  - AuditLog: action=INSURANCE_VERIFY_TIMEOUT, outcome=HOLD
  - Escalations: [] (no escalation yet)


Attempt 3: Timeout Third Time, Retries Exhausted
──────────────────────────────────────────────────

(Time passes; next FR3 invocation triggered)

FR3.3 STEP 2: Call insurance API third time
  - POST /api/v1/eligibility/verify
  - No response; timeout after 15s
  - AuditLog: action=INSURANCE_VERIFY_START

FR3.3 STEP 3C: Process timeout
  - Check retry_count: 2 < 3 → one more check
  - After processing this timeout: retry_count will be 3
  - Retries exhausted: 3 >= 3
  - Update IntakeWorkItem:
    - insurance_verification_retry_count = 3
    - status = ESCALATED (no more retries)
    - escalated_at = current UTC timestamp
    - escalated_to = FRONT_DESK
    - escalation_reason = "Insurance verification timeout after 3 retries"

  - Create Escalation:
    - escalation_id: (new UUID)
    - workitem_id: (from IntakeWorkItem)
    - escalation_category = INSURANCE_TIMEOUT
    - assigned_to = FRONT_DESK
    - status = OPEN
    - sla_deadline = now + 4 hours
    - AuditLog: action=ESCALATION_CREATED

  - AuditLog: action=INSURANCE_VERIFY_TIMEOUT, outcome=ESCALATED

Final State (After Attempt 3):
  - IntakeWorkItem.status = ESCALATED
  - insurance_verification_status = TIMEOUT
  - retry_count = 3
  - escalated_at = set
  - Escalations: 1 open escalation to FRONT_DESK

Audit Trail (Complete):
  - INTAKEWORKITEM_CREATED (initial)
  - INSURANCE_VERIFY_START (attempt 1)
  - INSURANCE_VERIFY_TIMEOUT (attempt 1, outcome=HOLD)
  - RETRY_ATTEMPT (prep for attempt 2)
  - INSURANCE_VERIFY_START (attempt 2)
  - INSURANCE_VERIFY_TIMEOUT (attempt 2, outcome=HOLD)
  - RETRY_ATTEMPT (prep for attempt 3)
  - INSURANCE_VERIFY_START (attempt 3)
  - INSURANCE_VERIFY_TIMEOUT (attempt 3, outcome=ESCALATED)
  - INTAKEWORKITEM_UPDATED (status → ESCALATED)
  - ESCALATION_CREATED (assigned to FRONT_DESK)

Pass Criteria:
  ✓ Attempt 1: timeout detected; status → HOLD; retry_count = 1; no escalation
  ✓ Attempt 2: timeout detected; status → HOLD; retry_count = 2; no escalation
  ✓ Attempt 3: timeout detected; retry_count = 3; status → ESCALATED
  ✓ Escalation created with category=INSURANCE_TIMEOUT, assigned_to=FRONT_DESK, status=OPEN
  ✓ SLA deadline set to now + 4 hours
  ✓ All AuditLog entries present; timestamps correct
  ✓ FRONT_DESK can see escalation and decide whether to verify manually or proceed without
```

---

## Validation Checklist

Before building, verify this spec passes these checks:

**Buildability:**
- [ ] Every requirement includes explicit acceptance criteria
- [ ] No vague words: "appropriate," "recent," "soon," "large," "if needed"
- [ ] All state transitions explicit with no ambiguous paths
- [ ] Integration contract complete: endpoint, auth, request/response, timeout, retry logic
- [ ] All edge cases named and handled

**Clinical Boundary:**
- [ ] No diagnosis, treatment, or medical triage delegated to agent
- [ ] Agent limited to: data plumbing, documentation, completeness checks, policy-based escalation triggers
- [ ] All clinical judgment preserved for human staff

**Determinism:**
- [ ] Same inputs produce same outputs (no randomness, no external state dependencies)
- [ ] Idempotent: same API call twice produces same result, no double-writes
- [ ] Timeout always treated consistently (no guessing on partial responses)

**Fail-Closed:**
- [ ] Ambiguity always escalates (not ignored or auto-resolved)
- [ ] Timeout after N retries always escalates (N=3 for FR3)
- [ ] All errors logged immutably for audit trail

**Audit & Governance:**
- [ ] Every action logged with timestamp, agent_id, entity_id, outcome
- [ ] Logs immutable; 90-day retention minimum
- [ ] Escalations tracked with SLA (4-hour acknowledgement)
- [ ] Human decisions captured and timestamped

---

## Ready for Build?

**Go-ahead criteria:**

1. ✓ Data model clear (state machines, constraints, immutability rules)
2. ✓ FR1 (Appointment Sync) fully specified with retry logic and idempotence guarantee
3. ✓ FR3 (Insurance Verification) fully specified with fail-closed escalation logic
4. ✓ Audit logging defined for all actions
5. ✓ Two test scenarios (V1 happy path, V3 timeout exhaustion) fully traced
6. ✓ Clinical boundaries enforced (no clinical judgment delegated)

**Next step:** Pass this file to Claude Code with build instructions (CLAUDE.md context file).

---

**Document Version:** 1.0  
**Last Updated:** 2026-04-20  
**Author:** FDE Program Delivery Assistant  
**Status:** Ready for Build
