<div align="center">

# OC-Orchestrator

Task router for OpenCode. Classifies your request as LIGHT, MEDIUM, HEAVY, or DESIGN and delegates to a specialist agent.

</div>

---

## How it works

You describe a task. The orchestrator classifies it and routes to the right agent.

| Classification | Routing | Models used |
|---|---|---|
| LIGHT: single file, surface change | `oc-writer` directly | Flash, M2.5 |
| MEDIUM: multi-file, moderate scope | `oc-writer` or optional `oc-planner` | Flash, V4 Pro |
| HEAVY: complex, 3+ files | `oc-planner` plans, workers execute | Flash, V4 Pro, M2.5 |
| DESIGN: UI, animations, frontend | `oc-explorer` scans, `oc-designer` plans | Flash, Kimi K2.6, M2.5 |

---

## Agents

| Agent | Model | Req/5hr | Role |
|---|---|---|---|
| `orchestrator` | DeepSeek V4 Flash | ~31K | Routes, classifies, delegates |
| `oc-planner` | DeepSeek V4 Pro | ~3.4K | Architectural planning |
| `oc-explorer` | DeepSeek V4 Flash | ~31K | Codebase structure scanning |
| `oc-writer` | MiniMax M2.5 | ~6K | Implementation, file edits |
| `oc-reviewer` | MiniMax M2.5 | ~6K | Code review |
| `oc-tester` | DeepSeek V4 Flash | ~31K | Test writing |
| `oc-designer` | Kimi K2.6 | ~1.1K | Frontend design planning |
| `ask` | DeepSeek V4 Flash | ~31K | Q&A, codebase explanation |

---

## Features

**Smart routing** — Classifies tasks from description alone. DESIGN tasks take priority over HEAVY so UI work routes to Kimi, not the generic planner.

**Model cost strategy** — Flash (~31K req) handles routing, scanning, testing. V4 Pro (~3.4K req) fires only for complex multi-file planning. M2.5 (~6K req) balances writing and review. Kimi K2.6 (~1.1K req) reserved for creative frontend.

**Plan convention** — Every task gets a documented plan in `.ai-plans/active/`. YAML format, ticked step-by-step, moved to `done/` on completion.

**200-line cap** — Files exceeding 200 lines are split into hooks, sub-components, utility modules, or CSS partials.

**Resume and recall** — The orchestrator stores each subagent's `task_id`. On resume, it picks up the exact session. On failure, a fresh session is spawned with no corrupted state reused.

**Orchestrator never writes code** — It creates plan files, routes reports, and synthesizes results. It does not read source files, write code, or run build commands.

---

## Execution flow

```
User input
  -> Phase 0: Prompt refinement (confirms with user)
  -> Phase 1: Classify (LIGHT | MEDIUM | HEAVY | DESIGN)
  -> Phase 2: Write plan to .ai-plans/active/
       DESIGN/HEAVY: user approves from options
  -> Phase 3: oc-explorer scans structure (if needed)
  -> Phase 4: oc-designer or oc-planner creates plan
  -> Phase 5: Execute via task tool, oc-writer implements
  -> Phase 6-8: Synthesize, validate, move plan to done/
```

---

## Install

### Project-local (recommended for sharing)

```bash
git clone https://github.com/your-org/oc-orchestrator.git
cd oc-orchestrator
opencode
```

### Global

```bash
mkdir -p ~/.config/opencode/agents ~/.config/opencode/skills/orchestrator-delegation ~/.config/opencode/skills/frontend-design
cp .opencode/agents/*.md ~/.config/opencode/agents/
cp .opencode/skills/orchestrator-delegation/SKILL.md ~/.config/opencode/skills/orchestrator-delegation/
cp .opencode/skills/frontend-design/SKILL.md ~/.config/opencode/skills/frontend-design/
```

---

## Project structure

```
OC-Orchestrator/
├── opencode.json          # Agent config, model, permissions
├── AGENTS.md              # AI onboarding
├── README.md
├── docs/
│   ├── architecture-for-humans.md
│   └── architecture-for-agents.md
└── .opencode/
    ├── agents/
    │   ├── orchestrator.md, oc-planner.md, oc-explorer.md
    │   ├── oc-writer.md, oc-reviewer.md, oc-tester.md
    │   ├── oc-designer.md, ask.md
    └── skills/
        ├── orchestrator-delegation/
        └── frontend-design/
```

---

## Usage

**Simple edit** — "Change the primary button color to teal"
  Classifies LIGHT -> oc-writer edits file -> 1 file changed

**Multi-file fix** — "Fix checkout flow, 5 files affected"
  Classifies HEAVY -> user approves -> oc-explorer scans -> oc-planner plans -> oc-writer implements -> tester -> reviewer

**UI redesign** — "Full plan for UI layout and animations"
  Classifies DESIGN -> oc-explorer scans components -> oc-designer creates plan -> oc-writer implements -> tester -> reviewer

---

## Adding agents

Create `.opencode/agents/<name>.md` with frontmatter (model, permissions, temperature) + system prompt. Add task permission in `orchestrator.md` (`"oc-*": allow`), routing logic, and entry in `AGENTS.md`.

---

## License

MIT
