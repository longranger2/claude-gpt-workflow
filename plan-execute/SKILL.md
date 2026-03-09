---
name: plan-execute
description: Execute a finalized plan by delegating coding to Codex and reviewing the output. Use when the user says "/plan-execute", "plan execute", "执行方案", "执行计划", "开始实施", "implement plan", "execute plan", "开始coding", "按计划执行". Takes a plan file path as argument. Claude orchestrates, Codex codes, Claude reviews, Codex fixes — iterating until quality passes.
---

# Plan Execute Skill

## Purpose

当用户输入 `/plan-execute {plan文件路径}` 时，启动"编排式计划执行"流程：
1. 我（Claude Code）调用 Codex 按照 plan 实现代码
2. Codex 完成后，我对生成的代码进行 review
3. 我将 review 写入 reviews/ 目录，再调用 Codex 检查并修复
4. 循环直到代码质量达标

**核心原则：我不写代码也不改代码。我只做两件事：审查代码 + 编排 Codex。所有代码变更（实现、修复）都由 Codex 执行。**

## Usage

```
/plan-execute plans/my-feature-plan.md
```

## Session 复用

每次调用 Codex 后，从脚本输出中提取 `session_id=xxx`，保存为当前任务的 session ID。在同一任务的后续 Codex 调用中，通过 `--session <id>` 传入以复用上下文，让 Codex 记住之前的实现和修复历史，避免每次都重新读取理解所有文件。

## My Workflow (Claude Code)

### Step 1: 读取 Plan，拆分执行步骤

读取指定的 plan 文件，理解：
- 方案的整体目标和范围
- 需要创建/修改的文件列表
- 实现步骤的先后顺序
- 相关的项目约定（参考 CLAUDE.md）

如果 plan 中已有 checklist（`- [ ]` / `- [x]`），以此为执行单元。
如果没有明确步骤，我先拆分为合理的执行批次（每批不超过 5 个文件变更）。

### Step 2: 调用 Codex 实现代码

使用 `/codex` skill，给 Codex 以下指令：

```
按照 {plan文件路径} 中的方案实现代码。

当前执行范围：{具体的步骤/批次描述}

要求：
- 严格按照 plan 中的设计实现，不要自行发挥
- 遵循项目 CLAUDE.md 中的 Code Quality Hard Limits
- 单文件 ≤ 800 行，单函数 ≤ 50 行，嵌套 ≤ 3 层
- 完成后运行 pnpm build 确认编译通过
- 如果 plan 中有 checklist，完成的步骤标记为 [x]

实现完成后，列出所有变更的文件及变更摘要。
```

### Step 3: Review Codex 产出的代码（我的核心职责）

Codex 完成后，我进行代码 review。**注意：我只读代码和写 review，绝不直接修改任何源代码文件。**

1. **读取所有变更的文件**，逐一审查
2. **对照 plan** 检查实现是否符合设计意图
3. **检查代码质量**：
   - 是否违反 Code Quality Hard Limits
   - 是否引入安全隐患
   - 是否遗漏错误处理
   - 命名和组织是否清晰
   - 是否符合项目现有模式
4. **运行 `pnpm build`** 确认编译状态

### Step 4: 写入 Review 并交给 Codex 修复

将 review 追加到 `reviews/{topic}-review.md`（与 plan-review 共用同一文件）：

```markdown
---

## Code Review Round {N} — {YYYY-MM-DD}

**Scope**: {本次 review 的代码范围}
**Build Status**: PASS / FAIL

### Issues

#### Issue 1 ({severity}): {标题}
**File**: {文件路径:行号}
{问题描述}
**Fix**: {具体修复建议}

...

### Verdict: NEEDS_FIX / APPROVED
```

如果 `Verdict: NEEDS_FIX`，调用 `/codex` 让 Codex 去修复（我不自己改）：

```
读取 {review文件路径} 中最新一轮的 Code Review，
逐条检查 issues，确认是问题的进行修复，不是问题的说明理由。
修复完成后运行 pnpm build 确认编译通过。
列出修复的 issues 和对应的变更。
```

如果 `Verdict: APPROVED`，跳到 Step 6。

### Step 5: 验证修复并迭代

Codex 修复后，我再次 review（仍然只读不改）：
- 检查每个 issue 是否真正修复
- 检查修复是否引入新问题
- 如仍有问题，写新一轮 review → 再交给 Codex 修复（重复 Step 4）
- 如全部通过，标记 `Verdict: APPROVED`

### Step 6: 更新 Plan 进度

每完成一个批次，调用 Codex 更新 plan 文件中的 checklist（`- [ ]` → `- [x]`）。
如果还有未完成的步骤，回到 Step 2 执行下一批次。
全部完成后进入收尾。

### Step 7: 收尾

向用户汇报：
- 完成了哪些步骤
- 经历了几轮 code review
- 主要修复了哪些问题
- 最终编译状态
- 变更的文件列表
- review 记录文件路径

## Review Severity Levels

| Level | 含义 | 是否必须修复 |
|-------|------|-------------|
| Critical | 会导致运行时错误或安全漏洞 | 必须 |
| High | 违反项目约定或明显的设计缺陷 | 必须 |
| Medium | 代码质量问题，可改进 | 建议修复 |
| Low | 风格或偏好问题 | 可选 |
| Suggestion | 优化建议 | 可选 |

**Verdict 判定规则**：
- 存在 Critical 或 High → `NEEDS_FIX`
- 仅 Medium 及以下 → `APPROVED`（附带改进建议）

## File Convention

- Review 文件与 plan-review 共用：`reviews/{topic}-review.md`
- `{topic}` 取 plan 文件名（去掉 `.md`）
- Plan review 轮次和 code review 轮次都追加在同一文件中
- 通过标题区分：`## Round {N}` (plan review) vs `## Code Review Round {N}` (code review)
