---
name: bug-fix-workflow
metadata:
  version: 2.1.0
  author: reeves_zd
  recommends:
    - superpowers:systematic-debugging
    - superpowers:verification-before-completion
    - superpowers:requesting-code-review
    - superpowers:receiving-code-review
    - superpowers:using-git-worktrees
    - superpowers:finishing-a-development-branch
    - agent-browser
description: |
  Bug 修复工作流，铁律: 无法复现 = 禁止修改代码。基于 Superpowers 技能体系。

  即使是简单的 bug，也应使用此技能确保修复质量。
  支持参数: --quick 快速模式（4 阶段流程）

  触发条件:
  - Bug fix: fix bug, debug, hotfix, troubleshoot, diagnose, patch, crash, exception, error, issue
  - 功能异常: "不工作了"、"报错了"、"挂了"、"崩溃"、"异常"、"失败"、"错误"
  - 测试失败: "测试挂了"、"CI 红了"、"test failed"、"tests broken"
  - 问题排查: 调试、排查、故障、线上问题、regression、incident

  不适用场景（使用 superpowers-workflow）:
  - 新功能开发、重构、性能优化、新增需求
---

# Iron Law

```
无法复现 = 禁止修改代码
```

## Red Flags (出现时立即停止)

- 还没复现就猜原因并改代码
- 用户说"看起来是这个问题"就信了
- 跳过测试直接写修复
- "试试看这个改动能不能修好"

**优先级**: CLAUDE.md/AGENTS.md > 本工作流 > 默认行为

---

## 参数

| 参数 | 说明 |
|-----|------|
| (无参数) | 完整 7 阶段流程 |
| `--quick` | 快速模式：跳过环境准备和代码审查，合并调试+修复阶段 |

---

## Web 验证工具选择

UI Bug 复现（阶段 2）和完成前验证（阶段 5）涉及浏览器操作，需先选择工具，选定后统一使用。

**默认: Playwright MCP 内置工具**
- 场景: 页面截图、快照、元素交互、Bug 复现、截图对比
- 调用: `browser_navigate` → `browser_snapshot` → `browser_take_screenshot`
- 优势: 零配置，调用链短

**切换 agent-browser: `Skill(skill="agent-browser")`**
- 触发条件（满足任一即切换）:
  - 需要已登录的会话（auth vault / state 持久化）
  - 多步骤链式操作（batch 更高效）
  - 并行复现多个场景
  - 需要抓取页面数据做断言

---

## 阶段过渡

每个阶段开始输出 `> **阶段 N 开始: [名称]** — [目标]`，完成时输出 `> **阶段 N 完成: [名称]** — [成果]`。

---

## Bug 严重程度与流程调整

在阶段 1 确认严重程度后，自动调整后续流程:

| 严重程度 | 响应时间 | 分支策略 | 审查要求 | 环境隔离 |
|---------|---------|---------|---------|---------|
| P0 紧急 | 立即 | hotfix → main | 可后补 | 可跳过 |
| P1 高 | 24h 内 | hotfix → main | 必须 | 可跳过 |
| P2 中 | 本周内 | feature → develop | 必须 | 建议 |
| P3 低 | 排期 | feature → develop | 必须 | 建议 |

**P0 紧急 bug 例外**: 阶段 3 (环境准备) 和阶段 6 (代码审查) 可跳过，但必须在修复完成后 24h 内补齐审查。

---

## 工作流检查清单

### 完整流程 (7 阶段)

```
Bug 修复进度:

- [ ] 阶段 1: 接收 Bug 报告 ⚠️ REQUIRED
  - [ ] 1.1 确认 Bug 描述 (期望 vs 实际)
  - [ ] 1.2 确认复现步骤
  - [ ] 1.3 确认环境信息 (版本/浏览器/OS)
  - [ ] 1.4 确认错误信息 (截图/日志/堆栈)
  - [ ] 1.5 评估严重程度 (P0-P3) 和影响范围
  - [ ] 1.6 最低信息检查 ⛔ BLOCKING
      必须具备: 期望行为 + 实际行为 + 至少一个复现步骤
      缺少任何一项 → 请求补充，不进入下一阶段

- [ ] 阶段 2: 复现 Bug ⚠️ REQUIRED ⛔ BLOCKING
  - [ ] 2.1 判定复现方式 → 加载 references/reproduction-methods.md
  - [ ] 2.2 执行复现步骤
  - [ ] 2.3 确认可稳定复现
  - [ ] 2.4 记录复现证据 (截图/日志/命令输出)
  - [ ] 2.5 (无法复现 → 无法复现处理流程)

- [ ] 阶段 3: 环境准备 (根据严重程度决定)
  - [ ] 3.1 智能判断是否需要隔离:
    - [ ] 检查 git log 作者数（单人/多人）
    - [ ] 单人项目 + 单文件修改 + P2-P3 → 直接创建 fix branch，跳过 worktree
    - [ ] 多人项目 / 3+ 文件修改 / P0-P1 → 调用 Skill(skill="superpowers:using-git-worktrees")
  - [ ] 3.2 (如果跳过 worktree) 创建 fix branch: `fix/<bug-id>`

- [ ] 阶段 4: 调试与修复 ⚠️ REQUIRED
  - [ ] 4.1 调用 superpowers:systematic-debugging (完整 4 阶段)
  - [ ] 4.2 Phase 1-3: 根因调查 → 模式分析 → 假设验证
  - [ ] 4.3 确认根本原因 ⛔ BLOCKING
  - [ ] 4.4 Phase 4: 创建失败测试 + 实现修复 (已内部包含 TDD)
  - [ ] 4.5 ⛔ systematic-debugging 完成后，回到主流程阶段 5

- [ ] 阶段 5: 完成前验证 ⚠️ REQUIRED ⛔ BLOCKING
  - [ ] 5.1 调用 superpowers:verification-before-completion
  - [ ] 5.2 运行全部测试，确认 0 failures
  - [ ] 5.3 运行 lint/type 检查
  - [ ] 5.4 使用原复现步骤验证 Bug 已修复
  - [ ] 5.5 (UI Bug) 基于 Playwright 的修复验证:
    - [ ] 用阶段 2 相同的 Playwright 步骤重新执行
    - [ ] 确认 Bug 不再出现（修复后截图 + 对比）
    - [ ] 失败时调用 Skill(skill="superpowers:systematic-debugging")
  - [ ] 5.6 提供修复前后对比证据

- [ ] 阶段 6: 代码审查 (P0 可后补)
  - [ ] 6.1 调用 superpowers:requesting-code-review
  - [ ] 6.2 调用 superpowers:receiving-code-review

- [ ] 阶段 7: 完成分支
  - [ ] 7.1 调用 superpowers:finishing-a-development-branch
  - [ ] 7.2 更新 Bug 报告状态
```

### 快速流程 (--quick)

适用于影响范围小、修复方案明确的 bug。跳过环境准备和代码审查。

```
Bug 快速修复进度:

- [ ] 阶段 1: 接收报告 ⚠️ REQUIRED
  - [ ] 确认复现步骤和 Bug 描述

- [ ] 阶段 2: 复现 Bug ⚠️ REQUIRED ⛔ BLOCKING
  - [ ] 确认可稳定复现
  - [ ] 记录复现证据

- [ ] 阶段 3: 调试与修复 ⚠️ REQUIRED
  - [ ] 调用 superpowers:systematic-debugging (完整流程，含 TDD)
  - [ ] 根因确认 + RED → GREEN 循环

- [ ] 阶段 4: 验证 ⚠️ REQUIRED ⛔ BLOCKING
  - [ ] 测试通过、Bug 已修复
  - [ ] (UI Bug) Playwright 验证修复
  - [ ] 提供修复前后证据

⚠️ 注意: 复现和验证阶段不可跳过
```

---

## 阶段详情

阶段详情仅包含输出模板，操作步骤见上方检查清单。

### 阶段 1 输出

```
━━━ 阶段 1: 接收 Bug 报告 ━━━
收集信息:
  期望行为: <用户期望的结果>
  实际行为: <实际发生的结果>
  复现步骤: <如何触发>
  环境: <版本/浏览器/OS>
  错误信息: <日志/截图/堆栈>
评估:
  严重程度: P?  影响范围: <描述>

⛔ 最低信息检查:
  [ ] 期望行为  [ ] 实际行为  [ ] 复现步骤 (至少一个)
  → 全部通过 → 阶段 2  → 缺少 → 请求补充
```

### 阶段 2 输出

UI bug 参考"Web 验证工具选择"，非 UI bug 用命令行。→ 加载 `references/reproduction-methods.md`

```
━━━ 阶段 2: 复现 Bug ━━━
复现方式: <UI/API/命令行/测试脚本>
尝试 N 次，成功复现 N 次
复现证据: <截图/日志/命令输出>
→ 可稳定复现 → 阶段 3  → 无法复现 → references/reproduction-fallback.md
```

### 阶段 3 输出

**智能判断**: 单人项目 + 单文件修改 + P2-P3 → 直接创建 fix branch，跳过 worktree。多人项目 / 3+ 文件 / P0-P1 → 调用 `superpowers:using-git-worktrees`。

### 阶段 4 输出

调用 `superpowers:systematic-debugging`，**完整运行 4 个 Phase**。Phase 4 已包含 TDD（创建失败测试 + 实现修复）。

→ Bug 修复特有的 TDD 模式参考 `references/tdd-fix-guide.md`

```
━━━ 阶段 4: 调试与修复 ━━━
⛔ 根因确认 (Phase 1-3):
  根本原因: <一句话>  涉及文件: <file:line>  验证证据: <描述>
→ 确认门: "根本原因是 X，准备进入修复阶段，确认吗？"

修复 (Phase 4, 已包含 TDD):
RED:  测试 <名称> — FAIL <原因>
GREEN: 修改 <file:line> — PASS
回归:  命令 <cmd> — N passed, 0 failed
```

### 阶段 5 输出

调用 `superpowers:verification-before-completion`。→ 加载 `references/verification-guide.md`

**UI Bug Playwright 验证**: 用阶段 2 相同的 Playwright 步骤重新执行，确认 Bug 不再出现。

```
━━━ 阶段 5: 完成前验证 ━━━
[ ] 全部测试: N passed, 0 failed
[ ] lint/type: 通过
[ ] 原复现步骤: Bug 不再出现
[ ] (UI Bug) Playwright 验证: navigate → snapshot → 交互 → 截图对比
[ ] 对比证据: 修复前 + 修复后
→ 确认门: "Bug 已修复，可以提交吗？"
```

### 阶段 6-7 输出

依次调用 `superpowers:requesting-code-review` + `receiving-code-review` + `finishing-a-development-branch`。

---

## 确认门

在以下节点必须停止并获得用户确认:

1. **信息不足** (阶段 1): "缺少 XX 信息，请补充"
2. **复现失败** (阶段 2): "无法复现，需要更多信息或进行代码审查分析"
3. **根因确认** (阶段 4): "根本原因是 X，准备进入修复阶段，确认吗？"
4. **修复完成** (阶段 5): "Bug 已修复，可以提交吗？"

## 转交条件

以下情况应转交 `superpowers-workflow`:
- 排查后发现不是 bug，而是需要新功能或需求变更
- 修复过程中发现需要大规模重构
- 用户在修复过程中追加了新需求

---

## 反模式 (核心 5 条)

| # | 禁止 | 原因 |
|---|-----|------|
| 1 | 还没复现就改代码 | 无法复现 = 无法确认修复 |
| 2 | 跳过测试直接写修复 | 没有失败测试证明不了修复正确 |
| 3 | 修复表面症状 | 必须修复根本原因，否则会复发 |
| 4 | 夹带其他功能改动 | 一个 bug 一个 PR，禁止夹带 |
| 5 | 没有证据就声称修好 | 必须提供修复前后对比证据 |

→ 加载 `references/anti-patterns.md` 获取完整列表

---

## 交付前检查

### 必须验证 (所有模式)

- [ ] **Bug 已复现** (有复现证据)
- [ ] **根本原因已确认** (不是表面症状)
- [ ] **运行了测试命令**，确认 0 failures
- [ ] **使用原复现步骤验证** Bug 已修复
- [ ] **提供了对比证据**: 修复前 + 修复后

### 完整模式额外检查

- [ ] lint/type 检查通过
- [ ] 代码审查通过
- [ ] (UI Bug) Playwright 修复前后截图对比

### 完成时输出总结

```
━━━ Bug 修复完成 ━━━

Bug: <简要描述>
严重程度: P?
根本原因: <一句话>
修复文件: <file:line>

修复证据:
  修复前: <证据>
  修复后: <证据>
  测试: N passed, 0 failed

阶段耗时:
  阶段 1-2 (复现): ✅
  阶段 3 (环境): ✅
  阶段 4 (调试与修复): ✅
  阶段 5 (验证): ✅
```
