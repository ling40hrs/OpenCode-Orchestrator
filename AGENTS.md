# OC-Orchestrator Project

This project is a **standalone** general-purpose coding orchestrator for OpenCode using Go models. It is fully self-contained — copy the folder, open it in OpenCode, and it works. No global config or external dependencies needed.

## Agents

### Primary Agent: `orchestrator` (default)
Uses DeepSeek V4 Flash for **cheap routing**. Classifies tasks as LIGHT/MEDIUM/HEAVY/DESIGN, then delegates to the right subagent. For complex tasks, escalates to `@oc-planner` (V4 Pro) for deep architectural thinking. Never writes code or reads source files itself. Switch with Tab key.

### Subagents (invoke with @mention or Task tool)

| Agent | Model | Purpose | Cost |
|---|---|---|---|
| `@oc-planner` | DeepSeek V4 Pro | Deep-thinking planner for complex multi-file tasks | ~3.4K req/5hr |
| `@oc-explorer` | DeepSeek V4 Flash | Read-only codebase exploration — structure only, no full dumps | ~31K req/5hr |
| `@oc-writer` | MiniMax M2.5 | Implementation and file edits | ~6K req/5hr |
| `@oc-reviewer` | MiniMax M2.5 | Code review and quality validation | ~6K req/5hr |
| `@oc-tester` | DeepSeek V4 Flash | Test writing and execution | ~31K req/5hr |
| `@oc-designer` | Kimi K2.6 | Frontend design PLANNER — creates design plans only, does NOT implement | ~1.1K req/5hr |
| `@ask` | DeepSeek V4 Flash | Q&A about codebase, explain code, answer questions | ~31K req/5hr |

## How to Use

1. Open OpenCode in this directory — the orchestrator loads by default
2. Describe a coding task naturally — even a short sentence is fine:
   ```
   Add login to the homepage
   ```
3. The orchestrator will **improve your prompt**, show you the refined version, and ask for confirmation
4. Confirm (or edit) — then the orchestrator delegates, validates, and reports

## Execution Flow

```
You: "do a full plan and initialization on how the ui should look like"

Phase 0: Prompt Improvement       ← Flash refines and confirms
Phase 1: Classify                 ← Flash labels it DESIGN (from title only)
Phase 2: Plan file                ← Write .ai-plans/ + approval gate
Phase 3: Scan                     ← task(oc-explorer, "scan structure only")
Phase 4: Plan                     ← task(oc-designer, "report: <scan>")
                                       │
                                       ▼
                                  oc-designer (Kimi) creates DESIGN PLAN
                                  • No codebase reading (read: deny)
                                  • May spawn oc-explorer for specific file details
                                  • Outputs: visual direction, component map, file list
                                       │
                                       ▼
Phase 5: Delegate                 ← task(oc-writer, "implement plan")
                                  → task(oc-tester) → task(oc-reviewer)
Phase 6-8: Synthesize → Validate → Report
```

## Enforced Rules

### AI Planning Convention (`.ai-plans/`)
If the project has a `.ai-plans/` directory, the orchestrator follows a structured planning workflow: check active plans → user approval gate (for features >1 file) → write plan → tick steps → close. Every code-changing task gets a plan file.

### 200-Line Cap (Web Dev)
- All web frontend files (.tsx, .jsx, .vue, .svelte, .css, .scss, .html) have a **hard 200-line cap**
- Files exceeding this must be split into reusable components, hooks, or utility modules
- This makes bugs easier to isolate and fix

### Prompt Improvement Loop
- Every user prompt is refined for clarity and specificity before any work begins
- The refined prompt is shown to you for confirmation
- You can edit and re-confirm until it's right
- Only then does the orchestrator proceed with delegation

## Model Cost Strategy

The orchestrator (DeepSeek V4 Flash, ~31K req/5hr) is cheap and handles task routing. DeepSeek V4 Pro (~3.4K req/5hr) is used ONLY for complex multi-file planning via `@oc-planner`. All actual work is done by MiniMax M2.5 (~6K req/5hr) and V4 Flash. Kimi K2.6 (~1.1K req/5hr) is reserved strictly for creative frontend design planning.

---

## Installation: Setting Up on a New OpenCode Machine

### What an AI agent must do to install this on a new machine

The OC-Orchestrator works in TWO modes:
1. **Project mode**: Open the folder → works immediately (subagents are local)
2. **Global mode**: Copy agents to global config → orchestrator available in every project

### Mode 1: Project-only (simplest, recommended for sharing)

```
1. Copy the OC-Orchestrator/ folder to the target machine
2. Open it in OpenCode (desktop app or CLI)
3. The orchestrator loads automatically via opencode.json:
   - default_agent: orchestrator
   - model: opencode-go/deepseek-v4-flash
   - All 8 agents are in .opencode/agents/ — no global setup needed
4. Connect to OpenCode Go via `/connect` for model access
5. Done
```

### Mode 2: Global install (orchestrator available in ALL projects)

An AI agent must perform these steps:

```
STEP 1: Create global config directories
  mkdir -p ~/.config/opencode/agents
  mkdir -p ~/.config/opencode/skills/orchestrator-delegation
  mkdir -p ~/.config/opencode/skills/frontend-design

STEP 2: Copy all agent files from the repo to global
  cp .opencode/agents/*.md ~/.config/opencode/agents/
  (Copies: orchestrator.md, oc-planner.md, oc-explorer.md,
           oc-writer.md, oc-reviewer.md, oc-tester.md,
           oc-designer.md, ask.md)

STEP 3: Copy skill files
  cp .opencode/skills/orchestrator-delegation/SKILL.md \
     ~/.config/opencode/skills/orchestrator-delegation/SKILL.md
  cp .opencode/skills/frontend-design/SKILL.md \
     ~/.config/opencode/skills/frontend-design/SKILL.md

STEP 4: Create or update global opencode.json
  File: ~/.config/opencode/opencode.json
  Content: {
    "$schema": "https://opencode.ai/config.json",
    "default_agent": "orchestrator",
    "model": "opencode-go/deepseek-v4-flash",
    "provider": { "opencode-go": {} },
    "permission": { "*": "ask", "webfetch": "allow" },
    "agent": {
      "build": {
        "permission": {
          "task": { "oc-*": "deny", "plan": "deny", "*": "allow" }
        }
      }
    }
  }

STEP 5: Verify installation
  - Open OpenCode in any project
  - The orchestrator should be the default agent (Tab to switch)
  - Type /models — ensure opencode-go provider is listed
  - Connect OpenCode Go via /connect if not already connected

STEP 6: Verify subagents are hidden from non-orchestrator agents
  - oc-explorer, oc-writer, oc-reviewer, oc-tester, oc-designer, oc-planner
    all have hidden: true — they won't appear in @mention for the build agent
  - The orchestrator can still spawn them via task()
  - The build agent's task permission denies oc-* and plan
```

### Files you need to copy (inventory for AI agents)

```
Required files for GLOBAL install:
  ~/.config/opencode/
  ├── opencode.json                    # Config (create from template above)
  ├── agents/
  │   ├── orchestrator.md              # PRIMARY — must be present
  │   ├── oc-planner.md                # Complex planner (V4 Pro)
  │   ├── oc-explorer.md               # Codebase scanner (V4 Flash)
  │   ├── oc-writer.md                 # Implementer (M2.5)
  │   ├── oc-reviewer.md               # Reviewer (M2.5)
  │   ├── oc-tester.md                 # Tester (V4 Flash)
  │   ├── oc-designer.md               # Design planner (Kimi K2.6)
  │   └── ask.md                       # Q&A (V4 Flash)
  └── skills/
      ├── orchestrator-delegation/SKILL.md  # Delegation strategies
      └── frontend-design/SKILL.md          # Design guidelines for Kimi
```

### Troubleshooting for AI agents

```
Problem: Orchestrator not loading as default
  Fix: Check ~/.config/opencode/opencode.json has default_agent: "orchestrator"
  Fix: Check orchestrator.md exists in ~/.config/opencode/agents/

Problem: Subagents not spawning when orchestrator calls task()
  Fix: Check orchestrator.md frontmatter has permission.task: { "oc-*": "allow" }
  Fix: Check each subagent .md file exists in ~/.config/opencode/agents/

Problem: Build agent can see oc-* subagents
  Fix: Check each subagent .md has hidden: true in frontmatter
  Fix: Check opencode.json has agent.build.permission.task: { "oc-*": "deny" }

Problem: Kimi (oc-designer) tries to read source files
  Fix: Check oc-designer.md has read: deny, glob: deny, grep: deny
  Fix: Check oc-designer.md prompt says "work from reports only"

Problem: oc-designer doesn't have the frontend-design skill
  Fix: Check .opencode/skills/frontend-design/SKILL.md (or global) exists
  Fix: Check oc-designer prompt calls skill({ name: "frontend-design" })
```

---

## Project Structure (Standalone)
```
OC-Orchestrator/
├── opencode.json              # Config: default agent, model, permissions
├── AGENTS.md                  # This file — project instructions
├── docs/
│   ├── architecture-for-humans.md   # Human-readable architecture diagram
│   └── architecture-for-agents.md   # Machine-parseable architecture reference
├── .opencode/
│   ├── agents/
│   │   ├── orchestrator.md    # Primary orchestrator agent (Flash)
│   │   ├── oc-planner.md      # Complex task planner (V4 Pro)
│   │   ├── oc-explorer.md     # Codebase explorer subagent
│   │   ├── oc-writer.md       # Implementation subagent
│   │   ├── oc-reviewer.md     # Code review subagent
│   │   ├── oc-tester.md       # Testing subagent
│   │   ├── oc-designer.md     # Frontend design planner (Kimi)
│   │   └── ask.md             # Q&A subagent
│   └── skills/
│       ├── orchestrator-delegation/SKILL.md  # Delegation strategies
│       └── frontend-design/SKILL.md          # Design guidelines
```

To share: zip this folder → send to anyone → they open in OpenCode → it works immediately.
