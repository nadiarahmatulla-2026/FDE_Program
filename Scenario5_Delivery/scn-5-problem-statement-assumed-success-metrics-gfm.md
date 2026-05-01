# Problem Statement and Proposed Success Metrics

## Purpose

This section defines the business problem and proposed success targets for the Scenario 5 patient intake workflow. These targets are intended to guide implementation and validation. Where the scenario does not provide a current baseline, targets must be treated as **proposed success targets pending stakeholder validation**.

## Problem Statement

A family medicine practice with **6 physicians** operating across **2 locations** sees approximately **180 patients per day**, but its **4-person front-desk team** is struggling to complete a complex pre-visit intake workflow reliably at that volume. Required intake work includes insurance verification, prior-authorisation checks for scheduled procedures, pre-visit questionnaire collection, medication update collection, allergy-flag surfacing, and reason-for-visit routing. As a result, physicians often begin visits with incomplete intake information. The most common failures identified in the scenario are **expired prior authorisations** and **unreviewed medication changes**.

The system to be built must reduce front-desk administrative burden and improve intake completeness for scheduled visits. It may automate only **non-clinical administrative coordination tasks**. It must not diagnose, perform medical triage, interpret symptom severity, judge medication significance, or assess allergy risk. Any visit reason that may imply urgency, ambiguity, or patient-safety risk must be routed to a human review path. Clinical judgment must remain human.

## Build Objective

Build a patient intake coordination workflow that:

- increases the percentage of visits that begin with complete administrative intake
- reduces missed administrative defects before visit start
- reduces front-desk handling burden across approximately **180 daily visits**
- preserves strict human control over all clinical judgment
- creates a complete audit trail for PHI access, workflow actions, escalations, and overrides

## Proposed Success Metrics

These are **proposed implementation targets** and must be validated with stakeholders before being treated as production commitments.

| Metric | Proposed Target | Baseline To Validate |
|:--|:--|:--|
| Intake completeness before visit start | **>=95%** of visits begin with all required administrative intake steps completed or correctly escalated | Current intake completeness rate |
| Missed prior-auth and medication-change defects at visit start | **>=50% reduction** from baseline | Current rate of expired prior auths and unreviewed medication changes discovered at visit start |
| Front-desk administrative workload per patient | **30-40% reduction** in average routine intake handling time | Current average front-desk intake time per patient |
| Same-day visit disruption caused by intake failures | **>=40% reduction** from baseline | Current rate of visit delays, rework, or interruptions caused by intake defects |
| Potentially urgent or ambiguous visit reasons routed to human review | **100%** | Confirm human review workflow and owning team |
| Clinical-boundary violations by the system | **0** | No baseline required; this is a hard safety requirement |
| Insurance and prior-auth status capture accuracy | **>=98%** | Current administrative accuracy rate if tracked |
| Audit logging coverage for PHI access, status changes, escalations, and overrides | **100%** | Confirm logging and retention requirements |

## Hard Constraints

The implementation must enforce the following constraints:

- clinical judgment must remain human
- the system must never diagnose, triage medically, or recommend treatment
- the system must never determine whether a medication change is clinically important
- the system must never determine whether an allergy flag is clinically significant
- the system must never suppress human review for urgent-looking or ambiguous visit reasons
- all PHI access and workflow actions must be logged

## Implementation Notes for Claude

When building from this section:

- treat all metrics as design targets unless explicitly confirmed by stakeholder input
- do not hard-code any metric as a guaranteed current-state improvement
- preserve the distinction between administrative workflow completion and clinical review
- implement all completion logic so that unresolved safety or ambiguity conditions result in escalation, not silent completion
- prefer false-positive escalation over unsafe autonomy in any boundary-sensitive case

## Sources

- `README-Participants-Week1-Scenarios.md`
- `README-Participants-Intro-Week1.md`