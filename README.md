# Agent Skills

面向 AI Agent（Claude Code 等）的专业工作流技能集合，基于 [Superpowers](https://github.com/sunsetshadow/superpowers) 技能体系构建。

## Skills 一览

| Skill | 版本 | 用途 | 流程 |
|-------|------|------|------|
| [superpowers-workflow](#superpowers-workflow-v150) | v1.5.0 | 新功能端到端开发 | 9 阶段 / 快速 4 阶段 |
| [bug-fix-workflow](#bug-fix-workflow-v120) | v1.2.0 | Bug 修复（先复现后修复） | 8 阶段 / 快速 4 阶段 |

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
| `superpowers:brainstorming` | 需求探索与设计 | 1.1 | - |
| `superpowers:writing-plans` | 编写实施计划 | 2.1 | - |
| `superpowers:executing-plans` | 执行实施计划 | 3.1 | - |
| `superpowers:systematic-debugging` | 系统化调试 | 6.7 | 3.1 |
| `superpowers:using-git-worktrees` | Git Worktree 隔离 | 5.1 | 4.2 |
| `superpowers:test-driven-development` | TDD 开发循环 | 6.1 | 5.1 |
| `superpowers:verification-before-completion` | 完成前验证 | 7.1 | 6.1 |
| `superpowers:requesting-code-review` | 发起代码审查 | 8.1 | 7.1 |
| `superpowers:receiving-code-review` | 接收代码审查反馈 | 8.2 | 7.2 |
| `superpowers:finishing-a-development-branch` | 完成分支 | 9.1 | 8.1 |

### 可选：Web 验证插件

当项目为 Web 应用时，以下插件提供浏览器自动化验证能力：

| 插件 | 类型 | 安装方式 | 使用场景 |
|------|------|---------|---------|
| **Playwright MCP** | MCP Server（内置） | 零配置，开箱即用 | 页面截图、快照、元素交互 |
| **agent-browser** | Skill | `skills install agent-browser` | 已登录会话验证、多步骤链式操作、数据抓取 |

**选择规则：** 默认使用 Playwright MCP；当需要登录态持久化、多步骤批量操作或数据抓取时，自动切换到 `agent-browser`。

---

## superpowers-workflow v1.5.0

端到端新功能开发工作流，强制 TDD（测试驱动开发）。

**铁律：设计批准前禁止编码。**

### 完整流程（9 阶段）

```
阶段 1: 头脑风暴          → superpowers:brainstorming
         ↓ 输出设计文档，用户审阅批准 ⛔ BLOCKING
阶段 2: 编写计划          → superpowers:writing-plans
阶段 3: 执行计划          → superpowers:executing-plans
阶段 4: 开发前验证        → 技术可行性 + Web 基线截图
阶段 5: 环境准备          → superpowers:using-git-worktrees（可选）
阶段 6: TDD 开发循环      → superpowers:test-driven-development
         ↓ RED → 验证 → GREEN → 验证 → REFACTOR
阶段 7: 完成前验证        → superpowers:verification-before-completion
         ↓ 必须提供命令+输出+退出码证据 ⛔ BLOCKING
阶段 8: 代码审查          → requesting + receiving-code-review
阶段 9: 完成分支          → superpowers:finishing-a-development-branch
```

### 快速流程（`--quick`，4 阶段）

适用于简单需求、单文件修改、小功能调整。

```
阶段 1: 简要确认需求 → 用户批准 ⛔ BLOCKING
阶段 2: TDD 开发     → RED → GREEN → REFACTOR
阶段 3: 验证         → 测试+lint 证据 ⛔ BLOCKING
阶段 4: 提交
```

### 触发词

`新需求` `新功能` `add feature` `implement` `build` `create` `develop` `新特性` `功能开发` `TDD`

---

## bug-fix-workflow v1.2.0

Bug 修复工作流，强调先复现后修复。

**铁律：无法复现 = 禁止修改代码。**

### 完整流程（8 阶段）

```
阶段 1: 接收 Bug 报告     → 确认期望/实际行为 + 复现步骤 + 环境
         ↓ 最低信息检查 ⛔ BLOCKING
阶段 2: 复现 Bug          → 选择复现方式，确认可稳定复现 ⛔ BLOCKING
         ↓ 无法复现 → 不进入下一阶段
阶段 3: 系统化调试        → superpowers:systematic-debugging（仅根因分析）
         ↓ 确认根本原因 ⛔ BLOCKING
阶段 4: 环境准备          → superpowers:using-git-worktrees（按需）
阶段 5: TDD 修复循环      → superpowers:test-driven-development
         ↓ RED（复现Bug） → GREEN（最小修复） → 回归测试
阶段 6: 完成前验证        → superpowers:verification-before-completion
         ↓ 原步骤验证 + 前后对比证据 ⛔ BLOCKING
阶段 7: 代码审查          → requesting + receiving-code-review（P0 可后补）
阶段 8: 完成分支          → superpowers:finishing-a-development-branch
```

### 快速流程（`--quick`，4 阶段）

适用于影响范围小、修复方案明确的 bug。

```
阶段 1: 确认报告         → 复现步骤 + Bug 描述
阶段 2: 复现 Bug         → 稳定复现 + 证据 ⛔ BLOCKING
阶段 3: 调试 + TDD 修复  → 根因 → RED → GREEN
阶段 4: 验证             → 测试通过 + 前后证据 ⛔ BLOCKING
```

### Bug 严重程度与流程调整

| 严重程度 | 响应时间 | 分支策略 | 审查要求 |
|---------|---------|---------|---------|
| P0 紧急 | 立即 | hotfix → main | 可后补 |
| P1 高 | 24h 内 | hotfix → main | 必须 |
| P2 中 | 本周内 | feature → develop | 必须 |
| P3 低 | 排期 | feature → develop | 必须 |

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
│       ├── tdd-guide.md                # TDD 详细指南
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
