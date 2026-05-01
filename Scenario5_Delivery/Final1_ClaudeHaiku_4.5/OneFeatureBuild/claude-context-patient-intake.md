# Claude Code Context: Patient Intake Coordination Agent

**Purpose:** Build-ready instructions for Claude Code to implement Patient Intake Coordination Agent (FR1 + FR3).

**Source of Truth:** `Patient-Intake-Coordination-Agent-Spec.md` — consult that file for any ambiguity.

---

## QUICK START

You are building a deterministic, fail-closed patient intake coordination system for a small clinic. Two features:

1. **FR1: Appointment Sync** — Fetch appointments from athenahealth every 15 minutes (idempotent)
2. **FR3: Insurance Verification** — Verify insurance eligibility (deterministic: VERIFIED or ESCALATED, never guessed)

**Non-negotiables:**
- No clinical judgment anywhere (diagnosis, treatment, triage remain human-only)
- All actions logged immutably (90-day retention)
- Timeout always escalates after 3 retries (fail-closed)
- Same inputs always produce same outputs (deterministic)

---

## PART 1: TYPE DEFINITIONS & SCHEMAS

Use these schemas as the authoritative data model. Implement in your language of choice (TypeScript, Python, Go).

### Appointment

```typescript
// Core entity: canonical scheduled visit record
interface Appointment {
  id: string; // UUID, primary key
  athena_appointment_id: string; // Immutable, unique, external reference
  patient_id: string; // UUID, immutable foreign key
  scheduled_datetime: string; // ISO 8601 UTC, immutable
  provider_id: string; // immutable
  location_id: string; // immutable
  appointment_type: "ROUTINE" | "URGENT" | "FOLLOWUP" | "PREVENTIVE"; // immutable
  status: "SCHEDULED" | "CONFIRMED" | "COMPLETED" | "CANCELLED" | "NO_SHOW";
  intake_workitem_id: string | null; // UUID, mutable once set (immutable after)
  sync_status: "SYNCED" | "SYNC_PENDING" | "SYNC_ERROR";
  created_at: string; // ISO 8601 UTC, immutable
  updated_at: string; // ISO 8601 UTC, mutable (only on status/sync_status change)
  last_sync_at: string; // ISO 8601 UTC
  synced_by: string; // "ATHENA_SYNC_AGENT", immutable
  last_sync_error: string | null; // max 500 chars, cleared on success

  // Validation constraints (enforce in code)
  // - athena_appointment_id is globally unique (UNIQUE constraint in DB)
  // - scheduled_datetime > created_at
  // - Cannot modify: patient_id, athena_appointment_id, scheduled_datetime, appointment_type after creation
  // - If status = CANCELLED, associated IntakeWorkItem must be archived
}

// Database schema (SQL/NoSQL equivalent)
CREATE TABLE appointments (
  id UUID PRIMARY KEY,
  athena_appointment_id VARCHAR(255) NOT NULL UNIQUE,
  patient_id UUID NOT NULL,
  scheduled_datetime TIMESTAMP NOT NULL,
  provider_id VARCHAR(100) NOT NULL,
  location_id VARCHAR(100) NOT NULL,
  appointment_type VARCHAR(20) NOT NULL,
  status VARCHAR(20) NOT NULL,
  intake_workitem_id UUID,
  sync_status VARCHAR(20) NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  last_sync_at TIMESTAMP NOT NULL,
  synced_by VARCHAR(100),
  last_sync_error TEXT,
  FOREIGN KEY (patient_id) REFERENCES patients(id),
  FOREIGN KEY (intake_workitem_id) REFERENCES intake_workitems(id),
  INDEX (athena_appointment_id),
  INDEX (patient_id),
  INDEX (sync_status)
);
```

### IntakeWorkItem

```typescript
// Workflow tracking: tracks administrative intake for one appointment
interface IntakeWorkItem {
  id: string; // UUID, primary key
  appointment_id: string; // UUID, immutable, UNIQUE per appointment
  patient_id: string; // UUID, immutable
  status: "PENDING" | "READY" | "ESCALATED" | "HOLD" | "COMPLETED" | "ARCHIVED";
  
  appointment_sync_error: string | null; // max 500 chars
  insurance_verification_status: "PENDING" | "VERIFIED" | "FAILED" | "TIMEOUT" | "ESCALATED";
  insurance_verification_timestamp: string | null; // ISO 8601 UTC
  insurance_provider_id: string | null;
  insurance_reference_id: string | null; // max 200 chars
  insurance_verification_error: string | null; // max 500 chars
  insurance_verification_retry_count: number; // 0-3, capped at 3
  
  created_at: string; // ISO 8601 UTC, immutable
  updated_at: string; // ISO 8601 UTC, mutable
  created_by: string; // "ATHENA_SYNC_AGENT", immutable
  updated_by: string; // agent_id or "FRONT_DESK_OVERRIDE"
  
  escalated_at: string | null; // ISO 8601 UTC, immutable once set
  escalated_to: "FRONT_DESK" | null;
  escalation_reason: string | null; // max 500 chars

  // Validation constraints (enforce in code)
  // - Once escalated_at is set, it is immutable (no "unescalate")
  // - insurance_verification_retry_count capped at 3
  // - If retry_count >= 3 AND status = PENDING: must transition to ESCALATED
  // - If status = HOLD, must have insurance_verification_error populated
  // - Cannot transition to READY unless insurance_verification_status = VERIFIED
  // - If Appointment becomes CANCELLED, this must transition to ARCHIVED
}

// Database schema
CREATE TABLE intake_workitems (
  id UUID PRIMARY KEY,
  appointment_id UUID NOT NULL UNIQUE,
  patient_id UUID NOT NULL,
  status VARCHAR(20) NOT NULL,
  
  appointment_sync_error TEXT,
  insurance_verification_status VARCHAR(20) NOT NULL,
  insurance_verification_timestamp TIMESTAMP,
  insurance_provider_id VARCHAR(100),
  insurance_reference_id VARCHAR(200),
  insurance_verification_error TEXT,
  insurance_verification_retry_count INT NOT NULL DEFAULT 0,
  
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  created_by VARCHAR(100) NOT NULL,
  updated_by VARCHAR(100),
  
  escalated_at TIMESTAMP,
  escalated_to VARCHAR(50),
  escalation_reason TEXT,
  
  FOREIGN KEY (appointment_id) REFERENCES appointments(id),
  FOREIGN KEY (patient_id) REFERENCES patients(id),
  INDEX (status),
  INDEX (patient_id),
  INDEX (escalated_at)
);
```

### Escalation

```typescript
// Audit trail: immutable record of all escalations to human teams
interface Escalation {
  id: string; // UUID, primary key
  workitem_id: string; // UUID, immutable, UNIQUE per workitem
  patient_id: string; // UUID, immutable
  appointment_id: string; // UUID, immutable
  
  escalation_reason: string; // max 500 chars, immutable
  escalation_category: 
    | "INSURANCE_TIMEOUT"
    | "INSURANCE_FAILED"
    | "APPOINTMENT_SYNC_FAILED"
    | "VERIFICATION_RETRY_EXHAUSTED"
    | "MANUAL_OVERRIDE";
  
  assigned_to: "FRONT_DESK"; // immutable
  status: "OPEN" | "IN_PROGRESS" | "RESOLVED" | "CANCELLED";
  
  human_decision: string | null; // max 500 chars, set on RESOLVED
  human_decision_timestamp: string | null; // ISO 8601 UTC, immutable once set
  resolved_by: string | null; // user_id, immutable once set
  
  created_at: string; // ISO 8601 UTC, immutable
  updated_at: string; // ISO 8601 UTC
  sla_deadline: string; // ISO 8601 UTC, immutable (created_at + 4 hours)
  sla_breached: boolean; // default false, set by nightly audit

  // Validation constraints (enforce in code)
  // - workitem_id is globally unique (UNIQUE in DB)
  // - Cannot set human_decision or resolved_by unless status = RESOLVED
  // - Immutable fields: all except status, human_decision, resolved_by, updated_at, sla_breached
}

// Database schema
CREATE TABLE escalations (
  id UUID PRIMARY KEY,
  workitem_id UUID NOT NULL UNIQUE,
  patient_id UUID NOT NULL,
  appointment_id UUID NOT NULL,
  
  escalation_reason TEXT NOT NULL,
  escalation_category VARCHAR(50) NOT NULL,
  
  assigned_to VARCHAR(50) NOT NULL,
  status VARCHAR(20) NOT NULL,
  
  human_decision TEXT,
  human_decision_timestamp TIMESTAMP,
  resolved_by VARCHAR(100),
  
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  sla_deadline TIMESTAMP NOT NULL,
  sla_breached BOOLEAN NOT NULL DEFAULT false,
  
  FOREIGN KEY (workitem_id) REFERENCES intake_workitems(id),
  FOREIGN KEY (patient_id) REFERENCES patients(id),
  FOREIGN KEY (appointment_id) REFERENCES appointments(id),
  INDEX (status),
  INDEX (assigned_to),
  INDEX (sla_deadline)
);
```

### AuditLog

```typescript
// Immutable audit trail of all agent actions
interface AuditLog {
  id: string; // UUID, primary key
  timestamp: string; // ISO 8601 UTC, immutable
  agent_id: string; // "ATHENA_SYNC_AGENT", "INSURANCE_VERIFY_AGENT"
  action: string; // see action constants below
  entity_type: "APPOINTMENT" | "INTAKEWORKITEM" | "ESCALATION" | "SYNC_CYCLE";
  entity_id: string; // UUID or sync_cycle_id
  details: Record<string, any>; // JSON object, action-specific
  outcome: "SUCCESS" | "FAILURE" | "HOLD" | "ESCALATED";

  // Validation constraints (enforce in code)
  // - Immutable: once created, never modified or deleted
  // - Retention: 90 days minimum; 1 year preferred
}

// Defined action constants
export const AuditActions = {
  APPOINTMENT_SYNC_START: "APPOINTMENT_SYNC_START",
  APPOINTMENT_SYNC_COMPLETE: "APPOINTMENT_SYNC_COMPLETE",
  APPOINTMENT_SYNC_ERROR: "APPOINTMENT_SYNC_ERROR",
  APPOINTMENT_CREATED: "APPOINTMENT_CREATED",
  APPOINTMENT_UPDATED: "APPOINTMENT_UPDATED",
  APPOINTMENT_SKIPPED: "APPOINTMENT_SKIPPED",
  
  INSURANCE_VERIFY_START: "INSURANCE_VERIFY_START",
  INSURANCE_VERIFY_SUCCESS: "INSURANCE_VERIFY_SUCCESS",
  INSURANCE_VERIFY_TIMEOUT: "INSURANCE_VERIFY_TIMEOUT",
  INSURANCE_VERIFY_FAILURE: "INSURANCE_VERIFY_FAILURE",
  
  INTAKEWORKITEM_CREATED: "INTAKEWORKITEM_CREATED",
  INTAKEWORKITEM_UPDATED: "INTAKEWORKITEM_UPDATED",
  
  ESCALATION_CREATED: "ESCALATION_CREATED",
  
  RETRY_ATTEMPT: "RETRY_ATTEMPT",
};

// Database schema
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY,
  timestamp TIMESTAMP NOT NULL,
  agent_id VARCHAR(100) NOT NULL,
  action VARCHAR(50) NOT NULL,
  entity_type VARCHAR(50) NOT NULL,
  entity_id VARCHAR(255) NOT NULL,
  details JSON NOT NULL,
  outcome VARCHAR(20) NOT NULL,
  
  INDEX (timestamp),
  INDEX (agent_id),
  INDEX (action),
  INDEX (entity_type),
  INDEX (entity_id)
);
```

### Patient (Reference)

```typescript
// Minimal Patient schema for context; assume this exists
interface Patient {
  id: string; // UUID, primary key
  firstName: string;
  lastName: string;
  dateOfBirth: string; // YYYY-MM-DD
  medical_record_number: string; // used for insurance API
  insuranceMemberId?: string;
  insuranceGroupNumber?: string;
  insurancePlanName?: string;
}
```

---

## PART 2: SERVICE INTERFACES & RESPONSIBILITIES

Organize code into services by responsibility. Each service has clear input/output contracts.

### AppointmentSyncService

Responsibility: Fetch appointments from athenahealth, upsert Appointment records, create IntakeWorkItems.

```typescript
interface AppointmentSyncService {
  /**
   * Execute one sync cycle (triggered every 15 minutes)
   * 
   * Preconditions:
   * - athenahealth API key available in secrets
   * - Clinic location IDs in config
   * 
   * Execution:
   * - Fetch appointments for next 30 days (today UTC → +30 days)
   * - For each appointment, idempotent upsert (FR1.3)
   * - Log all actions to AuditLog
   * 
   * Postconditions:
   * - All new Appointment records created with sync_status = SYNCED
   * - All new IntakeWorkItem records created with status = PENDING
   * - Result object populated with counts
   * 
   * Outcome:
   * - Returns SyncCycleResult with sync_status, counts, errors
   * - Logs SyncCycleResult at INFO (success) or ERROR (failed)
   * 
   * Timeout: 10 minutes (job-level timeout; individual API calls 30s)
   */
  executeSyncCycle(): Promise<SyncCycleResult>;

  /**
   * Internal: Fetch raw appointments from athenahealth
   * Handles retry logic (5xx: 2 retries exponential backoff; 429: pause 1h; 4xx: no retry)
   * Timeout: 30 seconds per request
   */
  private fetchAppointmentsFromAthena(
    startDate: string,
    endDate: string
  ): Promise<AthenaAppointment[]>;

  /**
   * Internal: Upsert one appointment (idempotent)
   * Logic (FR1.3):
   * - Lookup by athena_appointment_id
   * - If not found: create new Appointment + IntakeWorkItem
   * - If found + athena.status IN [CANCELLED, COMPLETED]: archive and escalate
   * - If found + athena.status IN [SCHEDULED, CONFIRMED]: update if changed
   * - Log all decisions to AuditLog
   */
  private upsertAppointment(athenaAppt: AthenaAppointment): Promise<void>;
}

interface SyncCycleResult {
  sync_cycle_id: string; // UUID generated at start
  sync_started_at: string; // ISO 8601 UTC
  sync_completed_at: string; // ISO 8601 UTC
  sync_duration_seconds: number;
  appointments_fetched: number;
  appointments_created: number;
  appointments_updated: number;
  appointments_skipped: number;
  appointments_cancelled_with_escalation: number;
  new_intakeworkitems_created: number;
  new_escalations_created: number;
  sync_status: "SUCCESS" | "PARTIAL_SUCCESS" | "FAILED";
  errors: Array<{
    timestamp: string;
    error_code: string;
    error_message: string;
    appointment_id?: string;
  }>;
}

interface AthenaAppointment {
  id: string; // athena.appointmentId
  patientId: string;
  providerId: string;
  departmentId: string;
  appointmentType: string; // "Routine", "Urgent", "FollowUp", "Preventive"
  scheduledDateTime: string; // ISO 8601 UTC
  status: string; // "Scheduled", "Confirmed", "Completed", "Cancelled"
}
```

### InsuranceVerificationService

Responsibility: Verify patient insurance eligibility (deterministic decision: VERIFIED or ESCALATED).

```typescript
interface InsuranceVerificationService {
  /**
   * Verify insurance for one IntakeWorkItem
   * 
   * Preconditions:
   * - IntakeWorkItem exists, status = PENDING or HOLD
   * - Associated Patient exists with demographics
   * 
   * Execution (FR3.3):
   * - STEP 1: Validate patient demographics (escalate if missing)
   * - STEP 2: Call insurance API (15s timeout, 2x retry on 5xx, no retry on 4xx)
   * - STEP 3: Process response (ACTIVE→VERIFIED; others→FAILED or TIMEOUT→retry or escalate)
   * - STEP 4: Log decision to AuditLog
   * - STEP 5: Update IntakeWorkItem
   * 
   * Postconditions:
   * - IntakeWorkItem.insurance_verification_status set
   * - IntakeWorkItem.status updated (READY, HOLD, or ESCALATED)
   * - Escalation created if needed
   * - AuditLog entries created
   * 
   * Outcome:
   * - Returns VerificationResult with status, error, escalation_id
   */
  verifyInsurance(workitemId: string): Promise<VerificationResult>;

  /**
   * Internal: Call insurance provider API
   * Timeout: 15 seconds
   * Retry: 5xx → 2 times exponential backoff (3s, 6s); 4xx/timeout → no retry
   */
  private callInsuranceAPI(
    patient: Patient
  ): Promise<InsuranceResponse | null>;

  /**
   * Internal: Determine next state based on insurance response
   * Logic (FR3.3 STEP 3A-3D):
   * - ACTIVE → VERIFIED, no escalation
   * - INACTIVE/PENDING/UNKNOWN → FAILED, escalate
   * - PATIENT_NOT_FOUND/etc 4xx → FAILED, escalate
   * - TIMEOUT/5xx + retry < 3 → HOLD
   * - TIMEOUT/5xx + retry >= 3 → ESCALATED
   */
  private determineNextState(
    workitem: IntakeWorkItem,
    response: InsuranceResponse | null,
    error: InsuranceError | null
  ): Promise<VerificationOutcome>;
}

interface VerificationResult {
  workitem_id: string;
  status: "SUCCESS" | "HOLD" | "ESCALATED";
  verification_status: "VERIFIED" | "FAILED" | "TIMEOUT";
  error?: string;
  escalation_id?: string;
}

interface InsuranceResponse {
  status: "ACTIVE" | "INACTIVE" | "PENDING" | "UNKNOWN";
  providerId: string;
  referenceId: string;
  effectiveDate: string; // YYYY-MM-DD
  terminationDate: string | null;
  copay: number; // in cents
  deductible: number; // in cents
  deductibleMet: number | null;
  coinsurance: number; // percentage
  outOfPocketMax: number; // in cents
  outOfPocketMet: number | null;
  planType: string;
  preventiveCareCovered: boolean;
}

interface InsuranceError {
  code: string; // "PATIENT_NOT_FOUND", "INVALID_MEMBER_ID", etc.
  message: string;
}

interface VerificationOutcome {
  next_status: "READY" | "HOLD" | "ESCALATED";
  insurance_verification_status: "VERIFIED" | "FAILED" | "TIMEOUT";
  insurance_verification_error?: string;
  escalation?: EscalationInput;
}

interface EscalationInput {
  workitem_id: string;
  patient_id: string;
  appointment_id: string;
  escalation_reason: string;
  escalation_category:
    | "INSURANCE_TIMEOUT"
    | "INSURANCE_FAILED"
    | "APPOINTMENT_SYNC_FAILED"
    | "VERIFICATION_RETRY_EXHAUSTED";
  assigned_to: "FRONT_DESK";
}
```

### AuditService

Responsibility: Log all agent actions immutably.

```typescript
interface AuditService {
  /**
   * Log any agent action
   * - Immutable: once created, never modified
   * - Retention: 90 days minimum
   */
  log(entry: AuditLogEntry): Promise<void>;

  /**
   * Convenience methods for common actions
   */
  logAppointmentCreated(appointment: Appointment, syncCycleId: string): Promise<void>;
  logAppointmentUpdated(before: Appointment, after: Appointment): Promise<void>;
  logInsuranceVerifyStart(workitemId: string, patientId: string, attempt: number): Promise<void>;
  logInsuranceVerifySuccess(workitemId: string, patientId: string, response: InsuranceResponse): Promise<void>;
  logInsuranceVerifyTimeout(workitemId: string, patientId: string, attempt: number, action: string): Promise<void>;
  logInsuranceVerifyFailure(workitemId: string, patientId: string, error: InsuranceError, attempt: number): Promise<void>;
  logEscalationCreated(escalation: Escalation): Promise<void>;
}

interface AuditLogEntry {
  timestamp: string; // ISO 8601 UTC, set by service if not provided
  agent_id: string;
  action: string;
  entity_type: "APPOINTMENT" | "INTAKEWORKITEM" | "ESCALATION" | "SYNC_CYCLE";
  entity_id: string;
  details: Record<string, any>;
  outcome: "SUCCESS" | "FAILURE" | "HOLD" | "ESCALATED";
}
```

### EscalationService

Responsibility: Create and manage escalations.

```typescript
interface EscalationService {
  /**
   * Create escalation (called when verification fails or appointment cancelled)
   * Sets sla_deadline = now + 4 hours
   * Logs to AuditLog
   */
  createEscalation(input: EscalationInput): Promise<Escalation>;

  /**
   * Fetch open escalations for FRONT_DESK review
   */
  getOpenEscalations(): Promise<Escalation[]>;

  /**
   * Mark escalation resolved (human action)
   * Requires: human_decision, resolved_by user_id
   */
  resolveEscalation(
    escalationId: string,
    decision: string,
    resolvedBy: string
  ): Promise<Escalation>;

  /**
   * Nightly SLA audit: mark sla_breached = true for escalations
   * that are OPEN past their sla_deadline (runs outside agent scope)
   */
  auditSLABreaches(): Promise<Escalation[]>;
}
```

---

## PART 3: ERROR HANDLING & RETRY LOGIC

Define how each service handles failures deterministically.

### Retry Strategy

```typescript
/**
 * Retry policy for external API calls
 * 
 * athenahealth API:
 * - HTTP 5xx: retry 2 times, backoff 5s, 10s
 * - HTTP 429: do not retry; pause sync 1h; escalate to operations
 * - HTTP 4xx (non-429): do not retry; escalate to operations
 * - Timeout (30s): do not retry; escalate to operations if >1h unavailable
 * 
 * Insurance API:
 * - HTTP 5xx: retry 2 times, backoff 3s, 6s
 * - HTTP 4xx: do not retry; escalate to FRONT_DESK (patient-facing issue)
 * - Timeout (15s): do not retry immediately; managed by retry_count in IntakeWorkItem
 */

interface RetryConfig {
  maxRetries: number;
  backoffMs: number[]; // [5000, 10000] for athena, [3000, 6000] for insurance
  timeoutMs: number;
}

interface RetryAttempt {
  attempt: number;
  retryAt?: string; // ISO 8601 UTC, for HOLD state
  error: string;
  statusCode?: number;
}

/**
 * Exponential backoff with jitter
 * delay = baseMs * (2 ^ attempt) + jitter
 */
function calculateBackoff(baseMs: number, attempt: number): number {
  const exponential = baseMs * Math.pow(2, attempt);
  const jitter = Math.random() * 1000;
  return exponential + jitter;
}
```

### Error Classification

```typescript
/**
 * Classify errors deterministically to decide next action
 */

type ErrorCategory =
  | "RETRYABLE" // retry with backoff
  | "HOLD" // wait and retry (IntakeWorkItem.status = HOLD)
  | "ESCALATE" // create Escalation, human decides
  | "SKIP" // log and continue (e.g., missing patient)
  | "SYSTEM_ERROR"; // escalate to operations, not clinic

function classifyError(
  error: Error | string,
  context: {
    source: "ATHENA" | "INSURANCE" | "DB";
    statusCode?: number;
    attempt: number;
    maxRetries: number;
  }
): ErrorCategory {
  if (context.source === "ATHENA") {
    if (context.statusCode === 429) return "ESCALATE"; // rate limit
    if (context.statusCode && context.statusCode >= 500) {
      if (context.attempt < context.maxRetries) return "RETRYABLE";
      return "ESCALATE";
    }
    if (context.statusCode === 404 || context.statusCode === 401) return "SYSTEM_ERROR";
    if (!context.statusCode) return "ESCALATE"; // timeout
  }

  if (context.source === "INSURANCE") {
    if (context.statusCode && context.statusCode >= 500) {
      if (context.attempt < context.maxRetries) return "HOLD"; // managed by retry_count
      return "ESCALATE";
    }
    if (context.statusCode && context.statusCode >= 400) {
      return "ESCALATE"; // patient-facing issue
    }
    if (!context.statusCode) return "HOLD"; // timeout; managed by retry_count
  }

  return "SKIP"; // unknown error; log and continue
}
```

---

## PART 4: TESTING REQUIREMENTS

Two validation scenarios MUST pass before delivery.

### Test Scenario V1: Happy Path

```typescript
/**
 * Test: Appointment Synced + Insurance Verified → Status READY
 * 
 * Setup:
 * - Mock athenahealth API returns 1 appointment (status=Scheduled)
 * - Mock insurance API returns status=ACTIVE
 * - Patient exists with complete demographics
 * 
 * Expected flow:
 * 1. executeSyncCycle() creates Appointment + IntakeWorkItem (PENDING)
 * 2. verifyInsurance() calls insurance API, gets ACTIVE
 * 3. IntakeWorkItem transitions to READY
 * 4. No Escalations created
 * 
 * Assertions:
 * - Appointment.id exists, sync_status = SYNCED
 * - IntakeWorkItem.id exists, status = READY
 * - insurance_verification_status = VERIFIED
 * - insurance_reference_id populated (saved for later auth)
 * - Escalations count = 0
 * - AuditLog entries:
 *   * APPOINTMENT_CREATED
 *   * INTAKEWORKITEM_CREATED
 *   * INSURANCE_VERIFY_START
 *   * INSURANCE_VERIFY_SUCCESS
 *   * INTAKEWORKITEM_UPDATED (status→READY)
 */

describe("V1: Happy Path", () => {
  it("should sync appointment and verify insurance → READY", async () => {
    // Setup
    const mockAthenaResponse = {
      appointments: [
        {
          id: "ATH-2026-04-25-001",
          patientId: "PAT-ATH-00123",
          providerId: "PROV-001",
          departmentId: "LOC-MAIN",
          appointmentType: "Routine",
          scheduledDateTime: "2026-04-25T14:00:00Z",
          status: "Scheduled",
        },
      ],
      totalCount: 1,
    };

    const mockInsuranceResponse = {
      status: "ACTIVE",
      providerId: "BLUE_CROSS_01",
      referenceId: "ELIGIBILITY-2026-04-20-12345",
      effectiveDate: "2025-01-01",
      terminationDate: null,
      copay: 2500,
      deductible: 50000,
      deductibleMet: 15000,
      coinsurance: 20,
      outOfPocketMax: 700000,
      outOfPocketMet: 8000,
      planType: "PPO",
      preventiveCareCovered: true,
    };

    // Mock external APIs
    mockAthenahealth.getAppointments.mockResolvedValue(mockAthenaResponse);
    mockInsuranceProvider.verify.mockResolvedValue(mockInsuranceResponse);

    // Execute
    const syncResult = await appointmentSyncService.executeSyncCycle();
    const workitem = await db.intakeWorkItems.findOne({ /* ... */ });
    const verificationResult = await insuranceVerificationService.verifyInsurance(workitem.id);

    // Assertions
    expect(syncResult.sync_status).toBe("SUCCESS");
    expect(syncResult.appointments_created).toBe(1);
    expect(syncResult.new_intakeworkitems_created).toBe(1);
    
    const appointment = await db.appointments.findOne({ athena_appointment_id: "ATH-2026-04-25-001" });
    expect(appointment.sync_status).toBe("SYNCED");
    expect(appointment.status).toBe("SCHEDULED");

    expect(workitem.status).toBe("PENDING"); // after sync
    
    const updatedWorkitem = await db.intakeWorkItems.findOne({ id: workitem.id });
    expect(updatedWorkitem.status).toBe("READY"); // after insurance verification
    expect(updatedWorkitem.insurance_verification_status).toBe("VERIFIED");
    expect(updatedWorkitem.insurance_reference_id).toBe("ELIGIBILITY-2026-04-20-12345");

    const escalations = await db.escalations.find({ workitem_id: workitem.id });
    expect(escalations).toHaveLength(0);

    const logs = await db.auditLogs.find({ entity_id: workitem.id });
    expect(logs.map(l => l.action)).toContain("INSURANCE_VERIFY_START");
    expect(logs.map(l => l.action)).toContain("INSURANCE_VERIFY_SUCCESS");
    expect(logs.map(l => l.action)).toContain("INTAKEWORKITEM_UPDATED");
  });
});
```

### Test Scenario V3: Timeout Exhaustion

```typescript
/**
 * Test: Insurance Timeout × 3 Retries → Status ESCALATED
 * 
 * Setup:
 * - Appointment synced
 * - Insurance API times out on all 3 attempts (15s each)
 * 
 * Expected flow:
 * 1. verifyInsurance() attempt 1: timeout → HOLD, retry_count=1
 * 2. verifyInsurance() attempt 2: timeout → HOLD, retry_count=2
 * 3. verifyInsurance() attempt 3: timeout → ESCALATED, retry_count=3
 * 4. Escalation created with category=INSURANCE_TIMEOUT
 * 
 * Assertions (after attempt 1):
 * - IntakeWorkItem.status = HOLD
 * - insurance_verification_status = TIMEOUT
 * - retry_count = 1
 * - Escalations count = 0 (not escalated yet)
 * 
 * Assertions (after attempt 3):
 * - IntakeWorkItem.status = ESCALATED
 * - insurance_verification_status = TIMEOUT
 * - retry_count = 3
 * - Escalations count = 1, status = OPEN, assigned_to = FRONT_DESK
 * - sla_deadline = now + 4 hours
 * - AuditLog entries:
 *   * 3× INSURANCE_VERIFY_TIMEOUT (outcomes: HOLD, HOLD, ESCALATED)
 *   * ESCALATION_CREATED
 *   * INTAKEWORKITEM_UPDATED (status→ESCALATED)
 */

describe("V3: Timeout Exhaustion", () => {
  it("should timeout 3x and escalate", async () => {
    // Setup
    const workitem = await setupWorkitem("PENDING");
    const patient = await setupPatient();

    // Mock insurance API to always timeout
    mockInsuranceProvider.verify.mockImplementation(
      () =>
        new Promise((_, reject) =>
          setTimeout(() => reject(new Error("TIMEOUT")), 16000) // > 15s timeout
        )
    );

    // Attempt 1
    const result1 = await insuranceVerificationService.verifyInsurance(workitem.id);
    expect(result1.status).toBe("HOLD");
    let updated = await db.intakeWorkItems.findOne({ id: workitem.id });
    expect(updated.status).toBe("HOLD");
    expect(updated.insurance_verification_status).toBe("TIMEOUT");
    expect(updated.insurance_verification_retry_count).toBe(1);

    let escalations = await db.escalations.find({ workitem_id: workitem.id });
    expect(escalations).toHaveLength(0);

    // Attempt 2
    const result2 = await insuranceVerificationService.verifyInsurance(workitem.id);
    expect(result2.status).toBe("HOLD");
    updated = await db.intakeWorkItems.findOne({ id: workitem.id });
    expect(updated.insurance_verification_retry_count).toBe(2);
    escalations = await db.escalations.find({ workitem_id: workitem.id });
    expect(escalations).toHaveLength(0);

    // Attempt 3
    const result3 = await insuranceVerificationService.verifyInsurance(workitem.id);
    expect(result3.status).toBe("ESCALATED");
    updated = await db.intakeWorkItems.findOne({ id: workitem.id });
    expect(updated.status).toBe("ESCALATED");
    expect(updated.insurance_verification_retry_count).toBe(3);
    expect(updated.escalated_at).toBeDefined();
    expect(updated.escalated_to).toBe("FRONT_DESK");

    escalations = await db.escalations.find({ workitem_id: workitem.id });
    expect(escalations).toHaveLength(1);
    const escalation = escalations[0];
    expect(escalation.status).toBe("OPEN");
    expect(escalation.assigned_to).toBe("FRONT_DESK");
    expect(escalation.escalation_category).toBe("INSURANCE_TIMEOUT");
    const slaDeadline = new Date(escalation.sla_deadline);
    const createdAt = new Date(escalation.created_at);
    expect(slaDeadline.getTime() - createdAt.getTime()).toBe(4 * 60 * 60 * 1000); // 4 hours

    const logs = await db.auditLogs.find({ entity_id: workitem.id });
    const timeoutLogs = logs.filter(l => l.action === "INSURANCE_VERIFY_TIMEOUT");
    expect(timeoutLogs).toHaveLength(3);
    expect(timeoutLogs[0].outcome).toBe("HOLD");
    expect(timeoutLogs[1].outcome).toBe("HOLD");
    expect(timeoutLogs[2].outcome).toBe("ESCALATED");
  });
});
```

### Test Infrastructure

```typescript
/**
 * Setup helpers for tests
 */

// Mock database
const mockDb = {
  appointments: {
    create: jest.fn(),
    findOne: jest.fn(),
    update: jest.fn(),
  },
  intakeWorkItems: {
    create: jest.fn(),
    findOne: jest.fn(),
    update: jest.fn(),
  },
  escalations: {
    create: jest.fn(),
    find: jest.fn(),
  },
  auditLogs: {
    create: jest.fn(),
    find: jest.fn(),
  },
  patients: {
    findOne: jest.fn(),
  },
};

// Mock external services
const mockAthenahealth = {
  getAppointments: jest.fn(),
};

const mockInsuranceProvider = {
  verify: jest.fn(),
};

// Helper: setup test patient
async function setupPatient() {
  const patient = {
    id: "patient-uuid",
    firstName: "Jane",
    lastName: "Smith",
    dateOfBirth: "1985-06-15",
    medical_record_number: "MRN-123",
    insuranceMemberId: "BC-987654321",
  };
  mockDb.patients.findOne.mockResolvedValue(patient);
  return patient;
}

// Helper: setup test workitem
async function setupWorkitem(status = "PENDING") {
  const workitem = {
    id: "workitem-uuid",
    appointment_id: "appointment-uuid",
    patient_id: "patient-uuid",
    status,
    insurance_verification_status: "PENDING",
    insurance_verification_retry_count: 0,
    created_at: new Date().toISOString(),
  };
  mockDb.intakeWorkItems.findOne.mockResolvedValue(workitem);
  return workitem;
}

// Helper: reset all mocks
function resetMocks() {
  Object.values(mockDb).forEach(service => {
    Object.values(service).forEach(fn => fn.mockReset());
  });
  mockAthenahealth.getAppointments.mockReset();
  mockInsuranceProvider.verify.mockReset();
}
```

---

## PART 5: CONFIGURATION & ENVIRONMENT

Provide these as environment variables or config file.

```typescript
/**
 * Configuration (env vars or config.yaml)
 */

export const config = {
  // athenahealth integration
  ATHENA_API_KEY: process.env.ATHENA_API_KEY,
  ATHENA_BASE_URL: process.env.ATHENA_BASE_URL || "https://api.athenahealth.com",
  ATHENA_DEPARTMENT_IDS: (process.env.ATHENA_DEPARTMENT_IDS || "LOC-MAIN").split(","),
  ATHENA_API_TIMEOUT_MS: 30000, // 30 seconds
  ATHENA_SYNC_INTERVAL_MS: 15 * 60 * 1000, // 15 minutes
  ATHENA_SYNC_JOB_TIMEOUT_MS: 10 * 60 * 1000, // 10 minutes

  // Insurance integration
  INSURANCE_API_KEY: process.env.INSURANCE_API_KEY,
  INSURANCE_BASE_URL: process.env.INSURANCE_BASE_URL || "https://api.insurance-provider.com",
  INSURANCE_API_TIMEOUT_MS: 15000, // 15 seconds
  INSURANCE_VERIFY_MAX_RETRIES: 3,
  INSURANCE_VERIFY_BACKOFF_MS: [3000, 6000], // [3s, 6s]

  // Database
  DB_HOST: process.env.DB_HOST || "localhost",
  DB_PORT: parseInt(process.env.DB_PORT || "5432"),
  DB_NAME: process.env.DB_NAME || "patient_intake",
  DB_USER: process.env.DB_USER,
  DB_PASSWORD: process.env.DB_PASSWORD,

  // Audit
  AUDIT_RETENTION_DAYS: 90,

  // SLA
  ESCALATION_SLA_HOURS: 4,
};

/**
 * Example .env file
 */
/*
ATHENA_API_KEY=your_api_key_here
ATHENA_BASE_URL=https://api.athenahealth.com
ATHENA_DEPARTMENT_IDS=LOC-MAIN,LOC-URGENT,LOC-PEDIATRICS

INSURANCE_API_KEY=your_insurance_key_here
INSURANCE_BASE_URL=https://api.insurance-provider.com

DB_HOST=localhost
DB_PORT=5432
DB_NAME=patient_intake
DB_USER=postgres
DB_PASSWORD=postgres
*/
```

---

## PART 6: CRITICAL DO's AND DON'Ts

### DO

- ✅ **Log every action** — Every sync, every verify, every escalation logged to AuditLog
- ✅ **Make it deterministic** — Same inputs always produce same outputs
- ✅ **Fail closed** — Ambiguity always escalates, never guesses
- ✅ **Preserve immutability** — Once escalated_at set, never change it
- ✅ **Validate before API calls** — Check patient demographics before calling insurance
- ✅ **Handle timeouts explicitly** — Timeout ≠ success; timeout + retry_count → decide next state
- ✅ **Test both scenarios** — V1 happy path AND V3 timeout exhaustion
- ✅ **Timestamp everything** — All timestamps in ISO 8601 UTC
- ✅ **Idempotent upserts** — Sync appointment twice = no double-writes

### DON'T

- ❌ **Delegate clinical judgment** — No diagnosis, treatment, triage to agent; route only
- ❌ **Guess on ambiguity** — Unknown insurance status? Escalate to FRONT_DESK
- ❌ **Retry indefinitely** — Cap retries at 3 for insurance; then escalate
- ❌ **Suppress errors** — Log every error, even if you retry
- ❌ **Modify immutable fields** — patient_id, athena_appointment_id, appointment_type never change
- ❌ **Create escalations in tests** — Mock them; test logic, not side effects
- ❌ **Use local time** — Always UTC in timestamps and API calls
- ❌ **Assume patient exists** — Validate before using; skip if missing

---

## PART 7: BUILD CHECKLIST

Before submitting code:

- [ ] All three entities (Appointment, IntakeWorkItem, Escalation) implemented with correct types
- [ ] AppointmentSyncService: FR1 fully implemented (fetch, upsert, idempotence)
- [ ] InsuranceVerificationService: FR3 fully implemented (validate, call API, decide next state)
- [ ] AuditService: all actions logged immutably; retention enforced
- [ ] EscalationService: escalations created and tracked
- [ ] Database schema created with correct indexes
- [ ] V1 test scenario passing (happy path → READY)
- [ ] V3 test scenario passing (timeout × 3 → ESCALATED)
- [ ] No clinical judgment anywhere (no diagnosis, treatment, triage)
- [ ] All external API calls have timeout + retry logic
- [ ] All timestamps ISO 8601 UTC
- [ ] Idempotent upsert logic verified (sync twice = no duplicates)
- [ ] Determinism verified (same inputs = same outputs)
- [ ] Code documented (inline comments for complex logic)
- [ ] Error handling deterministic (no random fallbacks)

---

## REFERENCE: Full Spec Document

For any ambiguity or edge case not covered here, consult:

**`Patient-Intake-Coordination-Agent-Spec.md`**

That file is the **authoritative source of truth**. This context file is a build roadmap; the spec is the contract.

---

**Document Version:** 1.0  
**Status:** Ready for Claude Build  
**Last Updated:** 2026-04-20
