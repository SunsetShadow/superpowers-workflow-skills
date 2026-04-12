---
name: superpowers-workflow
metadata:
  version: 1.9.0
  author: reeves_zd
description: |
  端到端功能开发工作流，铁律: 设计批准前禁止实现。TDD 强制 + 设计审阅门。

  用于任何新功能开发，从"加个按钮"到完整系统设计。即使是看似简单的需求也应触发。
  支持参数: --quick 快速模式（4 阶段流程）

  触发条件:
  - 新功能/新需求: 新功能、新需求、add feature、implement、build、develop
  - 开发类: 做个接口、写个服务、加个字段/页面/按钮/组件、实现/开发/搭建一个
  - 设计类: 功能设计、架构设计、需求分析

  不适用场景（使用 bug-fix-workflow）:
  - bug 修复、问题排查、故障诊断、hotfix
---

# IRON LAW

```
NO IMPLEMENTATION BEFORE DESIGN APPROVAL. NEVER.
```

## Red Flags (Stop Immediately If)

- 还没问清楚需求就开始写代码
- 用户说"先做着看"，没有明确需求
- 没有输出设计文档就开始实现
- 跳过测试直接写实现
- 想到 "这个很简单，不用设计"

## 执行模式选择

阶段 3 需要根据计划特征选择执行方式:

| 模式 | 技能 | 适用场景 |
|------|------|---------|
| **Subagent-Driven (推荐)** | `Skill(skill="superpowers:subagent-driven-development")` | 计划有 3+ 任务、任务间相对独立 |
| **Inline Execution** | `Skill(skill="superpowers:executing-plans")` | 计划只有 1-2 个任务、任务间高度依赖 |
| **Parallel Dispatch** | `Skill(skill="superpowers:dispatching-parallel-agents")` | 计划有 2+ 完全独立的任务可并行 |

选择依据:
- 默认推荐 Subagent-Driven（每个任务派发独立 agent，两阶段审查，上下文隔离）
- 任务数 ≤ 2 且高度耦合时用 Inline（减少派发开销）
- 前后端分离、多个独立模块可同时推进时用 Parallel Dispatch

---

## Web 验证工具选择

Web 应用验证阶段需先选择工具，选定后该阶段统一使用。

**默认: Playwright MCP 内置工具**
- 场景: 截图、快照、元素交互（点击/填写/提交）
- 调用: `browser_navigate` → `browser_snapshot` → `browser_take_screenshot`
- 优势: 零配置，调用链短

**切换 agent-browser: `Skill(skill="agent-browser")`**
- 触发条件（满足任一即切换）:
  - 验证需要已登录的会话（auth vault / state 持久化）
  - 多步骤链式操作（batch 更高效）
  - 并行验证多个页面
  - 需要抓取页面数据做断言

---

## 参数

| 参数 | 说明 |
|-----|------|
| (无参数) | 完整 9 阶段流程 |
| `--quick` | 快速模式：4 阶段流程，适合简单需求 |

---

## 工作流检查清单

### 完整流程 (9 阶段)

```
新需求开发进度:

- [ ] 阶段 1: 头脑风暴 ⚠️ REQUIRED
  - [ ] 1.1 调用 Skill(skill="superpowers:brainstorming")
  - [ ] 1.2 探索项目上下文 (文件、文档、最近提交)
  - [ ] 1.3 逐个提问澄清需求 (目的、约束、成功标准)
  - [ ] 1.4 提出 2-3 种方案及权衡
  - [ ] 1.5 按需加载文档模板 (参见"文档模板"章节)
  - [ ] 1.6 编写文档产出:
    - Proposal → `docs/changes/<change-id>/proposal.md`
    - Spec Delta → `docs/changes/<change-id>/specs/<area>/spec-delta.md`
    - (用户偏好路径优先)
  - [ ] 1.7 展示完整文档并请求审阅 ⛔ BLOCKING

- [ ] 阶段 2: 编写计划
  - [ ] 2.1 (可选) 如需隔离环境，先调用 Skill(skill="superpowers:using-git-worktrees") 设置 worktree
  - [ ] 2.2 调用 Skill(skill="superpowers:writing-plans")
  - [ ] 2.3 输出两份文档 (各有分工):
    - tasks.json → `docs/changes/<change-id>/tasks.json` (做什么 + 完成状态跟踪)
    - Implementation Plan → `docs/plans/YYYY-MM-DD-<feature>.md` (怎么做 + 代码/命令/TDD 步骤)

- [ ] 阶段 3: 执行计划
  - [ ] 3.1 选择执行模式 (参考"执行模式选择")
  - [ ] 3.2 按选定模式执行，记录偏差和调整

- [ ] 阶段 4: 开发前验证
  - [ ] 4.1 技术可行性检查 (依赖、兼容性)
  - [ ] 4.2 (仅 Web 应用) 参考"Web 验证工具选择"选择工具
  - [ ] 4.3 (仅 Web 应用) 验证现有 UI 并保存基线截图

- [ ] 阶段 5: 环境准备 (可已在阶段 2 完成)
  - [ ] 5.1 如阶段 2 未设置 worktree 且需隔离开发，调用 Skill(skill="superpowers:using-git-worktrees")

- [ ] 阶段 6: TDD 开发循环 ⚠️ REQUIRED
  - [ ] 6.1 调用 Skill(skill="superpowers:test-driven-development")
  - [ ] 6.2 RED: 编写失败的测试
  - [ ] 6.3 验证 RED: 确认测试失败
  - [ ] 6.4 GREEN: 写最小代码通过测试
  - [ ] 6.5 验证 GREEN: 确认测试通过
  - [ ] 6.6 REFACTOR: 重构优化 (可选)
  - [ ] 6.7 (遇到 Bug → 调用 Skill(skill="superpowers:systematic-debugging"))

- [ ] 阶段 7: 完成前验证 ⚠️ REQUIRED
  - [ ] 7.1 调用 Skill(skill="superpowers:verification-before-completion")
  - [ ] 7.2 运行全部测试，确认 0 failures
  - [ ] 7.3 运行 lint/type 检查
  - [ ] 7.4 (仅 Web 应用) 参考"Web 验证工具选择"选择工具
  - [ ] 7.5 (仅 Web 应用) 验证新功能并截图对比基线
  - [ ] 7.6 提供验证证据 (命令 + 输出 + 退出码)

- [ ] 阶段 8: 代码审查
  - [ ] 8.1 调用 Skill(skill="superpowers:requesting-code-review")
  - [ ] 8.2 调用 Skill(skill="superpowers:receiving-code-review")

- [ ] 阶段 9: 完成分支
  - [ ] 9.1 调用 Skill(skill="superpowers:finishing-a-development-branch")
```

### 快速流程 (--quick)

适用于：简单需求、单文件修改、小功能调整

```
快速开发进度:

- [ ] 阶段 1: 头脑风暴 ⚠️ REQUIRED
  - [ ] 简要确认需求
  - [ ] 获得用户批准 ⛔ BLOCKING

- [ ] 阶段 2: TDD 开发 ⚠️ REQUIRED
  - [ ] RED → GREEN → REFACTOR 循环

- [ ] 阶段 3: 验证 ⚠️ REQUIRED ⛔ BLOCKING
  - [ ] 运行测试命令，查看完整输出
  - [ ] 确认 0 failures
  - [ ] 运行 lint/type 检查
  - [ ] 提供命令 + 输出 + 退出码证据

- [ ] 阶段 4: 完成
  - [ ] 提交变更

跳过: 执行计划、开发前验证、环境准备、浏览器验证、代码审查

⚠️ 注意: 验证阶段不可跳过，即使简单需求也必须提供测试通过证据
```

---

## 阶段过渡

每个阶段开始输出 `> **阶段 N 开始: [名称]** — [目标]`，完成时输出 `> **阶段 N 完成: [名称]** — [成果]`。

---

## 核心原则

| 原则 | 说明 |
|-----|------|
| 优先级链 | CLAUDE.md/AGENTS.md > Superpowers > 默认行为 |
| TDD 强制 | 测试先行，禁止先实现后补测试 |
| 证据先行 | 完成前必须提供运行证据 |
| 调试规范 | 遇到 bug 调用 systematic-debugging |

---

## 确认门

在以下节点必须停止并获得用户确认:

1. **设计批准** (阶段 1): 展示完整设计文档，通过 AskUserQuestion 提供审阅选项
2. **验证失败**: "验证发现问题，需要调整计划"
3. **功能完成**: "功能已完成，可以进入下一步吗？"

## 转交条件

以下情况应转交 `bug-fix-workflow`:
- 开发过程中发现现有功能 bug（先修 bug 再继续开发）
- 需求分析时发现用户描述的实际是 bug 而非新需求
- CI 测试失败需要先修复

---

## 反模式

| 禁止 | 原因 |
|-----|------|
| 跳过头脑风暴直接写代码 | 设计批准前禁止实现 |
| 先写实现再补测试 | 后补测试证明不了任何事 |
| 使用 "应该没问题" | 必须提供验证证据 |
| 保留 "参考" 代码 | 会参考 = 后补测试 |
| 遇到 Bug 直接改代码 | 必须调用 systematic-debugging |

→ 加载 `references/anti-patterns.md` 获取完整列表

## 文档模板

按阶段按需加载，不要一次全部读取:

| 阶段 | 加载文件 | 内容 |
|------|---------|------|
| 阶段 1 (完整变更) | `references/doc-proposal.md` + `references/doc-spec-delta.md` | Proposal + Spec Delta 模板与示例 |
| 阶段 1 (简单需求) | `references/doc-plans.md` (设计文档部分) | 设计文档模板与示例 |
| 阶段 2 | `references/doc-tasks.md` + `references/doc-plans.md` (实施计划部分) | Tasks JSON + Implementation Plan 模板与示例 |

---

## 阶段详情

阶段详情补充检查清单，操作步骤见上方。

### 阶段 1: 头脑风暴

调用 `superpowers:brainstorming`。路径覆盖: 使用 `docs/changes/<change-id>/` 而非默认路径。

核心问题: 功能目的？技术约束？成功标准？→ 按需加载文档模板

#### 设计审阅流程 ⛔ BLOCKING

1. 直接输出完整文档到对话（不要只写路径）
2. 用 `AskUserQuestion` 提供: 通过 / 修改 / 其他
3. 用户选"通过" → 阶段 2；选"修改" → 返回重设计；选"其他" → 处理后重新询问

禁止: 只写路径不展示内容 / 省略文档直接问"可以吗" / 跳过 AskUserQuestion

### 阶段 2-3: 计划与执行

阶段 2: 调用 `superpowers:writing-plans` → 输出 `tasks.json` + Implementation Plan（格式参考 `references/doc-tasks.md` + `references/doc-plans.md`）

阶段 3: 选择执行模式（参考"执行模式选择"）

> **tasks.json**: 轻量跟踪（做什么 + 状态） | **Implementation Plan**: 详细指南（怎么做 + TDD 步骤）

### 阶段 4-5: 开发验证与准备

阶段 4: 技术可行性检查。Web 应用参考"Web 验证工具选择"保存基线截图。

阶段 5: 可选 worktree 隔离（`superpowers:using-git-worktrees`）。阶段 2 已设置则跳过。

### 阶段 6: TDD 开发循环

调用 `superpowers:test-driven-development`。遇到 bug 调用 `superpowers:systematic-debugging`。

→ 加载 `references/tdd-guide.md`

### 阶段 7: 完成前验证

调用 `superpowers:verification-before-completion`。必须提供: 命令 + 输出 + 退出码。Web 应用截图对比基线。

→ 加载 `references/verification-guide.md`

### 阶段 8-9: 代码审查与完成

依次调用: `requesting-code-review` → `receiving-code-review` → `finishing-a-development-branch`

---

## 交付前检查

### 必须验证 (所有模式)

**测试验证**:
- [ ] 测试命令已执行 (例如: `npm test`)
- [ ] 退出码为 0
- [ ] 输出包含 "0 failed" 或等效文字
- [ ] 无 placeholder 文本残留 (TODO, FIXME, xxx)

**代码质量**:
- [ ] Lint 命令已执行 (例如: `npm run lint`)
- [ ] 退出码为 0
- [ ] Type check 已执行 (TypeScript 项目)

**证据格式**:
```
命令: npm test
退出码: 0
输出: Tests: 15 passed, 0 failed
```

### 完整模式额外检查

- [ ] 文档已输出到 `docs/changes/<change-id>/` 目录 (proposal.md + specs/ + tasks.json)
- [ ] change-id 命名符合 kebab-case 规则 (add-/fix-/migrate-/update-)
- [ ] 文件名无 placeholder (无 "xxx", "topic")
- [ ] (仅 Web 应用) 浏览器验证新功能（参见"Web 验证工具选择"）
- [ ] (仅 Web 应用) 截图对比改动前后

### 禁止的模糊表述

| 禁止 | 替代 |
|-----|------|
| "应该没问题了" | "测试命令结果: 0 failures" |
| "可能通过了" | "实际运行输出: [粘贴]" |
| "看起来正常" | "退出码: 0" |

### 验证未通过时

1. 停止 → 调用 `Skill(skill="superpowers:systematic-debugging")`
2. 修复后重新执行验证
3. 禁止跳过验证声称完成
