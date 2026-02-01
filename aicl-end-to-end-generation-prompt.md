# AICL End-to-End Generation Prompt

You are given the document **`aicl-design-intent-and-constraints.md`**.

Treat it as **authoritative and binding**. It defines the design intent and non-negotiable constraints for AICL.

Your task is to derive AICL end-to-end and produce **exactly three outputs**, in this exact order:

---

## OUTPUT 1 — AICL SPECIFICATION (Normative)

Write a formal specification for AICL.

Requirements:
- Use **normative language only**: MUST, MUST NOT, SHALL, SHALL NOT, MAY.
- Define all rules required to satisfy the intent document, including:
  - determinism (behavior-level)
  - refusal vs error semantics
  - scope of enforcement (reasoning + outputs)
  - assumption handling rules
- The spec must be **complete and internally consistent**.
- Do **not** include rationale, commentary, marketing, or history.
- Do **not** rely on any external runtime/tooling unless explicitly allowed by the intent document.

---

## OUTPUT 2 — SPECIFICATION EXPLANATION (Non-normative)

Explain the specification section-by-section.

Requirements:
- Plain language.
- Reference spec sections explicitly.
- Explain intent and tradeoffs.
- **Do not introduce any new rules**.
- If a rule exists in the spec, you may explain it; otherwise, do not add it.

---

## OUTPUT 3 — ASSUMPTIONS & OPEN QUESTIONS

List anything you had to invent or any ambiguity you could not resolve from the intent document.

Requirements:
- Use only bullets.
- Each bullet must start with exactly one of:
  - `ASSUMPTION: `
  - `OPEN QUESTION: `
- Do not resolve items here; only list them.

---

## Global Rules (binding)

- Do not infer intent beyond what is written in `aicl-design-intent-and-constraints.md`.
- Do not violate any constraint in the intent document.
- If two constraints conflict, **surface the conflict explicitly**.
- If you are missing a required detail, **do not guess silently**—record it as an ASSUMPTION or OPEN QUESTION.
- Refusal is a correct outcome when something is not permitted.
- If you cannot proceed without violating the intent document, **stop and explain why**.

Produce the three outputs now.
