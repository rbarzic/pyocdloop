# Loop prompt authoring guide

Load this when the user wants a custom or advanced `.loop-prompt.md` beyond the
standard "do one action, update one file" pattern.

## Anatomy

The file is markdown with a small YAML frontmatter:

```
---
description: Execute loop
---

<body — instructions OpenCode follows each iteration>
```

- The frontmatter is required by OpenCode's prompt format; keep `description: Execute loop`.
- The body is sent once per session (one task per session).

## The one placeholder you must use

`{{PLAN_FILE}}` — pyocloop rewrites this token to the **absolute path** of the
plan at runtime. Use it everywhere the plan is read or written. Never hardcode a
path, and do not invent other `{{...}}` placeholders (only `{{PLAN_FILE}}` is
substituted).

## Sections that work well

```
1. Before starting      → Read {{PLAN_FILE}} fully
2. Task selection        → first pending task, in-order phases, skip [MANUAL]/[BLOCKED]
3. Execute               → the actual job for this task
4. After completion      → mark [x], write outputs, record blockers
5. Completion check      → append <plan-complete> when nothing is pending
```

## Customization patterns

- **Output to a specific file:** add an explicit step in `After completion`:
  `Append the result for this item to <path/to/ZZZZZZ.md>`. Create the file on
  the first task; append on subsequent ones.
- **Multiple output files:** list each file and what should go in it.
- **Research / external knowledge:** instruct OpenCode to capture non-obvious
  findings into a `docs/<topic>.md` file and reference them from `AGENTS.md`,
  so later sessions reuse them instead of re-discovering.
- **Batching within a phase:** by default, one task per session. Only allow
  batching if the user explicitly asks, and only for tasks in the same phase and
  same file that are logically coupled.
- **Human-gated steps:** mark them `- [MANUAL] ...` in the plan and tell the
  prompt to skip `[MANUAL]` items.
- **Blocking gracefully:** if a task can't be done (permissions, external
  service), the prompt should write `- [BLOCKED: reason]` on that line and move on.

## Marking done & finishing

Always include both:

```
After completion:
1. Update {{PLAN_FILE}} marking the finished item with [x]   # per task

Completion check:
- If all non-[MANUAL] tasks are [x] or [BLOCKED]:
  - Append `<plan-complete>SUMMARY</plan-complete>` to {{PLAN_FILE}}   # once, at end
  - Exit the session
```

The per-task `[x]` is how progress advances; the final `<plan-complete>` is how
the loop terminates. Missing either stalls the run.

## Tips

- Keep the prompt focused on **one task's** lifecycle — it runs once per session.
- Be concrete about file paths, but route the *plan* through `{{PLAN_FILE}}`.
- Prefer telling OpenCode *what* a good result looks like over micromanaging *how*.
