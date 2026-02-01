# Design Intent Kit

## What This Is

Design Intent Kit is a reusable methodology for collaborating with AI on complex work.
It separates **intent and constraints** from **generation**, so outcomes are reviewable,
comparable, and resistant to drift.

This kit is language-, tool-, and domain-agnostic.

---

## Core Idea

1. Humans declare **intent and non‑negotiable constraints**
2. AI generates artifacts **inside that box**
3. Assumptions and ambiguities are made explicit
4. Humans review and iterate

The kit optimizes for correctness, clarity, and control over speed or creativity.

---

## The Two Artifacts

### 1. Design Intent & Constraints Document

Purpose:
- Define *what must be achieved*
- Define *what must not happen*
- Bound all downstream interpretation

Characteristics:
- Short
- Load‑bearing
- No syntax
- No implementation
- Treated as authoritative

### 2. Generation Prompt

Purpose:
- Tell the AI *what to produce*
- Enforce structure and accountability
- Surface assumptions explicitly

Characteristics:
- Minimal
- Deterministic outputs
- Normative vs explanatory separation

---

## Why This Works

- Prevents silent invention
- Makes AI output comparable across runs/models
- Keeps humans in control of meaning
- Scales across projects and domains

---

## When to Use

Good fits:
- Language design
- Architecture and protocols
- Product specs
- Long‑running or high‑cost projects

Poor fits:
- One‑off questions
- Casual brainstorming
- Throwaway scripts

---

## Relationship to AICL (Future)

Design Intent Kit works today with plain text and prompts.

AICL will later provide infrastructure to:
- Encode this workflow
- Enforce phase gating
- Make violations explicit and machine‑checkable
