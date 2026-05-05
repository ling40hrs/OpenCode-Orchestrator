# OC-Orchestrator Architecture — For Humans

## Purpose

A general-purpose coding orchestrator for OpenCode. Instead of one giant AI doing everything, the orchestrator **routes your task to the right specialist agent** — saving money, improving quality, and keeping files under 200 lines.

---

## Agent Hierarchy

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    PRIMARY AGENT (Tab-switchable)                          │
│                                                                            │
│   orchestrator (V4 Flash ~31K req/5hr)  ← YOU TALK TO THIS ONE            │
│   ┌──────────────────────────────────────────────────────────────────┐   │
│   │ CLASSIFIES first: "Is this LIGHT, MEDIUM, HEAVY, or DESIGN?"     │   │
│   │ DESIGN → oc-designer (Kimi) for creative planning                  │   │
│   │ HEAVY  → oc-planner (V4 Pro) for deep architecture                 │   │
│   │ MEDIUM → direct or optional oc-planner                             │   │
│   │ LIGHT  → direct to workers                                         │   │
│   │                                                                    │   │
│   │ NEVER reads source code — always uses oc-explorer for scans        │   │
│   │ NEVER writes code — always uses oc-writer for implementation       │   │
│   │ Tracks task_ids for resume/recall                                  │   │
│   └──────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┬───────────────────┐
         │                    │                    │                   │
         ▼                    ▼                    ▼                   ▼
   ┌──────────┐    ┌──────────────────┐    ┌──────────────┐    ┌───────────────┐
   │  LIGHT   │    │     MEDIUM       │    │    HEAVY     │    │   DESIGN      │
   │  ─────── │    │     ──────       │    │    ─────     │    │   ──────      │
   │  oc-     │    │  oc-writer or    │    │  oc-         │    │  oc-          │
   │  writer  │    │  optionally      │    │  planner     │    │  explorer     │
   │  directly│    │  oc-planner      │    │  (V4 Pro)    │    │  (scan)       │
   └──────────┘    └──────────────────┘    │  then        │    │  oc-designer  │
                                           │  oc-writer   │    │  (Kimi plan)  │
                                           │  oc-tester   │    │  oc-writer    │
                                           │  oc-reviewer │    │  oc-tester    │
                                           └──────────────┘    │  oc-reviewer  │
                                                               └───────────────┘
```

---

## Flowchart: Every Scenario

### Scenario 1: LIGHT Task (single edit, <10 lines)

```
User: "change this button to blue"
         │
         ▼
┌─────────────────────┐
│ 0. Prompt Improve   │──→ User confirms
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 1. CLASSIFY         │──→ LIGHT (1 file, obvious change)
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 2. Plan file        │──→ Write minimal plan to .ai-plans/active/
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 3. Delegate         │──→ task(oc-writer, "change color...tick [x] in plan")
│   task(writer)      │      │
│   ┌──────────┐      │      ▼
│   │ oc-writer│      │   oc-writer edits file, ticks [x], returns
│   └──────────┘      │
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 4. Synthesize       │──→ Verify plan step was ticked, report done
└─────────────────────┘
```

### Scenario 2: HEAVY Task (multi-file, architecture change)

```
User: "fix the broken checkout flow affecting 5 files"
         │
         ▼
┌─────────────────────┐
│ 0. Prompt Improve   │──→ User confirms
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 1. CLASSIFY         │──→ HEAVY (5 files, cross-cutting)
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 2. Plan file        │──→ User Approval Gate: present 2-4 options
│   + Approval Gate   │    User picks → write to .ai-plans/active/
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 3. SCAN             │──→ task(oc-explorer, "scan structure only")
│   task(explorer)    │      │
│   ┌──────────┐      │      ▼
│   │ explorer │      │   Returns: file tree, imports, hierarchy (NO full dumps)
│   └──────────┘      │
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 4. PLAN             │──→ task(oc-planner, "plan from report:<scan>")
│   task(planner)     │      │
│   ┌──────────┐      │      ▼
│   │ oc-planner│     │   Returns: detailed impl plan, step-by-step
│   └──────────┘      │
└─────────────────────┘
         │
         ▼
┌──────────────────────────────────────────────────────┐
│ 5. EXECUTE (sequential chain)                        │
│                                                       │
│  task(oc-writer,  "impl step 1...tick [x]")          │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐   │
│  │ oc-writer│───→│ oc-tester│───→│ oc-reviewer  │   │
│  │ (M2.5)   │    │ (Flash)  │    │ (M2.5)       │   │
│  │ ticks [x]│    │ ticks [x]│    │ orchestrator │   │
│  │ in plan  │    │ in plan  │    │ ticks [x]    │   │
│  └──────────┘    └──────────┘    └──────────────┘   │
│                                                       │
│  Each step uses task_id stored for resume/recall      │
└──────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 6-8. Synthesize →   │──→ Plan moved to .ai-plans/done/
│      Validate →     │    closed: YYYY-MM-DD
│      Report         │
└─────────────────────┘
```

### Scenario 3: DESIGN Task (new UI/animation direction)

```
User: "do a full plan and initialization on how the ui should look like and the animations"
         │
         ▼
┌─────────────────────┐
│ 0. Prompt Improve   │──→ User confirms
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 1. CLASSIFY         │──→ DESIGN (UI + animations, takes priority over HEAVY)
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 2. Plan file        │──→ User Approval Gate: 2-4 options
│   + Approval Gate   │    User picks → .ai-plans/active/design-<target>.md
└─────────────────────┘
         │
         ▼
┌───────────────────────────────────────────────┐
│ 3. SCAN (UI-focused)                          │
│   task(oc-explorer,                           │
│     "scan UI structure: components, styles,   │
│      imports. No full file dumps.")           │
│   ┌──────────┐                                │
│   │ explorer │──→ Returns: file tree,         │
│   └──────────┘    component hierarchy         │
└───────────────────────────────────────────────┘
         │
         ▼
┌───────────────────────────────────────────────┐
│ 4. CHECK GUARDRAILS                           │
│   □ New page/redesign? ✅                     │
│   □ Creative decisions? ✅                    │
│   □ 3+ new files? ✅                          │
│   □ User asked for design? ✅                 │
│   ALL PASS → route to oc-designer             │
└───────────────────────────────────────────────┘
         │
         ▼
┌───────────────────────────────────────────────┐
│ 5. PLAN (Kimi creates design plan)             │
│   task(oc-designer,                            │
│     "Report:<scan>. Task:design UI + anim.")   │
│                                               │
│   ┌──────────────┐                            │
│   │ oc-designer  │──→ Kimi (K2.6)             │
│   │ (read: deny) │    • Uses report only       │
│   │              │    • May spawn oc-explorer  │
│   │              │      for specific file      │
│   └──────────────┘    • Writes plan to         │
│                         .ai-plans/active/      │
│                       • Returns design plan    │
└───────────────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────────────────────┐
│ 6. EXECUTE (design → implement → test → review)        │
│                                                         │
│  task(oc-writer,  "implement design plan...tick [x]")   │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐     │
│  │ oc-writer│───→│ oc-tester│───→│ oc-reviewer  │     │
│  │ (M2.5)   │    │ (Flash)  │    │ (M2.5)       │     │
│  │ ticks [x]│    │ ticks [x]│    │ orchestrator │     │
│  │ in plan  │    │ in plan  │    │ ticks [x]    │     │
│  └──────────┘    └──────────┘    └──────────────┘     │
└────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 7-8. Validate →     │──→ Plan moved to .ai-plans/done/
│      Report         │
└─────────────────────┘
```

### Scenario 4: Resume After User Stopped Midway

```
User stops the session after oc-writer completed step 2 of 5
         │
         ▼
┌──────────────────────────────────────────────┐
│ Resume flow:                                  │
│                                                │
│  1. Orchestrator checks: do I have stored     │
│     task_id for the interrupted subagent?     │
│                                                │
│     ┌─ Yes → task(oc-writer, "continue...",   │
│     │         task_id: "<stored_id>")          │
│     │         → Resumes EXACT session          │
│     │                                           │
│     └─ No  → task(oc-writer, "continue...")    │
│               → Fresh session (same work)      │
│                                                │
│  2. Orchestrator reads plan file to determine  │
│     which step to resume from                  │
│                                                │
│  3. Continues from step 3                      │
└────────────────────────────────────────────────┘
```

### Scenario 5: Subagent Failed — Fresh Session

```
task(oc-writer, "implement step 3") returns empty result or error
         │
         ▼
┌──────────────────────────────────────────────┐
│ 1. Check: did subagent return non-empty?     │
│    → No: it failed                            │
│                                                │
│ 2. Do NOT reuse the stored task_id            │
│    (corrupted session would fail again)       │
│                                                │
│ 3. Spawn COMPLETELY NEW session:              │
│    task(oc-writer, "same prompt...")          │
│    (no task_id parameter)                     │
│                                                │
│    ┌─ New session succeeds → continue         │
│    └─ New session also fails → escalate to    │
│       user with summary of the failure        │
└────────────────────────────────────────────────┘
```

### Scenario 6: Plan Lifecycle (Subagents Update Plan)

```
┌────────────────────────────────────────────────────────────┐
│ Plan file: .ai-plans/active/feat-dark-mode-ui.md            │
│                                                             │
│ ---                                                         │
│ goal: "Redesign UI with dark theme..."                      │
│ created: "2026-05-05"                                       │
│ ---                                                         │
│                                                             │
│ ## steps                                                    │
│ [x] 1. Install framer-motion          ← oc-tester ticked   │
│ [ ] 2. Create themeStore              ← oc-writer working  │
│ [ ] 3. Create ThemeToggle component                        │
│ ...                                                        │
│                                                             │
│ ## verify                                                   │
│ [ ] pnpm run lint                                           │
│ [ ] pnpm run build                                          │
└────────────────────────────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────────────────────────┐
│ oc-writer completes step 2:                                 │
│   → Opens plan file                                         │
│   → Changes [ ] to [x] on step 2                           │
│   → Adds comment in ## ctx if issues found                 │
│   → Returns summary to orchestrator                        │
│                                                             │
│ Orchestrator verifies:                                      │
│   ☐ Non-empty result?                                       │
│   ☐ Plan step ticked?                                       │
│   ☐ Expected files changed?                                 │
│                                                             │
│ If step is FINAL AND all verify pass:                       │
│   → Move plan to .ai-plans/done/                            │
│   → Add closed: "2026-05-05" to frontmatter                 │
└────────────────────────────────────────────────────────────┘
```

---

## The Subagents (Workers)

```
                    ┌───────────────────────────────┐
                    │        oc-explorer              │
                    │    V4 Flash ~31K/hr              │
                    │    Read-only search (structure) │
                    │    "Show directory tree, file   │
                    │     names, imports — NO full    │
                    │     file dumps"                 │
                    │    Orchestrator ticks plan      │
                    └───────────────────────────────┘

    ┌────────────────┐    ┌────────────────┐    ┌────────────────┐
    │   oc-writer     │    │  oc-reviewer   │    │   oc-tester    │
    │  MiniMax M2.5   │    │  MiniMax M2.5  │    │  V4 Flash      │
    │  ~6K req/5hr    │    │  ~6K req/5hr   │    │  ~31K req/5hr  │
    │  WRITES code    │    │  REVIEWS code  │    │  TESTS code    │
    │  TICKS its own  │    │  Read-only     │    │  TICKS its own │
    │  plan step [x]  │    │  Orch ticks    │    │  plan step [x] │
    └────────────────┘    └────────────────┘    └────────────────┘

    ┌──────────────────────┐    ┌──────────────────┐
    │    oc-designer        │    │     ask           │
    │   Kimi K2.6           │    │  V4 Flash ~31K   │
    │   ~1.1K req/5hr       │    │  Read-only Q&A   │
    │   DESIGN PLANNER only  │    │  "Explain this   │
    │   No codebase reading  │    │   code to me"    │
    │   No implementation    │    └──────────────────┘
    │   Writes plan file     │
    │   May spawn explorer   │
    └──────────────────────┘

    ┌────────────────────┐
    │    oc-planner       │
    │   V4 Pro ~3.4K/hr   │
    │   Read-only planner │
    │   Complex tasks     │
    │   only (expensive!) │
    │   Orch ticks plan   │
    └────────────────────┘
```

---

## Model Cost: Why Different Models?

| Agent | Model | Cost (req/5hr) | Used for |
|---|---|---|---|
| orchestrator | V4 Flash | ~31,650 | Routing, classification, simple delegation |
| oc-planner | V4 Pro | ~3,450 | Deep architectural thinking (complex only) |
| oc-writer | MiniMax M2.5 | ~6,300 | Writing code + ticking plan steps |
| oc-reviewer | MiniMax M2.5 | ~6,300 | Reviewing code |
| oc-tester | V4 Flash | ~31,650 | Writing/running tests + ticking plan steps |
| oc-explorer | V4 Flash | ~31,650 | Codebase scanning (structure only) |
| oc-designer | Kimi K2.6 | ~1,150 | Creative frontend DESIGN PLANNING only |
| ask | V4 Flash | ~31,650 | Q&A, code explanation |

**Key insight**: Flash (~31K/hr) is 10x cheaper than V4 Pro (~3.4K/hr). The orchestrator uses Flash by default and only pays for V4 Pro or Kimi when the task genuinely needs deep reasoning or creative design.

---

## Resume & Recall Summary

```
Subagent returns ──→ Check result
                        │
                   ┌────┴────┐
                   │         │
                SUCCESS    FAILURE
                   │         │
                   ▼         ▼
            Store task_id  IGNORE task_id
            Continue       Spawn FRESH session
                           (no task_id passed)
```

---

## File Structure

```
OC-Orchestrator/
├── opencode.json                     # Config: default_agent, model, permissions
├── AGENTS.md                         # Project instructions for AI
├── docs/
│   ├── architecture-for-humans.md    # ← You are here
│   └── architecture-for-agents.md    # Machine-parseable for AI agents
├── .opencode/
│   ├── agents/
│   │   ├── orchestrator.md           # Primary: Flash router (334 lines)
│   │   ├── oc-planner.md             # V4 Pro: complex task planning
│   │   ├── oc-explorer.md            # V4 Flash: structural scanning
│   │   ├── oc-writer.md              # M2.5: implementation + ticks plan
│   │   ├── oc-reviewer.md            # M2.5: code review
│   │   ├── oc-tester.md              # V4 Flash: testing + ticks plan
│   │   ├── oc-designer.md            # Kimi K2.6: design planning only
│   │   └── ask.md                    # V4 Flash: Q&A
│   └── skills/
│       ├── orchestrator-delegation/  # Delegation strategies + resume/recall
│       └── frontend-design/          # Design guidelines for Kimi
```

---

## 🔑 Quick Reference

| You type | Classification | What happens |
|---|---|---|
| "Explain how auth works" | LIGHT | `@ask` reads auth code, explains it |
| "Change button to blue" | LIGHT | `@oc-writer` makes the change, ticks plan |
| "Create a login page" | DESIGN | scan → `@oc-designer` plans → `@oc-writer` builds |
| "Fix broken checkout (4 files)" | HEAVY | scan → `@oc-planner` plans → workers execute |
| "Add tests for the cart" | MEDIUM | `@oc-tester` writes and runs tests, ticks plan |
| "Review my PR" | MEDIUM | `@oc-reviewer` analyzes code, orch ticks plan |
| "What files use the User model?" | LIGHT | `@oc-explorer` finds and reports, orch ticks plan |

The orchestrator never reads source files or writes code — it only classifies, routes, synthesizes, and tracks plan lifecycle.
