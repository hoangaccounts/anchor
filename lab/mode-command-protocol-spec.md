# Mode Command Protocol Spec

## Purpose
This document defines the user’s custom conversational command protocol for controlling assistant behavior with compact single-line command tokens. The system is designed to reduce drift, support long design/strategy sessions, and provide explicit mode control without requiring repeated natural-language instructions.

## Design Goals
- Keep commands short, memorable, and fast to type.
- Make commands deterministic by placing them at the start of the message.
- Support both one-shot mode commands and persistent lock mode commands.
- Provide a minimal helper/control set.
- Avoid punctuation-heavy syntax.
- Preserve a universal hard interrupt command.

## Core Rules

### 1. Command Position
A command must be the **first token** in the message.

Correct:

```text
MCRL Is this feature actually defensible?
```

Incorrect:

```text
I have a question MCRL is this feature defensible?
```

### 2. Single-Line Use
Commands are intended to be used on the same line as the prompt when applicable.

Example:

```text
MBU Write the minimal analytics spec for v1.
```

### 3. One-Shot vs Lock Behavior
- Non-lock commands apply to the **current message only**.
- Lock commands persist across messages until replaced or cleared.

### 4. Lock Clearing
A persistent lock remains active until one of the following occurs:
- another lock command is issued
- `MU` is issued
- `CG` is issued

### 5. Global Interrupt
`CG` is a universal hard interrupt. It is not namespaced under `M`.

`CG` must:
- stop momentum
- reset to neutral behavior
- clear any active lock
- wait for the next instruction

### 6. Local Override Rule
If a persistent lock is active, an explicit one-shot mode command at the start of a message overrides the lock **for that message only**. After the response, the prior lock resumes unless changed by a new lock, `MU`, or `CG`.

Example:
- active lock: `MBUL`
- next message starts with: `MCR`
- assistant critiques for that message only
- after that, `MBUL` resumes

## Namespace
The protocol uses the `M` prefix for mode-related commands.

Pattern:

```text
M + action abbreviation
M + action abbreviation + L   (lock form)
```

## Mode Commands

### Explore
- `MEX` = Explore for current message only
- `MEXL` = Explore lock

Purpose:
Generate options, alternatives, and directions without converging too early.

Typical use cases:
- brainstorming features
- UI directions
- naming options
- monetization options

Example:

```text
MEX Alternative ways to visualize denomination progress.
```

### Critique
- `MCR` = Critique for current message only
- `MCRL` = Critique lock

Purpose:
Stress-test the idea, attack assumptions, expose risks, weak logic, vague thinking, weak UX, or lack of defensibility.

Typical use cases:
- feature evaluation
- business risk review
- UX risk review
- challenge weak proposals

Example:

```text
MCRL Is adding history actually high ROI?
```

### Build
- `MBU` = Build for current message only
- `MBUL` = Build lock

Purpose:
Produce concrete output instead of debating.

Typical use cases:
- specs
- prompts
- implementation plans
- structured deliverables
- concrete copy

Example:

```text
MBU Write the minimal Firebase analytics event schema.
```

### Decide
- `MDE` = Decide for current message only
- `MDEL` = Decide lock

Purpose:
Force a recommendation based on available tradeoffs.

Typical use cases:
- choosing between options
- locking product direction
- resolving design indecision

Expected behavior:
- evaluate options
- recommend the best path
- explain tradeoffs
- state what loses

Example:

```text
MDE Should ChaChing add history in v2?
```

### Check
- `MCH` = Check for current message only
- `MCHL` = Check lock

Purpose:
Review for correctness, completeness, omissions, logic gaps, unclear wording, or missing edge cases.

Typical use cases:
- spec review
- prompt review
- UX logic review
- edge-case audits

Example:

```text
MCH Review this analytics spec for missing events and edge cases.
```

### Explore + Critique
- `MEXCR` = Explore then critique for current message only
- `MEXCRL` = Explore then critique lock

Purpose:
Generate options and then immediately stress-test them.

Typical use cases:
- product strategy ideation with immediate filtering
- UI option generation with immediate evaluation
- feature brainstorming without drifting into shallow idea spam

Expected behavior:
1. generate multiple options
2. critique each option
3. identify strongest direction

Example:

```text
MEXCR New ways to differentiate ChaChing from generic counter apps.
```

## Helper / Control Commands

### MM
Show the currently active mode.

Expected output examples:

```text
Mode: NONE
```

```text
Mode: CRITIQUE-LOCK (MCRL)
```

### MH
Show compact mode help.

`MH` should print:
1. current active mode
2. mode commands
3. control commands

Expected output format:

```text
Mode: NONE

Modes
MEX / MEXL     Explore ideas
MCR / MCRL     Critique ideas
MBU / MBUL     Build / produce output
MDE / MDEL     Decide between options
MCH / MCHL     Check correctness / completeness
MEXCR / MEXCRL Explore then critique

Controls
MM      Show active mode
MH      Show help
MU      Unlock mode
CG      Interrupt and clear mode
```

### MU
Unlock the current active mode and return to normal behavior.

Expected behavior:
- clear active lock
- return to default behavior

### CG
Global hard interrupt.

Expected behavior:
- stop momentum immediately
- reset to neutral
- clear any active lock
- do not continue advancing the idea until the next instruction

## Full Command Set

```text
MEX / MEXL       Explore
MCR / MCRL       Critique
MBU / MBUL       Build
MDE / MDEL       Decide
MCH / MCHL       Check
MEXCR / MEXCRL   Explore then critique

MM               Show current mode
MH               Show mode help
MU               Unlock mode
CG               Hard interrupt
```

## Behavioral Priority

When interpreting commands, use the following order:
1. `CG`
2. explicit command at the start of the current message
3. active lock mode
4. default behavior

## Recommended Usage Examples

### Example 1: One-shot critique
```text
MCR Is this pricing strategy too cheap to signal quality?
```

### Example 2: Persistent critique mode
```text
MCRL Is this feature actually defensible?
```

Followed by:

```text
What customer problem does it solve that competitors do not?
```

### Example 3: Switch to build mode
```text
MBUL Write the product spec for the winning option.
```

### Example 4: Temporary override while locked
Assume `MBUL` is active.

```text
MCH Check this spec for edge cases before continuing.
```

Result:
- current message uses Check mode
- after response, Build lock resumes

### Example 5: Interrupt
```text
CG
```

### Example 6: Query mode state
```text
MM
```

### Example 7: Get command help
```text
MH
```

## Anti-Patterns
Do not:
- put command tokens in the middle of a sentence
- create many new command families without real need
- add punctuation variants for the same command
- rely on previous message mode if the lock was cleared
- over-expand the protocol into a giant command shell

## Guidance for Future Expansion
Only add new commands if all of the following are true:
1. the command solves a repeated real workflow problem
2. it is meaningfully different from existing modes
3. it will be used often enough to justify memorization
4. it does not bloat the command language

Recommended principle:
- keep the protocol small
- prefer consistency over cleverness
- optimize for fast typing and strong recall

## Summary
This protocol creates a compact conversational control layer that:
- reduces AI drift
- supports focused long sessions
- allows explicit cognitive mode switching
- provides persistent lock modes
- includes helper and interrupt controls
- keeps the interface short enough for real daily use
