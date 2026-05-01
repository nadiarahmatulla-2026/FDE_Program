# FDE Program — Agent Instructions

This workspace contains **documentation only** — Markdown artefacts for the FDE (Forward Deployed Engineer) Accelerated Development Program. There is no application code to build or test.

## What this workspace is

A 5-week learning program teaching AI-native specification writing. Participants produce business artefacts (specs, delegation analyses, problem statements, validation designs) precise enough for AI coding agents to build from. See [README-Participants.md](./README-Participants.md) for the full index.

## File conventions

### Naming
Scenario deliverables follow this pattern:
```
scn-{N}-{artefact-type}-gfm.md
```
Examples: `scn-5-capability-specification-gfm.md`, `scn-5-delegation analysis-gfm.md`, `scn-5-BuildBrief-gfm.md`

Final/reviewed versions live under `ScenarioN_Delivery/Final/`.

### Format
All documents use **GitHub Flavored Markdown (GFM)**. Tables, checkboxes, and fenced code blocks are widely used. Do not convert to other formats.

## Key reference documents

These files define the standards used to evaluate participant work — link to them rather than duplicating their content:

| File | Purpose |
|---|---|
| [production-spec-checklist.md](./production-spec-checklist.md) | Criteria for a buildable, agent-ready spec |
| [spec-ambiguity-vs-builder-mistakes.md](./spec-ambiguity-vs-builder-mistakes.md) | Taxonomy for diagnosing build failures (4 categories) |
| [claude-md-examples-guide.md](./claude-md-examples-guide.md) | CLAUDE.md quality tiers and examples |
| [Week1-Thinking-Discipline-Primer.md](./Week1-Thinking-Discipline-Primer.md) | Thinking framework for FDE artefacts |

## Program terminology

| Term | Meaning |
|---|---|
| **FDE** | Forward Deployed Engineer — designs AI-native systems, writes buildable specs |
| **Build loop** | Cycle of spec → agent build → diagnose output → fix spec or accept |
| **Gate** | Timed, sealed assessment at end of each week |
| **Delegation analysis** | Which tasks are fully agentic / agent-led / human-led, and why |
| **Capability specification** | Precise agent spec (entities, behaviours, acceptance criteria) |
| **Build Brief** | Compressed, agent-ready version of a capability specification |
| **Shortened Spec** | Condensed spec optimised for a single focused build prompt |
| **Validation design** | What "working" looks like in testable terms + anticipated failure modes |
| **Assumptions & unknowns** | Explicit log of untested premises in a spec |
| **Virtual week** | Program's internal Mon–Fri structure, decoupled from physical calendar dates |
| **Closed build loop** | Completing at least one full spec → build → diagnose cycle against Claude Code |

## When editing artefacts

- Preserve assumption logs (`**Assumption:**`, `**Hypothesis:**`, `**How I'd test it:**`, `**Confidence:**` blocks) — these are scoreable deliverables, not notes to clean up.
- Ambiguous language (`should`, `may`, `eventually`, `as appropriate`) in specs is a defect — flag it or replace it with precise criteria.
- Tables in delegation analyses and validation designs are structural; do not flatten them into prose.
- Do not add content that isn't either stated or derivable from the scenario context — undisclosed assumptions are a failure mode in this program.
