This is for Gate 1 Exam Deliverable

Using all the artefacts in C:\Users\NadiaRahmatulla\FDE_Program\Gate1\ as a guide against the Scenario XXXXX help me produce 1 markdown specification file for the Validation design in this build name it as scn-5-validation-and-test-design-gfm.md

Final/reviewed versions live under `ScenarioN_Delivery/Final/

### Format
All documents use **GitHub Flavored Markdown (GFM)**. Tables, checkboxes, and fenced code blocks are widely used. Do not convert to other formats.

where possible use mermaid flow diagrams

focus on

**Validation design** — how do you know the agent is working? What do you test? What does failure look like?

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

- **Validation design** — you've named what "working" looks like in testable terms and what the most likely failure modes are. *Good:* failure modes tied to specific decisions in the spec. *Bad:* "we'll write tests" without specifying what behaviour they'd defend.

