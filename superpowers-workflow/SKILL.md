---
name: superpowers-workflow
metadata:
  version: 1.5.0
  author: reeves_zd
description: |
  End-to-end feature development workflow with TDD enforcement. Use when user
  asks to build, create, implement, develop, add, design, code, or ship new
  features, functionality, modules, components, services, APIs, endpoints,
  pages, or capabilities. Triggers on: 新需求、新功能、add feature、
  implement、build、create、develop、new module、新特性、功能开发、
  头脑风暴、TDD、测试驱动、代码审查. Supports --quick for simple tasks
  (4-phase flow). Projects: web app, backend service, CLI tool, library,
  API, microservice, monolith.
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

## 技能调用方式

本工作流调用 superpowers 插件技能，使用 `Skill` 工具：

```
Skill(skill="superpowers:brainstorming")
Skill(skill="superpowers:writing-plans")
Skill(skill="superpowers:executing-plans")
...
```

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
  - [ ] 1.5 编写设计文档到 docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md (用户偏好路径优先)
  - [ ] 1.6 展示完整设计文档并请求审阅 ⛔ BLOCKING

- [ ] 阶段 2: 编写计划
  - [ ] 2.1 调用 Skill(skill="superpowers:writing-plans")
  - [ ] 2.2 输出实施步骤、文件列表、风险点、测试策略到 docs/superpowers/plans/

- [ ] 阶段 3: 执行计划
  - [ ] 3.1 调用 Skill(skill="superpowers:executing-plans")
  - [ ] 3.2 按步骤执行，记录偏差和调整

- [ ] 阶段 4: 开发前验证
  - [ ] 4.1 技术可行性检查 (依赖、兼容性)
  - [ ] 4.2 (仅 Web 应用) 参考"Web 验证工具选择"选择工具
  - [ ] 4.3 (仅 Web 应用) 验证现有 UI 并保存基线截图

- [ ] 阶段 5: 环境准备 (可选)
  - [ ] 5.1 如需隔离开发，调用 Skill(skill="superpowers:using-git-worktrees")

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

## 阶段过渡提示规范

每个阶段开始和完成时，必须输出视觉标识提示。

### 阶段开始格式

```
> 🚀 **阶段 N 开始: [阶段名称]**
> 📍 [一句话描述本阶段目标]
```

### 阶段完成格式

```
> 🏆 **阶段 N 完成: [阶段名称]**
> ✅ [一句话总结阶段成果]
```

### 示例

**开始**:
> 🚀 **阶段 1 开始: 头脑风暴**
> 📍 探索需求，产出设计方案

**完成**:
> 🏆 **阶段 1 完成: 头脑风暴**
> ✅ 设计文档已通过审阅，路径: docs/superpowers/specs/2026-03-29-auth-design.md

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

---

## 阶段详情

### 阶段 1: 头脑风暴

**调用**: `Skill(skill="superpowers:brainstorming")`

**问题清单**:
- 这个功能的目的是什么？
- 有什么技术约束？
- 成功标准是什么？

**输出**: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

#### 设计审阅流程 ⛔ BLOCKING

设计文档完成后，必须执行以下流程：

**第一步**: 直接输出完整设计文档内容到对话中（不要只说"文档已写好"）

**第二步**: 使用 `AskUserQuestion` 工具提供以下选择：

```
问题: "请审阅上方设计文档，选择下一步操作："
选项:
  1. ✅ 通过审阅，继续下一阶段
  2. 🔄 提出修改意见，重新调整设计
  3. 💬 输入其他内容
```

**用户选择处理**:
- 选 1 → 输出 🏆 完成提示，进入阶段 2
- 选 2 → 收集修改意见，返回步骤 1.4 重新设计方案，再次进入审阅流程
- 选 3 → 根据用户输入内容处理，处理完毕后重新询问审阅选择

**禁止**:
- 禁止只写文件路径不展示内容
- 禁止省略设计文档直接问"可以吗？"
- 禁止跳过 AskUserQuestion 直接继续

### 阶段 2-3: 计划与执行

依次调用:
- `Skill(skill="superpowers:writing-plans")` → 输出计划到 `docs/superpowers/plans/`
- `Skill(skill="superpowers:executing-plans")` → 按计划执行

### 阶段 4: 开发前验证

**所有项目**: 技术可行性检查（依赖、兼容性）

**仅 Web 应用**: 参考"Web 验证工具选择"选择工具，验证现有 UI 并保存基线截图

### 阶段 5: 环境准备

**可选隔离**: `Skill(skill="superpowers:using-git-worktrees")`

### 阶段 6: TDD 开发循环

**调用**: `Skill(skill="superpowers:test-driven-development")`

```
🔴 RED → ⚡验证 → 🟢 GREEN → ⚡验证 → 🔵 REFACTOR
```

→ 加载 `references/tdd-guide.md` 获取详细指南

### 阶段 7: 完成前验证

**调用**: `Skill(skill="superpowers:verification-before-completion")`

**必须提供**: 命令 + 完整输出 + 退出码

**仅 Web 应用**: 参考"Web 验证工具选择"选择工具，验证新功能并截图对比基线

→ 加载 `references/verification-guide.md` 获取详细清单

### 阶段 8-9: 代码审查与完成

依次调用:
- `Skill(skill="superpowers:requesting-code-review")`
- `Skill(skill="superpowers:receiving-code-review")`
- `Skill(skill="superpowers:finishing-a-development-branch")`

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

- [ ] 设计文档已输出到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
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
