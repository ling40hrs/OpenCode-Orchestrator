# OC-Orchestrator Architecture — For AI Agents

> **Purpose**: Machine-parseable reference for AI agents to understand the orchestrator system. Contains full metadata, decision matrices, permission tables, cost data, and state machines. No prose fluff.

---

## 1. AGENT REGISTRY

### 1.1 Primary Agent

```yaml
orchestrator:
  mode: primary
  model: opencode-go/deepseek-v4-flash
  cost_req_per_5hr: 31650
  cost_tier: budget
  temperature: 0.2
  color: primary
  permissions:
    edit: allow
    bash: allow
    task:
      "oc-*": allow
      "plan": allow
      "*": deny
  files:
    - .opencode/agents/orchestrator.md
  description: "Coding orchestrator — routes tasks and delegates. Never writes code or reads source files. Uses Flash for cheap routing, escalates complex tasks to oc-planner."
  tags: [router, delegator, primary]
```

### 1.2 Subagents

```yaml
oc-planner:
  mode: subagent
  model: opencode-go/deepseek-v4-pro
  cost_req_per_5hr: 3450
  cost_tier: premium
  temperature: 0.1
  color: primary
  permissions:
    edit: deny
    bash: ask  # read-only commands only
    read: allow
    glob: allow
    grep: allow
    webfetch: allow
  files:
    - .opencode/agents/oc-planner.md
  description: "Deep-thinking planner for complex multi-file tasks. Uses V4 Pro. Read-only: investigates codebase, returns structured implementation plan."
  tags: [planner, read-only, expensive]

oc-explorer:
  mode: subagent
  model: opencode-go/deepseek-v4-flash
  cost_req_per_5hr: 31650
  cost_tier: budget
  temperature: 0.1
  color: info
  permissions:
    edit: deny
    bash: ask
    read: allow
    glob: allow
    grep: allow
  files:
    - .opencode/agents/oc-explorer.md
  description: "Fast read-only codebase explorer. Searches files, reads code, answers structural questions."
  tags: [search, read-only, cheap]

oc-writer:
  mode: subagent
  model: opencode-go/minimax-m2.5
  cost_req_per_5hr: 6300
  cost_tier: balanced
  temperature: 0.3
  color: success
  permissions:
    edit: allow
    bash: allow
  files:
    - .opencode/agents/oc-writer.md
  description: "Implementation subagent. Edits files, writes new code, handles feature implementation and bug fixes."
  tags: [implementer, editor]

oc-reviewer:
  mode: subagent
  model: opencode-go/minimax-m2.5
  cost_req_per_5hr: 6300
  cost_tier: balanced
  temperature: 0.1
  color: warning
  permissions:
    edit: deny
    bash: ask
    read: allow
    grep: allow
  files:
    - .opencode/agents/oc-reviewer.md
  description: "Code review subagent. Analyzes code for correctness, security, performance, and style. Returns structured PASS/FAIL feedback."
  tags: [reviewer, read-only, evaluator]

oc-tester:
  mode: subagent
  model: opencode-go/deepseek-v4-flash
  cost_req_per_5hr: 31650
  cost_tier: budget
  temperature: 0.2
  color: info
  permissions:
    edit: allow
    bash: allow
  files:
    - .opencode/agents/oc-tester.md
  description: "Testing subagent. Writes and runs tests, debugs failures. Works with any test framework."
  tags: [tester, automation]

oc-designer:
  mode: subagent
  model: opencode-go/kimi-k2.6
  cost_req_per_5hr: 1150
  cost_tier: premium
  temperature: 0.6
  color: accent
  permissions:
    edit: allow
    bash: ask
    read: deny
    glob: deny
    grep: deny
    webfetch: allow
    task:
      "oc-explorer": allow
      "*": deny
  files:
    - .opencode/agents/oc-designer.md
    - .opencode/skills/frontend-design/SKILL.md
  description: "Frontend design PLANNER using Kimi K2.6. Creates design plans only — does NOT implement. Works from codebase reports, never reads source files."
  tags: [designer, planner, creative, expensive]

ask:
  mode: all
  model: opencode-go/deepseek-v4-flash
  cost_req_per_5hr: 31650
  cost_tier: budget
  temperature: 0.1
  color: info
  permissions:
    edit: deny
    bash: deny
    read: allow
    glob: allow
    grep: allow
    webfetch: allow
  files:
    - .opencode/agents/ask.md
  description: "Ask questions about codebase, get explanations, explore ideas, understand how code works. Read-only."
  tags: [qa, explainer, read-only]
```

---

## 2. DECISION MATRICES

### 2.1 Complexity Routing Gate

Decision criteria for orchestrator (Flash) to route between light and heavy paths:

```yaml
complexity_routing:
  input: user_task_description
  output: LIGHT | MEDIUM | HEAVY
  model_running: opencode-go/deepseek-v4-flash

  LIGHT:
    criteria:
      files_changed: "1 file"
      lines_changed: "< 10 lines"
      task_type: ["single CSS property", "text change", "simple bug fix", "single element addition"]
      needs_architecture_decision: false
      cross_cutting_change: false
    action: "Delegate directly to workers. Skip oc-planner."

  MEDIUM:
    criteria:
      files_changed: "2-3 files"
      lines_changed: "10-50 lines"
      task_type: ["moderate feature addition", "multi-step bug fix", "small refactor"]
      needs_architecture_decision: false
      cross_cutting_change: false
    action: "Delegate directly, or optionally use oc-planner if uncertain."

  HEAVY:
    criteria:
      files_changed: "4+ files"
      lines_changed: "50+ lines"
      task_type: ["new feature", "refactor", "architecture change", "cross-cutting concern"]
      needs_architecture_decision: true
      cross_cutting_change: true
    action: "MUST delegate to oc-planner (V4 Pro) first. Wait for plan. Then execute."
```

### 2.2 oc-designer Guardrails

```yaml
designer_guardrails:
  dispatch_to: oc-designer
  condition: ALL_REQUIRED
  requirements:
    - task_involves: ["new page", "new section", "redesign"]
    - requires_creative_decisions: true
    - fields_needed: ["typography", "layout", "color_palette", "visual_hierarchy"]
    - files_created: "3+ new files OR major structural changes"
    - user_signal: ["design", "plan", "visual direction", "layout", "style"]

  skip_to: oc-writer
  condition: ANY_MATCH
  skip_conditions:
    - task_type: "single CSS property change"
    - task_type: "one-line JSX tweak"
    - task_type: "existing UI bug fix"
    - task_type: "single element addition to existing page"
    - task_type: "responsive adjustment for one breakpoint"
    - task_type: "copy/text changes only"
    - signal_words: ["tweak", "adjust", "fix", "change color", "update", "modify"]

  self_rejection:
    agent: oc-designer
    trigger: "task is smaller than threshold"
    response: "REJECTED: This task is too small for @oc-designer. Dispatch @oc-writer instead."

  output_type: "design plan only — visual direction, component architecture, file structure"
  implementation_by: oc-writer
  codebase_reading: "oc-designer has read:deny. Works from structural reports from oc-explorer."
  deep_dive: "oc-designer may spawn task(oc-explorer, 'read file X') for specific file details."
```

### 2.3 AI Planning Convention

```yaml
ai_plans_convention:
  enabled: "project has .ai-plans/ directory"
  trigger_criteria:
    all_required:
      - "task has >3 distinct code-changing steps OR touches >2 files"
      - "no existing plan in .ai-plans/active/ covers the same goal"
  skip_when:
    - single_command: ["npm install", "git status"]
    - informational_query: true
    - single_line_fix: "one file only"

  user_approval_gate:
    required: true
    steps:
      - "generate 2-4 implementation options with trade-offs"
      - "include 'Other / custom' option"
      - "use question tool — do not proceed until user picks one"
      - "plan.goal and plan.steps must reflect user's choice"

  file_naming:
    pattern: "<category>-<target>.md"
    categories:
      feat: "New features, capabilities"
      fix: "Bug fixes, error corrections"
      refactor: "Structural changes, no behavior change"
      chore: "Maintenance, deps, tooling, CI"
      docs: "Documentation, comments, READMEs"
      perf: "Performance optimization"
      test: "Test additions, test fixes"
      style: "UI styling, design, CSS"

  file_format:
    frontmatter:
      goal: "string (required) — one-line summary"
      branch: "string (optional) — feature/xyz"
      cwd: "string (optional) — working directory"
      files: "string[] (optional) — affected files"
      created: "YYYY-MM-DD (required)"
      closed: "YYYY-MM-DD (added on completion)"
    sections:
      - name: "steps"
        format: "- [ ] N. description → expected result"
      - name: "verify"
        format: "- [ ] command to run"
      - name: "ctx"
        format: "bullet list of constraints, decisions, gotchas"

  lifecycle:
    create: "write to .ai-plans/active/<category>-<target>.md"
    tick: "[x] only after executing AND verifying. One per action."
    done:
      condition: "all verify checkboxes ticked"
      action: "move to .ai-plans/done/ + add closed: YYYY-MM-DD"
    abandon:
      condition: "explicit cancellation"
      action: "move to .ai-plans/abandoned/ + add closed: + reason:"
```

### 2.4 200-Line Cap Enforcement

```yaml
line_cap:
  hard_limit: 200
  optimal: 100-150
  warning_threshold: 150
  applies_to:
    extensions: [".tsx", ".jsx", ".vue", ".svelte", ".css", ".scss", ".html", ".js", ".ts"]
    context: "frontend only"
  enforcement:
    before_write: "estimate final line count before writing"
    if_exceeds: "do NOT write as one file — split"
    splitting_strategies:
      hooks: "extract when >2 useState or useEffect >10 lines"
      sub_components: "split when JSX >60 lines"
      constants_types: "move to .constants.ts and .types.ts"
      css_partials: "split into CSS modules per component"
      html_partials: "extract sections into partials/templates"
      pure_functions: "extract into utility modules"

  violation_handler:
    orchestrator_rejects: true
    message: "Split this file. Extract X into a separate component/module."
    reason: "File exceeds 200-line cap. Bug isolation becomes harder beyond this limit."
    no_placeholders: true
    refactor_first: true
```

---

## 3. PROMPT IMPROVEMENT LOOP

```yaml
prompt_improvement:
  always_required: true
  steps:
    - phase: "raw_input"
      input: "user's original prompt (may be vague, short, incomplete)"
    - phase: "improve"
      actions:
        - "clarify scope — which files, which framework, which behavior"
        - "add technical context — existing patterns to follow"
        - "make specific — enough detail for a subagent to act on"
        - "identify edge cases — empty states, errors, loading"
    - phase: "show_user"
      format: """
        I'll refine that before proceeding. Here's my improved version:

          <refined prompt with bullet points>

        Does this capture what you want?
        """
    - phase: "confirm"
      wait_for: "user says yes or provides edits"
      on_edit: "incorporate edits and re-confirm iteratively"
      max_iterations: null  # unlimited, but use judgement
  skip_when:
    condition: "prompt already has 5+ specific instructions"
    action: "brief confirm, skip full loop"
```

---

## 4. TASK DELEGATION STATE MACHINE

```yaml
delegation_state_machine:
  initial_state: TASK_ARRIVED
  states:
    TASK_ARRIVED:
      transitions:
        - to: CHECK_AI_PLANS
          condition: ".ai-plans/ directory exists"
        - to: COMPLEXITY_GATE
          condition: "no .ai-plans/ directory"

    CHECK_AI_PLANS:
      on_entry: "ls .ai-plans/active/"
      transitions:
        - to: RESUME_PLAN
          condition: "matching plan found"
          action: "read plan, tick steps, resume work"
        - to: USER_APPROVAL_GATE
          condition: "trigger criteria met, no matching plan"
        - to: COMPLEXITY_GATE
          condition: "skip criteria met"

    USER_APPROVAL_GATE:
      on_entry: "present 2-4 options with trade-offs + 'Other' option"
      transitions:
        - to: WRITE_PLAN
          condition: "user picks an option"
          action: "write plan to .ai-plans/active/<category>-<target>.md"
        - to: COMPLEXITY_GATE
          condition: "user skips planning"

    WRITE_PLAN:
      on_entry: "write plan file with YAML frontmatter"
      transitions:
        - to: COMPLEXITY_GATE
          condition: "plan written"

    COMPLEXITY_GATE:
      on_entry: "classify task as LIGHT | MEDIUM | HEAVY"
      transitions:
        - to: DESIGNER_GUARDRAILS
          condition: "frontend design task"
        - to: PARALLEL_DELEGATE
          condition: "task is LIGHT or MEDIUM"
        - to: OC_PLANNER
          condition: "task is HEAVY"

    DESIGNER_GUARDRAILS:
      on_entry: "check all 4 guardrail conditions"
      transitions:
        - to: SCAN_CODEBASE
          condition: "ALL guardrails pass"
          action: "scan UI structure first, then pass report to oc-designer"
        - to: PARALLEL_DELEGATE
          condition: "any guardrail fails"
          target: oc-writer

    SCAN_CODEBASE:
      on_entry: "task(oc-explorer, 'scan structure only — files, imports, hierarchy. No full dumps.')"
      transitions:
        - to: DELEGATE_DESIGNER
          condition: "scan report received"
        - to: OC_PLANNER
          condition: "task is HEAVY (not DESIGN)"

    OC_PLANNER:
      on_entry: "task(oc-planner, 'analyze and plan from report...')"
      transitions:
        - to: EXECUTE_PLAN
          condition: "plan received from oc-planner"

    DELEGATE_DESIGNER:
      on_entry: "task(oc-designer, 'report: <scan>. Task: design plan.')"
      transitions:
        - to: EXECUTE_PLAN
          condition: "design plan received from oc-designer"

    EXECUTE_PLAN:
      on_entry: "design plan → task(oc-writer, implement) → task(oc-tester) → task(oc-reviewer)"
      transitions:
        - to: SYNTHESIZE
          condition: "all workers complete"

    PARALLEL_DELEGATE:
      on_entry: "determine parallel vs sequential dispatch"
      actions:
        parallel: "dispatch independent tasks simultaneously"
        sequential: "chain dependent tasks"
      transitions:
        - to: SYNTHESIZE
          condition: "all subagents return results"

    SYNTHESIZE:
      on_entry: "collect results, check for failures"
      transitions:
        - to: CHECK_RESULT
          condition: "subagent returned"
        - to: EVALUATOR_OPTIMIZER
          condition: "all workers completed"

    CHECK_RESULT:
      on_entry: "verify subagent actually worked"
      checks:
        - "non-empty result returned?"
        - "plan step ticked [x]?"
        - "expected files created/modified?"
      transitions:
        - to: STORE_TASK_ID
          condition: "all checks pass"
        - to: FRESH_SESSION
          condition: "any check fails"

    STORE_TASK_ID:
      on_entry: "save task_id alongside plan step for resume/recall"
      transitions:
        - to: TICK_PLAN
          condition: "task_id stored"

    FRESH_SESSION:
      on_entry: |
        do NOT reuse stored task_id
        spawn completely new session: task(oc-writer, "same prompt...")
        if new session also fails → escalate to user
      transitions:
        - to: DELEGATE_DESIGNER
          condition: "retry with fresh session"
        - to: REPORT_AND_CLOSE
          condition: "second fresh session also failed"
          action: "escalate to user with failure summary"

    EVALUATOR_OPTIMIZER:
      on_entry: "review combined output"
      transitions:
        - to: RE_DELEGATE_WITH_FEEDBACK
          condition: "review found issues"
          action: "include specific fix instructions, use FRESH SESSION"
        - to: TICK_PLAN
          condition: "review passed"

    RE_DELEGATE:
      on_entry: "fresh session (do NOT reuse task_id)"
      transitions:
        - to: DELEGATE_DESIGNER
          condition: "max 2 retries"

    RE_DELEGATE_WITH_FEEDBACK:
      on_entry: "fresh session with fix instructions (do NOT reuse task_id)"
      transitions:
        - to: DELEGATE_DESIGNER
          condition: "max 3 iterations"

    RESUME_SESSION:
      on_entry: |
        user stopped midway
        check if task_id is stored for the interrupted step
        - yes → task(oc-writer, ..., task_id: "<stored_id>")
        - no  → task(oc-writer, "same prompt...") (fresh)
      transitions:
        - to: SYNTHESIZE
          condition: "resumed session completes"

    TICK_PLAN:
      on_entry: |
        verify plan was ticked by subagent (oc-writer, oc-tester tick themselves)
        if not ticked but work was done → orchestrator ticks manually
        if not ticked and no work done → transition to FRESH_SESSION
      transitions:
        - to: REPORT_AND_CLOSE
          condition: "all steps done and verify checkboxes pass"
          action: "move plan to .ai-plans/done/ + closed: YYYY-MM-DD"
        - to: COMPLEXITY_GATE
          condition: "more steps remain"

    REPORT_AND_CLOSE:
      on_entry: "summarize files changed, tests, status"
      transitions:
        - to: DONE
          condition: "report delivered"
      on_exit:
        move_plan: ".ai-plans/active/ → .ai-plans/done/ + closed: YYYY-MM-DD"
```

---

## 5. ERROR RECOVERY

```yaml
error_recovery:
  worker_empty_result:
    action: "fresh_session — do NOT reuse stored task_id, spawn completely new session"
    max_retries: 2
    notes: "If fresh session also fails, escalate to user. Do not retry 3 times."

  worker_permission_denied:
    action: "try_different_approach_or_ask_user. Use fresh session."

  review_found_critical_issues:
    action: "re_delegate_with_specific_fix_instructions using fresh session (no task_id)"
    loop_limit: 3

  too_many_iterations:
    action: "summarize_state_and_ask_user_for_guidance"

  user_stopped_midway:
    action: |
      Check stored task_id for the interrupted step.
      - task_id found → resume: task(agent, ..., task_id: "<stored_id>")
      - no task_id → fresh session with same prompt

  subagent_plan_not_ticked:
    action: |
      Verify output: if work was done but plan wasn't updated → manually tick.
      If no work was done → fresh session (no task_id).

resume_and_recall:
  store_task_id: "After spawning each subagent, save the returned task_id alongside the plan step it corresponds to."
  resume_mechanism: "task(agent, prompt, task_id: '<stored_id>') — resumes exact session"
  fresh_session_mechanism: "task(agent, 'same prompt') — no task_id, new session"
  plan_file_reference: "plan file tells orchestrator which step to resume from"
```

---

## 6. COST OPTIMIZATION TABLE

```yaml
cost_optimization:
  principle: "Use cheapest model that can do the job correctly"
  matrix:
    task_type:
      - type: "task routing & classification"
        model: "V4 Flash"
        cost: 31650
        note: "orchestrator default"

      - type: "simple code edit (1 file, <10 lines)"
        model: "MiniMax M2.5"
        cost: 6300
        agent: oc-writer

      - type: "code review"
        model: "MiniMax M2.5"
        cost: 6300
        agent: oc-reviewer

      - type: "write & run tests"
        model: "V4 Flash"
        cost: 31650
        agent: oc-tester

      - type: "codebase search & exploration"
        model: "V4 Flash"
        cost: 31650
        agent: oc-explorer

      - type: "Q&A, explanations"
        model: "V4 Flash"
        cost: 31650
        agent: ask

      - type: "complex multi-file planning"
        model: "V4 Pro"
        cost: 3450
        agent: oc-planner
        usage: "HEAVY tasks only"

      - type: "creative frontend design"
        model: "Kimi K2.6"
        cost: 1150
        agent: oc-designer
        usage: "new pages only, guardrails enforced"
```

---

## 7. PLAN LIFECYCLE PER AGENT

```yaml
plan_lifecycle:
  file_location: ".ai-plans/active/<category>-<target>.md"
  done_location: ".ai-plans/done/<category>-<target>.md"
  abandoned_location: ".ai-plans/abandoned/<category>-<target>.md"

  agent_responsibilities:
    - agent: oc-explorer
      can_edit: false
      ticks_own_step: false
      ticks_by: orchestrator
      verifies: "output is non-empty and matches the scan request"

    - agent: oc-writer
      can_edit: true
      ticks_own_step: true
      ticks_by: self
      verifies: "files were created/modified as expected"
      move_to_done: "if last step AND all verify checkboxes pass"
      move_to_abandoned: "if step fails irrecoverably"

    - agent: oc-tester
      can_edit: true
      ticks_own_step: true
      ticks_by: self
      verifies: "tests pass and were written correctly"
      on_failure: "leave [ ] and add failure details in ## ctx"

    - agent: oc-reviewer
      can_edit: false
      ticks_own_step: false
      ticks_by: orchestrator
      verifies: "review passed (PASS verdict)"

    - agent: oc-planner
      can_edit: false
      ticks_own_step: false
      ticks_by: orchestrator
      verifies: "plan file was written and has all required sections"

    - agent: oc-designer
      can_edit: true
      ticks_own_step: true  # writes the plan file initially
      ticks_by: self
      verifies: "design plan is complete and written to .ai-plans/active/"

  task_prompt_format:
    always_include: "After completing, update .ai-plans/active/<file>.md — tick your step [x]."

  tick_instructions_for_editable_agents:
    - "After completing your work, open .ai-plans/active/<name>.md"
    - "Find your step and change '- [ ]' to '- [x]'"
    - "If the step FAILED, leave '[ ]' and add a note in ## ctx"
    - "If this is the final step AND all verify checkboxes pass, move to .ai-plans/done/ and add closed: 'YYYY-MM-DD' to frontmatter"

  resume_and_recall:
    store_task_id: true
    location: "store alongside the plan step it corresponds to"
    resume: "task(agent, prompt, task_id: '<stored_id>')"
    fresh_session: "task(agent, 'same prompt') — no task_id parameter"
    when_failed: "do NOT reuse stored task_id — spawn completely new session"
    when_stopped_midway: |
      - task_id available: resume exact session
      - no task_id: spawn fresh session with same prompt
      - plan file tells orchestrator which step to resume from
    verification_checks:
      - "non-empty result returned?"
      - "plan step ticked [x]?"
      - "expected files created/modified?"
      - "if any fails → re-delegate to fresh session (new task_id)"
```

---

## 8. FILE INDEX

```yaml
file_index:
  config:
    - path: "opencode.json"
      purpose: "Global config: default_agent, model, provider, permissions"
      schema: "https://opencode.ai/config.json"

  docs:
    - path: "AGENTS.md"
      purpose: "Project instructions for AI agents entering the repo"
    - path: "docs/architecture-for-humans.md"
      purpose: "Human-readable architecture overview"
    - path: "docs/architecture-for-agents.md"
      purpose: "Machine-parseable architecture reference"

  agents:
    - path: ".opencode/agents/orchestrator.md"
      purpose: "Primary orchestrator agent — routing + delegation logic"
      role: primary
    - path: ".opencode/agents/oc-planner.md"
      purpose: "Deep-thinking planner for complex tasks"
      role: subagent
    - path: ".opencode/agents/oc-explorer.md"
      purpose: "Codebase exploration"
      role: subagent
    - path: ".opencode/agents/oc-writer.md"
      purpose: "Implementation and edits"
      role: subagent
    - path: ".opencode/agents/oc-reviewer.md"
      purpose: "Code review"
      role: subagent
    - path: ".opencode/agents/oc-tester.md"
      purpose: "Test writing and execution"
      role: subagent
    - path: ".opencode/agents/oc-designer.md"
      purpose: "Frontend design and new page planning"
      role: subagent
    - path: ".opencode/agents/ask.md"
      purpose: "Q&A, code explanation"
      role: all

  skills:
    - path: ".opencode/skills/orchestrator-delegation/SKILL.md"
      purpose: "Delegation strategies for the orchestrator"
    - path: ".opencode/skills/frontend-design/SKILL.md"
      purpose: "Design guidelines loaded by oc-designer"
```

---

## 9. PROMPT IMPROVEMENT EXAMPLES

```yaml
prompt_examples:
  bad_vs_good:
    - user_says: "add login"
      improved: |
        Add a login form component to the homepage:
        - Create src/components/LoginForm.tsx with email + password fields
        - Validate inputs on submit
        - Show error messages inline
        - Style it to match the existing design

    - user_says: "fix the bug"
      improved: |
        Investigate and fix the bug in src/api/users.ts where:
        - When a user submits an empty email field, the app crashes
        - Expected: show a validation error instead
        - Add input validation before the API call
        - Write a test for the empty-email edge case

    - user_says: "change color"
      improved: |
        Update the primary button color in src/styles/theme.css:
        - Change --color-primary from #3b82f6 to #2563eb
        - Verify contrast ratio meets WCAG AA standard
        - Check all button variants (primary, secondary, ghost) are consistent
```
