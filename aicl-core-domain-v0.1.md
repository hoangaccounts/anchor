# AICL Core Domain v0.1 — Shape (Generator Scaffold)

This document freezes the **v0.1 shape** (domain objects, invariants, lifecycle, and deterministic semantics).
Modules MAY extend actions/commands/profiles/rules without changing this core.

---

## 1. Domain Objects (Canonical Schemas)

### 1.1 Contract
**Fields**
- `contract_id: string` (required; unique within a loaded file)
- `version: string` (required)
- `rules: Rule[]` (required; MAY be empty)
- `commands: Command[]` (optional; default empty)
- `metadata: map<string,string|bool|string[]>` (optional)

**Invariants**
- `contract_id` MUST be non-empty.
- Duplicate `contract_id` among loaded contracts MUST produce `ERROR`.

**Autoload**
- `metadata.autoload: boolean` (optional)
  - If `true`, the contract MUST be added to `active_contract_ids` at startup.

---

### 1.2 Rule
**Fields**
- `rule_id: string` (required; unique within its contract)
- `effect: "ALLOW" | "DENY" | "REQUIRE"` (required)
- `action_id: string` (required; references Action)
- `target: string | null` (optional; `null` treated as literal `"<null>"` for conflict keys)
- `scope_required: boolean` (optional; default `false`)
- `profile_id: string | null` (optional; default `null`)
- `note: string | null` (optional)

**Invariants**
- `effect` MUST be one of the closed set.
- If `action_id` does not resolve to a defined Action, the system MUST `ERROR` (fail closed).

---

### 1.3 Action
**Fields**
- `action_id: string` (required; unique across loaded modules/contracts)
- `kind: "read_only" | "change_existing" | "create_new" | "contract_state"` (required)
- `description: string` (optional)

**Invariants**
- Unknown `action_id` referenced by any Rule MUST `ERROR` (fail closed).

**Built-in minimum change-existing actions (v0.1)**
- `EDIT_EXISTING_ARTIFACT`
- `DELETE_CONTENT`
- `REORDER_CONTENT`
- `REGENERATE_ARTIFACT`

---

### 1.4 Command
**Fields**
- `command_id: string` (required; unique within its contract/module)
- `directive_name: string` (required; the `/name(...)` token)
- `args_schema: map<string,string>` (required; minimal typing such as `"string"|"bool"|"int"`)
- `effects: map<string,string|bool|string[]|object>` (required; declarative state mutations)
- `note: string | null` (optional)

**Invariants**
- A directive invoked in chat MUST match a known `directive_name` exactly, or `ERROR`.
- If more than one active contract defines the same `directive_name` and the directive is not namespaced, the system MUST `ERROR` with `AMBIGUOUS_COMMAND`.

**Core effect style (no directive-chaining)**
- Feature commands MUST NOT execute core directives by text expansion.
- Feature commands MAY declare effects that mutate core runtime state fields (typed effects), e.g.:
  - add/remove active profiles
  - set output format flags
  - activate/terminate contracts via state mutation
  - propose scope payloads

---

### 1.5 Directive
**Fields**
- `raw_text: string` (required)
- `directive_name: string` (required)
- `args: map<string, any>` (required; parsed)
- `is_valid: boolean` (required)
- `error_code: string | null` (optional)

**Directive recognition (strict)**
A message is a Directive only if ALL are true:
1) The **first character** is `/` (no leading whitespace/newlines).
2) It matches the exact form `/name(...)` OR `/ns.name(...)`.
3) `name` starts with a letter and contains only letters/digits/`_`/`-`.
4) If present, `ns` starts with a letter and contains only letters/digits/`_`/`-`.
5) Parentheses `(` and `)` are present as a matching pair.

If not satisfied, the message MUST NOT be treated as a directive and MUST NOT change state.

**Near-miss handling**
- If input resembles a directive but fails recognition/validation, it MUST be treated as a **near-miss**:
  - MUST NOT execute
  - MUST NOT change state
  - Turn outcome MUST be `REFUSE`
  - Assistant MAY provide gentle corrective feedback.

---
## 1.8 Directive Namespacing and Resolution (v0.1)

### Namespaced directive form
- A directive MAY use an optional namespace prefix: `/ns.name(...)`.
- `ns` SHOULD match a `contract_id`.

### Resolution rules (deterministic)
If a directive is namespaced:
- The system MUST resolve the command only within the contract identified by `ns`.
- If `ns` does not identify a loaded contract, the system MUST `ERROR` with `UNKNOWN_CONTRACT`.
- If the contract exists but does not define `name`, the system MUST `ERROR` with `UNKNOWN_COMMAND`.

If a directive is not namespaced:
- If exactly one **active** contract defines `name`, the system MUST resolve to that command.
- If zero active contracts define `name`, the system MUST `ERROR` with `UNKNOWN_COMMAND`.
- If more than one active contract defines `name`, the system MUST `ERROR` with `AMBIGUOUS_COMMAND`.

---

### 1.6 Scope
**Fields**
- `scope_id: string` (required)
- `target: string` (required)
- `operation: string` (required)
- `bounds: map<string, any>` (required)
- `immutability: map<string, any>` (required)
- `status: "proposed" | "approved" | "rejected" | "cleared"` (required)

**Minimum bounds support (v0.1)**
- Append-only:
  - `operation=append_section`
  - `bounds`: `{ "insert_after_heading": "<heading>" }` OR `{ "append_to_end": true }`
  - `immutability`: `{ "no_edits_outside_insertion": true }`
- Section-level:
  - `operation=edit_section`
  - `bounds`: `{ "section_id": "<id>" }` OR `{ "heading": "<heading>" }`
  - `immutability`: `{ "no_changes_outside_section": true }`
- Range-level (optional; allowed):
  - `operation=edit_range`
  - `bounds`: `{ "line_start": N, "line_end": M }`
  - `immutability`: `{ "no_changes_outside_range": true }`

**Scope directives (core)**
- `/proposeScope(<scope_payload>)` → registers scope as `proposed`
- `/approveScope(scope_id="<id>")` → sets scope `approved` and active
- `/rejectScope(scope_id="<id>")` → sets scope `rejected`
- `/clearScope()` → clears active scope

---

### 1.7 Policy
**Fields**
- `active_contract_ids: string[]` (required)
- `active_scope_id: string | null` (required)
- `active_profiles: string[]` (required)
- `effective_rules: Rule[]` (required; derived)
- `conflicts: map<string, any>[]` (optional; details if any)

**Invariants**
- If conflicts exist among active contracts’ rules for the same conflict key, the system MUST `ERROR`.

---

## 2. Directive Categories (Core)

Each directive MUST belong to exactly one category:

1) **Contract lifecycle**: changes `active_contract_ids`
2) **Scope control**: changes `active_scope_id` and scope statuses
3) **Mode/Profile control**: changes `active_profiles` and optional profile parameters

**Invariant**
- Directives MUST only change AICL runtime state (contracts, profiles, scope). They MUST NOT imply external execution.

---

## 3. Autoload (Startup Defaults)

### 3.1 Contract autoload
- If `Contract.metadata.autoload=true`, contract_id MUST be added to `active_contract_ids` at startup.

### 3.2 Profile autoload
A Contract MAY declare:
- `metadata.autoload_profiles: string[]`

Startup behavior:
1) Apply contract autoload to compute initial `active_contract_ids`.
2) For each **active** contract, union its `autoload_profiles` into `active_profiles`.
3) If any autoload profile is not defined by any loaded contract/module, the system MUST `ERROR` (fail closed).

---

## 4. Conflict Detection (Rule Conflicts)

### 4.1 Conflict key
Two rules are about the same thing if they share:
- `action_id`
- `target` (with `null` treated as literal `"<null>"`)

Conflict key:
- `K = (action_id, target_or_null)`

### 4.2 Conflict conditions
For each key `K`, collect all matching rules from all **active** contracts after profile filtering.

A conflict exists if either:
- **Type A (effect contradiction):** more than one distinct `effect` exists for `K`
- **Type B (scope_required mismatch):** same `effect` but differing `scope_required` exists for `K`

### 4.3 Conflict outcome
If any conflict exists:
- Turn outcome MUST be `ERROR`
- The turn MUST halt (atomic abort)
- Error payload MUST include:
  - `conflict_key` (action_id + target_or_null)
  - `conflict_of` (contract_ids)
  - `rules` (rule_ids + effects + scope_required)

---

## 5. Transactionality (Atomic Turns)

Each user turn MUST be atomic:

- If outcome is `ERROR`:
  - MUST perform no artifact changes
  - MUST perform no contract/profile/scope state changes
- If outcome is `REFUSE`:
  - MUST perform no artifact changes
  - Directive state changes MAY occur only if the user input was a valid directive
- If outcome is `ALLOW`:
  - MUST apply all permitted actions for the turn as a single unit
  - MUST NOT partially apply changes

Deterministic order of operations:
1) Parse directives (if any)
2) Validate directive(s) (invalid/unknown identities → `ERROR`)
3) Compute effective Policy + detect conflicts (conflict → `ERROR`)
4) Classify requested actions
5) Enforce permissions/requirements/scope (violation → `REFUSE`)
6) Execute permitted actions (all-or-nothing)

---

## 6. Outcomes (ALLOW / REFUSE / ERROR)

### 6.1 Outcome enum
Each turn MUST classify as exactly one of:
- `ALLOW`
- `REFUSE`
- `ERROR`

### 6.2 ERROR
`ERROR` means invalid state or broken invariant, including:
- Unknown contract/profile/action/command identity
- Missing/invalid required fields in loaded artifacts
- Rule conflicts (type A or B)
- Invalid directive structure that is not a near-miss classification case

**Semantics**
- MUST halt and be atomic abort (no changes).

### 6.3 REFUSE
`REFUSE` means valid system state, but request is not permitted or required conditions are unmet, including:
- Requested action not explicitly permitted (closed permission model)
- DENY rules
- REQUIRE rules with unmet conditions
- Missing approved scope for `change_existing` actions
- Near-miss directives

**Semantics**
- MUST not perform forbidden actions or artifact mutations.
- MAY propose scope or request required details.

### 6.4 ALLOW
`ALLOW` means all checks pass and requested action(s) are permitted.

**Semantics**
- MUST execute permitted actions atomically (all-or-nothing).

---

## 7. Policy Derivation Algorithm (Minimal)

Given runtime state:
- `active_contract_ids`
- `active_profiles`
- `active_scope_id`

Compute Policy:

1) Collect all Rules from **active** contracts.
2) Filter profile-gated rules:
   - If `rule.profile_id != null` and `profile_id` is not in `active_profiles`, exclude the rule.
3) Group remaining rules by conflict key `(action_id, target_or_null)`.
4) Detect conflicts:
   - Type A: multiple distinct effects within a group
   - Type B: same effect but differing `scope_required`
5) If any conflict exists → `ERROR` (atomic abort).
6) Else, set `effective_rules` as one representative rule per group key (duplicates allowed; collapse permitted).

---

## 8. Scope Enforcement (No Change Without Approved Scope)

- Before performing any action with `Action.kind="change_existing"`, the system MUST verify:
  - an active scope exists with `status="approved"`.
- If no approved active scope exists:
  - Turn outcome MUST be `REFUSE`
  - Assistant MAY propose a scope (machine-checkable) for explicit user approval
  - Assistant MUST NOT perform the change in the same turn unless scope is already approved.
