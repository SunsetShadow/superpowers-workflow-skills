# Tasks JSON 模板与示例

> **加载时机**: 阶段 2 — 编写 tasks.json 前
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
      "passes": null
    },
    {
      "number": 2,
      "category": "阶段 2：[阶段名称]",
      "task": "[任务简述]",
      "steps": [
        { "step": "[具体步骤描述]", "completed": false }
      ],
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

---

## tasks.json vs Implementation Plan

两份文档互补，不要只用一个：

- **tasks.json** — 轻量跟踪层。阶段划分 + 步骤列表 + 完成状态。人类和 agent 都可快速查看进度。
- **Implementation Plan** — 详细指南层。具体代码、命令、TDD RED/GREEN/REFACTOR 步骤。指导 agent 逐条执行。

路径:
- tasks.json → `docs/changes/<change-id>/tasks.json`
- Implementation Plan → `docs/plans/YYYY-MM-DD-<feature>.md`
