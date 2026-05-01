# PowerPoint Slide Deck — Gate 1 Defense
## FNOL Claims Processing Agent | Nadia Rahmatulla

---

## SLIDE 1: Title Slide
**Title:** FNOL Claims Processing Agent  
**Subtitle:** Gate 1 Specification Defense  
**Name:** Nadia Rahmatulla  
**Context:** Insurance Claims Intake Automation

---

## SLIDE 2: The Problem — Current State is Broken
**Visual:** Process flow diagram showing bottleneck

### Left Column: The Numbers
- 300 FNOLs/day
- 12 specialists
- 22 min/claim average
- **110 person-hours/day** (zero slack)

### Right Column: The Failures
- **18% routing errors** → 54 claims/day to wrong adjuster
- **31% SLA breaches** → 93 claimants not acknowledged in 2hrs
- **Zero capacity** for volume spikes or complexity

### Bottom: Why This Matters
> "FNOL is the highest-stakes moment. A distressed claimant expects rapid acknowledgment. Failing here breaks trust when they need it most."

**Visual element:** Red warning triangle on failure stats

---

## SLIDE 3: Solution Overview — Agent + Human Delegation
**Visual:** Two-column split with icons

### LEFT: Fully Agentic (70-85% of claims)
✅ **Ingest** all channels (email/phone/web)  
✅ **Extract** structured data (LLM + confidence)  
✅ **Validate** policy coverage (SOAP)  
✅ **Triage** LOW/MEDIUM severity (rules)  
✅ **Route** to best-match adjuster  
✅ **Acknowledge** within 15 minutes  

### RIGHT: Human Oversight (15-30% of claims)
🟡 **HIGH/CRITICAL severity** → specialist reviews dossier  
🟡 **Coverage declines** → specialist approves notice  
🟡 **Low-confidence extraction** → specialist completes  
🟡 **System failures** → escalate with context  

### Bottom Banner:
**Delegation Principle:** Fully agentic when logic is codifiable, errors detectable, no legal accountability required

---

## SLIDE 4: Honest Assumptions — Load-Bearing Risks
**Visual:** Table with confidence indicators

| Assumption | Confidence | Why It Matters | Test |
|---|---|---|---|
| **£50k = "high-value" threshold** | 🔴 LOW | Wrong threshold → wrong oversight boundary | Sprint 0 client sign-off |
| **SOAP responds in ≤5s** | 🔴 LOW | If 30s typical → SLA collapses | 50 staging calls before commit |
| **LLM API within build timeline** | 🔴 LOW | Insurance PII restrictions common | Sprint 0 blocker — written approval |
| **Policy # in 80% of FNOLs** | 🟡 MEDIUM | Below 50% → most claims escalate | 100-sample audit from client |
| **Automated ack acceptable** | 🟢 HIGH | Industry standard at FNOL stage | Review complaint logs |

### Callout Box:
> "No filler unknowns. Every assumption marked LOW confidence is a sprint blocker with a resolution path."

---

## SLIDE 5: Critical Unknowns — Must Resolve Before Build
**Visual:** Roadblock icons with resolution paths

| Unknown | Why It Blocks | Resolution | Sprint |
|---|---|---|---|
| **U.1: "High-value" definition** | Severity thresholds are placeholders | Client workshop + sign-off | 🔴 Sprint 0 |
| **U.2: CRM API schema** | Cannot build routing module | API docs + sandbox credentials | Sprint 1 |
| **U.3: SOAP latency profile** | Cannot confirm validation budget | WSDL + 90-day uptime logs | Sprint 1 |
| **U.7: FCA automated decline rules** | May require human sign-off by law | Legal/compliance review | 🔴 Sprint 0 |
| **U.8: LLM data residency** | PII restrictions may block extraction | Written provider approval | 🔴 Sprint 0 |

### Bottom:
**8 unknowns named. 3 are sprint 0 gates. Zero hidden blockers.**

---

## SLIDE 6: Spec Precision — Could an AI Build This?
**Visual:** Code-style contract boxes

### State Machine (Executable)
```
RECEIVED → EXTRACTING → VALIDATING → TRIAGING → AUTO_ROUTING → ROUTED
```
- Every transition: timestamp, actor, reason
- Max time in state: 30-90 seconds
- Timeout → auto-escalate to specialist queue
- **Zero silent hangs**

### Module Contract Example: M4 Severity Classifier
| Priority | Condition | Severity |
|---|---|---|
| 1 | Bodily injury mentioned | CRITICAL |
| 2 | Incident type = LIABILITY | HIGH |
| 3 | Damage ≥ £50k | HIGH |
| 8 | No match → default MEDIUM | MEDIUM* |

*Conservative: wrong LOW = unreviewed high-stakes claim

### Agent Rules (Never Violate)
1. Never auto-route HIGH/CRITICAL
2. Never send decline without specialist approval
3. Never retry SOAP > 2 times
4. claim_id set at ingestion, immutable

---

## SLIDE 7: Validation — How Do We Know It's Working?
**Visual:** Dashboard mockup with metrics

### "Working" = 5 Testable Conditions (5-day rolling window)
| # | Condition | Target | Measurement |
|---|---|---|---|
| 1 | Routing error rate | ≤ 5% | Re-assignments ÷ total claims |
| 2 | SLA compliance | ≥ 90% | `routed_at` − `received_at` ≤ 120min |
| 3 | Zero silent failures | 100% | Complete audit trail to terminal state |
| 4 | Zero unacknowledged | 100% | Every ROUTED has notification record |
| 5 | Escalation rate | 15-35% | Outside band → investigate |

### Quiet Failure Tests (Most Dangerous)
- ❌ Wrong policy # extracted with high confidence  
  **Test:** Weekly 5% audit. Target ≥ 97% accuracy
- ❌ HIGH claim misclassified as MEDIUM  
  **Test:** 20 synthetic high-value narrative claims
- ❌ Claim stuck in EXTRACTING (LLM hang)  
  **Test:** Watchdog moves to queue after 90s

---

## SLIDE 8: 30-Day Pilot Plan
**Visual:** Gantt-style timeline

### Shadow Mode (Weeks 1-2)
📊 60 claims/day, no live actions  
✅ Pass: Zero state machine errors  
✅ Pass: Extraction accuracy ≥ 95%  
✅ Pass: Severity accuracy ≥ 92% vs specialist

### Phased Live (Weeks 3-4)
**Week 3:** LOW claims live only  
- Pass: Routing error ≤ 5%, ack ≤ 15min

**Week 4:** LOW + MEDIUM live  
- Pass: Combined routing error ≤ 5%, SLA breach ≤ 12%

### Rollback Trigger (Automatic)
🚨 If routing error > 10% in any 48hr window  
→ Auto-move ALL claims to specialist queue

---

## SLIDE 9: Why This Spec Is Defensible
**Visual:** Three pillars with checkmarks

### 1️⃣ Delegation Boundaries Are Justified
- LOW/MEDIUM: deterministic rules + detectable errors + no legal accountability = agentic
- HIGH/CRITICAL: financial/legal stakes + client requirement = oversight
- **Can explain WHY every task is at its level**

### 2️⃣ Assumptions Are Honest
- 10 assumptions with hypothesis + test + confidence
- 3 marked LOW confidence (load-bearing risks)
- 8 unknowns with resolution path + sprint assignment
- **No "we'll figure it out later"**

### 3️⃣ Spec Is Build-Ready
- State machine is executable
- Module I/O schemas complete
- Integration contracts list knowns AND unknowns
- **If AI builder misreads, that's my spec bug**

### 4️⃣ Validation Is Testable
- "Working" has 5 numeric conditions
- Quiet failures named (the dangerous ones)
- Pilot has pass conditions + auto-rollback
- **These ARE the tests, not promises to write tests**

---

## SLIDE 10: Expected Challenges — Pre-Answered
**Visual:** Q&A format with coach questions

### Q: "Why is £50k threshold in spec if you don't know it?"
**A:** Hiding it is worse. U.1 shows where confidence is load-bearing. Sprint 0 blocker with client workshop resolution. Honest uncertainty beats plausible guess.

### Q: "What if SOAP is down 2 hours during business hours?"
**A:** Addressed in A.3 (LOW confidence) and U.8. Test: request 90-day uptime logs. If unreliable → policy cache layer or accept validation failure spikes.

### Q: "How prevent routing HIGH as MEDIUM?"
**A:** Rules 1-3 deterministic. Rule 8 defaults MEDIUM (not LOW). LLM cannot downgrade without specialist. **Severity escalation is one-way.**

### Q: "Single biggest risk?"
**A:** **A.8 — LLM procurement delay.** PII/data residency may block third-party LLM for 3+ months → re-scope extraction to regex/rules → 15% escalation rate instead of 5%. **Mitigation: Sprint 0 blocker, no code until written approval.**

---

## SLIDE 11: Close — What I'm Asking
**Visual:** Simple checklist

### ✅ This Spec Is Ready Because:
1. Every delegation boundary has named justification
2. 3 LOW-confidence assumptions, 8 sprint blockers — honest
3. Precise enough for AI coding agent to build from
4. Validation = 5 numeric conditions, quiet failures named

### 🤔 Feedback Requested:
- Are severity thresholds the right sprint 0 conversation?
- Is SOAP latency unknown a sprint 0 gate or sprint 1 acceptable risk?
- Have I missed a quiet failure mode?

---

## SLIDE 12: Backup — System Architecture (If Asked)
**Visual:** Architecture diagram

### Input Channels
📧 Email (IMAP/SMTP)  
📞 Phone Transcript (Webhook)  
🌐 Web Form (REST POST)

### Agent Core Modules
**M1:** Normaliser → Channel-agnostic text  
**M2:** Extractor → LLM structured output + confidence  
**M3:** Validator → SOAP policy check  
**M4:** Classifier → Severity rules  
**M5:** Router → Adjuster matching + load balance  
**M6:** Acknowledgment → Template engine  
**M7:** Escalation → Specialist queue + dossier  

### External Systems
🗂 CRM (REST)  
📋 Policy Admin (SOAP)  
📁 DMS (REST)  
📬 Notification (Email/SMS)

### State Machine
All transitions logged. Max time in state enforced. Zero silent failures.

---

## SLIDE 13: Backup — Delegation Visual (If Asked)
**Visual:** Three-zone flowchart

```
┌─────────────────────────────────────┐
│   ✅ FULLY AGENTIC (70-85%)        │
│   Ingest → Extract → Validate →    │
│   Triage LOW/MED → Route → Ack     │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   🟡 AGENT-LED, HUMAN OVERSIGHT     │
│   HIGH/CRITICAL severity            │
│   Coverage declines                 │
│   Low confidence extraction         │
│   System failures                   │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   🔴 HUMAN ONLY                     │
│   Fraud investigation               │
│   Coverage dispute resolution       │
│   Regulatory reporting              │
└─────────────────────────────────────┘
```

---

## SLIDE 14: Backup — Success Metrics Table (If Asked)
**Visual:** Before/After comparison table

| Metric | Baseline | Target | Business Impact |
|---|---|---|---|
| Routing error rate | 18% | ≤ 5% | 30 specialist-hours/week saved |
| SLA breach rate | 31% | ≤ 10% | 90% claimants acknowledged on time |
| Avg handling time | 22 min | ≤ 8 min (reviewed)<br>≤ 3 min (agentic) | 70% time reduction on LOW/MED |
| Claimant ack time | Unknown | ≤ 15 min | Trust at highest-stakes moment |
| Specialist capacity | 110 hrs/day | 60 hrs/day projected | 45% capacity freed for complex work |

**Business case threshold:** System pays for itself if routing errors < 8% AND ack SLA ≥ 90% within 60 days.

---

## Design Notes for Slides:

### Color Palette Recommendation:
- **Agentic actions:** Green (#51cf66)
- **Human oversight:** Orange/amber (#ffa94d)
- **Failures/risks:** Red (#ff6b6b)
- **Metrics/success:** Blue (#4dabf7)
- **Warnings/unknowns:** Purple (#9c36b5)

### Visual Elements to Include:
- **Icons:** Robot/gear for agentic, person icon for human oversight, warning triangles for risks
- **Flowcharts:** Use Mermaid-style diagrams for state machine and delegation zones
- **Tables:** Clean, readable tables with alternating row colors
- **Callout Boxes:** For critical points (delegation principle, honest uncertainty)
- **Progress Bars:** For pilot timeline and confidence levels

### Animation Suggestions:
- Slide 2: Fade in failure numbers one at a time
- Slide 3: Split-screen reveal (agentic left, human right)
- Slide 6: Build state machine left-to-right
- Slide 8: Timeline animation showing phased rollout

### Slide Timing (10 mins total):
- Slides 1-2: 90 seconds (problem)
- Slide 3: 2 minutes (solution)
- Slides 4-5: 90 seconds (assumptions)
- Slide 6: 2 minutes (precision)
- Slide 7-8: 2 minutes (validation)
- Slide 9: 90 seconds (why defensible)
- Slides 10-11: 90 seconds (challenges + close)
- Slides 12-14: Backup (only if questions arise)

---

*Ready for import to PowerPoint/Google Slides/Keynote*
*Markdown can be converted using Pandoc or manually transferred*
