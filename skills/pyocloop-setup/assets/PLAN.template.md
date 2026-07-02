# {{PLAN_TITLE}}

## Overview

{{GOAL_DESCRIPTION - one or two sentences on what this loop accomplishes.}}

Output / results are written to: `{{OUTPUT_FILE - e.g. results/summary.md}}`

## Backlog

### Phase 1: {{PHASE_NAME - e.g. Items 1-10}}

- [ ] **1** {{TASK_DESCRIPTION - one self-contained item; OpenCode sees only this}}
- [ ] **2** {{TASK_DESCRIPTION}}
- [ ] **3** {{TASK_DESCRIPTION}}
- [ ] **4** {{TASK_DESCRIPTION}}
- [ ] **5** {{TASK_DESCRIPTION}}
- [ ] **6** {{TASK_DESCRIPTION}}
- [ ] **7** {{TASK_DESCRIPTION}}
- [ ] **8** {{TASK_DESCRIPTION}}
- [ ] **9** {{TASK_DESCRIPTION}}
- [ ] **10** {{TASK_DESCRIPTION}}

### Phase 2: {{PHASE_NAME - next batch}}

- [ ] **11** {{TASK_DESCRIPTION}}
- [ ] **12** {{TASK_DESCRIPTION}}

<!--
Task line syntax (parser reads the checkbox between [ and ]):
  - [ ]                 pending      (OpenCode picks the FIRST of these)
  - [x]  / - [X]        completed
  - [MANUAL]            skipped by the loop (needs a human)
  - [BLOCKED: reason]   could not complete; reason recorded

Rules:
  - Execution order = file order (first pending line wins).
  - Add a new "### Phase N: ..." section per ~10 tasks.
  - Keep phases small for reliable TUI progress tracking.
  - The loop ends when the prompt appends a single line:
        <plan-complete>summary</plan-complete>
-->
