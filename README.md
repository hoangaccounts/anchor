# AI Contract Language (AICL)

**A deterministic contract language for AI systems**

Version: 0.1  
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

Commands are the explicit execution mechanism: only properly recognized command/update invocations can change state or trigger declared effects.

There is no guessing, no best-effort behavior, and no silent drift.


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

## Core Domain (v0.1)

AICL v0.1 defines a concrete, end-user-authorable contract language with the following primitives:

- **Modules** — Namespaced containers for contracts and identifiers.
- **Contracts** — Activatable policy bundles defining rules and state updates.
- **Rules** — Closed-vocabulary permissions (`ALLOW`, `DENY`, `REQUIRE`) over actions and targets.
- **Actions** — Explicitly declared operations (read-only or mutating).
- **StateUpdates (UpdateKeys)** — Deterministic, side-effect-free state mutations invoked via strict syntax.
- **Commands** — Deterministic invocations that may emit output and/or read/write declared state, but only through explicitly declared effects.
- **Policy** — A derived, conflict-checked view of all active rules.
- **Scope** — Machine-checkable bounds required for mutating actions.
- **Outcomes** — Every turn deterministically resolves to exactly one of `ALLOW`, `REFUSE`, or `ERROR`.

The full normative specification lives in `aicl-spec-v0.1.md`.


## Core Principles

### 1. **Commands Are the Only Execution Mechanism**

Free-form prose is allowed but has **zero** behavioral effect. Only properly formatted commands (`/command_name(args)`) cause the AI to take action. This eliminates the confusion of "was that an instruction or just discussion?"

### 2. **Contracts Over Conversations**

Define your expectations as machine-checkable **protocols** instead of hoping the AI understands context:

```yaml
[[PROTOCOL]]
protocols:
  - id: "solid_principles"
    rules:
      - id: "require_dependency_injection"
        check:
          type: "require_pattern"
          pattern: "dependency_injection"
```

### 3. **Self-Enforced by Design**

There is no external runtime or compiler. The AI assistant itself parses, validates, and enforces these contracts. The AI **treats these rules as binding constraints** - deterministically enforcing them on its own behavior.

### 4. **Type-Safe State Management**

Track and mutate state explicitly with a proper type system:

```yaml
[[STATE]]
name: currentPhase
type: WorkMode
default: research
[[/STATE]]
```

### 5. **Fail-Closed on Ambiguity**

Any ambiguity, unknown reference, or rule violation results in a deterministic ERROR. The system never "does its best" - it halts and reports exactly what went wrong.

---

## Key Concepts

### WorkMode: Engineering Workflow Phases

`WorkMode` describes **what phase of development** you're in, enabling phase-gate enforcement:

- `research` - Gather requirements, constraints, existing solutions
- `ideate` - Brainstorm approaches, explore alternatives
- `design` - Choose architecture, define interfaces and boundaries
- `plan` - Break into tasks, define implementation sequence
- `preview` - Review design, simulate outcomes before committing
- `implement` - Execute the plan, write code
- `test` - Verify correctness, validate assumptions

Each phase can have different rules - for example, code execution might be forbidden until `implement` phase.

### Modules: Scoped Symbol Spaces

All blocks (except `[[LANGUAGE_SPEC]]`) must live inside `[[MODULE]]` blocks. Each module has its own namespace for commands, state, enums, and protocols. Reference external symbols using qualified names: `ModuleName.SymbolName`.

### Protocols: Machine-Checkable Rules

Protocols define **deterministic, fail-closed contracts**. Every rule must be machine-checkable - no prose-based "guidelines":

```yaml
rules:
  - id: "follow_clean_architecture"
    check:
      type: "require_pattern"
      pattern: "clean_architecture_layers"
```

### Enabled Rules Policy

Commands **replace** (not append to) the set of enabled rules. Only explicitly enabled rules affect behavior.

---

## Quick Start: Engineering Workflow

### 1. Define Your Module

```yaml
[[MODULE]]
name: backend_development
description: "Clean architecture backend development workflow"
```

### 2. Declare Development State

```yaml
[[STATE]]
name: currentPhase
type: WorkMode
default: research
[[/STATE]]

[[STATE]]
name: architectureApproved
type: Bool
default: false
[[/STATE]]
```

### 3. Create Phase-Gate Commands

```yaml
[[COMMAND]]
name: start_design
description: "Move to design phase after research"
effects:
  set_state:
    - name: currentPhase
      value: "design"
  enable:
    protocols: ["no_implementation"]
[[/COMMAND]]

[[COMMAND]]
name: approve_architecture
description: "Approve design and enable implementation"
effects:
  set_state:
    - name: currentPhase
      value: "implement"
    - name: architectureApproved
      value: true
  enable:
    protocols: ["solid_principles", "clean_architecture"]
[[/COMMAND]]
```

### 4. Define Phase Protocols

```yaml
[[PROTOCOL]]
protocols:
  - id: "no_implementation"
    description: "Block code execution during research/design"
    rules:
      - id: "forbid_code"
        check:
          type: "forbid_tool"
          tool: "code_exec"
      
  - id: "solid_principles"
    description: "Enforce SOLID in implementation"
    rules:
      - id: "require_di"
        check:
          type: "require_pattern"
          pattern: "dependency_injection"
[[/PROTOCOL]]
```

### 5. Execute Workflow

```
User: "Let's build an auth system"
AI: "I'll start with research phase. What are the requirements?"

User: "OAuth2, JWT tokens, role-based access"
AI: *researches patterns, presents findings*

User: /start_design()
AI: "Moved to design phase. Here's a proposed architecture..."

User: /approve_architecture()
AI: "Architecture approved. Ready to implement following clean architecture..."
```

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

## Extending the Language

### Custom Enums

Define your own types:

```yaml
[[ENUM]]
name: ArchitecturePattern
values: [clean_architecture, hexagonal, onion, layered]
[[/ENUM]]
```

### Custom Workflows

Chain multiple steps together:

```yaml
[[WORKFLOW]]
workflows:
  - id: "safe_implementation_start"
    steps:
      - kind: enforce_protocol
        ref: "architecture_approved"
      - kind: enforce_protocol
        ref: "solid_principles"
      - kind: emit_response
        ref: "implementation_ready"
[[/WORKFLOW]]
```

### Response Templates

Define structured output formats:

```yaml
[[RESPONSE]]
responses:
  - id: "design_proposal"
    description: "Architecture design document"
    constraints:
      structure:
        sections: ["CONTEXT", "ARCHITECTURE", "PATTERNS", "TRADEOFFS"]
      must_include:
        - "Clean Architecture layers"
        - "Dependency flow"
        - "SOLID compliance"
[[/RESPONSE]]
```

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
