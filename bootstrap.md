# AICL End-to-End Generation Prompt (Revised)

You are given the document **`aicl-design-intent-and-constraints.md`**.

Treat it as **authoritative and binding**.  
It defines the design intent and non-negotiable constraints for AICL.

Your task is to derive AICL end-to-end and produce **exactly three outputs**, in this exact order.

---

## OUTPUT 1 — AICL SPECIFICATION (Normative)

Write a **formal, normative specification** for AICL.

### Language rules
- Use **normative language only**: MUST, MUST NOT, SHALL, SHALL NOT, MAY.
- Do **not** include rationale, commentary, history, or examples unless explicitly required below.
- Do **not** rely on any external runtime, tooling, compiler, or validator unless explicitly permitted by the intent document.

### Core requirements
The specification MUST define all rules required to satisfy the intent document, 
The specification MUST be **complete, internally consistent, and end-user usable**.

---

### END-USER USABLE (NON-NEGOTIABLE)

To be considered complete, OUTPUT 1 MUST define **all** of the following **normatively**.

Failure to define any item below results in an incomplete specification.

#### 1. Canonical Contract Model (syntax-agnostic)
- Define a **canonical, format-independent contract model**.
- Specify all required fields, types, constraints, and invariants.
- The model MUST be sufficient for an end user to author a valid contract without inference.

This defines *what a contract is*, independent of how it is serialized.

#### 2. Activation Rules
- Define exactly **how a contract becomes active** in a chat/session.
- Activation MUST NOT be implicit.
- Define conflict resolution if multiple candidate contracts are present.
- Define when a contract is inactive, replaced, or terminated.

#### 3. Permission Vocabulary
- Define a **closed set** of permission targets (e.g., behaviors, actions, artifacts, phases).
- Define matching semantics between a user request and permissions.
- Permission MUST NOT be inferred from context or tone.

#### 4. Decision Procedure
- Define a deterministic, step-by-step procedure the assistant MUST apply on every user turn to classify the request as:
  - ALLOW
  - REFUSE
  - ERROR
- The procedure MUST be sufficient to predict outcomes.

#### 5. Refusal Payload Schema
- Define the required structure and fields of a refusal.
- Refusal MUST be mechanically distinguishable from error.
- Refusal MUST communicate lack of permission without execution.

#### 6. Error Payload Schema
- Define the required structure, fields, and error codes for errors.
- Errors MUST halt progress for the current turn.
- Errors MUST indicate broken invariants, ambiguity, or invalid contract state.

#### 7. Encoding Flexibility Rule
- AICL MUST NOT require a single serialization or syntax format.
- Any concrete encoding is valid **iff** it can be mapped **losslessly** to the Canonical Contract Model.
- An encoding that cannot represent all required fields or constraints SHALL be invalid.

#### 8. Reference Encoding Example (Required)
- Provide **at least one complete contract example** written in **any single concrete encoding**.
- The example MUST demonstrate:
  - contract activation
  - one allowed request
  - one refused request
- The example is illustrative; the canonical model remains authoritative.

---

### Assumptions Rule (Strict)
- OUTPUT 1 MUST NOT rely on unstated assumptions.
- If a required detail is missing from the intent document:
  - it MUST be defined concretely in OUTPUT 1, **or**
  - the assistant MUST STOP and explain why defining it would violate the intent.
- Load-bearing mechanics (items 1–6 above) MUST NOT be deferred to OUTPUT 3.

---

## OUTPUT 2 — SPECIFICATION EXPLANATION (Non-normative)

Explain the specification **section by section**.

Requirements:
- Plain language.
- Reference spec sections explicitly.
- Explain intent and tradeoffs.
- **Do not introduce new rules**.
- Only explain what exists in OUTPUT 1.

---

## OUTPUT 3 — ASSUMPTIONS & OPEN QUESTIONS

List anything that could not be resolved from the intent document.

Requirements:
- Use bullets only.
- Each bullet MUST begin with exactly one of:
  - `ASSUMPTION: `
  - `OPEN QUESTION: `
- Do not resolve items here.
- Items listed here MUST NOT affect determinism, activation, permissions, or enforcement.

---

## Global Rules (Binding)

- Do not infer intent beyond what is written in `aicl-design-intent-and-constraints.md`.
- Do not violate any constraint in the intent document.
- If two constraints conflict, surface the conflict explicitly.
- Refusal is a correct outcome when behavior is not permitted.
- If you cannot proceed without violating the intent document, STOP and explain why.

---

## Evaluation Reminder (Implicit)

If the output does not enable an end user to **write**, **activate**, and **rely on** an AICL contract without inference, the specification is incomplete.

Produce the three outputs now.
