---
name: pyocloop-setup
description: Generate pyocloop loop files (a PLAN.md task list and a .loop-prompt.md instruction file) so OpenCode can iteratively process a list of items through opencode. Use when the user wants to run the same job for each item in a list/batch, asks to loop over a backlog with opencode or OpenCode, or mentions pyocloop, ocloop, PLAN.md, loop-prompt, or orchestrating a repetitive task across many items.
license: MIT
compatibility: Requires pyocloop installed (ocloop on PATH) and OpenCode (opencode on PATH). Python 3.11+.
metadata:
  author: rbarzic
  version: "1.0"
  project: pyocloop
---

# Set up a pyocloop run

pyocloop (`ocloop`) is a loop harness that drives [OpenCode](https://opencode.ai)
(`opencode`) to work through a backlog **one task per session**: it reads a
`PLAN.md`, sends the next pending task to OpenCode, and repeats until the plan is
done. pyocloop does **not** call any LLM itself — it delegates all AI work to the
`opencode` binary.

Your job when this skill is activated: turn the user's request into the **two
files** pyocloop needs, then tell them how to launch the loop.

## The two files you must produce

| File (default name)         | Role                                                                 |
| --------------------------- | -------------------------------------------------------------------- |
| `PLAN.md`                   | The backlog — a markdown task list using checkbox syntax.            |
| `.loop-prompt.md`           | Instructions OpenCode follows on **every** iteration for one task.   |

Both must live in the directory the user specifies (the loop's working directory).
Note `.loop-prompt.md` is a **dotfile** — `ocloop run` defaults its `--prompt`
to this exact hidden name, and `ocloop bootstrap` creates it with this name.

## The request pattern this skill handles

A typical request looks like:

> "For each **XXXXX** items do **YYYYYY** and update **ZZZZZZ.md** file
> accordingly. Put the files in the directory **DDDDDDDDDD**."

Map it like this:

- **XXXXX** (the item list) → one `- [ ]` task line per item in `PLAN.md`
- **YYYYYY** (the action) → the `Execute:` body in `.loop-prompt.md`
- **ZZZZZZ.md** (the output) → a step in `.loop-prompt.md` that updates this file; also referenced in the plan's Overview
- **DDDDDDDDDD** (the location) → where both files are written

## Procedure

1. **Create the target directory** (DDDDDDDDDD) if it does not exist.

2. **Write `PLAN.md`** — start from `assets/PLAN.template.md` and:
   - Replace the title and Overview with the real goal.
   - Add **one** `- [ ]` task line per item from XXXXX. Each task must be
     self-contained (OpenCode sees only one task per session).
   - Group tasks into `### Phase N: ...` sections of ~10 tasks each. Phases are
     worked **strictly in order** — finish phase N before N+1.
   - If a task needs a human, mark it `- [MANUAL] description` (the loop skips it).
   - Use stable numbering (`**1**`, `**2**`, …) so tasks are easy to reference.

3. **Write `.loop-prompt.md`** — start from `assets/loop-prompt.template.md` and:
   - Keep the YAML frontmatter (`---\ndescription: Execute loop\n---`).
   - **Always** use the literal placeholder `{{PLAN_FILE}}` wherever the plan is
     referenced — pyocloop injects the absolute path at runtime. Never hardcode a path.
   - In `Execute:`, put the YYYYYY action the user asked for.
   - In `After completion:`, add a step that updates the ZZZZZZ.md output file.

4. **Tell the user how to run it** (see "Launch" below). Do **not** run the loop
   yourself unless explicitly asked — `ocloop run` opens an interactive TUI.

## Key rules (these break the loop if violated)

- **`{{PLAN_FILE}}` is mandatory.** The loop prompt must read and update the plan
  via this placeholder; pyocloop replaces it with the absolute path. If you write
  a literal path, the prompt stops working when the directory moves.
- **Mark `[x]` after EACH task**, not all at once at the end. The loop picks the
  *first* pending (`- [ ]`) line; if none is updated it loops forever on the same task.
- **`<plan-complete>` ends the run.** When every non-`[MANUAL]` task is `[x]` or
  `[BLOCKED: reason]`, the prompt must append a single line to `PLAN.md`:
  `<plan-complete>short summary</plan-complete>`. pyocloop stops on this tag.
- **Exact checkbox spellings** (parsed by pyocloop, see Gotchas):
  `- [ ]` pending · `- [x]` or `- [X]` done · `- [MANUAL]` skip ·
  `- [BLOCKED: reason]` cannot do.
- **One task per session** is the reliable default. Do not instruct OpenCode to
  batch tasks unless the user explicitly asks.

## Gotchas

- `.loop-prompt.md` starts with a **dot** (hidden file). Don't name it
  `loop-prompt.md` unless you also pass `--prompt` explicitly — the default
  `--prompt` value is the dotfile `.loop-prompt.md`.
- `ocloop run` **validates** that `PLAN.md` exists and refuses to start without
  it (use `--debug` to skip, but normally just create the file).
- `<plan-complete>` is matched anywhere on its own line; keep it on one line.
- Tasks are matched in **file order**, first pending wins — so phasing order in
  the file IS the execution order.
- If you'd rather start from pyocloop's own starter files, run
  `ocloop bootstrap <dir>` (creates both files), then customize them with the
  steps above. Use `-f` to overwrite.

## Launch

After writing both files, give the user the command. Run from inside the
directory (defaults apply):

```bash
cd DDDDDDDDD
ocloop run --model <provider/model> --run
```

…or run from outside by passing explicit paths:

```bash
ocloop run \
  --plan   DDDDDDDDD/PLAN.md \
  --prompt DDDDDDDDD/.loop-prompt.md \
  --model  <provider/model> --run
```

- `<provider/model>`: list with `opencode models`. Common: `zai-coding-plan/glm-5.1`,
  `openai/gpt-5.5`. Let the user pick if not specified.
- `--run` starts immediately; omit it to start manually by pressing `S` in the TUI.
- TUI keys: `S` start · `Space` pause/resume · `R` retry · `Q` quit.

## References (load on demand)

- Read `references/PLAN-FORMAT.md` when you need the **exact checkbox / blocked /
  manual syntax and parsing rules**, or details on phasing and the completion tag.
- Read `references/PROMPT-AUTHORING.md` when the user wants a **custom or advanced
  loop prompt** (multi-file output, research capture, batching, manual steps).

Both reference files are optional for a standard "for each item do X" request —
the templates in `assets/` plus the rules above are usually enough.
