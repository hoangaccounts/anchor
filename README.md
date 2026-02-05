# AI Contract Language (AICL)

**A constraint-based contract language for disciplined AI-assisted engineering**

Version: 0.2  
Status: Active research project

---

## Quick User Story (30 seconds)

You’re working with an AI assistant on a real codebase.

1. You start in **analysis/design**.
2. The assistant **jumps to implementation**, changes files you didn’t approve, and invents details.
3. You correct it… and 10 minutes later it drifts again.

AICL is how you make that workflow **repeatable**:

- You declare what’s allowed *right now* (e.g., “design only; no code”).
- You declare what state can change (e.g., `phase = design`).
- Every turn resolves to a simple outcome: **ALLOW / REFUSE / ERROR**.
- When drift appears, you **re-anchor** by reasserting the contract.

---

### In plain English

AICL is a lightweight way to **put guardrails around AI-assisted work**.

It lets you define:

- what the assistant may do in the current phase (design vs implement),
- what scope is required for edits,
- and how violations are handled deterministically.

> In AICL, a “contract” is a declarative bundle of constraints that bounds and re-anchors behavior within a scoped context — not a guarantee of global correctness.

---

## What This Project Is (and Is Not)

AICL is **not** an attempt to fully control or formalize AI behavior in the general case.

Through building and using AICL, a few hard constraints became clear:

- AI behavior is **probabilistic and entropy-driven**
- Long-running sessions **drift**, even with strong initial constraints
- Natural language instructions alone cannot reliably enforce discipline
- No static contract can permanently “lock” behavior without re-anchoring

Rather than treating these as failures, AICL embraces them as **design facts**.

### What AICL *Is*

AICL is a **personal constraint DSL** used to:

- Encode *known AI failure modes* (“AI smells”)
- Reapply structural constraints when those smells appear
- Make violations **obvious and recoverable**
- Reduce repeated negotiation with the assistant
- Capture and reuse *taste*, *workflow discipline*, and *guardrails*

In practice, AICL behaves more like:

- `.editorconfig` for AI behavior  
- Lint rules for conversational workflows  
- A re-anchoring mechanism when the model drifts  

### What AICL Is *Not*

- Not a guarantee of global determinism
- Not a replacement for judgment or oversight
- Not a general-purpose governance system
- Not a commercial compliance or enforcement product

The assistant still interprets the language.  
The value comes from **making interpretation explicit, constrained, and repeatable**.

---

## You Know This Pain

**What you want:**
```
You: "Let's analyze the requirements for this auth system"
AI: *does research, presents findings*
You: "Good. Now let's design the architecture"
AI: *proposes clean arch with clear boundaries*
You: "Approved. Implement the user repository"
AI: *writes code following DDD patterns*
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

## The Core Idea

AI Contract Language (AICL) treats AI interaction as a **constraint-driven workflow**, not a free-form conversation.

Instead of relying on inferred intent or polite instructions, you author **explicit, machine-checkable contracts** that define:

- what actions are allowed or denied,
- what state may change,
- what scope is required,
- and how conflicts and errors are resolved.

Within an active contract and scope, outcomes are **locally deterministic**:
`ALLOW`, `REFUSE`, or `ERROR`.

Determinism here is **scoped and contextual**, not absolute.

---

## Ongoing Work

AICL is being explored along two parallel tracks:

### 1) AI Smells

A catalog of recurring failure modes observed during real LLM-assisted engineering work, such as:

- Context drift
- Phase jumping
- Premature implementation
- Spec erosion
- Over-helpfulness
- Render leakage

The smell catalog lives in a separate repository:  
https://github.com/hoangaccounts/ai-smells

### 2) Constraint Libraries

Reusable AICL modules designed to counter specific smells.

These libraries are:

- loaded selectively, not globally
- used to re-anchor behavior when drift appears
- designed to reduce repeated negotiation

---

## Minimal example (v0.2 shape)

```yaml
[[MODULE]]
module_name: engineering_workflow
module_version: "0.2"
module_namespace: eng
description: "Example module"

[[CONTRACT]]
contract_id: phase_gate
version: "0.2"
rules:
  - rule_id: "require_scope_for_edits"
    effect: "REQUIRE"
    action_id: "EDIT_EXISTING_ARTIFACT"
    target: null
    scope_required: true
updates:
  - update_id: "set_phase"
    update_key: "phase"
    args_schema:
      value: string
    effects:
      active_profiles: ["{value}"]
metadata:
  autoload: true
[[/CONTRACT]]

[[COMMAND]]
command_id: eng.status
command_key: status
args_schema:
  summary: bool
result_schema:
  phase: string
effects:
  - read_state:
      paths: ["active_profiles"]
render:
  format: "text"
[[/COMMAND]]

[[/MODULE]]
```

```
/phase = design
/status(summary: true)
```

---

## Core Domain (v0.2)

AICL defines a concrete, end-user-authorable contract language with the following primitives:

- **Modules** — Namespaced containers for contracts and identifiers
- **Contracts** — Activatable policy bundles defining rules and state updates
- **Rules** — Closed-vocabulary permissions (`ALLOW`, `DENY`, `REQUIRE`) over actions and targets
- **Actions** — Explicitly declared operations (read-only or mutating)
- **StateUpdates (UpdateKeys)** — Deterministic, side-effect-free state mutations
- **Commands** — First-class objects invoked via strict syntax
- **Policy** — A derived, conflict-checked view of all active rules
- **Scope** — Machine-checkable bounds required for mutating actions
- **Outcomes** — Every turn resolves to exactly one of `ALLOW`, `REFUSE`, or `ERROR`

The full normative specification lives in `aicl-spec-v0.2.md`.

---

## License

[Your license here]

## Author

[Your details here]
