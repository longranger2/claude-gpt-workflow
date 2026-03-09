# Claude-GPT 工作流 Skills

[English](./README.md) | 中文

Claude Code Skills 集合，实现 Claude 与 GPT (Codex) 自动协作工作流。

## Skills

### 1. [codex](./codex/)
将编码任务委托给 Codex CLI 执行。CodeX 是一个成本效益高、能力强的 coder — 非常适合批量重构、代码生成、多文件修改和多轮实现任务。

基于 [oil-oil/codex](https://github.com/oil-oil/codex)。

**用法：**
```bash
~/.claude/skills/codex/scripts/ask_codex.sh "Your request"
```

### 2. [plan-review](./plan-review/)
通过 Codex 对技术方案进行评审并迭代优化。在实施前使用对抗式评审提升方案质量。

**触发：** `/plan-review <plan-file-path>`

### 3. [plan-execute](./plan-execute/)
通过委托 Codex 执行来实施最终方案。Claude 编排、Codex 编码、Claude 评审、Codex 修复 — 迭代直到质量达标。

**触发：** `/plan-execute <plan-file-path>`

## 安装

### 方式一：npx add-skill（推荐）

**前置条件：** 先安装 add-skill CLI：
```bash
npm install -g add-skill
```

然后安装 skills：
```bash
npx add-skill longranger2/claude-gpt-workflow
```

### 方式二：单独安装

单独安装各个 skill：
```bash
npx add-skill longranger2/claude-gpt-workflow/plan-review
npx add-skill longranger2/claude-gpt-workflow/plan-execute
npx add-skill longranger2/claude-gpt-workflow/codex
```

### 方式三：手动安装

复制 skills 到你的 Claude Code skills 目录：
```bash
cp -r plan-review/ ~/.claude/skills/
cp -r plan-execute/ ~/.claude/skills/
cp -r codex/ ~/.claude/skills/
```

## 工作流

```
用户 → plan-review → 方案迭代 → APPROVED
                                  ↓
                      plan-execute → Codex (GPT) → 代码评审 → 修复迭代
                                  ↓
                              完成
```

## License

MIT
