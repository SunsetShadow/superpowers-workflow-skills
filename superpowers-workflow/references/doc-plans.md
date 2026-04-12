# 设计文档与实施计划模板

> **加载时机**: 阶段 1 (简单需求) 加载设计文档部分，阶段 2 加载实施计划部分
> **注意**: 示例中的技术栈和路径来自参考项目，使用时替换为当前项目的实际路径

---

## 目录

1. [设计文档模板 (简单需求)](#设计文档模板)
2. [设计文档示例 — Health Check](#设计文档示例)
3. [实施计划模板](#实施计划模板)
4. [实施计划示例 — Health Check](#实施计划示例)

---

## 设计文档模板

> 当变更较简单时 (bug 修复、小调整、重构) 使用此模板替代完整 Proposal + Spec Delta。
> 路径: `docs/specs/<area>/YYYY-MM-DD-<topic>-design.md`

```markdown
# [功能名称] 设计文档

> **For agentic workers:** 本文档经用户审阅批准后方可进入实施阶段。

**Change ID**: `<kebab-case-id>`
**Created**: YYYY-MM-DD
**Status**: Draft

---

## 目的

[一段话说明这个功能要解决什么问题]

## 技术约束

- [约束 1]
- [约束 2]

## 方案

### 方案 A: [名称] (推荐)

**描述**: [2-3 句话]

**优点**:
- [优点 1]

**缺点**:
- [缺点 1]

### 方案 B: [名称]

**描述**: [2-3 句话]

**优点/缺点**: [简述]

## 实施要点

1. [关键步骤 1]
2. [关键步骤 2]

## 验收标准

- [ ] [可验证的标准 1] — GIVEN [上下文] WHEN [操作] THEN [预期结果]
- [ ] [可验证的标准 2] — GIVEN [上下文] WHEN [操作] THEN [预期结果]
```

---

## 设计文档示例

```markdown
# Health Check 端点设计文档

> **For agentic workers:** 本文档经用户审阅批准后方可进入实施阶段。

**Change ID**: `add-health-check`
**Created**: 2026-04-12
**Status**: Draft

---

## 目的

NestJS 后端缺少健康检查端点，K8s/Docker 部署时无法探测服务存活状态。需要一个 `/health` 端点返回服务和数据库连接状态。

## 技术约束

- 必须兼容 NestJS 已有模块结构
- 响应时间 < 500ms（含数据库探测）
- 不引入新依赖

## 方案

### 方案 A: 独立 Controller (推荐)

**描述**: 新建 `HealthController` + `HealthService`，通过 TypeORM 的 DataSource 检测数据库连接。

**优点**:
- 无新依赖
- 与现有代码风格一致
- 可扩展（未来加 Redis、外部 API 检测）

**缺点**:
- 手动实现数据库探测逻辑

### 方案 B: @nestjs/terminus

**描述**: 使用官方 terminus 模块提供标准健康检查。

**优点/缺点**: 功能更全但引入新依赖，当前只需基础检查没必要。

## 实施要点

1. 在 `HealthModule` 中注册 Controller 和 Service
2. Service 通过注入 DataSource 执行 `SELECT 1` 探测
3. 返回 `{ status: "ok", db: "connected" }` 或 `{ status: "error", db: "disconnected" }`

## 验收标准

- [ ] GET /health 返回 200 — GIVEN 服务正常运行 WHEN 请求 /health THEN 返回 `{"status":"ok","db":"connected"}`
- [ ] 数据库断连时返回 503 — GIVEN 数据库不可用 WHEN 请求 /health THEN 返回 503 和 `{"status":"error","db":"disconnected"}`
```

---

## 实施计划模板

> 对应工作流阶段 2 的输出。每个 task 是 2-5 分钟可完成的原子操作。
> 路径: `docs/plans/YYYY-MM-DD-<feature>.md`

    # [Feature Name] Implementation Plan

    > **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

    **Goal:** [一句话描述目标]

    **Architecture:** [2-3 句话描述架构方案]

    **Tech Stack:** [关键技术/库]

    ---

    ### Task N: [组件名称]

    **Files:**
    - Create: `exact/path/to/file.py`
    - Modify: `exact/path/to/existing.py:123-145`
    - Test: `tests/exact/path/to/test.py`

    - [ ] **Step 1: Write the failing test**

    [代码]

    - [ ] **Step 2: Run test to verify it fails**

    Run: `pytest tests/path/test.py::test_name -v`
    Expected: FAIL with "function not defined"

    - [ ] **Step 3: Write minimal implementation**

    [代码]

    - [ ] **Step 4: Run test to verify it passes**

    Run: `pytest tests/path/test.py::test_name -v`
    Expected: PASS

    - [ ] **Step 5: Commit**

        git add tests/path/test.py src/path/file.py
        git commit -m "feat: add specific feature"

---

## 实施计划示例

    # Health Check Implementation Plan

    > **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

    **Goal:** 添加 /health 端点，返回服务和数据库连接状态

    **Architecture:** 新建 HealthModule 包含 Controller 和 Service，Service 通过 TypeORM DataSource 探测数据库

    **Tech Stack:** NestJS, TypeORM

    ---

    ### Task 1: HealthService

    **Files:**
    - Create: `src/health/health.service.ts`
    - Create: `src/health/health.service.spec.ts`

    - [ ] **Step 1: Write the failing test**

        describe('HealthService', () => {
          it('should return ok when db is connected', async () => {
            const service = new HealthService(mockDataSource);
            const result = await service.check();
            expect(result.status).toBe('ok');
            expect(result.db).toBe('connected');
          });
        });

    - [ ] **Step 2: Run test to verify it fails**

    Run: `npx jest src/health/health.service.spec.ts`
    Expected: FAIL

    - [ ] **Step 3: Write minimal implementation**

        @Injectable()
        export class HealthService {
          constructor(private dataSource: DataSource) {}
          async check() {
            await this.dataSource.query('SELECT 1');
            return { status: 'ok', db: 'connected' };
          }
        }

    - [ ] **Step 4: Run test to verify it passes**

    Run: `npx jest src/health/health.service.spec.ts`
    Expected: PASS

    - [ ] **Step 5: Commit**

        git add src/health/health.service.ts src/health/health.service.spec.ts
        git commit -m "feat(health): add HealthService with db check"

    ### Task 2: HealthController

    **Files:**
    - Create: `src/health/health.controller.ts`
    - Create: `src/health/health.controller.spec.ts`

    - [ ] **Step 1: Write the failing test**

        describe('HealthController', () => {
          it('GET /health returns 200 when healthy', async () => {
            const result = await request(app.getHttpServer())
              .get('/health');
            expect(result.status).toBe(200);
            expect(result.body).toEqual({ status: 'ok', db: 'connected' });
          });
        });

    - [ ] **Step 2: Run test to verify it fails**

    Run: `npx jest src/health/health.controller.spec.ts`
    Expected: FAIL (route /health not found)

    - [ ] **Step 3: Write minimal implementation**

        @Controller('health')
        export class HealthController {
          constructor(private healthService: HealthService) {}
          @Get()
          async check() { return this.healthService.check(); }
        }

    - [ ] **Step 4: Run test to verify it passes**

    Run: `npx jest src/health/health.controller.spec.ts`
    Expected: PASS

    - [ ] **Step 5: Commit**

        git add src/health/health.controller.ts src/health/health.controller.spec.ts
        git commit -m "feat(health): add HealthController with GET /health"

    ### Task 3: Module Registration

    **Files:**
    - Create: `src/health/health.module.ts`
    - Modify: `src/app.module.ts`

    - [ ] **Step 1: Create HealthModule and register in AppModule imports**
    - [ ] **Step 2: Verify endpoint** — `curl http://localhost:3000/health`
    - [ ] **Step 3: Commit**

---

## 路径映射规则

| 工作流阶段 | 产出文档 | 路径格式 |
|-----------|---------|---------|
| 阶段 1 (完整变更) | Proposal + Spec Delta | `docs/changes/<change-id>/proposal.md` + `specs/<area>/spec-delta.md` |
| 阶段 1 (简单需求) | Design Document | `docs/specs/<area>/YYYY-MM-DD-<topic>-design.md` |
| 阶段 2 | Tasks JSON + Implementation Plan | `docs/changes/<change-id>/tasks.json` + `docs/plans/YYYY-MM-DD-<feature>.md` |
| 持续维护 | Reference Materials | `docs/references/<topic>.md` |

**用户偏好路径优先**: 如果 CLAUDE.md 或用户指定了其他文档路径，以用户偏好为准。
