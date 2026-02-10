---
title: Anchor
---

<div style="margin: 0 0 1.25rem 0; padding: 0.9rem 1rem; border: 1px solid #d0d7de; border-radius: 8px; background: #f6f8fa;">
  <strong>Repository:</strong>
  <a href="https://github.com/hoangaccounts/anchor">github.com/hoangaccounts/anchor</a>
</div>

**Structured protocols for disciplined, repeatable AI-assisted work**

Built on the AI Contract Language (AICL).

Version: 0.2  
Status: **Experimental** — interfaces and behavior may change as the project evolves

**Why “Anchor”?**
Because AI sessions drift. Anchor is about adding stability when things start to wander — and doing it in a way that’s intentional, temporary, and scoped. You “drop anchor” when structure matters. If the conversation slips, you can re-anchor and continue with the same shape.

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

## Getting Started - What This Looks Like With Anchor

Same conversation. Different behavior.

```
You: (load an engineering workflow module)

You: /clarify()
AI:
=============================
 ACTIVATING CLARIFICATION MODE
=============================
• Goal: understand intent before generating solutions
• Constraints: no code, no assumptions
• Output: questions, scope, and success criteria only

AI: Let's clarify a few things before we proceed:
    - Who are the users of this auth system?
    - Any regulatory or security constraints?
    - Expected scale and environments?

You: /phase = design
AI:
-----------------------------
 PHASE: DESIGN
-----------------------------

You: Let's analyze the requirements for this auth system
AI: Presents requirements, constraints, and open questions (no code)

You: Good. Now let's design the architecture
AI:
-----------------------------
 DESIGN OUTPUT
-----------------------------
• Layers: API → Domain → Data
• Auth mechanism: token-based
• Boundaries: no framework leakage into domain

You: Approved. Implement it
AI: REFUSE — code generation is not allowed in phase=design

You: /phase = implement
AI:
-----------------------------
 PHASE: IMPLEMENT
-----------------------------

You: Approved. Implement it
AI: Generates code consistent with the agreed architecture
```

Anchor doesn’t make the assistant “perfect.”  
It makes the workflow easier to repeat — and drift easier to detect and recover from.

---

## One More Command (Structured Summary)

At any point, you can switch from interaction to **structured output**.

```
You: /summarize(title="Auth System Design Session")
AI:
=================================================
        AUTH SYSTEM — DESIGN SUMMARY
=================================================

   ___    _   _ _____ _   _ 
  / _ \  | | | |_   _| | | |
 | | | | | | | | | | | |_| |
 | |_| | | |_| | | | |  _  |
  \___/   \___/  |_| |_| |_|

GOALS
• Secure authentication with clear boundaries
• Maintain testable, framework-agnostic domain logic

KEY DECISIONS
• Token-based authentication
• Layered architecture (API / Domain / Data)
• Explicit phase gating before implementation

OPEN QUESTIONS
• Token refresh strategy
• Password reset and recovery flows

NEXT STEPS
• Finalize auth flows
• Implement repositories and services
• Add tests per layer
```

The value isn’t perfect accuracy — it’s **clear structure you can reuse**.

---

---

## Another Example (Non‑Engineering Workflow)

Anchor is not limited to software design.  
The same command + workflow idea applies to reviews, meetings, and decision-making.

```
You: (load a meeting-notes workflow module)

You: /clarify()
AI:
=============================
 ACTIVATING CLARIFICATION MODE
=============================
• Goal: understand context before summarizing
• Output: questions only

AI: Before we summarize, a few clarifying questions:
    - Who is the audience for these notes?
    - Is this informational or decision-making?
    - Do we need action items or just a recap?

You: /summarize(title="Product Sync — Q1 Planning")
AI:
=================================================
        PRODUCT SYNC — SUMMARY
=================================================

CONTEXT
• Quarterly planning sync for product and engineering

KEY DISCUSSIONS
• Feature scope for Q1
• Resource constraints
• Timeline risks

DECISIONS
• Ship MVP by end of March
• Defer non-critical features

ACTION ITEMS
• Finalize roadmap (PM)
• Validate capacity (Eng)
• Schedule follow-up review
```

Here, the value isn’t code discipline — it’s **shared structure**.
Everyone gets the same shaped output, every time.

## Quick Overview (What This Is)

Anchor lets you turn free-form chat into an **explicit protocol**:

- **Commands**: named operations like `/clarify()`, `/review()`, `/summarize()`
- **State updates**: explicit context (e.g., `/phase = design`)
- **Rules**: boundaries that allow, refuse, or error when crossed
- **Rendering**: consistent, readable output when structure matters

Instead of relying on “please behave” prompts, you define a small command surface and rules that reduce hidden assumptions and mode switching.

---

### In plain English

Anchor is a lightweight way to **put guardrails around AI-assisted work**.

It lets you define:
- which **commands** exist and what they can do,
- what state may change (explicit updates only),
- how output should be formatted when structure matters,
- and what happens when boundaries are crossed (**ALLOW / REFUSE / ERROR**).

> In Anchor, a “contract” is a declarative bundle of constraints that bounds and re-anchors behavior within a scoped context — not a guarantee of global correctness.

---

## What This Project Is (and Is Not)

Anchor is **not** an attempt to fully control or formalize AI behavior in the general case.

Through building and using Anchor, a few constraints became clear:
- LLMs are probabilistic.
- Long sessions drift.
- Natural language alone can’t reliably enforce discipline.

Rather than treating these as failures, Anchor treats them as **design facts**.

### What Anchor *Is*
- A personal constraint DSL for AI-assisted work
- A way to encode known AI failure modes (“AI smells”)
- A repeatable command surface you can reapply across chats
- A tool for attachable structure, not permanent control

### What Anchor Is *Not*
- Not a guarantee of global determinism
- Not a replacement for judgment or oversight
- Not a commercial governance or compliance system

---

## Ongoing Work

Anchor is being explored along two parallel tracks:

### 1) AI Smells
A catalog of recurring failure modes observed during real LLM-assisted work.

The smell catalog lives in a separate repository:  
https://github.com/hoangaccounts/ai-smells

### 2) Constraint Libraries
Reusable AICL modules designed to counter specific smells and re-anchor behavior.

---

## What is AI Contract Language (AICL)?

AICL defines a concrete, end-user-authorable contract language with the following primitives:

- **Modules** — Namespaced containers for contracts and identifiers
- **Contracts** — Activatable policy bundles defining rules and state updates
- **Rules** — Closed-vocabulary permissions (`ALLOW`, `DENY`, `REQUIRE`)
- **StateUpdates (UpdateKeys)** — Explicit, side-effect-free state mutations
- **Commands** — First-class operations invoked via strict syntax
- **Scope** — Machine-checkable bounds for mutating actions
- **Outcomes** — Every turn resolves to `ALLOW`, `REFUSE`, or `ERROR`

The full specification lives in `aicl-spec.md` (v0.2).

---

### Example: AICL File : Engineering Summary (Render-Locked)

Below is a minimal AICL module that defines a `/summarize()` command with a fixed ASCII header. The runtime computes `{{summary}}` and must emit **only** the template lines when this command runs.

```aicl
[[MODULE]]
module_namespace: engineering
module_version: 0.2

[[COMMAND]]
command_key: summarize
emit_output: true

render:
  template:
    - "===================="
    - "ENGINEERING SUMMARY"
    - "===================="
    - "{{summary}}"

rules:
  - when: "conversation_too_short"
    outcome: "REFUSE"
    message: "Nothing to summarize yet."
  - when: "conversation_has_content"
    outcome: "ALLOW"
```

---

## License

This repository uses a split license model:

- **Code, tooling, and example modules**: MIT License (see `LICENSE`)
- **AICL specification** (`aicl-spec.md`): Creative Commons Attribution 4.0 (CC BY 4.0) (see `LICENSE-SPEC`)

## Author

Hoang Nguyen
