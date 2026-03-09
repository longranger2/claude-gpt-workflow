---
name: plan-review
description: Review a technical plan via Codex and iteratively refine it. Use when the user says "/plan-review", "plan review", "review方案", "方案review", "PRD review", "review一下方案", "评审方案". Takes a plan file path as argument, delegates critical review to Codex, then refines the plan based on feedback.
---

# Plan Review Skill

## Purpose

当用户输入 `/plan-review {plan文件路径}` 时，启动"方案对抗式迭代"流程：
1. 我（Claude Code）调用 Codex 对指定方案进行批判性 review
2. 我阅读 Codex 产出的 review，评估其建议的合理性
3. 我基于合理的建议修改方案，更新回原 plan 文件
4. 如 review 状态为 NEEDS_REVISION，自动再次调用 Codex review
5. 循环直到达成共识（MOSTLY_GOOD 或 APPROVED）

## Usage

```
/plan-review plans/my-feature-plan.md
```

## Session 复用

每次调用 Codex 后，从脚本输出中提取 `session_id=xxx`，保存为当前任务的 session ID。在同一任务的后续 Codex 调用中，通过 `--session <id>` 传入以复用上下文，让 Codex 记住之前的 review 历史，提升多轮 review 的效率和一致性。

## My Workflow (Claude Code)

### Step 1: 确定 review 文件

从 plan 文件名派生 review 文件路径：
- `plans/auth-refactor.md` → `reviews/auth-refactor-review.md`
- 规则：`reviews/{plan文件名去掉.md}-review.md`

如果 review 文件已存在，说明不是第一轮，Codex 需要追踪上轮 issue 修复状态。

### Step 2: 调用 Codex Review

使用 `/codex` skill，给 Codex 以下指令：

```
读取 {plan文件路径} 的内容，作为独立第三方审稿人进行批判性 review。

要求：
- 至少提出 10 个具体可操作的改进点
- 每个 issue 包含：问题描述 + 方案中的具体位置/引用 + 改进建议
- 按严重程度分级：Critical > High > Medium > Low > Suggestion
- 如果 {review文件路径} 已存在，先读取它，在新轮次中追踪上轮 issue 的修复状态

分析维度（根据方案类型选择适用项）：
- 架构合理性：过度设计 vs 设计不足、模块边界、职责单一
- 技术选型：选型依据、替代方案对比、与项目现有栈的兼容性
- 完整性：遗漏的场景、未考虑的边界情况、依赖和影响范围
- 可行性：实现复杂度、性能隐患、迁移/兼容性风险
- 工程质量：是否符合项目 CLAUDE.md 中的 Code Quality Hard Limits
- 用户体验：交互流程、错误/加载状态、i18n（如适用）
- 安全性：认证授权、数据校验（如适用）

将本轮 review 追加到 {review文件路径} 中（如文件不存在则创建）。
每轮用 --- 分隔，新轮次追加在文件末尾。格式如下：

---

## Round {N} — {YYYY-MM-DD}

### Overall Assessment
{2-3 句总体评价}
**Rating**: {X}/10

### Previous Round Tracking (R2+ only)
| # | Issue | Status | Notes |
|---|-------|--------|-------|

### Issues
#### Issue 1 ({severity}): {标题}
**Location**: {方案中的位置}
{问题描述}
**Suggestion**: {改进建议}
...（至少 10 个）

### Positive Aspects
- ...

### Summary
{Top 3 关键问题}
**Consensus Status**: NEEDS_REVISION / MOSTLY_GOOD / APPROVED

关键原则：做挑刺的人，不做 yes-man。每个 issue 必须具体到让人知道怎么改。
```

review 文件首次创建时，在文件开头添加 header：

```markdown
# Plan Review: {方案标题}

**Plan File**: {plan文件路径}
**Reviewer**: Codex
```

### Step 3: 阅读 Review 并修改方案

Codex 完成后，我读取 review 文件中最新一轮的内容：

1. **逐条评估** Codex 提出的每个 issue
2. **采纳合理建议**，修改 plan 文件内容
3. **拒绝不合理建议**时，在 plan 中简要注明理由（可选）
4. **更新原 plan 文件**（不创建新文件，直接修改原路径）

### Step 4: 判断是否继续迭代

根据 Codex 给出的 `Consensus Status`：

| Status | 我的行动 |
|--------|---------|
| `NEEDS_REVISION` | 修改方案后，自动再次调用 Codex review（回到 Step 2） |
| `MOSTLY_GOOD` | 修改方案后，告知用户方案基本成熟，询问是否需要再 review |
| `APPROVED` | 告知用户方案已通过 review，可以进入实施阶段 |

### Step 5: 收尾

迭代完成后，向用户汇报：
- 经历了几轮 review
- 主要改进了哪些方面
- 最终方案文件路径
- review 记录文件路径

## File Convention

- 单个 plan 对应单个 review 文件：`reviews/{topic}-review.md`
- `{topic}` 取 plan 文件名（去掉 `.md`）
- 所有轮次追加在同一文件中，用 `---` 分隔
- 示例：`plans/auth-refactor.md` → `reviews/auth-refactor-review.md`
