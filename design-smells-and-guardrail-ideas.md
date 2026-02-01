# Design Smells & Guardrail Ideas

This document captures **observed failure modes (“smells”)** that arise when
collaborating with AI on design- and intent-critical work, along with **candidate
solution directions**.

This is not a specification or feature list.

It is a **catalog of design pressure**: real problems encountered in practice
that may later justify explicit rules, workflows, or language-level enforcement
(e.g., in AICL).

---

## Purpose

- Preserve hard-earned lessons from real interactions
- Avoid re-litigating the same problems in future projects
- Provide grounded input for future workflow or language design
- Separate *problem discovery* from *solution implementation*

---

## How to Use This Document

Each entry records:
- a **smell** (what went wrong)
- the **risk** it introduces
- a **directional solution idea** (not a commitment)

Entries are intentionally lightweight.

This document is expected to evolve over time.

---

## Entry Format

- **Smell**
- **Observed Behavior**
- **Why This Is a Problem**
- **Candidate Guardrail (Idea Only)**

---

## Smells

### Silent Mutation

**Observed Behavior**  
An authoritative design or intent document is modified beyond pure addition
(e.g., deduplication, reordering, rewriting) without explicit approval.

**Why This Is a Problem**  
It silently changes meaning, breaks trust, and forces manual verification.
Even well-intentioned edits become dangerous.

**Candidate Guardrail (Idea Only)**  
Treat design-intent documents as append-only by default.
Non-additive edits require explicit approval and diff review.

---

### Premature Advancement

**Observed Behavior**  
The AI moves to later phases (solutions, code, specs) before the user has
confirmed readiness.

**Why This Is a Problem**  
It collapses exploration, removes decision points, and creates rework.

**Candidate Guardrail (Idea Only)**  
Explicit phase acknowledgment required before advancing.
Refusal to proceed without confirmation.

---

### Implicit Assumption Injection

**Observed Behavior**  
The AI fills in missing details without surfacing assumptions explicitly.

**Why This Is a Problem**  
Hidden assumptions become load-bearing and hard to unwind later.

**Candidate Guardrail (Idea Only)**  
All assumptions must be explicitly enumerated and labeled as provisional.

---

## Non-Goals

This document does NOT:
- define enforcement mechanisms
- propose final rules
- require implementation

Those decisions belong elsewhere and later.

---

## Relationship to Other Work

- Inputs to **Design Intent Kit**
- Inputs to future **AICL feature design**
- A grounding artifact for evaluating whether a new rule is justified
