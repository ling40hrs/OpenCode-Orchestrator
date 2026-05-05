---
name: orchestrator-delegation
description: Orchestrator delegation strategies — when to delegate in parallel, when to chain sequentially, and how to recover from worker failures.
license: MIT
compatibility: opencode
metadata:
  audience: agents
  pattern: orchestrator-workers
---

## AI Planning Convention (`.ai-plans/`)

If the project has `.ai-plans/` at the root, follow this before any delegation:

1. **Check**: Run `ls .ai-plans/active/` — if a matching plan exists, resume it
2. **Create**: If trigger criteria met (multi-step or >2 files):
   - Present 2-4 implementation options to user → let them pick
   - Write plan to `.ai-plans/active/<category>-<target>.md` with YAML frontmatter
   - Categories: `feat-`, `fix-`, `refactor-`, `chore-`, `docs-`, `perf-`, `test-`, `style-`
3. **Tick**: Mark each step `[x]` only after execution + verification
4. **Close**: Move to `done/` or `abandoned/` when finished

### Plan Lifecycle: Who Updates What

| Subagent | Can edit? | Who ticks plan step? | Who handles done/abandoned? |
|---|---|---|---|
| oc-explorer | edit: deny | Orchestrator ticks after verifying | Orchestrator |
| oc-writer | edit: allow | oc-writer ticks itself | oc-writer (if last step) or Orchestrator |
| oc-tester | edit: allow | oc-tester ticks itself | oc-tester (if last step) or Orchestrator |
| oc-reviewer | edit: deny | Orchestrator ticks after verifying | Orchestrator |
| oc-planner | edit: deny | Orchestrator ticks after plan returned | Orchestrator |
| oc-designer | edit: allow | oc-designer writes the plan file initially | oc-designer or Orchestrator |

### Resume & Recall

When spawning a subagent via `task()`, always store the returned `task_id` alongside the plan step.

**User stops midway → resume**:
1. Check the stored `task_id` for the interrupted step
2. Resume: `task(oc-writer, ..., task_id: "<stored_id>")`
3. If no task_id stored → spawn fresh session with same prompt

**Subagent failed (empty result, error, no plan tick) → fresh session**:
1. Do NOT use the stored `task_id` — session may be corrupted
2. Spawn completely new: `task(oc-writer, "same prompt...")` (no task_id param)
3. If fresh session also fails → escalate to user

**Verify the subagent actually worked**:
1. Non-empty result returned?
2. Plan step ticked `[x]`?
3. Expected files created/modified?
4. If any check fails → re-delegate to fresh session

## Delegation Patterns

### Parallel Dispatch
Use when subtasks are **independent** (no output dependency):
```
@oc-writer "implement feature X in file A"
@oc-tester "write tests for feature X"
```
Next step: `@oc-reviewer "review both the implementation and tests"`

### Sequential Chain
Use when each step depends on the previous:
```
@oc-explorer "find all files related to the auth system"
→ (analyze output)
→ @oc-writer "refactor auth according to the exploration findings"
→ @oc-tester "update tests for the refactored auth"
→ @oc-reviewer "validate the final result"
```

### Evaluator-Optimizer Loop
After initial implementation + review:
```
if @oc-reviewer output is FAIL:
  → delegate @oc-writer with specific fix instructions
  → delegate @oc-reviewer to re-verify
  → max 3 iterations
```

### Recovery
- **Worker returns empty**: do NOT reuse task_id — spawn fresh session with same prompt
- **Worker hits bash permission denied**: try a different approach that doesn't need that command
- **Worker times out**: split the task into smaller pieces and delegate separately (fresh sessions)
- **Review found HIGH severity**: treat as FAIL and re-delegate immediately (fresh session)
- **Plan step not ticked after work**: verify output — if work was done, tick manually. If not, fresh session.

## Prohibited Patterns
- Never delegate tests to `@oc-writer` or implementation to `@oc-tester`
- Never skip review for changes to shared/utility code
- Never run `@oc-reviewer` and `@oc-writer` on the same file in parallel (race condition)
