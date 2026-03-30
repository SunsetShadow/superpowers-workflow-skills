# Bug 复现方法指南

## 目录
- [选择复现方式](#选择复现方式)
- [方案 A: UI Bug — 浏览器自动化](#方案-a-ui-bug--浏览器自动化)
- [方案 B: API/后端 Bug — API 调用](#方案-b-api后端-bug--api-调用复现)
- [方案 C: 逻辑 Bug — 单元测试](#方案-c-逻辑-bug--单元测试复现)
- [方案 D: 间歇性 Bug — 脚本循环](#方案-d-间歇性-bug--脚本循环复现)
- [快速判定流程](#快速判定流程)

## 选择复现方式

```
Bug 类型判定:
  ├── UI 相关 (页面显示、交互、样式)
  │   └── → 方案 A: 浏览器自动化复现
  ├── API/后端 (接口错误、数据异常)
  │   └── → 方案 B: API 调用复现
  ├── 逻辑/计算 (结果不正确)
  │   └── → 方案 C: 单元测试复现
  └── 间歇性 (偶发、时序相关)
      └── → 方案 D: 脚本循环复现
```

---

## 方案 A: UI Bug — 浏览器自动化

**适用**: 页面崩溃、样式错乱、交互异常、显示错误

### 判定标准

- 涉及浏览器渲染
- 需要用户交互触发
- 与页面布局/样式相关
- 需要截图作为证据

### 工具选择

参考 SKILL.md "Web 验证工具选择" 章节选择工具，选定后本阶段统一使用。

**默认 Playwright MCP**: `browser_navigate` → `browser_snapshot` → `browser_take_screenshot`
**切换 agent-browser**: 需要登录态、复杂链式操作时调用 `Skill(skill="agent-browser")`

```
[工具选择]
  UI bug → 参考"Web 验证工具选择"章节
  判定: 使用 <Playwright MCP / agent-browser>
```

### 复现步骤

1. **打开目标页面**
   ```
   # Playwright MCP
   browser_navigate <url>
   browser_snapshot

   # 或 agent-browser
   agent-browser open <url>
   agent-browser snapshot
   ```

2. **执行触发操作**（根据用户提供的复现步骤）
   ```
   # Playwright MCP
   browser_click ref=<ref>
   browser_type ref=<ref> text="<value>"
   browser_snapshot

   # 或 agent-browser
   agent-browser click @<element>
   agent-browser fill @<element> "<value>"
   agent-browser snapshot
   ```

3. **确认 Bug 出现**
   - 截图保存 (`browser_take_screenshot` 或 `agent-browser screenshot`)
   - 检查控制台错误 (`browser_console_messages` 或 `agent-browser console`)
   - 记录错误信息

4. **多次复现验证**
   - 重复步骤 1-3 至少 2 次
   - 确认每次都能复现

### 证据格式

```
UI Bug 复现证据:

步骤:
  1. 打开 https://example.com/login
  2. 输入用户名: (空)
  3. 点击"提交"

结果:
  期望: 显示"请输入用户名"提示
  实际: 页面崩溃，白屏

控制台错误:
  TypeError: Cannot read property 'trim' of undefined
    at handleSubmit (LoginPage.js:23)

截图:
  复现截图: [保存路径]

复现稳定性: 3/3 次
```

### 浏览器工具不可用时

如果浏览器工具加载失败：
1. 请用户手动操作并提供截图
2. 使用 curl/fetch 测试相关 API
3. 编写组件级渲染测试（如 Jest + React Testing Library）

---

## 方案 B: API/后端 Bug — API 调用复现

**适用**: 接口返回错误、数据异常、权限问题、服务端崩溃

### 判定标准

- 涉及 HTTP 接口
- 服务端日志有错误
- 数据库状态异常
- 不依赖浏览器渲染

### 复现步骤

1. **确认 API 端点和参数**
   ```bash
   # 使用 curl 复现
   curl -X POST https://api.example.com/orders \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <token>" \
     -d '{"items": [], "coupon": "INVALID"}'
   ```

2. **检查响应**
   ```
   期望: 400 Bad Request, {"error": "购物车为空"}
   实际: 500 Internal Server Error
   ```

3. **查看服务端日志**
   ```bash
   # 查看相关日志
   grep "orders" /var/log/app.log | tail -20
   ```

### 证据格式

```
API Bug 复现证据:

请求:
  POST /api/orders
  Body: {"items": [], "coupon": "INVALID"}
  Headers: Authorization: Bearer xxx

响应:
  状态码: 500 (期望: 400)
  Body: Internal Server Error

服务端日志:
  TypeError: Cannot read property 'length' of undefined
    at validateOrder (order-service.js:45)

复现稳定性: 3/3 次
```

---

## 方案 C: 逻辑 Bug — 单元测试复现

**适用**: 计算错误、边界条件、数据处理异常

### 判定标准

- 纯函数/方法级别的问题
- 输入输出可明确界定
- 不依赖外部服务

### 复现步骤

直接编写失败测试复现 bug：

```javascript
it('calculates discount correctly for edge case', () => {
  // Bug: 金额为 0 时返回 NaN
  const result = calculateDiscount(0, 0.1);
  expect(result).toBe(0);  // 实际得到 NaN
});
```

### 证据格式

```
逻辑 Bug 复现证据:

测试:
  calculateDiscount(0, 0.1) 期望返回 0
  实际返回 NaN

失败信息:
  Expected: 0
  Received: NaN

涉及代码: src/utils/discount.js:15
```

---

## 方案 D: 间歇性 Bug — 脚本循环复现

**适用**: 竞态条件、时序问题、偶发崩溃

### 判定标准

- 不是每次都出现
- 与时序/并发相关
- 需要多次执行才能复现

### 复现步骤

1. **编写循环脚本**
   ```bash
   # 循环执行，捕获偶发错误
   for i in $(seq 1 50); do
     result=$(curl -s -X POST http://localhost:3000/api/transfer \
       -H "Content-Type: application/json" \
       -d '{"from": "A", "to": "B", "amount": 100}')
     if echo "$result" | grep -q "error"; then
       echo "第 $i 次失败: $result"
     fi
   done
   ```

2. **或并发测试**
   ```bash
   # 使用 ab 或 wrk 进行并发测试
   ab -n 100 -c 10 -p data.json -T application/json \
     http://localhost:3000/api/transfer
   ```

### 证据格式

```
间歇性 Bug 复现证据:

测试方法: 循环执行 50 次
失败次数: 3/50 (6%)
失败模式: 并发请求时出现

失败日志:
  第 17 次失败: {"error": "余额不足"}
  第 23 次失败: {"error": "余额不足"}
  第 41 次失败: {"error": "余额不足"}

分析: 竞态条件，两个并发请求同时读取余额
```

---

## 快速判定流程

在阶段 2 开始时，按以下问题快速判定复现方式：

```
问: Bug 涉及页面显示或浏览器交互吗？
  是 → 方案 A (浏览器自动化)
  否 ↓

问: Bug 涉及 API 接口或服务端处理吗？
  是 → 方案 B (API 调用)
  否 ↓

问: Bug 是纯逻辑/计算错误吗？
  是 → 方案 C (单元测试)
  否 ↓

问: Bug 不是每次都出现吗？
  是 → 方案 D (循环/并发)
  否 → 根据上下文选择最接近的方案
```
