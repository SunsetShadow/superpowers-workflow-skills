---
name: superpowers-workflow
metadata:
  version: 3.2.0
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

---

## 执行模式选择

阶段 4（执行）需要根据计划特征选择执行方式:

| 模式 | 技能 | 适用场景 |
|------|------|---------|
| **Subagent-Driven (推荐)** | `Skill(skill="superpowers:subagent-driven-development")` | 计划有 3+ 任务、任务间相对独立 |
| **Inline Execution** | `Skill(skill="superpowers:executing-plans")` | 计划只有 1-2 个任务、任务间高度依赖 |

选择依据:
- 默认推荐 Subagent-Driven（每个任务派发独立 agent，两阶段审查，上下文隔离）
- 任务数 ≤ 2 且高度耦合时用 Inline（减少派发开销）
- 多个完全独立的模块需要并行 → 在阶段 1 分解为子项目，每个独立走完整流程

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
| (无参数) | 完整 6 阶段流程 |
| `--quick` | 快速模式：4 阶段流程，适合简单需求 |

---

## 工作流检查清单

### 完整流程 (6 阶段)

```
新需求开发进度:

- [ ] 阶段 1: 需求分流 ⚠️ REQUIRED
  - [ ] 1.1 判断: 新功能还是 bug? → bug 转交 bug-fix-workflow
  - [ ] 1.2 判断: 需要分解成子项目吗? → 大项目先分解，每个子项目独立走流程
  - [ ] 1.3 判断: 完整流程还是 --quick? → 简单需求走快速流程
  - [ ] 1.4 判断: 是否 Web 应用? → 标记后续阶段需要 UI 验证

- [ ] 阶段 2: 环境准备
  - [ ] 2.1 智能判断是否需要 worktree:
    - [ ] 检查 git log 作者数（单人/多人）
    - [ ] 检查是否已有活跃的 feature branch
    - [ ] 单人项目 + 无并行需求 → 跳过 worktree，直接创建 feature branch，输出原因
    - [ ] 多人项目或需要隔离 → 调用 Skill(skill="superpowers:using-git-worktrees")
  - [ ] 2.2 (如果跳过 worktree) 创建 feature branch: `feature/<change-id>`

- [ ] 阶段 3: 设计 ⚠️ REQUIRED
  - [ ] 3.1 调用 Skill(skill="superpowers:brainstorming") — ⛔ 必须使用 Skill 工具调用，禁止手工模拟
  - [ ] 3.2 brainstorming 完成探索 → 提问 → 方案 → 设计 → 文档写入 → 用户审阅批准
  - [ ] 3.3 brainstorming 自动调用 writing-plans，产出实施计划（含 TDD 步骤）
  - [ ] 3.4 按 references/doc-tasks.md 格式额外产出 tasks.json（含 testCases 字段）
  - [ ] 3.5 ⛔ 产出物验证门 — 以下文件必须存在，缺少任一不可进入阶段 4:
    - [ ] 设计文档: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
    - [ ] 实施计划: `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`
    - [ ] 任务跟踪: `docs/changes/<change-id>/tasks.json`
  - [ ] 3.6 ⛔ writing-plans 完成后，不要提供执行选择，回到主流程阶段 4

- [ ] 阶段 4: 执行 ⚠️ REQUIRED
  - [ ] 4.1 选择执行模式 (参考"执行模式选择")
  - [ ] 4.2 ⛔ 严格遵守执行技能的全部流程，不允许跳过任何审查步骤
  - [ ] 4.3 每完成一个任务，提供以下证据后才能更新 tasks.json:
    - [ ] 测试运行命令 + 输出（证明 RED → GREEN）
    - [ ] 更新 tasks.json: 所有 steps.completed = true → passes = true
  - [ ] 4.4 TDD 合规检查 (每个任务完成后):
    - [ ] 确认存在对应的测试文件
    - [ ] 确认测试在实现代码之前创建（或至少同步创建）
    - [ ] 没有测试文件 → STOP，不允许进入下一个任务
  - [ ] 4.5 从 tasks.json 取下一个 passes != true 的任务，重复 4.2-4.4 直到所有 passes = true
  - [ ] 4.6 ⛔ 所有任务完成后，不要调用 finishing-a-development-branch，回到主流程阶段 5

- [ ] 阶段 5: 最终集成验证 ⚠️ REQUIRED
  - [ ] 5.1 调用 Skill(skill="superpowers:verification-before-completion")
  - [ ] 5.2 运行全部测试，确认 0 failures
  - [ ] 5.3 运行 lint/type 检查
  - [ ] 5.4 (仅 Web 应用) 基于 tasks.json 的 Playwright 功能验证:
    - [ ] 从 tasks.json 筛选 testCases.type == "e2e" 的用例
    - [ ] 对每个 e2e 用例:
      - [ ] Playwright 执行: navigate → snapshot → 交互 → 断言
      - [ ] 记录验证结果（通过/失败 + 截图）
      - [ ] 失败时调用 Skill(skill="superpowers:systematic-debugging")
    - [ ] 输出 e2e 验证汇总表: 用例ID | 给定条件 | 操作 | 预期结果 | 实际结果 | 截图
  - [ ] 5.5 提供验证证据 (命令 + 输出 + 退出码)

- [ ] 阶段 6: 完成分支
  - [ ] 6.1 调用 Skill(skill="superpowers:finishing-a-development-branch")
```

### 快速流程 (--quick)

适用于：简单需求、单文件修改、小功能调整

```
快速开发进度:

- [ ] 阶段 1: 需求分流
  - [ ] 确认是简单需求（单文件修改、小功能调整）
  - [ ] 获得用户批准走快速流程

- [ ] 阶段 2: TDD 开发 ⚠️ REQUIRED
  - [ ] RED → GREEN → REFACTOR 循环

- [ ] 阶段 3: 验证 ⚠️ REQUIRED ⛔ BLOCKING
  - [ ] 运行测试命令，查看完整输出
  - [ ] 确认 0 failures
  - [ ] 运行 lint/type 检查
  - [ ] (如果改动了 UI) Playwright 验证改动区域
  - [ ] 提供命令 + 输出 + 退出码证据

- [ ] 阶段 4: 完成
  - [ ] 提交变更

跳过: 环境准备、头脑风暴、编写计划、代码审查

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
| 薄包装 | 只加子技能没有的能力，让子技能链自然运行 |
| TDD 强制 | 测试先行，禁止先实现后补测试 |
| 证据先行 | 完成前必须提供运行证据 |
| 调试规范 | 遇到 bug 调用 systematic-debugging |

---

## 确认门

在以下节点必须停止并获得用户确认:

1. **需求分流结论** (阶段 1): 告知用户流程选择和理由
2. **设计批准** (阶段 3): 由 brainstorming 技能内部处理，用户批准后方可继续
3. **验证失败** (阶段 5): "验证发现问题，需要调试修复"
4. **功能完成** (阶段 6): 由 finishing-a-development-branch 提供选项

## 转交条件

以下情况应转交 `bug-fix-workflow`:
- 开发过程中发现现有功能 bug（先修 bug 再继续开发）
- 需求分析时发现用户描述的实际是 bug 而非新需求
- CI 测试失败需要先修复

---

## 反模式

| 禁止 | 原因 |
|-----|------|
| 跳过需求分流直接设计 | 可能走错流程（bug 当功能做、大项目没分解） |
| 跳过头脑风暴直接写代码 | 设计批准前禁止实现 |
| 手工模拟 brainstorming 流程 | brainstorming = 文档产出 + writing-plans 调用，跳过它 = 跳过整个质量链 |
| 覆盖 brainstorming 的流程 | brainstorming 已包含完整的探索+设计+审阅 |
| 缺少产出物就进入阶段 4 | 设计文档 + 实施计划 + tasks.json 缺一不可 |
| 单独调用 TDD 技能 | 执行技能已内部处理 TDD |
| 单独调用代码审查技能 | 执行技能已包含 per-task 审查 + 最终审查 |
| 跳过执行技能直接手写代码 | 执行技能包含 TDD + 审查，跳过 = 跳过全部质量保障 |
| 没有测试文件就进入下一个任务 | TDD 合规检查：无测试 = STOP |
| 执行完计划就停止 | 阶段 4 完成后必须继续阶段 5-6 |
| 使用 "应该没问题" | 必须提供验证证据 |
| 保留 "参考" 代码 | 会参考 = 后补测试 |
| 遇到 Bug 直接改代码 | 必须调用 systematic-debugging |

→ 加载 `references/anti-patterns.md` 获取完整列表

---

## 文档模板

按阶段按需加载，不要一次全部读取:

| 阶段 | 加载文件 | 内容 |
|------|---------|------|
| 阶段 3 | `references/doc-tasks.md` | Tasks JSON 模板（进度跟踪 + testCases） |
| 阶段 5 | `references/verification-guide.md` | 验证流程（测试 + lint + Playwright e2e） |

注: 阶段 3 的设计文档和实施计划由 brainstorming + writing-plans 自动产出。路径:
- 设计文档 → brainstorming 默认路径 (`docs/superpowers/specs/`)
- 实施计划 → writing-plans 默认路径 (`docs/superpowers/plans/`)
- tasks.json → `docs/changes/<change-id>/tasks.json` (workflow 额外产出)

以下模板仅在用户明确要求非默认文档格式时使用:
- `references/doc-proposal.md` + `references/doc-spec-delta.md` — Proposal + Spec Delta 模板
- `references/doc-plans.md` — 设计文档模板

---

## 阶段详情

阶段详情补充检查清单，操作步骤见上方。

### 阶段 1: 需求分流

纯决策阶段，不做探索不写代码。快速判断走哪条路。

**必须回答 4 个问题**: 功能还是 bug？需要分解吗？完整还是快速？是否 Web 应用？

根据回答的结论：
- bug → 转交 `bug-fix-workflow`，本流程终止
- 大项目 → 先分解成子项目，每个子项目独立走完整流程
- 简单需求 → 切换到 `--quick` 快速流程
- Web 应用 → 标记后续阶段 5 需要 UI 验证

禁止: 在此阶段做技术探索或代码阅读 / 跳过分流直接进入头脑风暴

### 阶段 2: 环境准备

**智能判断**: 先评估项目是否真的需要 worktree。

- 单人项目（git log 作者 ≤ 1）→ 直接创建 feature branch，跳过 worktree
- 多人项目或需要隔离 → 调用 `superpowers:using-git-worktrees`

**为什么提前**: brainstorming、writing-plans、执行技能都期望在隔离环境中运行。

### 阶段 3: 设计

调用 `superpowers:brainstorming`，**⛔ 必须使用 Skill 工具调用，禁止手工模拟**。完全遵循其自带流程（探索、提问、提方案、写设计文档、审阅）。

brainstorming 会自动调用 `writing-plans`，产出实施计划（含 TDD RED/GREEN/REFACTOR 步骤）。

**额外产出**: 按 `references/doc-tasks.md` 格式额外生成 tasks.json（含 testCases 字段），保存到 `docs/changes/<change-id>/tasks.json`。tasks.json 提供轻量进度跟踪和 e2e 测试案例定义。

**⛔ 产出物验证门** — 以下文件必须存在，缺少任一不可进入阶段 4:
1. 设计文档: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
2. 实施计划: `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`
3. 任务跟踪: `docs/changes/<change-id>/tasks.json`

**⛔ 两个覆盖指令**:
1. writing-plans 完成后，**不要**提供执行选择，回到主流程阶段 4
2. 接受 writing-plans 的默认文档格式和路径，不覆盖

禁止: 跳过 brainstorming 的任何步骤 / 手工模拟 brainstorming / 重复执行 brainstorming 已完成的探索 / 覆盖 brainstorming 的文档路径 / 缺少产出物就进入阶段 4

### 阶段 4: 执行

选择执行模式（参考"执行模式选择"），**⛔ 严格遵守执行技能的全部流程，不允许跳过任何审查步骤**。

执行技能已包含:
- TDD 循环（RED → GREEN → REFACTOR）
- per-task spec 合规审查
- per-task 代码质量审查
- 最终整体代码审查

**任务跟踪循环**:
1. 从 tasks.json 取下一个 `passes != true` 的任务
2. 执行该任务（通过执行技能的 TDD 流程）
3. TDD 合规检查: 确认存在对应的测试文件，没有测试 → STOP
4. 提供测试运行证据（命令 + 输出），更新 tasks.json
5. 重复直到所有任务 `passes = true`

**⛔ 覆盖指令**: 所有任务完成后，**不要**调用 `finishing-a-development-branch`，回到主流程阶段 5。

禁止: 单独调用 TDD 技能（已包含）/ 单独调用代码审查技能（已包含）/ 跳过执行技能直接手写代码 / 没有测试文件就进入下一个任务

### 阶段 5: 最终集成验证

调用 `superpowers:verification-before-completion`。

**为什么需要**: 执行技能的审查是单任务维度。这里做全量集成验证:
- 所有测试一起跑（可能有跨任务的交互问题）
- lint/type 全量检查
- (Web 应用) 基于 tasks.json 的 Playwright 功能验证
- 证据输出（命令 + 输出 + 退出码）

→ 加载 `references/verification-guide.md`

**Playwright 功能验证流程** (仅 Web 应用):
1. 从 tasks.json 筛选 `testCases.type == "e2e"` 的用例
2. 对每个 e2e 用例: Playwright 执行 navigate → snapshot → 交互 → 断言
3. 失败时调用 `Skill(skill="superpowers:systematic-debugging")`
4. 输出 e2e 验证汇总表

验证未通过时:
1. 停止 → 调用 `Skill(skill="superpowers:systematic-debugging")`
2. 修复后重新执行验证
3. 禁止跳过验证声称完成

### 阶段 6: 完成分支

调用 `superpowers:finishing-a-development-branch`。

该技能会: 验证测试 → 提供 4 个选项（合并/PR/保留/丢弃）→ 执行选择 → 清理 worktree。

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

- [ ] tasks.json 已输出到 `docs/changes/<change-id>/tasks.json`（含 testCases）
- [ ] 设计文档已由 brainstorming 产出
- [ ] 实施计划已由 writing-plans 产出
- [ ] tasks.json 所有任务 passes = true
- [ ] 每个新增/修改的源文件有对应的测试文件
- [ ] (仅 Web 应用) tasks.json 中 e2e 用例已通过 Playwright 验证

### 禁止的模糊表述

| 禁止 | 替代 |
|-----|------|
| "应该没问题了" | "测试命令结果: 0 failures" |
| "可能通过了" | "实际运行输出: [粘贴]" |
| "看起来正常" | "退出码: 0" |
