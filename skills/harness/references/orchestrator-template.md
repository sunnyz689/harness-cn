# 编排器技能模板

编排器是统筹整个团队的顶层技能。按执行模式提供 3 种模板：

- **模板 A：智能体团队模式（默认）** — 2 人及以上协作时的首选
- **模板 B：子智能体模式（备选）** — 无需团队通信的场景
- **模板 C：混合模式** — 跨 Phase 混合使用不同模式

---

## 模板 A：智能体团队模式（默认 · 首选）

当 2 个及以上智能体协作时**优先考虑的默认模式**。使用 `TeamCreate` 组建团队，通过共享任务列表与 `SendMessage` 进行协调。

```markdown
---
name: {domain}-orchestrator
description: "{도메인} 에이전트 팀을 조율하는 오케스트레이터. {초기 실행 키워드}. 후속 작업: {도메인} 결과 수정, 부분 재실행, 업데이트, 보완, 다시 실행, 이전 결과 개선 요청 시에도 반드시 이 스킬을 사용."
---

# {领域} 编排器

统一技能：协调 {领域} 的智能体团队，产出 {最终产物}。

## 执行模式：智能体团队

## 智能体构成

| 成员 | 智能体类型 | 角色 | 技能 | 输出 |
|------|-----------|------|------|------|
| {teammate-1} | {自定义或内置} | {角色} | {skill} | {output-file} |
| {teammate-2} | {自定义或内置} | {角色} | {skill} | {output-file} |
| ... | | | | |

## 工作流

### Phase 0：上下文确认（支持后续作业）

通过检查既有产物是否存在，决定执行模式：

1. 检查 `_workspace/` 目录是否存在
2. 决定执行模式：
   - **`_workspace/` 不存在** → 初始执行。进入 Phase 1
   - **`_workspace/` 存在 + 用户请求部分修改** → 部分重跑。仅重新调用指定的智能体，覆盖既有产物中需要修改的部分
   - **`_workspace/` 存在 + 提供新输入** → 全新执行。将既有 `_workspace/` 移动到 `_workspace_{YYYYMMDD_HHMMSS}/`，再进入 Phase 1
3. 部分重跑时：在智能体提示词中包含先前产物的路径，指示智能体读取既有结果并吸纳反馈

### Phase 1：准备
1. 分析用户输入 — {需要厘清什么}
2. 在工作目录创建 `_workspace/`
   - **初始执行**：创建新的 `_workspace/`
   - **全新执行**：将既有 `_workspace/` 移动到 `_workspace_{YYYYMMDD_HHMMSS}/` 后，立即重建新的 `_workspace/`
3. 将输入数据保存到 `_workspace/00_input/`

### Phase 2：组建团队

1. 创建团队：
   ```
   TeamCreate(
     team_name: "{domain}-team",
     members: [
       { name: "{teammate-1}", agent_type: "{type}", model: "opus", prompt: "{역할 설명 및 작업 지시}" },
       { name: "{teammate-2}", agent_type: "{type}", model: "opus", prompt: "{역할 설명 및 작업 지시}" },
       ...
     ]
   )
   ```

2. 注册任务：
   ```
   TaskCreate(tasks: [
     { title: "{작업1}", description: "{상세}", assignee: "{teammate-1}" },
     { title: "{작업2}", description: "{상세}", assignee: "{teammate-2}" },
     { title: "{작업3}", description: "{상세}", depends_on: ["{작업1}"] },
     ...
   ])
   ```

   > 每位成员 5~6 个任务较为合适。存在依赖关系的任务通过 `depends_on` 显式声明。

### Phase 3：{主要作业 — 例如：调研/生成/分析}

**执行方式：** 团队成员自行协调

团队成员从共享任务列表中认领（claim）任务并独立执行。
领导负责监控进度，必要时介入。

**团队成员间的通信规则：**
- {teammate-1} 通过 SendMessage 将 {什么信息} 传递给 {teammate-2}
- {teammate-2} 完成作业后将结果保存为文件并通知领导
- 团队成员如需其他成员的结果，通过 SendMessage 请求

**产物保存：**

| 成员 | 输出路径 |
|------|----------|
| {teammate-1} | `_workspace/{phase}_{teammate-1}_{artifact}.md` |
| {teammate-2} | `_workspace/{phase}_{teammate-2}_{artifact}.md` |

**领导监控：**
- 团队成员进入空闲状态时会自动收到通知
- 特定成员卡住时，通过 SendMessage 指示或重新分配任务
- 通过 TaskGet 查看整体进度

### Phase 4：{后续作业 — 例如：验证/整合}
1. 等待所有成员任务完成（通过 TaskGet 确认状态）
2. 用 Read 收集每位成员的产物
3. {整合/验证逻辑}
4. 生成最终产物：`{output-path}/{filename}`

### Phase 5：收尾
1. 向团队成员发出终止请求（SendMessage）
2. 解散团队（TeamDelete）
3. 保留 `_workspace/` 目录（不删除中间产物 — 用于事后验证与审计追溯）
4. 向用户汇报结果摘要

> **需要重组团队时：** 若不同 Phase 需要不同的专家组合，先用 TeamDelete 解散当前团队，再用 TeamCreate 组建下一 Phase 的团队。既有团队的产物保留在 `_workspace/`，新团队可通过 Read 访问。

## 数据流

```
[领导] → TeamCreate → [teammate-1] ←SendMessage→ [teammate-2]
                          │                           │
                          ↓                           ↓
                    artifact-1.md              artifact-2.md
                          │                           │
                          └───────── Read ────────────┘
                                     ↓
                              [领导：整合]
                                     ↓
                              最终产物
```

## 错误处理

| 情况 | 策略 |
|------|------|
| 1 名成员失败/停止 | 领导检测到 → 通过 SendMessage 确认状态 → 重启或创建替代成员 |
| 过半成员失败 | 通知用户并确认是否继续 |
| 超时 | 使用截至当前已收集的部分结果，终止未完成成员 |
| 成员间数据冲突 | 注明来源后并列保留，不删除 |
| 任务状态延迟 | 领导通过 TaskGet 确认后手动 TaskUpdate |

## 测试场景

### 正常流程
1. 用户提供 {输入}
2. 在 Phase 1 得出 {分析结果}
3. 在 Phase 2 组建团队（{N} 名成员 + {M} 个任务）
4. 在 Phase 3 团队成员自行协调完成作业
5. 在 Phase 4 整合产物，生成最终结果
6. 在 Phase 5 解散团队
7. 预期结果：生成 `{output-path}/{filename}`

### 错误流程
1. 在 Phase 3 {teammate-2} 因错误停止
2. 领导收到空闲通知
3. 通过 SendMessage 确认状态 → 尝试重启
4. 重启失败时将 {teammate-2} 的作业重新分配给 {teammate-1}
5. 用其余结果继续 Phase 4
6. 在最终报告中注明 "{teammate-2} 区域部分未收集"
```

---

## 模板 B：子智能体模式（备选）

适用于无需团队通信开销的场景。通过 `Agent` 工具直接调用，收集返回值。

```markdown
---
name: {domain}-orchestrator
description: "{도메인} 에이전트를 조율하는 오케스트레이터. {초기 실행 키워드}. 후속 작업 키워드 포함."
---

## 执行模式：子智能体

## 智能体构成

| 智能体 | subagent_type | 角色 | 技能 | 输出 |
|---------|--------------|------|------|------|
| {agent-1} | {内置或自定义} | {角色} | {skill} | {output-file} |
| {agent-2} | ... | ... | ... | ... |

## 工作流

### Phase 0：上下文确认
（与模板 A 相同 — 按 `_workspace/` 是否存在进行分支）

### Phase 1：准备
1. 分析输入
2. 创建 `_workspace/`（初始执行时；或在全新执行中，将既有 `_workspace/` 移到归档目录 `_workspace_{YYYYMMDD_HHMMSS}/` 后立即创建，与模板 A 一致）

### Phase 2：并行执行
在单条消息中并发调用 N 个 Agent 工具：

| 智能体 | 输入 | 输出 | model | run_in_background |
|---------|------|------|-------|-------------------|
| {agent-1} | {来源} | `_workspace/{phase}_{agent}_{artifact}.md` | opus | true |
| {agent-2} | {来源} | `_workspace/{phase}_{agent}_{artifact}.md` | opus | true |

### Phase 3：整合
1. 收集各智能体的返回值
2. 文件型产物用 Read 收集
3. 应用整合逻辑 → 最终产物

### Phase 4：收尾
1. 保留 `_workspace/`
2. 汇报结果摘要

## 错误处理
- 单个智能体失败：重试 1 次。仍失败则注明缺失后继续
- 过半失败：通知用户并确认是否继续
- 超时：使用截至当前已收集的部分结果
```

---

## 模板 C：混合模式

各 Phase 使用不同的执行模式。在每个 Phase 顶部注明 `**执行模式：** {团队 | 子}`。

```markdown
---
name: {domain}-orchestrator
description: "{도메인} 오케스트레이터 (하이브리드). {키워드}. 후속 작업 키워드 포함."
---

## 执行模式：混合

| Phase | 模式 | 理由 |
|-------|------|------|
| Phase 2（并行采集） | 子智能体 | 独立的资料采集，无需团队通信 |
| Phase 3（共识整合） | 智能体团队 | 需要对冲突数据进行讨论并达成一致 |
| Phase 4（独立验证） | 子智能体 | 1 名 QA 智能体进行客观验证 |

## 工作流

### Phase 2：并行资料采集
**执行模式：** 子智能体

在单条消息中通过 Agent 工具并发调用 N 个智能体（`run_in_background: true`）。
各结果保存到 `_workspace/02_{agent}_raw.md`。

### Phase 3：协商整合
**执行模式：** 智能体团队

1. 通过 `TeamCreate` 组建整合团队（editor + fact-checker + synthesizer）
2. 通过 `TaskCreate` 分配任务 — 全部 Read Phase 2 的 `_workspace/02_*` 文件
3. 团队成员通过 `SendMessage` 讨论冲突数据，基于文件形成一致方案
4. 生成最终整合稿 `_workspace/03_integrated.md`
5. 通过 `TeamDelete` 解散团队

### Phase 4：独立验证
**执行模式：** 子智能体

单个 QA 子智能体读取 `_workspace/03_integrated.md` 作为输入，生成验证报告。
```

**混合模式切换规则：**
- 团队 → 子智能体：必须先通过 `TeamDelete` 解散团队，再调用 Agent 工具
- 子智能体 → 团队：将子智能体的文件型产物以 Read 路径方式传递给团队成员
- 团队 → 团队：解散既有团队后新建 `TeamCreate`（每个会话只能有 1 个活跃团队）

---

## 编写原则

1. **先声明执行模式** — 在编排器顶部从"智能体团队"/"子智能体"/"混合"中选择其一注明。混合模式必须附带按 Phase 列示的模式表
2. **团队模式要具体说明 TeamCreate/SendMessage/TaskCreate 用法** — 团队组建、任务注册、通信规则
3. **子智能体模式要完整列出 Agent 工具参数** — name、subagent_type、prompt、run_in_background、model
4. **文件路径使用绝对形式** — 禁止相对路径；明确以 `_workspace/` 为基准的路径
5. **标明 Phase 间依赖** — 哪个 Phase 依赖哪个 Phase 的结果。混合模式需特别强调模式切换点
6. **错误处理要贴合实际** — 不要假设"一切都会成功"
7. **必须包含测试场景** — 至少 1 个正常 + 1 个错误

## 编写 description 时的后续作业关键词

编排器的 description 仅靠初始执行关键词远远不够。必须包含以下后续作业表达：

- 重跑/再次执行/更新/修改/补做
- "只重新做 {领域} 的 {部分}"
- "基于先前结果"、"改进结果"
- 领域相关的日常请求（例如：若为上线策略 Harness，则包含"上线"、"宣传"、"热点"等）

如果缺少后续关键词，Harness 在首次执行后事实上将沦为死代码。

## 真实编排器参考

扇出/扇入（Fan-out/Fan-in）模式的编排器基本结构：
准备 → Phase 0（上下文确认）→ TeamCreate + TaskCreate → N 个成员并行执行 → Read + 整合 → 收尾。
参考 `references/team-examples.md` 中的调研团队示例。
