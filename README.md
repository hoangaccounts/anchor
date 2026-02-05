# AI Contract Language (AICL)

**A constraint-based contract language for disciplined AI-assisted engineering**

Version: 0.2  
Status: Active research project

---

## You Know This Pain

**What you want:**
```
You: "Let's analyze the requirements for this auth system"
AI: *does research, presents findings*
You: "Good. Now let's design the architecture"
AI: *proposes clean arch with clear boundaries*
You: "Approved. Implement it"
AI: *implements it, consistent with the agreed design*
```

**What actually happens:**
```
You: "Let's analyze the requirements for this auth system"
AI: "Sure! Here's a complete implementation with JWT tokens and a user table..."
    *dumps 200 lines of code*
You: "Stop. I didn't ask for code. We haven't designed anything yet."
AI: "You're right! Let me refactor this to use clean architecture..."
    *dumps more code*
You: 😤
```

---

## What This Looks Like With AICL

Same conversation. Different behavior.

```
You: (load an engineering workflow module)

You: /phase = design
AI: OK — phase set to design

You: Let's analyze the requirements for this auth system
AI: Presents requirements, constraints, and open questions (no code)

You: Good. Now let's design the architecture
AI: Proposes a clean architecture with clear boundaries (no code)

You: Approved. Implement it
AI: REFUSE — code generation is not allowed in phase=design

You: /phase = implement
AI: OK — phase set to implement

You: Approved. Implement it
AI: Generates code consistent with the agreed architecture
```

AICL doesn’t make the assistant “perfect.”  
It makes the workflow easier to repeat — and drift easier to detect and recover from.

---

## Quick Overview (What This Is)

AICL lets you turn free-form chat into an **explicit protocol**:

- **Commands**: operations you invoke (e.g., `/requirements(...)`, `/architecture(...)`, `/code(...)`, `/summarize(...)`)
- **State updates**: explicit context (e.g., `/phase = design`)
- **Rules**: boundaries that allow, refuse, or error when crossed
- **Rendering**: consistent output shapes when you want them (summaries, checklists, reports)

Instead of relying on “please behave” prompts, you define a small command surface and rules that reduce hidden assumptions and mode switching.

---

### In plain English

AICL is a lightweight way to **put guardrails around AI-assisted work**.

It lets you define:
- which **commands** exist and what they can do,
- what state may change (explicit updates only),
- how output should be formatted when structure matters,
- and what happens when boundaries are crossed (**ALLOW / REFUSE / ERROR**).

> In AICL, a “contract” is a declarative bundle of constraints that bounds and re-anchors behavior within a scoped context — not a guarantee of global correctness.

---

## Another Example (Pretty Output)

AICL is also useful when you want reusable structure, not just workflow discipline.

```
You: (load a summarize/review module)

You: /summarize(title="Auth System Design")
AI:
====================
  AUTH SYSTEM SUMMARY
====================

• Goal
  - Secure user authentication with clear separation of concerns

• Decisions
  - Token-based auth
  - Layered architecture (API / domain / data)

• Open Questions
  - Token refresh strategy
  - Password reset flow

• Next Steps
  - Finalize auth flows
  - Move to implementation phase
```

The value isn’t perfect accuracy — it’s **consistent structure you can reuse**.

---

## What This Project Is (and Is Not)

AICL is **not** an attempt to fully control or formalize AI behavior in the general case.

Through building and using AICL, a few constraints became clear:
- AI behavior is probabilistic and entropy-driven
- Long-running sessions drift, even with strong initial constraints
- Natural language instructions alone cannot reliably enforce discipline

Rather than treating these as failures, AICL treats them as **design facts**.

### What AICL *Is*
- A personal constraint DSL for AI-assisted work
- A way to encode known AI failure modes (“AI smells”)
- A repeatable command surface you can reapply across chats
- A tool for attachable structure, not permanent control

### What AICL Is *Not*
- Not a guarantee of global determinism
- Not a replacement for judgment or oversight
- Not a commercial governance or compliance system

---

## Ongoing Work

AICL is being explored along two parallel tracks:

### 1) AI Smells
A catalog of recurring failure modes observed during real LLM-assisted work.

The smell catalog lives in a separate repository:  
https://github.com/hoangaccounts/ai-smells

### 2) Constraint Libraries
Reusable AICL modules designed to counter specific smells and re-anchor behavior.

---

## Core Domain (v0.2)

AICL defines a concrete, end-user-authorable contract language with the following primitives:

- **Modules** — Namespaced containers for contracts and identifiers
- **Contracts** — Activatable policy bundles defining rules and state updates
- **Rules** — Closed-vocabulary permissions (`ALLOW`, `DENY`, `REQUIRE`)
- **StateUpdates (UpdateKeys)** — Explicit, side-effect-free state mutations
- **Commands** — First-class operations invoked via strict syntax
- **Scope** — Machine-checkable bounds for mutating actions
- **Outcomes** — Every turn resolves to `ALLOW`, `REFUSE`, or `ERROR`

The full normative specification lives in `aicl-spec-v0.2.md`.

---

## License

[Your license here]

## Author

[Your details here]
