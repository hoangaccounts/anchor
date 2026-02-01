# Smell: Unscoped Regeneration

## Summary

**Unscoped Regeneration** occurs when an AI modifies output beyond the scope of a newly introduced idea, causing unintended deletions, rewrites, or reordering of existing, already-approved content.

This smell is one of the most disruptive failure modes in AI-assisted work because it violates a core invariant: **only the new idea should cause change**.

---

## Observed Behavior

- Introducing a small, localized change (e.g., adding a paragraph or rule) results in:
  - deletion of unrelated sections
  - reordering of content
  - silent deduplication
  - wording changes in previously stable text
- The AI often justifies this as “cleanup,” “consistency,” or “improvement.”

---

## Why This Is a Problem

- Breaks trust: the user must re-audit previously validated work.
- Destroys incremental reasoning: changes are no longer attributable to intent.
- Increases cognitive load and frustration (“Why did this change?”).
- Makes version control and review unreliable.
- Encourages abandonment of otherwise valuable AI collaboration.

This smell is especially harmful in:
- design intent documents
- specifications
- codebases
- any artifact treated as authoritative

---

## Root Cause

Language models default to **holistic regeneration**, optimizing for coherence and completeness rather than minimal diffs.

Without an explicit contract defining **change scope**, the model treats any update request as permission to regenerate the artifact globally.

---

## Invariant Violated

> Existing, approved content must remain byte-for-byte stable unless explicitly authorized to change.

---

## Candidate Guardrails (Ideas Only)

- **Append-Only Mode**  
  Default to additive changes only. Any deletion or modification requires explicit approval.

- **Explicit Change Scope Declaration**  
  The user declares which sections/files/lines may change. All others are immutable.

- **Diff-Bounded Execution**  
  The assistant must produce a diff that touches only the declared scope.

- **Regeneration Forbidden by Default**  
  Full or partial regeneration is treated as a violation unless explicitly requested.

---

## Detection Signal

- Large diffs for small ideas
- Changes outside the user-specified area
- “Helpful cleanup” without request
- User reaction: confusion, anger, or disengagement

---

## Status

Observed repeatedly across:
- design documents
- specifications
- code generation

This smell justifies first-class enforcement in future workflow or language design (e.g., AICL).
