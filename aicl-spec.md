# AICL Core Domain v0.1 — Shape (Generator Scaffold)

This document freezes the **v0.1 shape** (domain objects, invariants, lifecycle, and deterministic semantics).
Modules MAY extend actions/commands/profiles/rules without changing this core.

---

## 0. Encoding and File Structure (v0.1)

### 0.1 Fixed block markers
- A serialized AICL artifact MUST use only the following block markers:
  - `[[MODULE]]` ... `[[/MODULE]]`
  - `[[CONTRACT]]` ... `[[/CONTRACT]]`
- A file MAY contain one or more `[[MODULE]]` blocks.
- Text outside `[[MODULE]]` blocks MUST be ignored by the runtime and MUST be ignored by validators unless a tool explicitly declares otherwise.

### 0.2 Module envelope rules
- `[[MODULE]]` blocks MUST contain:
  - Module metadata lines (`module_name`, `module_version`, `module_namespace`, optional `description`), AND
  - one or more `[[CONTRACT]]` blocks.
- `[[MODULE]]` blocks MUST NOT contain any other top-level content besides the metadata lines and `[[CONTRACT]]` blocks.
- Within a single file, `module_namespace` values MUST be unique; duplicates MUST produce `ERROR`.

### 0.3 Contract encoding rules (YAML only)
- Each `[[CONTRACT]]` block body MUST be exactly one YAML document that encodes a single Contract per the canonical schema.
- The YAML encoding MUST use a restricted YAML 1.2 subset:
  - mappings, sequences, and scalars (string, boolean, integer) only
  - MUST NOT use anchors, aliases, merge keys, tags, or multi-document markers (`---`)
- A validator MUST treat any violation of these encoding rules as `FAIL`.

---

## 1. Domain Objects (Canonical Schemas)

### 1.1 Module
**Fields**
- `module_name: string` (required)
- `module_version: string` (required)
- `module_namespace: string` (required; unique within a loaded file)
- `description: string` (optional)
- `contracts: Contract[]` (required; one or more)

**Invariants**
- `module_name` MUST be non-empty.
- `module_namespace` MUST be non-empty.
- Duplicate `module_namespace` among loaded modules MUST produce `ERROR**Module namespace and canonicalization (v0.1)**

- A `[[MODULE]]` MUST be a namespace container; there MUST be no global identifiers for items defined inside a module.
- Within a loaded `[[MODULE]]` block with `module_namespace = ns`:
  - The system MUST treat as identifier fields:
    - any field name that ends with `_id`, and
    - `StateUpdate.update_key` (as stored in the contract file).
  - The system MUST canonicalize identifier-field values defined inside the module by prefixing with `ns`.
  - The system MUST canonicalize identifier-field values to `{ns}.{value}`.
  - Identifier-field values inside a module MUST NOT contain `.`.
  - The system MUST canonicalize `StateUpdate.update_key` to `/{ns}.{value}`.
- Field-level exemptions:
  - The system MUST NOT canonicalize any `target` field value.
  - The system MUST NOT canonicalize any `effects` field key or value.

**UpdateKey invocation resolution (deterministic)**

- A message that matches UpdateKey recognition MUST be resolved against the set of active `StateUpdate.update_key` values.
- If an invocation uses `/ns.value(...)`, the system MUST resolve only within the loaded module with `module_namespace = ns`.
  - If `ns` does not identify a loaded module namespace, the system MUST `ERROR` with `UNKNOWN_MODULE`.
- If an invocation uses `/value(...)` (no namespace), the system MUST resolve across all loaded modules.
- If exactly one active `StateUpdate.update_key` matches the invocation, the system MUST resolve to that StateUpdate.
- If zero active `StateUpdate.update_key` values match, the system MUST `ERROR` with `UNKNOWN_UPDATE_KEY`.
- If more than one active `StateUpdate.update_key` matches, the system MUST `ERROR` with `AMBIGUOUS_UPDATE_KEY`.


---

### 1.2 Contract
**Fields**
- `contract_id: string` (required; unique within a loaded file)
- `version: string` (required)
- `rules: Rule[]` (required; MAY be empty)
- `commands: StateUpdate[]` (optional; default empty)
- `metadata: map<string,string|bool|string[]>` (optional)

**Invariants**
- `contract_id` MUST be non-empty.
- Duplicate `contract_id` among loaded contracts MUST produce `ERROR`.

**Autoload**
- `metadata.autoload: boolean` (optional)
  - If `true`, the contract MUST be added to `active_contract_ids` at startup.

---

### 1.3 Rule
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

### 1.4 Action
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

### 1.5 StateUpdate
**Fields**
- `update_id: string` (required; unique within its contract/module)
- `update_key: string` (required; the `/name(...)` token)
- `args_schema: map<string,string>` (required; minimal typing such as `"string"|"bool"|"int"`)
- `effects: map<string,string|bool|string[]|object>` (required; declarative state mutations)
- `note: string | null` (optional)

**Invariants**
- An update_key invoked in chat MUST match a known `update_key` exactly, or `ERROR`.
- If more than one active contract defines the same `update_key` and the update_key is not namespaced, the system MUST `ERROR` with `AMBIGUOUS_UPDATE_KEY`.

**Core effect style (no update_key-chaining)**
- Feature commands MUST NOT execute core update_keys by text expansion.
- Feature commands MAY declare effects that mutate core runtime state fields (typed effects), e.g.:
  - add/remove active profiles
  - set output format flags
  - activate/terminate contracts via state mutation
  - propose scope payloads

---

### 1.6 UpdateKey
**Fields**
- `raw_text: string` (required)
- `update_key: string` (required)
- `args: map<string, any>` (required; parsed)
- `is_valid: boolean` (required)
- `error_code: string | null` (optional)

**UpdateKey recognition (strict)**
A message is a UpdateKey only if ALL are true:
1) The **first character** is `/` (no leading whitespace/newlines).
2) It matches the exact form `/name(...)` OR `/ns.name(...)`.
3) `name` starts with a letter and contains only letters/digits/`_`/`-`.
4) If present, `ns` starts with a letter and contains only letters/digits/`_`/`-`.
5) Parentheses `(` and `)` are present as a matching pair.

If not satisfied, the message MUST NOT be treated as an update_key and MUST NOT change state.

**Near-miss handling**
- If input resembles an update_key but fails recognition/validation, it MUST be treated as a **near-miss**:
  - MUST NOT execute
  - MUST NOT change state
  - Turn outcome MUST be `REFUSE`
  - Assistant MAY provide gentle corrective feedback.

---
### 1.7 Scope
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

**Scope update_keys (core)**
- `/proposeScope(<scope_payload>)` → registers scope as `proposed`
- `/approveScope(scope_id="<id>")` → sets scope `approved` and active
- `/rejectScope(scope_id="<id>")` → sets scope `rejected`
- `/clearScope()` → clears active scope

---

### 1.8 Policy
**Fields**
- `active_contract_ids: string[]` (required)
- `active_scope_id: string | null` (required)
- `active_profiles: string[]` (required)
- `effective_rules: Rule[]` (required; derived)
- `conflicts: map<string, any>[]` (optional; details if any)

**Invariants**
- If conflicts exist among active contracts’ rules for the same conflict key, the system MUST `ERROR`.

---

## 2. UpdateKey Categories (Core)

Each update_key MUST belong to exactly one category:

1) **Contract lifecycle**: changes `active_contract_ids`
2) **Scope control**: changes `active_scope_id` and scope statuses
3) **Mode/Profile control**: changes `active_profiles` and optional profile parameters

**Invariant**
- UpdateKeys MUST only change AICL runtime state (contracts, profiles, scope). They MUST NOT imply external execution.

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
  - UpdateKey state changes MAY occur only if the user input was a valid update_key
- If outcome is `ALLOW`:
  - MUST apply all permitted actions for the turn as a single unit
  - MUST NOT partially apply changes

Deterministic order of operations:
1) Parse update_keys (if any)
2) Validate update_key(s) (invalid/unknown identities → `ERROR`)
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
- Unknown contract/profile/action/state update identity
- Missing/invalid required fields in loaded artifacts
- Rule conflicts (type A or B)
- Invalid update_key structure that is not a near-miss classification case

**Semantics**
- MUST halt and be atomic abort (no changes).

### 6.3 REFUSE
`REFUSE` means valid system state, but request is not permitted or required conditions are unmet, including:
- Requested action not explicitly permitted (closed permission model)
- DENY rules
- REQUIRE rules with unmet conditions
- Missing approved scope for `change_existing` actions
- Near-miss update_keys

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
