# Closed Build Loop — Quick Diagnosis Reference Card

**Use this during the build loop to quickly classify issues and know what to do.**

---

## One-Minute Diagnosis Process

**When a test fails or code doesn't match spec:**

1. **Read the test failure** (what was expected vs. actual)
2. **Find the relevant spec section** (use spec search)
3. **Read the actual code** (what did the agent build?)
4. **Answer these 3 questions:**

```
Q1: Does the code match the spec as written?
    YES → Q1B: Does it match your intent?
           YES → ✅ PASS (no issue)
           NO  → 🔴 CATEGORY 1: Spec Ambiguity (fix spec)
    NO  → Q2: Is the code defensible under any interpretation?
           YES → 🔴 CATEGORY 1: Spec Ambiguity (fix spec)
           NO  → Q3: Does spec address this?
                 YES → 🔴 CATEGORY 2: Builder Misread (re-prompt)
                 NO  → 🟡 CATEGORY 4: Design Gap (update spec, then re-build)
```

---

## The Four Categories (Super Short Version)

| Category | What Happened | What to Fix | Time |
|:--|:--|:--|:--|
| **1. Spec Ambiguity** | Agent chose a valid interpretation you didn't intend | **Rewrite the spec** (don't re-prompt) | 5–10 min |
| **2. Builder Misread** | Agent ignored or contradicted clear spec text | **Re-prompt** with relevant section highlighted | 2–5 min |
| **3. Test Problem** | Code is correct, test is wrong | **Fix the test** | 2–5 min |
| **4. Design Gap** | Spec is silent on something critical | **Add to spec explicitly**, then re-build | 10–20 min |

---

## Diagnosis Checklists

### Checklist: Is This Spec Ambiguity?

- [ ] The code is logically sound
- [ ] The code contradicts the spec OR doesn't match your intent
- [ ] Two smart people could interpret the spec differently
- [ ] The spec uses fuzzy words: "appropriate," "valid," "reasonable," "handle," "if needed"
- [ ] The agent's interpretation is defensible, just not the one you wanted

→ **Fix:** Rewrite the spec to remove ambiguity

---

### Checklist: Is This Builder Misread?

- [ ] The spec is clear and explicit about this behavior
- [ ] The code does the opposite of what the spec says
- [ ] A sentence in the spec directly contradicts the code
- [ ] This is something a code review would catch: "Did you read the part about...?"
- [ ] The agent ignored a constraint, field, default, or requirement

→ **Fix:** Re-prompt with relevant spec section highlighted

---

### Checklist: Is This Test Problem?

- [ ] The code matches the spec exactly
- [ ] The code is logically correct
- [ ] The test checks for specific values, not properties
- [ ] The test makes assumptions not in the spec
- [ ] Re-running with different test data might pass

→ **Fix:** Update the test to match spec + code

---

### Checklist: Is This Design Gap?

- [ ] The spec is clear about what it does specify
- [ ] The spec is silent about something critical
- [ ] A production system obviously needs X (error handling, logging, retry logic)
- [ ] The gap is something that should be true across most systems
- [ ] The agent built exactly what was asked for, but it's incomplete

→ **Fix:** Update spec to explicitly require the missing piece, then re-build

---

## Quick Decision Tree (For Impatient Situations)

```
Does code match spec literally?
├─ YES
│  └─ Does it match your intent?
│     ├─ YES → PASS ✅
│     └─ NO → Spec Ambiguity 🔴
└─ NO
   ├─ Does spec address this behavior?
   │  ├─ YES → Builder Misread 🔴
   │  └─ NO → Design Gap 🟡
   └─ Is code defensible anyway?
      ├─ YES → Spec Ambiguity 🔴
      └─ NO → Builder Misread 🔴
```

---

## Response Templates (Copy-Paste)

### Spec Ambiguity Response

```markdown
I've identified a spec ambiguity. The original wording:
"[quote fuzzy spec text]"

Could be interpreted as:
A) [interpretation 1]
B) [interpretation 2]

I intended (A). Here's the clarified spec:

Before:
[original text]

After:
[precise text with examples or acceptance criteria]
```

### Builder Misread Response

```markdown
The spec says (§ X, [section]):
> [quote exact spec text]

Your code [what it actually does]. This violates the spec.

Please revise to [what it should do instead].

Example: [walk through with concrete input/output]
```

### Test Problem Response

```markdown
The spec says: "[spec text]"
Your code does: [correct behavior per spec]
The test checks: [brittle or wrong assertion]

The test is incorrect. Here's the fix:

Before:
[old test]

After:
[new test that matches spec]
```

### Design Gap Response

```markdown
The spec is clear about [what it specifies]. But it's silent on [missing requirement].

A production system needs [requirement] because [reasoning].

I'm adding this to the spec:

[new section describing requirement]

Then we'll re-build.
```

---

## Red Flags for Each Category

### Spec Ambiguity Red Flags 🚩
- "The agent interpreted X as Y, which is valid but not what I meant"
- "Two people could reasonably read the spec differently"
- "The spec uses 'if needed' or 'appropriate'"
- "The agent's code is sensible; I just want something different"

### Builder Misread Red Flags 🚩
- "The spec clearly says X, but the code does Y"
- "The agent missed a constraint I see right there in the spec"
- "This is obvious from the spec; how did the agent miss it?"
- "The agent ignored a required field or default value"

### Test Problem Red Flags 🚩
- "The code is correct, but the test expects something different"
- "The test works with one dataset but fails with another"
- "The test checks for specific values instead of properties"

### Design Gap Red Flags 🚩
- "The spec doesn't mention error handling, but the code should handle errors"
- "The spec works fine, but it's missing a category of requirements"
- "This is obvious to any production system, but the spec is silent"

---

## Common Misclassifications (Don't Make These Mistakes)

| Mistake | What It Leads To | Right Answer |
|:--|:--|:--|
| Treating Builder Misread as Spec Ambiguity | You rewrite the spec; agent re-builds; same issue appears | Classify as Builder Misread; re-prompt agent |
| Treating Spec Ambiguity as Builder Misread | You re-prompt; agent re-reads ambiguous spec; same issue appears | Classify as Spec Ambiguity; rewrite spec |
| Treating Design Gap as Builder Misread | You re-prompt; agent adds code; it's not production-ready | Classify as Design Gap; update spec with missing requirement |
| Treating Test Problem as Builder Misread | You re-prompt; agent re-builds; tests still fail | Classify as Test Problem; fix the test |

---

## Time Estimates (To Plan Your Build Loop)

| Category | Time | Action |
|:--|:--|:--|
| Spec Ambiguity | 5–10 min | Rewrite 1–2 sentences, clarify with examples |
| Builder Misread | 2–5 min | Highlight spec section, re-prompt |
| Test Problem | 2–5 min | Fix 1 assertion or test setup |
| Design Gap | 10–20 min | Add new requirement to spec, trigger re-build |

**Total expected for Scenario 5 closed loop: 2–3 hours**
- Initial build: 30–45 min (data model + FRs 1–3)
- Review + diagnosis: 60–90 min (identify issues, classify them)
- Re-prompts: 20–40 min (fix each issue)
- Final test run: 10–15 min
- Buffer: 20 min

---

## The Discipline

**Why this matters:**

Every time you classify an issue correctly, you're training yourself to think like a production engineer:

- ✅ Spec ambiguity → you learn to write clearer specs
- ✅ Builder misread → you learn to highlight critical constraints
- ✅ Test problem → you learn to write robust tests
- ✅ Design gap → you learn to think about completeness upfront

**Week 1 skill:** Defensible delegation boundaries.  
**Week 2+ skill:** Understanding where AI is and isn't appropriate.  
**This skill:** Knowing how to diagnose and fix build failures precisely.

**Coaches will see:** You're not just building; you're thinking about how to build better.

---

## When in Doubt: Ask the Agent

If you're unsure whether something is:
- Spec Ambiguity vs. Builder Misread

Ask the agent:

```
I'm diagnosing a build issue. Here's the spec text:
"[spec excerpt]"

Here's your code:
"[code excerpt]"

Is your code a valid interpretation of the spec, or did you misread?
What did you think the spec was asking for?
```

The agent's response will often clarify whether the spec is ambiguous or the agent misread.

---

## Your Goal in the Closed Build Loop

By the end:

1. ✅ You've seen the spec in action (discovered ambiguities, gaps)
2. ✅ You've refined the spec based on the build (spec improved)
3. ✅ You've categorized each fix (learned the diagnosis skill)
4. ✅ You have working code that passes all tests (production-ready)
5. ✅ You can explain each iteration to coaches (disciplined thinking visible)

This is exactly what coaches are looking for in Week 1.

---

**Print this. Keep it open during the build loop. Reference it every time you hit a failure.**

**Most common issue you'll hit:** Builder Misread (agent didn't read a constraint carefully). Most common fix: highlight the exact sentence and re-prompt.

**You got this.**
