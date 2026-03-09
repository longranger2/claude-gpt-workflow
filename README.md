# Claude-GPT Workflow Skills

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

```
User → plan-review → Plan iteration → APPROVED
                                    ↓
                            plan-execute → Codex (GPT) → Code Review → Fix iteration
                                    ↓
                                Complete
```

## License

MIT
