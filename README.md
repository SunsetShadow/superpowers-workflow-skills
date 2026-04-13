# Agent Skills

面向 AI Agent（Claude Code 等）的专业工作流技能集合，基于 [Superpowers](https://github.com/sunsetshadow/superpowers) 技能体系构建。

## Skills 一览

| Skill | 版本 | 用途 | 流程 |
|-------|------|------|------|
| [superpowers-workflow](#superpowers-workflow-v320) | v3.2.0 | 新功能端到端开发 | 完整 6 阶段 / 快速 4 阶段 |
| [bug-fix-workflow](#bug-fix-workflow-v210) | v2.1.0 | Bug 修复（先复现后修复） | 完整 7 阶段 / 快速 4 阶段 |

---

## 前置依赖

### 必需：Superpowers 技能插件

两个工作流均依赖 [Superpowers](https://github.com/sunsetshadow/superpowers) 技能体系，需通过 `skills` CLI 安装：

```bash
# 安装 Superpowers 技能包（包含所有子技能）
skills install superpowers
```

**依赖的 Superpowers 子技能明细：**

| 子技能 | 用途 | superpowers-workflow | bug-fix-workflow |
|--------|------|:---:|:---:|
| `superpowers:brainstorming` | 需求探索与设计 | 阶段 3 | - |
| `superpowers:writing-plans` | 编写实施计划 | 阶段 3（brainstorming 内部调用） | - |
| `superpowers:executing-plans` | 执行实施计划 | 阶段 4（Inline 模式） | - |
| `superpowers:subagent-driven-development` | Subagent 并行执行 | 阶段 4（推荐模式） | - |
| `superpowers:systematic-debugging` | 系统化调试（含 TDD） | 阶段 5（验证失败时） | 阶段 4 |
| `superpowers:using-git-worktrees` | Git Worktree 隔离 | 阶段 2 | 阶段 3 |
| `superpowers:verification-before-completion` | 完成前验证 | 阶段 5 | 阶段 5 |
| `superpowers:requesting-code-review` | 发起代码审查 | 阶段 4（执行技能内部） | 阶段 6 |
| `superpowers:receiving-code-review` | 接收代码审查反馈 | 阶段 4（执行技能内部） | 阶段 6 |
| `superpowers:finishing-a-development-branch` | 完成分支 | 阶段 6 | 阶段 7 |

### 可选：Web 验证插件

当项目为 Web 应用时，以下插件提供浏览器自动化验证能力：

| 插件 | 类型 | 安装方式 | 使用场景 |
|------|------|---------|---------|
| **Playwright MCP** | MCP Server（内置） | 零配置，开箱即用 | 页面截图、快照、元素交互 |
| **agent-browser** | Skill | `skills install agent-browser` | 已登录会话验证、多步骤链式操作、数据抓取 |

**选择规则：** 默认使用 Playwright MCP；当需要登录态持久化、多步骤批量操作或数据抓取时，自动切换到 `agent-browser`。

---

## superpowers-workflow v3.2.0

端到端新功能开发工作流，强制 TDD（测试驱动开发）。

**铁律：设计批准前禁止实现。**

### 完整流程（6 阶段）

```
阶段 1: 需求分流          → 功能还是 bug？需要分解吗？完整还是快速？是否 Web 应用？
         ↓ 纯决策，不做探索不写代码
阶段 2: 环境准备          → superpowers:using-git-worktrees（智能判断，单人项目可跳过）
阶段 3: 设计              → superpowers:brainstorming → writing-plans
         ↓ 产出: 设计文档 + 实施计划 + tasks.json ⛔ BLOCKING（缺少任一不可继续）
阶段 4: 执行              → superpowers:subagent-driven-development 或 executing-plans
         ↓ TDD 循环 + per-task 审查 + 进度跟踪
阶段 5: 最终集成验证      → superpowers:verification-before-completion
         ↓ 全量测试 + lint + Playwright e2e ⛔ BLOCKING
阶段 6: 完成分支          → superpowers:finishing-a-development-branch
```

**执行模式选择（阶段 4）：**

| 模式 | 适用场景 |
|------|---------|
| **Subagent-Driven（推荐）** | 计划有 3+ 任务、任务间相对独立 |
| **Inline Execution** | 计划只有 1-2 个任务、任务间高度依赖 |

### 快速流程（`--quick`，4 阶段）

适用于简单需求、单文件修改、小功能调整。

```
阶段 1: 需求分流 → 确认简单需求，用户批准
阶段 2: TDD 开发 → RED → GREEN → REFACTOR
阶段 3: 验证     → 测试 + lint 证据 ⛔ BLOCKING
阶段 4: 完成     → 提交变更
```

跳过：环境准备、头脑风暴、编写计划、代码审查。

### 转交条件

以下情况应转交 `bug-fix-workflow`：
- 开发过程中发现现有功能 bug
- 需求分析时发现用户描述的实际是 bug
- CI 测试失败需要先修复

### 触发词

`新需求` `新功能` `add feature` `implement` `build` `create` `develop` `新特性` `功能开发` `TDD`

---

## bug-fix-workflow v2.1.0

Bug 修复工作流，强调先复现后修复。

**铁律：无法复现 = 禁止修改代码。**

### 完整流程（7 阶段）

```
阶段 1: 接收 Bug 报告     → 确认期望/实际行为 + 复现步骤 + 环境
         ↓ 最低信息检查 ⛔ BLOCKING
阶段 2: 复现 Bug          → 选择复现方式，确认可稳定复现 ⛔ BLOCKING
         ↓ 无法复现 → 不进入下一阶段
阶段 3: 环境准备          → superpowers:using-git-worktrees（智能判断，按需）
阶段 4: 调试与修复        → superpowers:systematic-debugging（完整 4 Phase，含 TDD）
         ↓ Phase 1-3 根因分析 → Phase 4 创建失败测试 + 实现修复 ⛔ BLOCKING
阶段 5: 完成前验证        → superpowers:verification-before-completion
         ↓ 原步骤验证 + 前后对比证据 ⛔ BLOCKING
阶段 6: 代码审查          → requesting + receiving-code-review（P0 可后补）
阶段 7: 完成分支          → superpowers:finishing-a-development-branch
```

### 快速流程（`--quick`，4 阶段）

适用于影响范围小、修复方案明确的 bug。

```
阶段 1: 接收报告         → 复现步骤 + Bug 描述
阶段 2: 复现 Bug         → 稳定复现 + 证据 ⛔ BLOCKING
阶段 3: 调试与修复       → systematic-debugging（含 TDD） → RED → GREEN
阶段 4: 验证             → 测试通过 + 前后证据 ⛔ BLOCKING
```

跳过：环境准备、代码审查。复现和验证不可跳过。

### Bug 严重程度与流程调整

| 严重程度 | 响应时间 | 分支策略 | 审查要求 | 环境隔离 |
|---------|---------|---------|---------|---------|
| P0 紧急 | 立即 | hotfix → main | 可后补 | 可跳过 |
| P1 高 | 24h 内 | hotfix → main | 必须 | 可跳过 |
| P2 中 | 本周内 | feature → develop | 必须 | 建议 |
| P3 低 | 排期 | feature → develop | 必须 | 建议 |

### 转交条件

以下情况应转交 `superpowers-workflow`：
- 排查后发现不是 bug，而是需要新功能或需求变更
- 修复过程中发现需要大规模重构
- 用户在修复过程中追加了新需求

### 触发词

`bug修复` `修bug` `fix bug` `debug` `调试` `排查` `hotfix` `紧急修复` `报错了` `不工作了` `测试挂了`

---

## 安装

```bash
# 安装技能（假设已配置 skills CLI）
skills install superpowers-workflow
skills install bug-fix-workflow
```

## 项目结构

```
AGENT-SKILLS/
├── superpowers-workflow/
│   ├── SKILL.md                        # 主技能定义
│   └── references/
│       ├── anti-patterns.md            # 反模式列表
│       ├── doc-proposal.md             # Proposal 文档模板
│       ├── doc-spec-delta.md           # Spec Delta 文档模板
│       ├── doc-plans.md                # 设计文档模板
│       ├── doc-tasks.md                # Tasks JSON 模板（进度跟踪 + testCases）
│       └── verification-guide.md       # 验证清单
├── bug-fix-workflow/
│   ├── SKILL.md                        # 主技能定义
│   └── references/
│       ├── anti-patterns.md            # 反模式列表
│       ├── reproduction-methods.md     # 复现方式选择指南
│       ├── reproduction-fallback.md    # 无法复现时的回退流程
│       ├── tdd-fix-guide.md            # TDD 修复指南
│       └── verification-guide.md       # 验证清单
└── README.md
```

## License

MIT
