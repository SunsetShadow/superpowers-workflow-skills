# Tasks JSON 模板与示例

> **加载时机**: 阶段 3 — 编写 tasks.json 前（brainstorming 完成后）
> **定位**: 轻量跟踪层 (做什么 + 完成状态)，与 Implementation Plan (怎么做) 互补

---

## 模板

```json
{
  "changeId": "<kebab-case-id>",
  "tasks": [
    {
      "number": 1,
      "category": "阶段 1：[阶段名称]",
      "task": "[任务简述]",
      "steps": [
        { "step": "[具体步骤描述]", "completed": false },
        { "step": "[具体步骤描述]", "completed": false }
      ],
      "testCases": [
        {
          "id": "TC-001",
          "type": "unit",
          "given": "[前置条件]",
          "when": "[触发动作]",
          "then": "[预期结果]"
        },
        {
          "id": "TC-002",
          "type": "e2e",
          "given": "[前置条件]",
          "when": "[触发动作]",
          "then": "[预期结果]"
        }
      ],
      "passes": null
    },
    {
      "number": 2,
      "category": "阶段 2：[阶段名称]",
      "task": "[任务简述]",
      "steps": [
        { "step": "[具体步骤描述]", "completed": false }
      ],
      "testCases": [],
      "passes": null
    }
  ]
}
```

---

## 示例 — Chat Input Tasks

```json
{
  "changeId": "add-chat-input-features",
  "tasks": [
    {
      "number": 1,
      "category": "阶段 1：基础设施",
      "task": "后端文件上传支持",
      "steps": [
        { "step": "创建 backend/app/api/v1/upload.py 上传端点", "completed": true },
        { "step": "创建 backend/app/services/upload_service.py 文件处理服务", "completed": true },
        { "step": "配置文件上传目录和大小限制", "completed": true },
        { "step": "添加文件类型验证逻辑", "completed": true }
      ],
      "testCases": [
        {
          "id": "TC-001",
          "type": "unit",
          "given": "upload_service 已初始化",
          "when": "上传一个有效的 PNG 文件 (size < 10MB)",
          "then": "返回文件 ID 和 URL"
        },
        {
          "id": "TC-002",
          "type": "unit",
          "given": "upload_service 已初始化",
          "when": "上传一个 .exe 文件",
          "then": "抛出 '不支持的文件类型' 错误"
        },
        {
          "id": "TC-003",
          "type": "e2e",
          "given": "服务运行中",
          "when": "POST /api/v1/upload 带有效 PNG 文件",
          "then": "返回 200 和附件元数据 { id, filename, type, url, size }"
        }
      ],
      "passes": true
    },
    {
      "number": 2,
      "category": "阶段 2：前端 UI",
      "task": "输入框工具栏 UI",
      "steps": [
        { "step": "在 MessageInput.vue 添加工具栏容器", "completed": true },
        { "step": "添加附件上传按钮（📎）", "completed": true },
        { "step": "添加联网开关按钮（🌐）", "completed": true },
        { "step": "添加思考过程开关按钮（🧠）", "completed": true }
      ],
      "testCases": [
        {
          "id": "TC-004",
          "type": "e2e",
          "given": "用户打开聊天页面",
          "when": "查看输入框工具栏",
          "then": "可见三个按钮: 📎 🌐 🧠"
        },
        {
          "id": "TC-005",
          "type": "e2e",
          "given": "用户在聊天页面",
          "when": "点击联网开关按钮 🌐",
          "then": "按钮变为激活状态（高亮），再次点击恢复默认"
        }
      ],
      "passes": true
    },
    {
      "number": 3,
      "category": "阶段 3：API 集成",
      "task": "聊天 API 扩展",
      "steps": [
        { "step": "扩展 SendMessageRequest schema 支持新参数", "completed": true },
        { "step": "修改 chat_service.py 处理附件", "completed": false },
        { "step": "修改 chat_service.py 处理联网搜索参数", "completed": false }
      ],
      "testCases": [
        {
          "id": "TC-006",
          "type": "unit",
          "given": "chat_service 收到带 attachments 的请求",
          "when": "处理消息",
          "then": "附件被正确解析并传入 LLM"
        },
        {
          "id": "TC-007",
          "type": "e2e",
          "given": "用户已上传附件并发送消息",
          "when": "消息发送完成",
          "then": "AI 回复包含对附件内容的分析"
        }
      ],
      "passes": false
    },
    {
      "number": 4,
      "category": "阶段 4：验证",
      "task": "功能验证",
      "steps": [
        { "step": "测试文件上传功能", "completed": true },
        { "step": "测试联网开关功能", "completed": true },
        { "step": "运行 pnpm build 确认无错误", "completed": true }
      ],
      "testCases": [],
      "passes": true
    }
  ]
}
```

---

## 任务编写规则

1. **阶段划分**: 按依赖关系分阶段 (基础设施 → 功能 → 集成 → 验证)
2. **步骤粒度**: 每个步骤是一个可独立验证的操作
3. **初始状态**: 新建时 `completed` 全为 `false`，`passes` 为 `null`
4. **完成标准**: 一个 task 所有 `steps.completed = true` 时，标记 `passes: true`
5. **JSON 格式**: 使用 2 空格缩进，保证可读性
6. **testCases 字段**: 每个任务定义测试案例，分 unit 和 e2e 两种类型

## testCases 编写规则

1. **id 命名**: `TC-NNN` 格式，全局唯一，递增编号
2. **type**: `unit`（单元测试，阶段 4 TDD 中覆盖）或 `e2e`（端到端，阶段 5 Playwright 覆盖）
3. **given/when/then**: BDD 风格，每个用例必须覆盖一个明确的场景
4. **覆盖要求**: 每个任务至少包含一个 `unit` 用例；Web 应用功能至少包含一个 `e2e` 用例
5. **纯验证类任务**（如"功能验证"）可不写 testCases，其验证由依赖任务的 e2e 用例覆盖

---

## tasks.json vs Implementation Plan

两份文档互补，不要只用一个：

- **tasks.json** — 轻量跟踪层 + 测试案例层。阶段划分 + 步骤列表 + 完成状态 + testCases（unit/e2e）。人类和 agent 都可快速查看进度和验证标准。
- **Implementation Plan** — 详细指南层。具体代码、命令、TDD RED/GREEN/REFACTOR 步骤。指导 agent 逐条执行。

路径:
- tasks.json → `docs/changes/<change-id>/tasks.json`
- Implementation Plan → `docs/plans/YYYY-MM-DD-<feature>.md`

testCases 消费方式:
- `type: unit` → 阶段 4 执行技能的 TDD 循环中覆盖
- `type: e2e` → 阶段 5 Playwright 功能验证中覆盖
