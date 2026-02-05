# std.io.summarize — Documentation (v0.1.0)

This document is the **human-facing** companion to `stdlib-std.io.summarize.aicl`.

- The `.aicl` file is intentionally minimal (token-efficient).
- This `.md` file contains detailed behavior, conventions, and examples.
- Keep the API surface in this doc **in sync** with the `.aicl` module.

---

## What it does

`/summarize(...)` produces:

1) A **summary artifact** (Markdown) intended to be saved as a downloadable file by the wrapper/runtime.
2) A **Continuation Prompt** (printed to screen by default, and also included in the file content by default).

This exists because repeatedly re-explaining summarization preferences is expensive and error-prone.

---

## Invocation

Depending on your runtime resolution rules, you may call:

- `/summarize(...)`
- `/std.io.summarize(...)`

Both refer to:
- `module_namespace: std.io`
- `command_key: summarize`

---

## Runtime Requirement (Scope = chat)

`scope="chat"` is the default and is **runtime-enforced**:

- The runtime/wrapper MUST supply the full chat transcript (or the approved transcript slice) as the summarization input.
- If the transcript is unavailable, the runtime should fail deterministically rather than silently summarizing partial context.

This prevents “summarize only the last topic” behavior.

---

## Arguments (v0.1)

All arguments are optional; defaults apply.

### `mode`
- `brief` (default): structured summary optimized for quick restart.
- `full`: exhaustive capture, including important lists/tables/backlogs when present.

### `scope`
- `chat` (default): summarize the current conversation (runtime must supply transcript).
- `text`: summarize the provided `input`.

### `input`
- Required when `scope="text"`.
- Ignored when `scope="chat"`.

### `output`
- `download` (default): file artifact is produced; screen prints **continuation prompt only**.
- `preview`: screen prints summary + continuation prompt; wrapper MAY ignore the file block.
- `both`: file artifact is produced AND screen prints summary + continuation prompt.

> Note: `output` is a policy hint for the wrapper/runtime. The module always emits the file block for determinism; wrappers can ignore it for `preview`.

### `includeContinuationPrompt`
- `true` (default): include continuation prompt content and print it to screen as appropriate.
- `false`: omit continuation prompt sections.

### `fileName`
- Optional file name override.
- If omitted, wrapper should choose a reasonable default, e.g.:
  - `AICL_SUMMARY_brief.md`
  - `AICL_SUMMARY_full.md`

---

## Output Format (Deterministic)

The command output must contain **only**:

1) A file block:
```
BEGIN_AICL_SUMMARY_FILE <file_name>
<file_content>
END_AICL_SUMMARY_FILE
```

2) A screen output block:
- For `output="download"`: continuation prompt only
- For `output="preview"|"both"`: summary + continuation prompt

No extra commentary should be emitted outside these sections.

---

## Summary Content Requirements

### Brief mode (`mode="brief"`)
Recommended sections:
- Title / Topic
- Key takeaways
- Decisions made
- Open questions
- Next steps
- Terminology clarified (only if discussed)

### Full mode (`mode="full"`)
Everything from brief mode, plus:
- Important detailed reasoning that led to decisions
- Important examples used
- Important lists or tables (especially ranked backlogs)

---

## Continuation Prompt Requirements

Default behavior is ON (for both `brief` and `full`) unless disabled.

Recommended continuation prompt content:
- One-liner objective for the next chat
- Explicit definition(s) to focus on (if relevant)
- The decision to be made next (binary or ranked)
- Constraints (e.g., “stay minimal, no token bloat”)

---

## Examples

### Default (brief + download + continuation prompt)
```
/summarize()
```

### Full capture (download by default)
```
/summarize(mode="full")
```

### Preview only (no download expected)
```
/summarize(output="preview")
```

### Both preview + download
```
/summarize(output="both")
```

### Summarize arbitrary text
```
/summarize(scope="text", input="...long text...")
```

---

## Sync Guard (Recommended)

To reduce doc drift, add a lightweight CI check:

- Parse the `.aicl` to extract:
  - `module_namespace`
  - `module_version`
  - command keys
  - args + defaults
- Assert this `.md` contains those values.

This keeps the API reference consistent even if narrative evolves.

---

## Version
- Module version: **0.1.0**
- File: `stdlib-std.io.summarize.aicl`
- Docs: `stdlib-std.io.summarize.md`
