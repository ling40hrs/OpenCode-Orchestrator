<div align="center">
  <img src="https://img.shields.io/badge/status-active--development-2ea44f?style=for-the-badge" alt="Status" />
  <img src="https://img.shields.io/badge/license-MIT-blue?style=for-the-badge" alt="License" />
  <img src="https://img.shields.io/badge/OpenCode-Go-brightgreen?style=for-the-badge" alt="OpenCode Go" />
  <img src="https://img.shields.io/badge/agents-8-purple?style=for-the-badge" alt="8 Agents" />
  <img src="https://img.shields.io/badge/models-4-important?style=for-the-badge" alt="4 Models" />
  <br/>
  <img src="https://img.shields.io/badge/V4%20Flash-31K%20req%2F5hr-lightgrey?style=flat-square" alt="Flash" />
  <img src="https://img.shields.io/badge/V4%20Pro-3.4K%20req%2F5hr-lightgrey?style=flat-square" alt="Pro" />
  <img src="https://img.shields.io/badge/MiniMax%20M2.5-6.3K%20req%2F5hr-lightgrey?style=flat-square" alt="M2.5" />
  <img src="https://img.shields.io/badge/Kimi%20K2.6-1.1K%20req%2F5hr-lightgrey?style=flat-square" alt="K2.6" />
</div>

<br/>

<div align="center">
  <h1>⚡ OC-Orchestrator</h1>
  <p><strong>Intelligent coding agent router for OpenCode — powered by open-source Go models</strong></p>
  <p>Decompose → Delegate → Synthesize → Validate</p>
</div>

<br/>

<div align="center">
  <table>
    <tr>
      <td align="center">🎯 <b>Smart Routing</b><br/><sub>Classifies tasks as LIGHT, HEAVY, or DESIGN</sub></td>
      <td align="center">🧩 <b>8 Specialized Agents</b><br/><sub>Each with a purpose-built model</sub></td>
      <td align="center">💰 <b>Cost Optimized</b><br/><sub>Flash for routing, Pro for thinking</sub></td>
    </tr>
    <tr>
      <td align="center">📋 <b>Plan Convention</b><br/><sub>Every task gets a documented plan</sub></td>
      <td align="center">🔄 <b>Resume & Recall</b><br/><sub>Pick up where you left off</sub></td>
      <td align="center">📏 <b>200-Line Cap</b><br/><sub>Clean, maintainable files</sub></td>
    </tr>
  </table>
</div>

---

## 🚀 Overview

OC-Orchestrator is a **general-purpose coding orchestrator** for [OpenCode](https://opencode.ai). Instead of a single AI model doing everything, it routes your task to the right specialist — saving money, improving quality, and keeping your codebase clean.

Built entirely on open-source **OpenCode Go models** — no proprietary API keys required.

### How it works

```
You describe a task → Orchestrator classifies it → Delegates to specialists → Validates → Reports
```

| You say | Orchestrator routes | Models used |
|---|---|---|
| "Change this button to blue" | **LIGHT** → `@oc-writer` directly | Flash → M2.5 |
| "Fix the broken checkout (5 files)" | **HEAVY** → `@oc-planner` plans → workers execute | Flash → V4 Pro → M2.5/Flash |
| "Design the dashboard with animations" | **DESIGN** → `@oc-designer` plans → `@oc-writer` builds | Flash → Kimi K2.6 → M2.5 |
| "How does auth work?" | **LIGHT** → `@ask` explains | Flash |
| "Add tests for the cart" | **MEDIUM** → `@oc-tester` | Flash |

---

## 🧠 Agent Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    PRIMARY AGENT (Tab-switchable)                          │
│                                                                            │
│   orchestrator (V4 Flash ~31K req/5hr)  ← YOU TALK TO THIS ONE            │
│   ┌──────────────────────────────────────────────────────────────────┐   │
│   │ CLASSIFIES first: LIGHT | MEDIUM | HEAVY | DESIGN                │   │
│   │ NEVER reads source code — delegates scanning to oc-explorer       │   │
│   │ NEVER writes code — delegates implementation to oc-writer         │   │
│   │ Tracks task_ids for resume/recall                                 │   │
│   └──────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┬───────────────────┐
         │                    │                    │                   │
         ▼                    ▼                    ▼                   ▼
   ┌──────────┐    ┌──────────────────┐    ┌──────────────┐    ┌───────────────┐
   │  LIGHT   │    │     MEDIUM       │    │    HEAVY     │    │   DESIGN      │
   │  ─────── │    │     ──────       │    │    ─────     │    │   ──────      │
   │  Writer  │    │  Writer or       │    │  Planner     │    │  Explorer     │
   │  Direct  │    │  optional Plan   │    │  → Writer    │    │  → Designer   │
   └──────────┘    └──────────────────┘    │  → Tester    │    │  → Writer     │
                                           │  → Reviewer  │    │  → Tester     │
                                           └──────────────┘    │  → Reviewer   │
                                                               └───────────────┘
```

### The Agents

| Agent | Model | Cost | Role |
|---|---|---|---|
| **`orchestrator`** | DeepSeek V4 Flash | ~31K req/5hr | Routes, classifies, delegates |
| **`oc-planner`** | DeepSeek V4 Pro | ~3.4K req/5hr | Deep architectural thinking |
| **`oc-explorer`** | DeepSeek V4 Flash | ~31K req/5hr | Structural codebase scanning |
| **`oc-writer`** | MiniMax M2.5 | ~6K req/5hr | Implementation & file edits |
| **`oc-reviewer`** | MiniMax M2.5 | ~6K req/5hr | Code review & validation |
| **`oc-tester`** | DeepSeek V4 Flash | ~31K req/5hr | Test writing & execution |
| **`oc-designer`** | Kimi K2.6 | ~1.1K req/5hr | Creative frontend design planning |
| **`ask`** | DeepSeek V4 Flash | ~31K req/5hr | Q&A & codebase explanation |

---

## ✨ Key Features

### 🎯 Smart Task Routing
The orchestrator classifies your task from the description alone — no source code reading needed. **DESIGN tasks take priority** over HEAVY tasks, so a UI redesign with animations goes to Kimi, not to the generic planner.

### 💰 Cost-Optimized Model Strategy
- **V4 Flash** (~31K req/5hr): Cheap routing, cheap scanning, cheap testing — use freely
- **V4 Pro** (~3.4K req/5hr): Only fires for complex multi-file planning
- **MiniMax M2.5** (~6K req/5hr): Balanced for code writing and review
- **Kimi K2.6** (~1.1K req/5hr): Reserved strictly for creative frontend design

### 📋 AI Planning Convention
Every task — even a one-line change — gets a documented plan in `.ai-plans/active/`. Plans use a standardized YAML format, are ticked step-by-step, and move to `done/` or `abandoned/` on completion.

### 📏 200-Line Cap
All web frontend files enforce a hard 200-line limit. Files exceeding this are split into hooks, sub-components, utility modules, and CSS partials. This makes bugs easier to locate, code changes less risky, and reviews faster.

### 🔄 Resume & Recall
If you stop midway, the orchestrator stores each subagent's `task_id`. When you resume, it picks up the exact session. If a subagent fails, a completely fresh session is spawned — no corrupted state reused.

### 🚫 Orchestrator Never Writes Code
The orchestrator is a **manager, not an engineer**. It creates plan files, routes reports, and synthesizes results — but never reads source files, writes code, or runs build commands. All implementation goes to subagents.

---

## 🏗️ Installation

### Prerequisites
- [OpenCode](https://opencode.ai) (desktop app, CLI, or IDE extension)
- [OpenCode Go](https://opencode.ai/go) subscription ($5 first month, then $10/month)
- Connect via `/connect` in the OpenCode terminal or settings

### Option 1: Project-only (recommended for sharing)

```bash
# Clone or copy the OC-Orchestrator folder
git clone https://github.com/your-org/oc-orchestrator.git
cd oc-orchestrator

# Open in OpenCode
opencode
```

Opens with the orchestrator loaded automatically. All 8 agents are local to the project.

### Option 2: Global install (available in every project)

An AI agent can perform this install:

```bash
# 1. Create global directories
mkdir -p ~/.config/opencode/agents
mkdir -p ~/.config/opencode/skills/orchestrator-delegation
mkdir -p ~/.config/opencode/skills/frontend-design

# 2. Copy agent files
cp .opencode/agents/*.md ~/.config/opencode/agents/

# 3. Copy skill files
cp .opencode/skills/orchestrator-delegation/SKILL.md \
   ~/.config/opencode/skills/orchestrator-delegation/SKILL.md
cp .opencode/skills/frontend-design/SKILL.md \
   ~/.config/opencode/skills/frontend-design/SKILL.md

# 4. Create global config (see opencode.json for reference)
```

---

## 🗺️ Project Structure

```
OC-Orchestrator/
├── opencode.json                    # Config: default agent, model, permissions
├── AGENTS.md                        # AI agent onboarding instructions
├── README.md                        # ← You are here
├── docs/
│   ├── architecture-for-humans.md   # Human-readable architecture (with flowcharts)
│   └── architecture-for-agents.md   # Machine-parseable reference for AI
├── .opencode/
│   ├── agents/
│   │   ├── orchestrator.md          # Primary: Flash router
│   │   ├── oc-planner.md            # V4 Pro: complex task planning
│   │   ├── oc-explorer.md           # V4 Flash: structural scanning
│   │   ├── oc-writer.md             # M2.5: implementation
│   │   ├── oc-reviewer.md           # M2.5: code review
│   │   ├── oc-tester.md             # V4 Flash: testing
│   │   ├── oc-designer.md           # Kimi K2.6: design planning
│   │   └── ask.md                   # V4 Flash: Q&A
│   └── skills/
│       ├── orchestrator-delegation/ # Delegation strategies
│       └── frontend-design/         # Creative design guidelines
```

---

## 🔧 Usage Examples

### Simple Edit
```
User: "Change the primary button color to teal"
  → orchestrator classifies: LIGHT
  → task(oc-writer, "update CSS variable --color-primary to #0d9488")
  → oc-writer edits file, ticks plan [x]
  → Done: 1 file changed
```

### Multi-file Bug Fix
```
User: "Fix the checkout flow — 5 files affected"
  → orchestrator classifies: HEAVY
  → User Approval Gate: 3 options presented → user picks
  → task(oc-explorer, "scan the checkout files")
  → task(oc-planner, "plan from scan report")
  → task(oc-writer, "impl step 1") → task(oc-tester) → task(oc-reviewer)
  → Plan moved to .ai-plans/done/
```

### Full UI Redesign
```
User: "do a full plan and initialization on how the ui should look like and the animations"
  → orchestrator classifies: DESIGN (priority over HEAVY)
  → Guardrails check: ALL pass
  → task(oc-explorer, "scan UI components, styles, imports")
  → task(oc-designer, "report: <scan> — create design plan")
  → Kimi writes plan to .ai-plans/active/ with colors, typography, animations
  → task(oc-writer, "implement design plan") → tester → reviewer
```

---

## 🧪 Execution Flow

```
User input
    │
    ▼
┌─────────────────────────────────────────────┐
│ Phase 0: Prompt Improvement                  │
│   "I'll refine that... Does this match?"     │
│   User confirms → proceed                    │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│ Phase 1: Classify (Complexity Gate)          │
│   From task description alone (no source)    │
│   → DESIGN | HEAVY | MEDIUM | LIGHT          │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│ Phase 2: Write Plan File                     │
│   → .ai-plans/active/<category>-<target>.md  │
│   DESIGN/HEAVY: User Approval Gate (options) │
│   LIGHT: Minimal plan directly               │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│ Phase 3: Scan (if needed)                    │
│   task(oc-explorer, "structure only")       │
│   → Returns file tree, imports, hierarchy    │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│ Phase 4: Plan                                │
│   DESIGN: task(oc-designer) → creates plan   │
│   HEAVY:  task(oc-planner) → creates plan    │
│   LIGHT:  Skip — brief plan shown to user    │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│ Phase 5: Execute (via Task tool)             │
│   Each subagent ticks its own plan step [x]  │
│   task_id stored for resume/recall           │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│ Phase 6-8: Synthesize → Validate → Report    │
│   Verify plan ticks, check quality           │
│   Move plan to .ai-plans/done/ on success    │
└─────────────────────────────────────────────┘
```

---

## 🤝 Contributing

All agents are defined as markdown files in `.opencode/agents/`. Each agent has:
- **Frontmatter**: Model, permissions, temperature, mode
- **System prompt**: Instructions that define its behavior

To add a new agent:
1. Create `.opencode/agents/<name>.md` with frontmatter + prompt
2. Add task permission in `orchestrator.md` (`"oc-*": allow` covers all)
3. Add routing logic in the delegation decision tree
4. Add to `AGENTS.md` and docs

---

## 📄 License

MIT — use freely, modify, share. Built for [OpenCode](https://opencode.ai) with ❤️.

---

<div align="center">
  <sub>Powered by <a href="https://opencode.ai/go">OpenCode Go</a> — DeepSeek V4 Flash, DeepSeek V4 Pro, MiniMax M2.5, Kimi K2.6</sub>
  <br/>
  <sub>Inspired by <a href="https://www.anthropic.com/engineering/building-effective-agents">Anthropic's Building Effective Agents</a></sub>
</div>
