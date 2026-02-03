# AI Contract Language (AICL)

**A deterministic contract language for AI systems**

Version: 0.2  
Status: Core domain specified and stable

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

You can't say "just don't implement yet" because:
- It "understands" but doesn't **enforce**
- Next message it forgets
- Next chat you start from scratch
- No way to make it **structurally impossible** to jump ahead

Sound familiar?

---

## The Real Problem

If you're a software engineer trying to work with AI, you're constantly fighting:

1. **Phase jumping** - AI skips research/design and jumps straight to implementation
2. **SOLID violations** - Tight coupling, no dependency injection, God classes everywhere
3. **Workflow chaos** - No way to enforce "we're in design phase, code is forbidden"
4. **Inconsistency** - Chat A follows patterns, Chat B is complete chaos
5. **Chattiness** - You want a simple "/next" but get a essay explaining everything

Traditional AI prompting relies on:
- **Intent guessing** - The AI infers what you meant
- **Implicit behavior** - Hidden assumptions about how it should act  
- **Non-deterministic interpretation** - Same prompt, different behaviors

You can write custom instructions, but they're suggestions, not enforcement. The AI "understands" but doesn't comply structurally.

This works for creative conversations, but fails when you need **engineering discipline**.

---

## The Solution

AI Contract Language (AICL) treats AI interaction as a **deterministic contract**, not a conversation.

Instead of relying on inferred intent or polite instructions, you author **explicit, machine-checkable contracts** that define:
- what actions are allowed or denied,
- what state may change,
- what scope is required,
- and how conflicts and errors are resolved.

These contracts are parsed and **self-enforced by the AI assistant itself**, producing deterministic outcomes (`ALLOW`, `REFUSE`, or `ERROR`) for every turn.

Commands and UpdateKeys are the explicit execution mechanisms: only properly recognized invocations can change state or trigger declared effects.

### Minimal example (v0.2 shape)

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

Deterministic enforcement.



---

## Philosophy: Why This Matters
---

## Philosophy: Why This Matters

The AI revolution is giving us powerful assistants, but we're still controlling them with the equivalent of shouting suggestions across a room. As AI becomes more capable, we need **precision control mechanisms** that match that capability.

This language is an experiment in treating AI assistance as **infrastructure** rather than **conversation**. Just as we don't write prose to configure databases or deploy servers, we shouldn't need creative persuasion to get deterministic AI behavior.

The goal isn't to replace natural conversation - it's to provide an **alternative mode** for contexts where reliability matters more than flexibility.

---

## Critical: Self-Enforced Architecture

**This language is embedded directly in AI chat context and self-enforced by the AI assistant itself.**

There is **no external compiler, runtime, or enforcement engine**. The AI reads these rules, parses them, and **treats them as binding constraints on its own behavior**. The assistant becomes both the interpreter and the executor.

This is not a tool that *controls* the AI from outside. This is a **behavioral contract** that the AI enforces on itself deterministically for the duration of the chat session.

Trust, but verify.

---


---

## Core Domain (v0.2)

AICL v0.1 defines a concrete, end-user-authorable contract language with the following primitives:

- **Modules** — Namespaced containers for contracts and identifiers.
- **Contracts** — Activatable policy bundles defining rules and state updates.
- **Rules** — Closed-vocabulary permissions (`ALLOW`, `DENY`, `REQUIRE`) over actions and targets.
- **Actions** — Explicitly declared operations (read-only or mutating).
- **StateUpdates (UpdateKeys)** — Deterministic, side-effect-free state mutations invoked via strict syntax.
- **Commands** — First-class core objects, defined via `[[COMMAND]]` blocks and invoked deterministically with `/name(args)`; they may emit output and/or read/write state only through explicitly declared effects.
- **Policy** — A derived, conflict-checked view of all active rules.
- **Scope** — Machine-checkable bounds required for mutating actions.
- **Outcomes** — Every turn deterministically resolves to exactly one of `ALLOW`, `REFUSE`, or `ERROR`.

The full normative specification lives in `aicl-spec-v0.2.md`.


## Core Principles

### 1. **Contracts Are the Enforcement Mechanism**

AICL enforcement comes only from loaded `[[MODULE]]` / `[[CONTRACT]]` artifacts. Free-form prose may exist, but it MUST NOT change policy or state.

### 2. **Deterministic Outcomes**

Every user turn MUST resolve to exactly one outcome:
- `ALLOW` — all checks pass; permitted actions execute atomically.
- `REFUSE` — valid state, but request is not permitted or required conditions are unmet.
- `ERROR` — invalid state or invariant violation; atomic abort.

### 3. **Fail-Closed Identity and Conflicts**

Unknown identifiers, broken invariants, and policy conflicts MUST halt deterministically (`FAIL` at load-time or `ERROR` at runtime, per spec).

### 4. **Explicit State Mutation Only**

State changes are permitted only via declared **StateUpdates (UpdateKeys)** or declared **Commands** effects. There are no implicit mutations.

### 5. **Scope-Gated Mutation**

Actions that change existing artifacts MUST require an approved scope, or the turn MUST `REFUSE`.


---

## Key Concepts
---

## Key Concepts

### Modules

A `[[MODULE]]` is a namespace container. Identifiers inside a module are canonicalized with `module_namespace` for determinism.

### Contracts

A `[[CONTRACT]]` is a YAML document containing:
- `rules` (permissions over actions/targets)
- `updates` (StateUpdates invoked via UpdateKeys)
- optional metadata like `autoload`

### Rules (ALLOW / DENY / REQUIRE)

Rules use a closed vocabulary and are conflict-checked across active contracts.

### StateUpdates and UpdateKeys

A StateUpdate defines an `update_key` invoked via strict assignment syntax:

```
/name = <rhs>
/ns.name = <rhs>
```

RHS parses under a restricted YAML subset and is validated against `args_schema`. Near-misses `REFUSE` and do not change state.

### Commands

Commands are first-class core objects, defined in `[[COMMAND]]` blocks and invoked via `/name(args)`. They are deterministic, may perform only declared effects, and are not free-form “tools”.


---

## Quick Start: Engineering Workflow
---

## Quick Start: Engineering Workflow

### 1. Author a module + contract

```yaml
[[MODULE]]
module_name: backend_development
module_version: "0.1"
module_namespace: backend
description: "Example workflow module"

[[CONTRACT]]
contract_id: workflow_core
version: "0.1"
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
[[/MODULE]]
```

### 2. Load and activate

If `metadata.autoload=true`, the contract activates at startup. Otherwise, activation is performed via declared state mutations (see spec).

### 3. Drive state with an UpdateKey

```
/phase = design
```

From here, policy (rules + scope) deterministically gates what the assistant may do.


---

## Design Trade-offs
---

## Design Trade-offs

### What You Gain

- **Deterministic behavior** - Same input = same output
- **Phase enforcement** - AI CANNOT skip ahead structurally
- **Engineering discipline** - SOLID, Clean Arch, DDD enforced by protocol
- **Explicit state** - No hidden context or assumptions
- **Repeatable workflows** - Same process across all chats
- **Clear error messages** - Exact violation when rules break

### What You Give Up

- **Conversational flexibility** - Can't just chat casually about implementation
- **Implicit understanding** - Must explicitly define phases and transitions
- **Fuzzy intent** - The AI won't "figure out what you meant"
- **Learning curve** - More complex than natural language prompting

---

## When to Use This

### Good Fits

- **Engineering projects requiring discipline** - Following SOLID, Clean Arch, DDD patterns
- **Multi-phase workflows** - Research → Design → Implement → Test cycles
- **Collaborative development** - Teams need shared, documented AI behavior
- **Regulated environments** - Auditable, deterministic AI interactions
- **Complex state management** - Tracking phases, approvals, constraints across sessions

### Poor Fits

- **One-off questions** - "How do I reverse a string?" doesn't need formal commands
- **Creative brainstorming** - Conversational flow is actually helpful
- **Exploratory conversations** - You don't know what you want yet
- **Quick scripts** - Overhead isn't worth it for throwaway code

---

## Architecture

### Parsing & Validation

1. **Block markers** - `[[BLOCK_TYPE]]` ... `[[/BLOCK_TYPE]]` must be properly nested
2. **YAML mode** - Structural blocks parsed as YAML 1.2
3. **Text mode** - Guidance blocks treated as raw text
4. **Symbol resolution** - Deterministic, order-independent lookup
5. **Type checking** - Primitives, Lists, Sets, Maps, and custom Enums

### Execution Model

```
User input → Parse commands → Resolve symbols → 
Check types → Validate protocols → Execute effects → 
Update state → Emit response
```

Any failure at any step produces a structured ERROR and halts.

### Error Handling

All errors follow a deterministic format:

```
ERROR: ai-contract-language
code: PROTOCOL_VIOLATION
message: "Tool 'code_exec' forbidden in current phase"
details:
  tool: "code_exec"
  phase: "design"
  protocol: "no_implementation"
```

---

## Roadmap

The v0.1 spec freezes the core domain shape: Modules, Contracts, Rules, Actions, StateUpdates (UpdateKeys), Scope, Policy, and deterministic ALLOW/REFUSE/ERROR outcomes.

Additional block types and higher-level authoring conveniences may be introduced in later versions, but are not part of v0.1.


---

## Contributing
---

## Contributing

This is an experimental specification with a stable core philosophy but evolving implementation details.

**Especially valuable feedback:**

- Do the core philosophy and self-enforced architecture resonate? Make sense?
- Where do you see contradictions in the implementation details?
- What syntax or patterns feel natural vs. awkward?
- What's missing from the type system or protocol mechanics?
- Are there fundamental flaws in the execution model?
- Real-world use cases that would break the current design?
- **Engineering workflow patterns** - What other phase gates or protocols would be useful?

The goal is a language that's both **human-authored** and **machine-enforced**, bridging the gap between natural language and formal verification. Your perspectives on what works and what doesn't are invaluable.

---

## License

[Your license here]

## Author

[Your details here]
