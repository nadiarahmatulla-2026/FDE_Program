This is for Gate 1 Exam Deliverable

Using all the artefacts in C:\Users\NadiaRahmatulla\FDE_Program\Gate1\ as a guide against the Scenario XXXXX help me produce 1 markdown specification file for the Agent specification in this build name it as scn-5-capability-specification-gfm.md

Final/reviewed versions live under `ScenarioN_Delivery/Final/

### Format
All documents use **GitHub Flavored Markdown (GFM)**. Tables, checkboxes, and fenced code blocks are widely used. Do not convert to other formats.

where possible use mermaid flow diagrams

focus on

**Agent specification** — purpose, scope, inputs/outputs, decision logic, escalation triggers, integration points. Precise enough that an AI coding agent could start building from it

## What coaches are looking for

Three things more than anything else:

1. **Delegation boundaries are defensible, not arbitrary.** If you can't explain why a specific task is fully agentic versus human-overseen, you haven't done the thinking yet.
2. **The spec is precise enough that an AI coding agent wouldn't need to ask a clarifying question.** The critique session and your own closed build loop are both designed to teach you what this feels like. Use them.
3. **Assumptions and unknowns are honest.** "I don't know" beats a plausible-sounding guess. At least 5 genuine unknowns, not filler.

Thinking-Discipline-Primer
> **Assumption:** [what you're taking as given]
> **Hypothesis:** If [X is true], then [Y will happen], because [reasoning].
> **How I'd test it:** [the coach session, prototype, or data probe that would confirm/refute]
> **Confidence:** low / medium / high — and why.

- **Agent specification** — precise enough that an AI coding agent wouldn't need to ask a clarifying question. Your closed build loop against Claude Code is how you test this directly. Every time the builder produces something unintended, you classify it: spec ambiguity (you own the fix), builder misread (the builder owns the fix), or unjustified addition (ask for rollback).

