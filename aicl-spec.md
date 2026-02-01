# ai-context-language-spec v0.4.20 (proposed)
This document defines **ai-context-language**, a deterministic context language
for AI systems. The language is designed to eliminate intent guessing,
implicit behavior, and non-deterministic interpretation.

Free-form prose is allowed outside blocks.
Only BLOCKS and COMMAND calls have meaning.

---

[[LANGUAGE_SPEC]]
version: 0.4.20
language: "ai-context-language"
status: "stable"

principles:
  - No implicit behavior
  - No intent inference
  - Commands are the only execution mechanism
  - All ambiguity results in errors

execution_model:
  prose_is_non_executable: true
  command_only_execution: true

terminology:
  enum_metatype:
    definition: >
      Enum is a metatype (a "kind of type").
      Each [[ENUM]] declaration defines a NEW concrete enum type (e.g., WorkMode, DayOfWeek).
      Values are instances of that enum type.
    invariants:
      - "Enum types are referenced by their declared name (e.g., WorkMode)."
      - "Enum values are referenced by their literal value tokens as declared in ENUM.values."

  workmode_intent_model:
    definition: >
      WorkMode describes the user's intent: what kind of work the user is doing.
      WorkMode does not grant permissions or directly enforce behavior.
      Behavioral constraints MUST be expressed via protocols/rules, not via WorkMode.
    canonical_values_example:
      - research
      - ideate
      - design
      - plan
      - preview
      - implement
      - test
    meanings:
      research: "Gather information, learn unknowns, collect constraints."
      ideate: "Generate options and alternative directions."
      design: "Choose an approach and define its shape/specification."
      plan: "Define implementation steps and execution order."
      preview: "Simulate/inspect outcomes before committing; no final artifacts."
      implement: "Execute changes to realize the design."
      test: "Verify correctness and validate assumptions."

  enabled_rules:
    definition: >
      Enabled Rules are the only rules currently ON.
      Only enabled rules affect behavior.
      All other defined rules are ignored.

enabled_rules_policy:
  mode: replace
  explanation: >
    Each command call REPLACES the enabled rules set.
    No rule remains enabled unless the command explicitly enables it.

# -------------------------
# Parsing + framing
# -------------------------

parsing:
  block_markers:
    open: "[[<BLOCK_TYPE>]]"
    close: "[[/<BLOCK_TYPE>]]"
    marker_rules:
      - "Markers MUST appear alone on a line (ignoring surrounding whitespace)."
      - "BLOCK_TYPE names are case-sensitive."
      - "Blocks MUST be properly nested and closed; otherwise raise PARSE_ERROR."

  nesting_rules:
    - "[[LANGUAGE_SPEC]] is the root block and MUST NOT be nested inside any other block."
    - "All blocks except [[LANGUAGE_SPEC]] MUST be contained within a [[MODULE]] block."
    - "[[MODULE]] blocks MUST NOT be nested inside other [[MODULE]] blocks."

  yaml_mode:
    applies_to_blocks: [LANGUAGE_SPEC, MODULE, ENUM, STATE, COMMAND, RESPONSE, WORKFLOW, PROTOCOL]
    yaml_version: "1.2"
    rules:
      - "YAML content MUST parse as exactly one YAML document; otherwise raise PARSE_ERROR."
      - "YAML key order is irrelevant."
      - "Duplicate YAML keys are forbidden; otherwise raise SCHEMA_VIOLATION."
      - "Unknown YAML keys are forbidden for schema-defined blocks; otherwise raise SCHEMA_VIOLATION."

  text_mode:
    applies_to_blocks: [COMMANDS, ENGINEERING, CHECKLISTS, GUIDELINES]
    rules:
      - "Content is treated as raw text (no YAML parsing)."
      - "Only block framing + scoping + nesting rules apply."
      - "Text-mode blocks MUST NOT be used for deterministic behavior until schemas are defined."

text_mode_policy:
  strict: false
  rule: >
    Text-mode blocks are allowed for human guidance but MUST NOT be relied on for deterministic parsing
    beyond framing/scoping.

block_types:
    - ENGINEERING
    - CHECKLISTS
    - GUIDELINES
  allowed_floating_block_types:
    - LANGUAGE_SPEC
  on_violation:
    code: SCHEMA_VIOLATION
    message: "Block must be contained within a MODULE."

# -------------------------
# Symbol + precedence
# -------------------------

precedence_order:
  - RESPONSE
  - WORKFLOW
  - PROTOCOL
  - ENGINEERING
  - CHECKLISTS
  - GUIDELINES
  - COMMAND
  - STATE
  - ENUM

conflict_rules:
  same_precedence: error
  cross_precedence: higher_precedence_wins

validation_policy:
  duplicate_id: error
  unknown_id: error
  invalid_command: error
  conflicting_enabled_rules: error
  invalid_yaml: error
  schema_violation: error

module_and_symbol_scoping:
  module_name_uniqueness:
    rule: "[[MODULE]].name MUST be globally unique within the combined document."
    on_violation: SCHEMA_VIOLATION

  symbol_uniqueness_within_module:
    rules:
      - "Within a module, STATE.name MUST be unique."
      - "Within a module, ENUM.name MUST be unique."
      - "Within a module, COMMAND.name MUST be unique."
      - "Within a module, RESPONSE.responses[*].id MUST be unique."
      - "Within a module, WORKFLOW.workflows[*].id MUST be unique."
      - "Within a module, protocol IDs MUST be unique across all [[PROTOCOL]] blocks (i.e., across every protocols[*].id entry)."
    on_violation: SCHEMA_VIOLATION

  reference_qualification:
    default: qualified
    allow_local_unqualified_inside_same_module: true
    rules:
      - "References to external-module symbols MUST be fully qualified as <ModuleName>.<SymbolName>."
      - "References to same-module symbols MAY be unqualified when allow_local_unqualified_inside_same_module is true."
    on_unqualified_external_reference: REFERENCE_ERROR

# -------------------------
# Types + state
# -------------------------

types:
  primitives: [Bool, Int, String]
  type_names_case_sensitive: true
  list_syntax: "List<Type>"
  set_syntax: "Set<Type>"
  map_syntax: "Map<String, Type>"
  enum_reference: "<EnumName> # declared via [[ENUM]]"

defaults:
  implicit:
    Bool: false
    Int: 0
    String: ""
    List<T>: []
    Set<T>: []
    Map<String, T>: {}
  enum_requires_default: true

typing_rules:
  list_is_homogeneous: true
  set_is_homogeneous: true
  set_requires_unique_elements: true
  map_requires_string_keys: true
  map_values_must_typecheck: true
  list_enum_elements_must_be_valid: true
  set_enum_elements_must_be_valid: true

state_declarations:
  meaning: >
    Each [[STATE]] block declares exactly ONE runtime variable.
    Runtime state persists across command calls within the same chat session.
    Commands may mutate runtime state via effects.set_state.
  structure:
    one_variable_per_block: true
    required_keys: [name, type]
    optional_keys: [default]
    forbid_extra_keys: true
  merge_policy:
    duplicate_variable_name: error

# -------------------------
# Command calls
# -------------------------

command_syntax:
  prefix: "/"
  must_start_at_column_0: true
  require_parentheses: true
  allow_empty_parentheses: true
  allow_multiple_args: true
  allow_mixing_positional_and_named: false
  call_forms:
    unqualified_pattern: "/<command_name>(<args...>)"
    qualified_pattern: "/<module_name>.<command_name>(<args...>)"


ux_layer:
  note: >
    UX-layer helpers may warn and suggest corrections but MUST NOT execute anything.
    Execution remains command-only.

  command_like_text_warning:
    hard_rule: >
      If user input resembles a command call but does not match command_syntax (e.g., missing leading '/'),
      the interpreter MUST NOT execute or mutate state. It MAY emit a warning and a suggested corrected command.
      The suggested command MUST be copy/paste exact. Execution occurs ONLY if the user later submits the exact
      corrected command that conforms to command_syntax.
    warning_format:
      prefix: "WARNING: ai-context-language"
      message_template: "Not a command. Did you mean: <suggested_command> ? To run it, type it exactly as shown."
    examples:
      - input: "mode(design)"
        warning_suggest: "/mode(design)"

symbol_resolution:
  qualified_syntax:
    delimiter: "."
    pattern: "<module_name>.<symbol_name>"
  algorithm:
    description: >
      Resolution is deterministic and does NOT depend on module order, file order,
      or any “current module” context.
      Unqualified references are allowed only when globally unique for that symbol type.

  errors:
    UNKNOWN_ID:
      description: "No matching symbol for the reference (or invalid qualification)."
    AMBIGUOUS_ID:
      description: "Multiple symbols share the same name; qualification required."
      include_candidates: true

# -------------------------
# Effects + deterministic expressions
# -------------------------

effects_model:
  note: >
    Commands are the only execution mechanism.
    A command's effects fully define what becomes enabled after that command call.
    Enabled rules are REPLACED (not merged) on each command call.

  enable_effect:
    keys:
      responses: "List<String> # RESPONSE.responses[*].id"
      workflows: "List<String> # WORKFLOW.workflows[*].id"
      protocols: "List<String> # IDs from [[PROTOCOL]].protocols[*].id"
      guidelines: "List<String> # GUIDELINES ids (text-mode; non-deterministic)"
      checklists: "List<String> # CHECKLIST ids (text-mode; non-deterministic)"
      engineering: "List<String> # ENGINEERING ids (text-mode; non-deterministic)"

  set_state_effect:
    shape: "List<{name: String, value: ValueExpr}>"
    keys:
      name: "String # STATE.name"
      value: "ValueExpr"

value_expr:
  description: >
    Deterministic, side-effect-free value expressions used by effects.set_state.
    No functions, no conditionals, no loops, no external reads.
  forms:
    - literal: "String | Int | Bool"
    - arg_ref: "$arg0, $arg1, ... (positional args)"
    - named_arg_ref: "$<argName> (named args)"
    - state_ref: "$state.<stateName>"
    - string_template:
        description: >
          A string literal MAY include zero or more placeholders of the form ${<ref>}.
          <ref> MUST be one of: arg_ref | named_arg_ref | state_ref.
          Placeholders are substituted left-to-right with their string value.
          No nested placeholders; no escape sequences besides standard YAML string escaping.
        examples:
          - "mode=${$arg0}"
          - "mode=${$state.userWorkMode}"
          - "hello ${$value}"

# -------------------------
# Deterministic errors
# -------------------------

error_handling:
  behavior: fail_closed
  response_id: "ai_context_error"

error_model:
  definition: >
    ERROR = Exception + mandatory standardized printing + no catch.
    An ERROR stops execution immediately and emits a fixed error payload.
    Recovery, fallback behavior, and intent inference are not permitted.

error_payload:
  header_line: "ERROR: ai-context-language"
  body_format: yaml
  required_keys:
    - code
    - message
    - details
  details_shape:
    location_keys:
      - block_type
      - id_optional
      - line_optional
    enabled_rules_keys:
      - responses_optional
      - workflows_optional
  allow_hint: true
  allow_additional_prose: false

error_codes:
  - AMBIGUOUS_ID
  - SCHEMA_VIOLATION
  - UNKNOWN_ID
  - INVALID_COMMAND
  - INVALID_YAML
  - PARSE_ERROR
  - CONFLICT
  - DUPLICATE_ID
  - REFERENCE_ERROR

invariants:
  - "Commands do not add rules."
  - "Commands replace enabled rules."
  - "Only enabled rules affect behavior."
  - "Everything else is ignored."

activation:
  implicit_activation: false
  activation_method: command_only

# -------------------------
# Schemas (YAML-backed blocks)
# -------------------------

block_schemas:
  MODULE:
    allowed_keys: [name, description]
    required_keys: [name, description]
    key_types:
      name: String
      description: String

  ENUM:
    required_keys: [name, values]
    optional_keys: [description]
    forbid_extra_keys: true
    values_type: string_array
    values_must_be_unique: true
    merge_policy:
      duplicate_enum_name: error

  STATE:
    required_keys: [name, type]
    optional_keys: [default]
    forbid_extra_keys: true
    merge_policy:
      duplicate_variable_name: error

  COMMAND:
    required_keys: [name]
    optional_keys:
      - description
      - args
      - reads_state
      - writes_state
      - effects
    forbid_extra_keys: true
    merge_policy:
      duplicate_command_name: error

    args_shape:
      description: >
        args is optional. If present, it declares the signature for validation.
        kind determines whether positional OR named arguments are allowed.
      required_keys: [kind]
      optional_keys: [positional, named]
      kind_values: [positional, named]
      positional_item_keys: [name, type]
      named_item_keys: [name, type]
      forbid_extra_keys: true

    effects_shape:
      optional_keys: [enable, set_state]
      enable_keys: [responses, workflows, protocols, guidelines, checklists, engineering]
      set_state_item_keys: [name, value]
      forbid_extra_keys: true

  RESPONSE:
    required_keys: [responses]
    optional_keys: []
    forbid_extra_keys: true
    responses_item_schema:
      required_keys: [id]
      optional_keys: [description, constraints]
      forbid_extra_keys: true

  WORKFLOW:
    required_keys: [workflows]
    optional_keys: []
    forbid_extra_keys: true
    workflows_item_schema:
      required_keys: [id, steps]
      optional_keys: [description]
      forbid_extra_keys: true
    workflow_step_schema:
      required_keys: [kind]
      optional_keys: [ref]
      forbid_extra_keys: true
      kind_values:
        - emit_response      # ref -> RESPONSE id
        - enforce_protocol   # ref -> PROTOCOL id
        - enforce_checklist  # ref -> CHECKLIST id (text-mode; non-deterministic)
        - enforce_guideline  # ref -> GUIDELINE id (text-mode; non-deterministic)
        - enforce_engineering # ref -> ENGINEERING id (text-mode; non-deterministic)



  PROTOCOL:
    required_keys: [protocols]
    optional_keys: []
    forbid_extra_keys: true
    description: "Deterministic-only contract: protocol rules are ALWAYS enforced (fail-closed). Rules MUST be machine-checkable checks; free-text rules are not allowed. Any violation emits ERROR and halts."
    protocols_item_schema:
      required_keys: [id, rules]
      optional_keys: [description]
      forbid_extra_keys: true
    rules_type: "List<Rule>"
    rule_schema:
      required_keys: [id, check]
      optional_keys: []
      forbid_extra_keys: true
    check_schema:
      required_keys: [type]
      optional_keys: [tool, block_type, response_id]
      forbid_extra_keys: true

    type_definitions:
      Rule:
        description: "A deterministic, machine-checkable constraint that is evaluated when a PROTOCOL is enforced."
        required_keys: [id, check]
        properties:
          id: "String # unique within a protocol definition"
          check: "Check"
      Check:
        description: "A machine-checkable predicate. The engine evaluates it against observable artifacts (context AST, emitted blocks, tool calls)."
        required_keys: [type]
        properties:
          type: "CheckType # closed set"
          tool: "String # only used when type=forbid_tool or require_tool"
          block_type: "String # only used when type=forbid_block or require_block"
          response_id: "String # only used when type=require_response_id"

[[/LANGUAGE_SPEC]]

---

## Example (minimal)

[[MODULE]]
name: exampleModule
description: "Minimal example module showing declarations and one command."

[[RESPONSE]]
responses:
  - id: "ai_context_error"
    description: "Deterministic ai-context-language error output."
    constraints:
      structure:
        sections: ["ERROR"]
      must_include:
        - "ERROR: ai-context-language"
        - "code:"
        - "message:"
        - "details:"
      formatting:
        allow_code_blocks: false
        forbid_tables: true
[[/RESPONSE]]

[[PROTOCOL]]
protocols:
  - id: "tool_policy"
    description: "Deterministic tool constraints."
    rules:
      - id: "no_web_run"
        check:
          type: "forbid_tool"
          tool: "web.run"
      - id: "no_code_exec"
        check:
          preset: "no_code_execution"
[[/PROTOCOL]]

[[WORKFLOW]]
workflows:
  - id: "chat_start"
    description: "Minimal startup workflow."
    steps:
      - kind: emit_response
        ref: "ai_context_error"
      - kind: enforce_protocol
        ref: "no_intent_inference"
[[/WORKFLOW]]

[[ENUM]]
name: WorkMode
values: [research, ideate, design, plan, preview, implement, test]
[[/ENUM]]



## CheckType (deterministic checks)

Each CheckType defines a machine-checkable predicate. The engine evaluates the check against observable artifacts (e.g., emitted blocks, selected response IDs, tool invocations).

- forbid_tool: FAIL if the specified tool is invoked.
- require_tool: FAIL unless the specified tool is invoked at least once.
- forbid_block: FAIL if the specified block_type is present in output.
- require_block: FAIL unless the specified block_type is present in output.
- require_response_id: FAIL unless the output uses the specified response_id (from RESPONSE.responses[*].id).


[[ENUM]]
name: CheckType
values:
  - forbid_tool
  - require_tool
  - forbid_block
  - require_block
  - require_response_id
[[/ENUM]]

## Presets (deterministic macros)

Presets are a **documentation-friendly authoring layer** for protocols.
A preset is a named macro that **expands deterministically** into one or more concrete checks.

- Preset expansion happens **after parsing/validation** and **before** checks are evaluated.
- The checker evaluates only the **expanded** concrete checks (never the preset name).
- Missing preset ID ⇒ UNKNOWN_ID error (fail-closed).
- Presets MUST expand to concrete check objects only (no prose, no heuristics).
- Presets MUST NOT reference other presets in v1 (no recursion).

### Rule.check schema (v1)

`Rule.check` is one of:

- **Concrete check**
  - `type: <CheckType>` plus the required fields for that check type.
- **Preset reference**
  - `preset: <PresetId>`

### Built-in preset catalog (v1)

The language defines a minimal built-in preset catalog. Engines MUST expand these IDs exactly as specified.

- `no_browsing`
  - expands to: `{ type: forbid_tool, tool: web.run }`
- `no_code_execution`
  - expands to:
    - `{ type: forbid_tool, tool: python.exec }`
    - `{ type: forbid_tool, tool: python_user_visible.exec }`
    - `{ type: forbid_tool, tool: container.exec }`
- `require_code_execution`
  - expands to: `{ type: require_tool, tool: python.exec }`
- `no_file_output`
  - expands to:
    - `{ type: forbid_block, block_type: FILE }`
    - `{ type: forbid_block, block_type: ARTIFACT }`
    - `{ type: forbid_block, block_type: DOWNLOAD }`
    - `{ type: forbid_tool, tool: python_user_visible.exec }`
    - `{ type: forbid_tool, tool: container.download }`

[[ENUM]]
name: PresetId
values:
  - no_browsing
  - no_code_execution
  - require_code_execution
  - no_file_output
[[/ENUM]]


[[ENUM]]
name: OutputStyle
values: [compact, strict, chatty]
[[/ENUM]]

[[STATE]]
name: userWorkMode
type: WorkMode
default: design
[[/STATE]]

[[STATE]]
name: responseMode
type: OutputStyle
default: compact
[[/STATE]]

[[STATE]]
name: verbose
type: Bool
default: false
[[/STATE]]

[[STATE]]
name: maxTokens
type: Int
default: 1200
[[/STATE]]

[[COMMAND]]
name: mode
description: "Set the current work mode."
args:
  kind: positional
  positional:
    - name: value
      type: WorkMode
effects:
  set_state:
    - name: userWorkMode
      value: "$arg0"
[[/COMMAND]]

[[COMMAND]]
name: summarize
description: "Summarize input using current responseMode."
reads_state: [userWorkMode, responseMode, verbose, maxTokens]
effects:
  enable:
    responses: ["ai_context_error"]
    workflows: ["chat_start"]
    protocols: ["no_intent_inference"]
[[/COMMAND]]

[[/MODULE]]


## Intent & Execution Model (Critical)

This AI Context Language is designed to be **embedded directly into an AI chat context**
and interpreted and enforced by the AI assistant itself.

There is **no external runtime, compiler, or enforcement engine**.
The context file is not an instruction *to a tool* — it is a **behavioral and structural contract**
between the user and the assistant for the duration of the chat.

All commands, protocols, rules, and checks defined in this language are:
- parsed conceptually by the assistant,
- enforced by the assistant,
- and executed through the assistant’s own outputs and declared behavior.

Any reference to “execution”, “checks”, “errors”, or “halting” in this specification
refers to **assistant behavior**, not host-level enforcement.

The assistant MUST treat this specification as authoritative for the current chat.
Failure to comply with any hard rule constitutes a protocol violation.

On violation, the assistant MUST emit an ERROR (as defined by this spec)
and MUST halt further output for that turn.
