# Claude-GPT Workflow Skills

[中文](./README_zh.md) | English

A collection of Claude Code skills for Claude + GPT automatic collaboration workflows.

## Skills

### 1. [codex](./codex/)
Delegates coding tasks to Codex CLI for execution. CodeX is a cost-effective, strong coder — great for batch refactoring, code generation, multi-file changes, and multi-turn implementation tasks.

Based on [oil-oil/codex](https://github.com/oil-oil/codex).

**Usage:**
```bash
~/.claude/skills/codex/scripts/ask_codex.sh "Your request"
```

### 2. [plan-review](./plan-review/)
Reviews a technical plan via Codex and iteratively refines it. Uses adversarial review to improve plan quality before implementation.

**Trigger:** `/plan-review <plan-file-path>`

### 3. [plan-execute](./plan-execute/)
Executes a finalized plan by delegating coding to Codex. Claude orchestrates, Codex codes, Claude reviews, Codex fixes — iterating until quality passes.

**Trigger:** `/plan-execute <plan-file-path>`

> **Note**
>
> After Claude enters `plan` mode, plan files are stored under `~/.claude` by default. You can change the plan storage directory in `~/.claude/settings.json`; for example, the config below stores plans in the current project's `./plans` directory:
>
> ```json
> {
>   "env": {},
>   "plansDirectory": "./plans"
> }
> ```

## Installation

### Option 1: npx add-skill (Recommended)

**Prerequisite:** Install the add-skill CLI first:
```bash
npm install -g add-skill
```

Then install the skills:
```bash
npx add-skill longranger2/claude-gpt-workflow
```

### Option 2: Per-skill installation

Install individual skills separately:
```bash
npx add-skill longranger2/claude-gpt-workflow/plan-review
npx add-skill longranger2/claude-gpt-workflow/plan-execute
npx add-skill longranger2/claude-gpt-workflow/codex
```

### Option 3: Manual installation

Copy skills to your Claude Code skills directory:
```bash
cp -r plan-review/ ~/.claude/skills/
cp -r plan-execute/ ~/.claude/skills/
cp -r codex/ ~/.claude/skills/
```

## Workflow

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'clusterBkg': 'transparent', 'background': 'transparent', 'primaryColor': '#FFF7E6', 'primaryTextColor': '#5A3A00', 'primaryBorderColor': '#F59E0B', 'lineColor': '#6366F1', 'secondaryColor': '#E8F5FF', 'tertiaryColor': '#F3E8FF'}}}%%
flowchart TB
  %% ---------- Styles ----------
  classDef user fill:#E8F5FF,stroke:#1B6EF3,stroke-width:2px,color:#0B2A5B,stroke-dasharray: 0;
  classDef claude fill:#FFF7E6,stroke:#F59E0B,stroke-width:2px,color:#5A3A00,stroke-dasharray: 0;
  classDef codex fill:#F3E8FF,stroke:#7C3AED,stroke-width:2px,color:#2E1065,stroke-dasharray: 0;
  classDef decision fill:#FFFFFF,stroke:#6366F1,stroke-width:2px,color:#111827,stroke-dasharray: 5 5;

  %% ---------- Phase 1: Plan Review ----------
  subgraph P1["Phase 1: Plan Review (Adversarial Iteration)"]
    direction TB
    U1["User: Submit Plan"]:::user
    C1["Claude: Delegate to Codex for Critical Review"]:::claude
    X1["Codex: Output Review (10+ issues, Critical/High/Medium/Low)"]:::codex
    C2["Claude: Evaluate & Refine Plan"]:::claude
    D1{"Status?"}:::decision

    U1 --> C1 --> X1 --> C2 --> D1
    D1 -- "NEEDS_REVISION (Loop A)" --> C1
  end

  %% ---------- Phase 2: Plan Execute ----------
  subgraph P2["Phase 2: Plan Execute (Orchestrated Implementation)"]
    direction TB
    C3["Claude: Dispatch batch to Codex for implementation"]:::claude
    X2["Codex: Implement code changes (Executor)"]:::codex
    C4["Claude: Read-only Code Review (Alignment / Quality / Build)"]:::claude
    C5["Claude: Write Review (issue list)"]:::claude
    D2{"Verdict?"}:::decision
    D3{"More steps remaining?"}:::decision
    Done([Complete]):::decision

    C3 --> X2 --> C4 --> C5 --> D2
    D2 -- "NEEDS_FIX (Loop B)" --> X2
    D2 -- "APPROVED" --> D3
    D3 -- "YES (Loop C: next batch)" --> C3
    D3 -- "NO" --> Done
  end

  %% ---------- Phase Transition ----------
  D1 -- "APPROVED / MOSTLY_GOOD" --> C3
```

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Adversarial** | Codex acts as a "nitpicker", not an assistant — its job is to find flaws |
| **Iterative** | Not one-shot; multiple rounds of back-and-forth until quality gates pass |
| **Role Separation** | User defines what, Claude orchestrates how, Codex executes |
| **Feedback Loops** | Review → Fix → Re-review cycles (Loops A, B, C) |

### The Three Loops

- **Loop A (Plan Refinement)**: Review finds issues → Refine plan → Re-review → ... → APPROVED
- **Loop B (Code Fixing)**: Code review finds bugs → Codex fixes → Re-review → ... → APPROVED  
- **Loop C (Batch Processing)**: Complete current batch → Next batch → ... → All done

## License

MIT
