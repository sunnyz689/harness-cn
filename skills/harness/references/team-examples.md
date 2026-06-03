# Agent Team Examples

---

## 示例 1：研究团队（智能体团队模式）

### 团队架构：扇出/扇入（Fan-out/Fan-in）
### 执行模式：智能体团队

```
[领导者/编排器]
    ├── TeamCreate(research-team)
    ├── TaskCreate(4 个调研作业)
    ├── 团队成员自行协调 (SendMessage)
    ├── 收集结果 (Read)
    └── 生成综合报告
```

### 智能体构成

| 团队成员 | 智能体类型 | 角色 | 输出 |
|------|-------------|------|------|
| official-researcher | general-purpose | 官方文档/博客 | research_official.md |
| media-researcher | general-purpose | 媒体/投资 | research_media.md |
| community-researcher | general-purpose | 社区/社交媒体 | research_community.md |
| background-researcher | general-purpose | 背景/竞品/学术 | research_background.md |
| (领导者 = 编排器) | — | 综合报告 | 综合报告.md |

> 调研智能体使用 `general-purpose` 内置类型，但必须通过 `.claude/agents/{name}.md` 文件进行定义。文件中应明确角色、调研范围与团队通信协议，以保证复用性与协作质量。

### 编排器工作流（智能体团队）

```
Phase 1: 准备
  - 分析用户输入 (识别主题、调研模式)
  - 创建 _workspace/

Phase 2: 组建团队
  - TeamCreate(team_name: "research-team", members: [
      { name: "official", prompt: "调研官方渠道..." },
      { name: "media", prompt: "调研媒体/投资动向..." },
      { name: "community", prompt: "调研社区反响..." },
      { name: "background", prompt: "调研背景/竞争环境..." }
    ])
  - TaskCreate(tasks: [
      { title: "官方渠道调研", assignee: "official" },
      { title: "媒体动向调研", assignee: "media" },
      { title: "社区反响调研", assignee: "community" },
      { title: "背景环境调研", assignee: "background" }
    ])

Phase 3: 执行调研
  - 4 名团队成员独立调研
  - 出现有趣发现时通过 SendMessage 在成员间共享
    (例如：media 发现的投资新闻转发给 background)
  - 出现信息冲突时成员间直接讨论
  - 每位成员完成后保存文件并通知领导者

Phase 4: 整合
  - 领导者 Read 4 份产物
  - 生成综合报告
  - 冲突信息并列标注出处

Phase 5: 收尾
  - 请求团队成员终止
  - 清理团队
  - 保留 _workspace/ (供事后验证与审计追踪)
```

### 团队通信模式

```
official ──SendMessage──→ background  (共享相关官方公告)
media ────SendMessage──→ background  (共享投资/收购信息)
community ─SendMessage──→ media      (社区反响中与媒体相关的信息)
所有成员 ──TaskUpdate──→ 共享任务列表  (更新进度)
领导者 ←───── 空闲通知 ──── 已完成的成员   (自动)
```

---

## 示例 2：科幻小说写作团队（智能体团队模式）

### 团队架构：流水线（Pipeline） + 扇出
### 执行模式：智能体团队

```
Phase 1 (并行 — 智能体团队): worldbuilder + character-designer + plot-architect
  → 彼此通过 SendMessage 协调一致性
Phase 2 (顺序): prose-stylist (写作)
Phase 3 (并行 — 智能体团队): science-consultant + continuity-manager (审阅)
  → 彼此通过 SendMessage 共享发现
Phase 4 (顺序): prose-stylist (根据审阅反馈修改)
```

### 智能体构成

| 团队成员 | 智能体类型 | 角色 | 技能 |
|------|-------------|------|------|
| worldbuilder | 自定义 | 世界观构建 | world-setting |
| character-designer | 自定义 | 角色设计 | character-profile |
| plot-architect | 自定义 | 情节结构 | outline |
| prose-stylist | 自定义 | 文风编辑 + 写作 | write-scene, review-chapter |
| science-consultant | 自定义 | 科学验证 | science-check |
| continuity-manager | 自定义 | 一致性验证 | consistency-check |

### 智能体文件完整示例：`worldbuilder.md`

```markdown
---
name: worldbuilder
description: "构建科幻小说世界观的专家。设计物理法则、社会结构、技术水平与历史。"
---

# Worldbuilder — 科幻世界观设计专家

你是科幻小说的世界观设计专家。基于科学事实并扩展想象力，构建故事得以展开的物理、社会与技术基础。

## 核心角色
1. 定义世界的物理法则与技术水平
2. 设计社会结构、政治体系、经济系统
3. 建立历史脉络与当前冲突结构
4. 描绘各场所的环境与氛围

## 工作原则
- 内部一致性最优先 — 设定之间不能矛盾
- 通过"如果存在这种技术呢？"的连锁追问推导世界的连带影响
- 服务于故事的世界观 — 避免妨碍情节的过度设定

## 输入/输出协议
- 输入：用户的世界观概念、类型要求
- 输出：`_workspace/01_worldbuilder_setting.md`
- 格式：Markdown，按章节划分 (物理/社会/技术/历史/场所)

## 团队通信协议
- 向 character-designer：通过 SendMessage 提供社会结构、阶级体系、职业信息
- 向 plot-architect：通过 SendMessage 提供世界的主要冲突结构与危机要素
- 从 science-consultant：接收科学错误反馈 → 修正设定
- 世界观变更时向相关团队成员全员广播

## 错误处理
- 概念模糊时提出 3 个方向并请求选择
- 发现科学错误时一并给出替代方案

## 协作
- 向 character-designer 提供社会结构信息
- 向 plot-architect 提供冲突结构信息
- 根据 science-consultant 的反馈修正设定
```

### 团队工作流详解

```
Phase 1: TeamCreate(team_name: "novel-team", members: [worldbuilder, character-designer, plot-architect])
         TaskCreate([世界观构建, 角色设计, 情节结构])
         → 团队成员自行协调并并行作业
         → worldbuilder 完成社会结构时通过 SendMessage 通知 character-designer
         → character-designer 完成主角设定时通过 SendMessage 通知 plot-architect

Phase 2: 清理 Phase 1 的团队 → 以子智能体方式调用 prose-stylist (单独写作，无需团队)
         prose-stylist Read _workspace/ 中的 3 份产物后开始写作
         → 结果保存到 _workspace/02_prose_draft.md

Phase 3: 创建新团队 — TeamCreate(team_name: "review-team", members: [science-consultant, continuity-manager])
         (每个会话仅一个活跃团队，但已清理 Phase 1 团队，因此可创建新团队)
         → 两位审阅者检查 draft，相互共享发现
         → science-consultant 发现物理错误时同步告知 continuity-manager
         → 审阅完成后清理团队

Phase 4: 以子智能体方式调用 prose-stylist，根据审阅结果完成最终修改
```

---

## 示例 3：漫画制作团队（子智能体模式）

### 团队架构：生产-验证
### 执行模式：子智能体

> 在生产-验证模式中，智能体只有 2 个，且重心在于传递结果而非通信，因此子智能体更为合适。

```
Phase 1: Agent(webtoon-artist) → 生成分镜
Phase 2: Agent(webtoon-reviewer) → 审阅
Phase 3: Agent(webtoon-artist) → 问题分镜重新生成 (最多 2 次)
```

### 智能体构成

| 智能体 | subagent_type | 角色 | 技能 |
|---------|--------------|------|------|
| webtoon-artist | 自定义 | 分镜图像生成 | generate-webtoon |
| webtoon-reviewer | 自定义 | 质量审阅 | review-webtoon, fix-webtoon-panel |

### 智能体文件完整示例：`webtoon-reviewer.md`

```markdown
---
name: webtoon-reviewer
description: "审阅漫画分镜质量的专家。评估构图、角色一致性、文本可读性与演出效果。"
---

# Webtoon Reviewer — 漫画质量审阅专家

你是审阅漫画分镜质量的专家。以视觉完成度、故事传达力、角色一致性为标准评估分镜。

## 核心角色
1. 评估每个分镜的构图与视觉完成度
2. 验证角色外观在分镜之间的一致性
3. 评估对话气泡文本的可读性与排布
4. 检查整集的演出流程与节奏感

## 工作原则
- 按 PASS/FIX/REDO 三档明确判定
- FIX 适用于可通过局部修改解决的情况；REDO 表示需要全面重做
- 依据客观标准 (一致性、可读性、构图) 而非主观偏好作出判断

## 输入/输出协议
- 输入：`_workspace/panels/` 目录下的分镜图像
- 输出：`_workspace/review_report.md`
- 格式：
  ```
  ## Panel {N}
  - 判定: PASS | FIX | REDO
  - 原因: [具体理由]
  - 修改指示: [FIX/REDO 时给出具体修改方向]
  ```

## 错误处理
- 图像加载失败时将该分镜判定为 REDO
- 经 2 次重做仍为 REDO 的分镜，附加警告后作 PASS 处理

## 协作
- 向 webtoon-artist 传达修改指示书 (基于结果文件)
- 对重新生成的分镜再次审阅 (最多 2 轮循环)
```

### 错误处理

```
重试策略：
- REDO 判定的分镜 → 请求 artist 重新生成 (附带具体修改指示)
- 最多 2 轮循环后强制 PASS
- 若 REDO 比例超过总分镜的 50%，向用户建议修改提示词
```

---

## 示例 4：代码评审团队（智能体团队模式）

### 团队架构：扇出/扇入 + 讨论
### 执行模式：智能体团队

> 代码评审是智能体团队最具代表性的发光场景。不同视角的审阅者共享发现并相互挑战，从而获得更深入的评审。

```
[领导者] → TeamCreate(review-team)
    ├── security-reviewer: 安全漏洞排查
    ├── performance-reviewer: 性能影响分析
    └── test-reviewer: 测试覆盖率验证
    → 审阅者之间相互共享发现 (SendMessage)
    → 领导者整合结果
```

### 团队通信模式

```
security ──SendMessage──→ performance  ("此 SQL 查询可能被注入，请从性能角度也确认一下")
performance ──SendMessage──→ test      ("发现 N+1 查询，请确认相关测试是否存在")
test ────SendMessage──→ security      ("认证模块缺少测试，请从安全视角给出优先级意见")
```

要点：审阅者**绕过领导者**直接沟通，快速捕捉跨领域问题。

---

## 示例 5：监督者模式 — 代码迁移团队（智能体团队模式）

### 团队架构：监督者（Supervisor）
### 执行模式：智能体团队

```
[supervisor/领导者] → 分析文件清单 → 分配批次
    ├→ [migrator-1] (batch A)
    ├→ [migrator-2] (batch B)
    └→ [migrator-3] (batch C)
    ← 接收 TaskUpdate → 追加分配或重新分配
```

### 智能体构成

| 团队成员 | 角色 |
|------|------|
| (领导者 = migration-supervisor) | 文件分析、批次分发、进度管理 |
| migrator-1~3 | 迁移分配到的文件批次 |

### 监督者的动态分配逻辑（结合智能体团队）

```
1. 收集全部目标文件清单
2. 估算复杂度 (文件大小、import 数量、依赖关系)
3. 通过 TaskCreate 将文件批次登记为作业 (包含依赖关系)
4. 团队成员自行请求作业 (claim)
5. 成员通过 TaskUpdate 报告完成时：
   - 成功 → 自动请求下一个作业
   - 失败 → 领导者通过 SendMessage 确认原因 → 重新分配或指派给其他成员
6. 全部作业完成 → 领导者运行集成测试
```

与扇出的区别：作业并非事先固定，而是**在运行时动态分配**。共享任务列表的自我请求 (claim) 功能与监督者模式天然契合。

---

## 产物模式小结

### 智能体定义文件
位置：`项目/.claude/agents/{agent-name}.md`
必备章节：核心角色、工作原则、输入/输出协议、错误处理、协作
团队模式追加章节：**团队通信协议** (消息收发、作业请求范围)

### 技能文件结构
位置：`项目/.claude/skills/{skill-name}/SKILL.md` (项目级)
或：`~/.claude/skills/{skill-name}/SKILL.md` (全局级)

### 整合技能（编排器）
负责调度整个团队的上层技能。按场景定义智能体构成与工作流。
模板：参见 `references/orchestrator-template.md`。
**必须明确执行模式** — 智能体团队 (默认) 或子智能体。
