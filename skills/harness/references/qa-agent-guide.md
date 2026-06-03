# QA 智能体设计指南

构建 Harness（Harness 工具套件）时，若需在流水线中包含 QA 智能体，可参考本指南。基于实际项目（SatangSlide）中发现的 Bug 模式及其根本原因分析，提供系统地捕获 QA 容易遗漏缺陷的验证方法论。

---

## 目录

1. QA 智能体遗漏缺陷的模式
2. 集成一致性验证（Integration Coherence Verification）
3. QA 智能体设计原则
4. 验证清单模板
5. QA 智能体定义模板

---

## 1. QA 智能体遗漏缺陷的模式

### 1-1. 接口边界不一致（Boundary Mismatch）

最频繁出现的缺陷。两个组件各自"正确"实现，但在连接点上契约不一致。

| 接口边界 | 不一致示例 | 遗漏原因 |
|---------|-----------|---------|
| API 响应 → 前端 hook | API 返回 `{ projects: [...] }`，hook 期望 `SlideProject[]` | 各自单独验证时正常，未做交叉比较 |
| API 响应字段名 → 类型定义 | API 为 `thumbnailUrl`（camelCase），类型为 `thumbnail_url`（snake_case） | 用 TypeScript 泛型强制转换时编译器无法捕获 |
| 文件路径 → 链接 href | 页面位于 `/dashboard/create`，但链接指向 `/create` | 未交叉比较文件结构与 href |
| 状态转移映射 → 实际 status 更新 | 映射中定义了 `generating_template → template_approved`，代码中缺失该转移 | 只检查映射是否存在，未跟踪所有更新代码 |
| API 端点 → 前端 hook | API 存在但无对应 hook（未被调用） | API 列表与 hook 列表未做 1:1 映射 |
| 立即响应 → 异步结果 | API 立即返回 `{ status }`，前端访问 `data.failedIndices` | 不区分同步/异步响应，仅检查类型 |

### 1-2. 为什么静态代码审查无法捕获

- **TypeScript 泛型的局限**：`fetchJson<SlideProject[]>()` —— 即使运行时响应是 `{ projects: [...] }`，编译仍然通过
- **`npm run build` 通过 ≠ 正常运行**：一旦使用类型强制转换、`any`、泛型，构建成功但运行时会失败
- **存在性验证 vs 连接性验证的差异**："API 是否存在？"和"API 的响应是否与调用方期望一致？"是完全不同的验证

---

## 2. 集成一致性验证（Integration Coherence Verification）

QA 智能体中必须包含的**交叉比较验证**领域。

### 2-1. API 响应 ↔ 前端 hook 类型交叉验证

**方法**：比较每个 API route 的 `NextResponse.json()` 调用处与对应 hook 的 `fetchJson<T>` 类型参数。

```
验证步骤：
1. 从 API route 的 NextResponse.json() 调用中提取对象 shape
2. 在对应 hook 中确认 fetchJson<T> 的 T 类型
3. 比较 shape 与 T 是否一致
4. 检查是否存在包装（若 API 返回 { data: [...] }，hook 是否解出 .data）
```

**需要特别关注的模式：**
- 分页 API：`{ items: [], total, page }` vs 前端期望数组
- snake_case DB 字段 → camelCase API 响应 → 前端类型定义 之间不一致
- 立即响应（202 Accepted）与最终结果的 shape 差异

### 2-2. 文件路径 ↔ 链接/路由路径映射

**方法**：从 `src/app/` 下的 page 文件提取 URL 路径，并与代码中所有的 `href`、`router.push()`、`redirect()` 值进行对照。

```
验证步骤：
1. 从 src/app/ 下的 page.tsx 文件路径中提取 URL 模式
   - (group) → 从 URL 中移除
   - [param] → 动态段
2. 收集代码中所有 href=、router.push(、redirect( 的值
3. 确认每条链接是否匹配真实存在的 page 路径
4. 注意 route group 内部页面的 URL 前缀（例如：dashboard/ 下）
```

### 2-3. 状态转移完整性追踪

**方法**：从代码中提取所有 `status:` 更新并与状态转移映射对照。

```
验证步骤：
1. 从状态转移映射（STATE_TRANSITIONS）中提取允许的转移列表
2. 在所有 API route 中搜索 .update({ status: "..." }) 模式
3. 确认每个转移是否已在映射中定义
4. 识别映射中已定义但代码中未执行的转移（死转移）
5. 特别注意：中间状态（如 generating_template）到最终状态（template_approved）的转移是否缺失
```

### 2-4. API 端点 ↔ 前端 hook 1:1 映射

**方法**：列出所有 API route 与前端 hook，确认是否成对。

```
验证步骤：
1. 从 src/app/api/ 下的 route.ts 中按 HTTP 方法提取端点列表
2. 从 src/hooks/ 下的 use*.ts 中提取 fetch 调用的 URL 列表
3. 识别未被 hook 调用的 API 端点 → 标记为"未使用"
4. 判断"未使用"是出于设计意图（管理 API 等）还是调用遗漏
```

---

## 3. QA 智能体设计原则

### 3-1. 使用 general-purpose 类型而非 Explore 类型

如果 QA 智能体是 `Explore` 类型则只能读取。但有效的 QA 还需要：
- 用 Grep 搜索模式（提取所有 `NextResponse.json()`）
- 通过脚本执行自动对照（API shape vs hook 类型）
- 必要时可直接修改

**建议**：配置为 `general-purpose` 类型，并在智能体定义中明确"验证 → 报告 → 修复请求"协议。

### 3-2. 清单应优先"交叉比较"而非"存在性确认"

| 弱清单 | 强清单 |
|------|------|
| API 端点是否存在？ | API 端点的响应 shape 与对应 hook 的类型是否一致？ |
| 状态转移映射是否定义？ | 所有 status 更新代码是否与映射中的转移一致？ |
| 页面文件是否存在？ | 代码中所有链接是否指向真实存在的页面？ |
| 是否启用 TypeScript strict mode？ | 是否存在被泛型强制转换绕过的类型安全性问题？ |

### 3-3. "两侧同时读取"原则

QA 要捕获接口边界 Bug，不能只读一侧。必须：
- 同时读取 API route **和**对应 hook
- 同时读取状态转移映射 **和**实际更新代码
- 同时读取文件结构 **和**链接路径

将此原则明确写入智能体定义中。

### 3-4. QA 应该在每个模块完成后立即运行，而非构建完成之后

如果编排器只把 QA 放在"Phase 4：全部完成后"：
- Bug 不断累积，修复成本上升
- 早期的接口边界不一致会传播到后续模块

**推荐模式**：每个后端 API 完成后立即执行该 API + 对应 hook 的交叉验证（增量式 QA）。

---

## 4. 验证清单模板

需纳入 QA 智能体定义的 Web 应用集成一致性清单。

```markdown
### 集成一致性验证（Web 应用）

#### API ↔ 前端连接
- [ ] 所有 API route 的响应 shape 与对应 hook 的泛型类型一致
- [ ] 包装后的响应（{ items: [...] }）在 hook 中正确解包
- [ ] snake_case ↔ camelCase 转换一致地应用
- [ ] 立即响应（202）与最终结果的 shape 在前端被正确区分
- [ ] 所有 API 端点都有对应的前端 hook，且确实被调用

#### 路由一致性
- [ ] 代码中所有 href/router.push 值与真实 page 文件路径匹配
- [ ] 考虑 route group ((group)) 在 URL 中被移除后的路径验证
- [ ] 动态段（[id]）是否被正确的参数填充

#### 状态机一致性
- [ ] 所有已定义的状态转移在代码中都被执行（无死转移）
- [ ] 代码中所有 status 更新都在转移映射中定义（无未授权转移）
- [ ] 中间状态到最终状态的转移没有遗漏
- [ ] 前端中基于状态的条件分支（if status === "X"）中的 X 真实可达

#### 数据流一致性
- [ ] DB schema 字段名与 API 响应字段名的映射保持一致
- [ ] 前端类型定义与 API 响应的字段名一致
- [ ] 可选字段的 null/undefined 处理在两侧保持一致
```

---

## 5. QA 智能体定义模板

构建 Harness 工具套件的 QA 智能体需包含的核心章节。

```markdown
---
name: qa-inspector
description: "QA 验证专家。验证规格符合度、集成一致性、设计质量。"
---

# QA Inspector

## 核心角色
对照规格验证实现质量与**模块间集成一致性**。

## 验证优先级

1. **集成一致性**（最高）—— 接口边界不一致是运行时错误的主要原因
2. **功能规格符合度** —— API / 状态机 / 数据模型
3. **设计质量** —— 颜色 / 字体 / 响应式
4. **代码质量** —— 未使用代码、命名规范

## 验证方法："两侧同时读取"

接口边界验证必须**同时打开两侧代码**进行比较：

| 验证对象 | 左侧（生产者） | 右侧（消费者） |
|--------|--------------|--------------|
| API 响应 shape | route.ts 的 NextResponse.json() | hooks/ 的 fetchJson<T> |
| 路由 | src/app/ page 文件路径 | href, router.push 值 |
| 状态转移 | STATE_TRANSITIONS 映射 | .update({ status }) 代码 |
| DB → API → UI | 表列名 | API 响应字段 → 类型定义 |

## 团队通信协议

- 一经发现立即向对应智能体发起具体修改请求（文件:行 + 修改方法）
- 接口边界问题需通知两侧智能体
- 向领导汇报：验证报告（区分通过/失败/未验证项）
```

---

## 实际案例：SatangSlide 中发现的 Bug

本指南所有内容均提炼自以下真实 Bug：

| Bug | 接口边界 | 原因 |
|-----|---------|------|
| `projects?.filter is not a function` | API→hook | API 返回 `{projects:[]}`，hook 期望数组 |
| 仪表盘所有链接 404 | 文件路径→href | 缺少 `/dashboard/` 前缀 |
| 主题图片不显示 | API→组件 | `thumbnailUrl` vs `thumbnail_url` |
| 主题选择未保存 | API→hook | select-theme API 存在，无 hook |
| 生成页面永久等待 | 状态转移→代码 | `template_approved` 转移代码缺失 |
| `data.failedIndices` 崩溃 | 立即响应→前端 | 在立即响应中访问后台结果 |
| 完成后查看幻灯片 404 | 文件路径→href | `/projects/` → `/dashboard/projects/` |
