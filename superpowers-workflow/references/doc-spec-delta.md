# Spec Delta 模板与示例

> **加载时机**: 阶段 1 (完整变更模式) — 编写 spec delta 前
> **注意**: Spec Delta 使用 BDD 风格 (GIVEN/WHEN/THEN)，每个需求至少包含一个正常场景和一个异常场景

---

## 模板

```markdown
# Spec Delta: [功能名称]

**Change ID**: `<kebab-case-id>`
**Target Spec**: `docs/specs/<area>/spec.md`

---

## ADDED Requirements

### Requirement: [需求名称]
WHEN [触发条件],
系统 SHALL [预期行为]。

#### Scenario: [正常场景名称]
GIVEN [前置条件]
WHEN [触发动作]
THEN [预期结果]
AND [附加验证]

#### Scenario: [异常场景名称] (边界/异常场景必须包含)
GIVEN [前置条件]
WHEN [触发异常动作]
THEN [错误提示]
AND [系统行为]

## MODIFIED Requirements

### Requirement: [修改的现有需求]
WHEN [触发条件],
系统 SHALL [新行为]。

#### Scenario: [兼容场景]
GIVEN [旧条件]
WHEN [触发动作]
THEN [兼容行为]

---

## API Changes

### METHOD /api/path (新增/修改)

[接口描述]

**请求**:
- Content-Type: `application/json`
- Body:

    {"field": "value"}

**响应**:

    {"id": "xxx", "result": "value"}
```

---

## 示例 — Chat Input Spec Delta

```markdown
# Spec Delta: Chat Input Features

**Change ID**: `add-chat-input-features`
**Target Spec**: `openspec/specs/core-features/spec.md`

---

## ADDED Requirements

### Requirement: 附件上传
WHEN 用户点击附件按钮并选择文件,
系统 SHALL 上传文件并显示预览。

#### Scenario: 上传图片
GIVEN 用户在聊天输入框
WHEN 用户点击附件按钮并选择一张图片
THEN 系统验证文件类型和大小
AND 上传文件到服务器
AND 显示图片缩略图预览
AND 附件与下一条消息一起发送

#### Scenario: 文件类型不支持
GIVEN 用户在聊天输入框
WHEN 用户选择不支持的文件类型（如 .exe）
THEN 系统显示错误提示 "不支持的文件类型"
AND 不上传文件

#### Scenario: 文件大小超限
GIVEN 用户在聊天输入框
WHEN 用户选择超过 10MB 的文件
THEN 系统显示错误提示 "文件大小超过限制（最大 10MB）"
AND 不上传文件

### Requirement: 联网搜索
WHEN 用户开启联网搜索开关并发送消息,
系统 SHALL 在 AI 回答前搜索互联网获取最新信息。

#### Scenario: 开启联网搜索
GIVEN 用户在聊天输入框
WHEN 用户点击联网按钮（🌐）
THEN 按钮变为激活状态（高亮）
AND 下一条消息将启用联网搜索

#### Scenario: 联网搜索消息处理
GIVEN 联网搜索已开启
WHEN 用户发送消息
THEN 系统在请求中包含 `web_search: true` 参数
AND AI 使用互联网信息回答问题

## MODIFIED Requirements

### Requirement: 发送消息 API
WHEN 用户发送带有附件或功能开关的消息,
系统 SHALL 将所有参数传递给后端。

#### Scenario: 发送带附件的消息
GIVEN 用户已上传附件
WHEN 用户点击发送按钮
THEN 系统发送消息内容包括：
  - `content`: 文本内容
  - `attachments`: 附件 ID 列表
  - `web_search`: 联网搜索开关状态
  - `thinking`: 思考过程开关状态
AND 清空输入框和附件预览

#### Scenario: 发送纯文本消息 (兼容)
GIVEN 用户未上传附件且所有开关关闭
WHEN 用户点击发送按钮
THEN 系统发送消息内容仅包含 `content` 字段
AND 行为与现有实现一致

---

## API Changes

### POST /api/v1/upload (新增)

上传文件并返回附件 ID。

**请求**:
- Content-Type: `multipart/form-data`
- Body: `file` (文件)

**响应**:

    {"id": "att_abc123", "filename": "document.pdf", "type": "document", "url": "/api/v1/upload/att_abc123", "size": 102400}

### POST /api/v1/chat/completions (修改)

扩展请求体：

    {"content": "请分析这个文档", "attachments": ["att_abc123"], "web_search": true, "thinking": true}
```
