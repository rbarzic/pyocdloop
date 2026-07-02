# PLAN.md format reference

Detailed rules for `PLAN.md`, derived from pyocloop's parser
(`src/pyocloop/plan_parser.py`). Load this when you need exact syntax.

## Task line syntax

Every task is a single markdown list item starting with `- [`. The parser reads
the **checkbox** (text between `[` and the first `]`) and the **description**
(text after `]`):

```
- [ ]              pending      ← OpenCode will pick this
- [x]   or - [X]   completed
- [MANUAL]         manual       ← skipped by the loop (human task)
- [BLOCKED: reason] blocked     ← cannot complete; reason recorded
```

Equivalent inline spellings are also accepted when the checkbox is empty:

```
- [] [MANUAL] description
- [] [BLOCKED: reason] description
```

Prefer the primary forms (`- [ ]`, `- [MANUAL]`, `- [BLOCKED: reason]`).

## Rules

- **Match is line-based.** A line only counts as a task if, when stripped, it
  starts with `- [`. Indented sub-bullets (`  - [ ]`) are still parsed as tasks,
  so keep tasks at the same indent level to avoid surprises.
- **Order is execution order.** The loop always selects the *first* pending
  (`- [ ]`) line in the file. Whatever appears first is what runs next.
- **`[x]` must be set by the prompt, immediately after a task finishes.** The
  parser recomputes progress (total / completed / pending / percent) from the
  checkboxes each iteration. `[MANUAL]` items are excluded from the percentage.
- **Marking `[BLOCKED: reason]`** lets OpenCode record a task it could not do and
  move on; blocked tasks count as "settled" for completion purposes.

## Phasing

Group tasks under `### Phase N: <name>` headings. The recommended loop prompt
instructs OpenCode to:

- finish all tasks in phase N before touching phase N+1,
- pick the first pending task in the earliest incomplete phase.

Because execution order is file order, simply listing phases sequentially is
enough to enforce ordering. Keep phases small (≈10 tasks) for reliable progress
tracking in the TUI.

## Completion

The run ends when the loop prompt appends a single line anywhere in the plan:

```
<plan-complete>Short summary of work done and any remaining MANUAL tasks</plan-complete>
```

The parser matches the *last* `<plan-complete>…</plan-complete>` block in the
file, on its own, so keep the tag to one line. pyocloop detects this and stops.

## Progress math (for context)

```
total        = all task lines
pending      = total - completed - manual - blocked
percent      = round(completed / (total - manual) * 100)
```

`[MANUAL]` items never count against completion, so a plan of only `[MANUAL]`
tasks reads as 100% complete.
