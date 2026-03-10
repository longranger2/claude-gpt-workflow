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

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'background': 'transparent', 'primaryColor': '#FFF7E6', 'primaryTextColor': '#5A3A00', 'primaryBorderColor': '#F59E0B', 'lineColor': '#6366F1', 'secondaryColor': '#E8F5FF', 'tertiaryColor': '#F3E8FF'}}}%%
flowchart TB
  %% ---------- 样式 ----------
  classDef user fill:#E8F5FF,stroke:#1B6EF3,stroke-width:2px,color:#0B2A5B,stroke-dasharray: 0;
  classDef claude fill:#FFF7E6,stroke:#F59E0B,stroke-width:2px,color:#5A3A00,stroke-dasharray: 0;
  classDef codex fill:#F3E8FF,stroke:#7C3AED,stroke-width:2px,color:#2E1065,stroke-dasharray: 0;
  classDef decision fill:#FFFFFF,stroke:#6366F1,stroke-width:2px,color:#111827,stroke-dasharray: 5 5;

  %% ---------- Phase 1: 方案评审 ----------
  subgraph P1["Phase 1: Plan Review（方案对抗式迭代）"]
    direction TB
    U1["User: 提交 Plan"]:::user
    C1["Claude: 将 Plan 交给 Codex 进行批判性 Review"]:::claude
    X1["Codex: 输出 Review（10+ 个 issue，Critical/High/Medium/Low）"]:::codex
    C2["Claude: 逐条评估并优化 Plan"]:::claude
    D1{"Status?"}:::decision

    U1 --> C1 --> X1 --> C2 --> D1
    D1 -- "NEEDS_REVISION（循环 A）" --> C1
  end

  %% ---------- Phase 2: 方案执行 ----------
  subgraph P2["Phase 2: Plan Execute（编排式代码实现）"]
    direction TB
    C3["Claude: 按批次拆分，派发本批次给 Codex 实现"]:::claude
    X2["Codex: 实现代码变更（执行者）"]:::codex
    C4["Claude: 只读 Code Review（对齐 Plan / 质量 / Build）"]:::claude
    C5["Claude: 写入 Review（issue 列表）"]:::claude
    D2{"Verdict?"}:::decision
    D3{"还有未完成步骤?"}:::decision
    Done([Complete]):::decision

    C3 --> X2 --> C4 --> C5 --> D2
    D2 -- "NEEDS_FIX（循环 B）" --> X2
    D2 -- "APPROVED" --> D3
    D3 -- "YES（循环 C：下一批次）" --> C3
    D3 -- "NO" --> Done
  end

  %% ---------- 阶段转换 ----------
  D1 -- "APPROVED / MOSTLY_GOOD" --> C3
```

### 核心概念

| 概念 | 说明 |
|------|------|
| **对抗性 (Adversarial)** | Codex 不是助手而是"挑刺者"——它的工作是找毛病 |
| **迭代性 (Iterative)** | 不是一次通过，而是多轮往返直到质量门通过 |
| **角色分离 (Role Separation)** | User 定义做什么，Claude 编排怎么做，Codex 执行 |
| **反馈闭环 (Feedback Loops)** | Review → Fix → Re-review 的循环（循环 A、B、C）|

### 三个循环

- **循环 A（方案精修）**: Review 发现问题 → 优化方案 → 再 Review → ... → APPROVED
- **循环 B（代码修复）**: Code Review 发现 bug → Codex 修复 → 再 Review → ... → APPROVED
- **循环 C（批次处理）**: 完成当前批次 → 下一批次 → ... → 全部完成

## License

MIT
