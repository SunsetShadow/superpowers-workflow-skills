# TDD 修复指南

## 目录
- [核心原则](#核心原则)
- [RED-GREEN 循环](#red-green-循环)
- [常见错误](#常见错误)
- [证据输出格式](#证据输出格式)
- [无法单元测试时的替代方案](#无法单元测试时的替代方案)

## 核心原则

```
没有失败的测试 = 禁止写修复代码
```

---

## RED-GREEN 循环

### Step 1: 编写失败测试 (RED)

测试必须：
- 明确复现 Bug 的行为
- 因 Bug 功能缺失而失败（不是语法错误）
- 有清晰的断言

```javascript
// 示例：登录页崩溃 Bug
it('should show error message when username is empty', () => {
  render(<LoginPage />);
  fireEvent.click(screen.getByText('Submit'));
  // Bug: 页面崩溃，这个测试会失败
  expect(screen.getByText('请输入用户名')).toBeInTheDocument();
});
```

### Step 2: 验证 RED

运行测试，确认：
- [ ] 测试因 Bug 失败（不是语法错误）
- [ ] 错误信息与 Bug 表现一致
- [ ] 其他测试仍通过

```
❌ LoginPage › should show error message when username is empty
   TypeError: Cannot read property 'trim' of undefined
```

### Step 3: 编写最小修复 (GREEN)

只写让测试通过的最小代码，不多写一行。

```javascript
// 修复：添加空值检查
const handleSubmit = (username) => {
  if (!username?.trim()) {  // 添加空值检查
    setError('请输入用户名');
    return;
  }
  // ... 原有逻辑
};
```

### Step 4: 验证 GREEN

- [ ] 新测试通过
- [ ] 所有旧测试仍通过
- [ ] 用原复现步骤验证 Bug 已修复

---

## 常见错误

| 错误 | 修正 |
|-----|------|
| 测试因语法错误失败 | 先修复测试语法，确保测试逻辑正确 |
| 先写修复再补测试 | 删除修复，先写测试确认失败，再修复 |
| 写了多余的修复代码 | 只写让测试通过的最小代码 |
| 测试没有复现 Bug | 重写测试，确保它因 Bug 失败 |

---

## 证据输出格式

```
TDD 修复证据:

RED 阶段:
  测试: should show error message when username is empty
  结果: ❌ TypeError: Cannot read property 'trim' of undefined

GREEN 阶段:
  修改: src/components/LoginPage.js:23
  测试结果: ✓ 1 passed

回归测试:
  命令: npm test
  结果: 42 passed, 0 failed
```

---

## 无法单元测试时的替代方案

并非所有 bug 都能写传统单元测试。根据 bug 类型选择合适的验证方式：

### 判断标准

```
能否隔离为纯函数/组件级别测试？
  → 能: 使用标准 TDD 流程
  → 不能: 根据类型选择替代方案
```

### 替代验证方案

#### 1. UI 视觉 bug（样式错乱、布局偏移）

**验证方式**: 截图对比 + 组件渲染测试

```javascript
// 方案 A: 组件渲染快照
it('renders button with correct styles', () => {
  const { container } = render(<Button />);
  expect(container.firstChild).toMatchSnapshot();
});

// 方案 B: CSS 属性断言
it('applies correct margin', () => {
  render(<Button />);
  const button = screen.getByRole('button');
  expect(button).toHaveStyle({ marginTop: '8px' });
});
```

**修复证据**: 修复前截图 vs 修复后截图 + 快照测试通过

#### 2. 时序/并发问题

**验证方式**: 日志验证 + 延迟注入

```javascript
// 方案: 注入延迟强制触发竞态条件
it('handles rapid successive clicks without race condition', async () => {
  jest.spyOn(api, 'fetch').mockImplementation(() =>
    new Promise(resolve => setTimeout(() => resolve({ data: [] }), 100))
  );
  render(<SearchBox />);
  // 快速连续点击
  fireEvent.click(screen.getByText('Search'));
  fireEvent.click(screen.getByText('Search'));
  await waitFor(() => {
    expect(screen.queryByText('Error')).not.toBeInTheDocument();
  });
});
```

**修复证据**: 测试通过 + 添加日志后时序正确

#### 3. 性能问题

**验证方式**: 性能基准测试

```javascript
it('renders large list within 100ms', () => {
  const start = performance.now();
  render(<List items={Array(10000).fill(item)} />);
  const duration = performance.now() - start;
  expect(duration).toBeLessThan(100);
});
```

**修复证据**: 修复前耗时 vs 修复后耗时

#### 4. 外部依赖问题

**验证方式**: Mock + 录制回放

```javascript
// Mock 外部服务返回错误
it('handles payment gateway timeout gracefully', async () => {
  jest.spyOn(paymentGateway, 'charge').mockRejectedValue(
    new Error('ETIMEDOUT')
  );
  render(<Checkout />);
  fireEvent.click(screen.getByText('Pay'));
  await waitFor(() => {
    expect(screen.getByText('支付超时，请重试')).toBeInTheDocument();
  });
});
```

**修复证据**: Mock 测试通过 + 集成环境验证

#### 5. 配置/环境相关 bug

**验证方式**: 参数化测试

```javascript
// 测试多种配置组合
describe.each([
  ['production', 'https://api.example.com'],
  ['staging', 'https://staging.api.example.com'],
  ['development', 'http://localhost:3000'],
])('API config for %s', (env, expectedUrl) => {
  it(`uses correct API URL for ${env}`, () => {
    process.env.NODE_ENV = env;
    expect(getApiUrl()).toBe(expectedUrl);
  });
});
```

### 无法写任何测试的极端情况

如果确实无法写任何形式的自动化测试：

1. **记录手动验证步骤** — 详细的操作步骤和预期结果
2. **截图/录屏对比** — 修复前后的视觉证据
3. **添加防护日志** — 在关键路径添加日志，用于未来排查
4. **标记 TODO** — 在代码中标记需要后续补充测试的位置

```
无法测试的证据格式:

手动验证:
  步骤 1: <操作>
  步骤 2: <操作>
  预期: <结果>
  实际: 修复前 <结果> → 修复后 <结果>

截图对比:
  修复前: [截图]
  修复后: [截图]

防护措施:
  - 添加了日志: <位置>
  - TODO: 补充自动化测试
```
