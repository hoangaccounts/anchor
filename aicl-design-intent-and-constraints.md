# AI Contract Language (AICL)

**A deterministic contract language for AI systems**

Status: Core philosophy stable, implementation intentionally omitted

> **This document defines the design intent and non-negotiable constraints for AICL and must be treated as authoritative input that bounds all downstream interpretation, specification, and implementation.**

---

## What This Is

**AICL is a deterministic behavior contract between a user and an AI assistant.**

The user explicitly declares **what the AI assistant is allowed to do**.  
The assistant must **refuse everything else**.

There is:
- No inference
- No intent guessing
- No “helpful” extrapolation
- No behavioral drift

If an action is not explicitly permitted by the contract, the assistant must not perform it.

---

## What This Is *Not*

AICL is **not**:
- A prompt-engineering technique
- A suggestion framework
- A best-practices guide
- A UX pattern
- An agent system
- A workflow recommendation

It does not attempt to make the AI “smarter,” “friendlier,” or “more helpful.”

It attempts to make the AI **bound**.

---

## The Actual Problem

There is currently **no deterministic, enforceable mechanism** that allows a user to declare an explicit behavior contract with an AI assistant and have that contract be treated as binding.

As a result:
- AI behavior is inferred rather than declared
- Constraints are advisory rather than mandatory
- Violations are ambiguous rather than explicit
- Behavior varies across chats even with identical intent

These failures are **not the core problem**.  
They are **symptoms of the missing contract**.

---

## Why Existing Prompting Fails

Traditional prompting relies on:
- Intent guessing
- Implicit expectations
- Cooperative interpretation
- Non-deterministic compliance

That is acceptable for casual or creative use.

It is insufficient when the user requires:
- Explicit control
- Phase gating
- Predictable refusal
- Repeatable collaboration rules

---

## The Core Goal (Canonical)

> **Provide a deterministic behavior contract between a user and an AI assistant, where the user explicitly declares allowed behavior and the assistant must refuse all other behavior without inference or drift.**

This is the **load-bearing goal** of AICL.

Everything else is downstream.

## End-to-End Completeness (Required)

A valid derivation of AICL MUST result in a complete, self-contained system
that is directly usable by an end user.

Outputs that define only meta-rules, conceptual contracts, or partial frameworks
without specifying a concrete, usable language SHALL be considered incomplete.

If achieving end-to-end completeness requires introducing structure, syntax,
or mechanics not explicitly defined in this document, the assistant MUST do so
explicitly and justify those choices against the stated design intent and
constraints.


## Determinism (Design-Level Definition)

For the purposes of AICL, *deterministic* means that given the same contract,
inputs, and declared phase, the assistant must produce the same **class of behavior**
(e.g., allowed vs refused, error vs success, phase-appropriate artifact types),
though not necessarily byte-for-byte identical wording.

Determinism applies to behavior and enforcement, not stylistic expression.


---

## Refusal and Error Semantics

Refusal is a correct and expected outcome when a requested action is not
permitted by the active contract.

Errors indicate contract violations, ambiguity, or invalid states and must
halt further progress for the current turn.

Refusals communicate lack of permission; errors communicate broken invariants.

## Consequences of This Goal

If such a contract exists, then—*as consequences, not objectives*—it becomes possible to:

- Enforce phases (e.g., design without implementation)
- Prevent premature execution
- Maintain architectural discipline
- Eliminate behavioral drift across sessions
- Produce explicit, structured refusals on violation

These are **outcomes**, not motivations.

---

## Critical: Self-Enforced by the Assistant

AICL is designed to be:
- Embedded directly in chat context
- Interpreted and enforced by the assistant itself
- Binding for the duration of the session

No external runtime, compiler, or validator is required for the concept to function.

If the assistant violates the contract, that is a **contract breach**, not a misunderstanding.

## Scope of Enforcement

The AICL contract constrains the assistant’s reasoning and decision-making,
not only its visible outputs.

The assistant must not internally pursue plans or actions that are forbidden
by the contract, even if those actions would not be emitted in the response.

---

## Phase Gating (Derivable Capability)

AICL does not prescribe specific phases of work.

Instead, it enables phase gating as a derivable capability by allowing the
contract to constrain which categories of actions or artifacts are permitted
at any given time.

Any specific phase model (e.g., research, design, implementation) is an
application of the contract, not a requirement of the language itself.

## Tradeoffs

**What you gain**
- Deterministic behavior
- Explicit control
- Clear refusal semantics
- Repeatable collaboration

**What you give up**
- Implicit understanding
- Conversational flexibility
- “Guess what I meant”
- Casual improvisation

This is an intentional trade.

---

## When This Makes Sense

### Good fits
- Multi-phase engineering work
- Architecture-sensitive collaboration
- Long-running or high-cost projects
- Teams that need consistency

### Poor fits
- One-off questions (“how do I reverse a string?”)
- Pure brainstorming
- Casual exploration
- Situations where ambiguity is desirable

---

## Assumptions and Gaps

When required details are not specified by the contract, the assistant may
introduce assumptions only if those assumptions are made explicit,
enumerated, and treated as provisional.

Unstated assumptions must not be relied upon for correctness or enforcement.

## Roadmap (Conceptual)

- Define a minimal, stable behavioral core
- Enable composable contract rules
- Provide reusable contract templates
- Optional tooling (non-required, non-authoritative)

---

## Contributing / Feedback

Especially valuable feedback:
- Where does this model break in real workflows?
- What contracts do you wish you could enforce?
- What should be in the smallest viable “core”?
- What should be explicitly out of scope forever?

---

## License

[Your license here]

## Author

[Your details here]
