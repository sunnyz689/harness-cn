# 更新日志

本项目遵循 [Semantic Versioning](https://semver.org/)。

## [Unreleased]

### Added
- 新增智能体/技能创建前的重复审查步骤（Phase 3-0、Phase 4-0）
- `references/agent-design-patterns.md` "智能体复用设计" 章节
- `references/skill-writing-guide.md` §9 "技能复用设计"

### Changed
- Phase 选择矩阵中显式标注 3-0/4-0
- Phase 2-3 中添加复用审查步骤指针
- 产物检查清单新增 2 项复用审查条目

---

## [1.2.1] - 2026-04-18

### Fixed

- **版本一致性同步** — README.md / README_KO.md / README_JA.md 徽章为 `v1.0.1`、`.claude-plugin/marketplace.json` 为 `1.1.0`、`.claude-plugin/plugin.json` 为 `1.2.0`，三处不一致 → 全部统一为 **v1.2.0**（以 plugin.json 为准）
- **零打 tag 状态准备解除** — 为 v1.0.0 / v1.0.1 / v1.1.0 / v1.2.0 制定追溯打 tag 计划（详见 `_workspace/release/audit-2026-04-18.md` §4）

### Added

- **定位声明："harness factory"** — 在 README 顶部加入类别自我界定语句。以"按领域批量生产智能体+技能的 Harness 工厂"抢占类别（区别于单 Agent / 提示词框架）
- **CONTRIBUTING.md** — 贡献指南与 SLA 说明（PR 首次响应 72h，Issue 分诊 48h），降低社区上手门槛
- **docs/ 目录** — 新增长期文档（架构、迁移、模式目录）迁移空间，避免 README 膨胀并提升可检索性
- **Issue #3 响应策略** — 为社区 issue 增加官方响应模板与分诊流程

### Changed

- `.claude-plugin/marketplace.json` version：`1.1.0` → `1.2.0`
- README 徽章（EN/KO/JA 三个版本）：`Version-1.0.1` → `Version-1.2.0`
- **`.claude-plugin/plugin.json` description 重写** — `"Agent Team & Skill Architect — Meta-skill that designs..."` → `"The team-architecture factory for Claude Code — a meta-skill that turns a domain description into an agent team and the skills they use, with six pre-defined team-architecture patterns..."`（英韩双语，体现 L3 Meta-Factory 定位）
- **`.claude-plugin/plugin.json` keywords 扩展** — 5 个 → 17 个（新增 `harness-factory`、`team-architecture-factory`、`claude-code-plugin`、`agent-scaffolding`、`multi-agent` 以及 6 种架构模式关键字）

## [1.2.0] - 2026-04-08

### Changed

- **CLAUDE.md 注册策略精简（去重）** — Phase 5-4 由 "上下文注册" 改为 "指针注册"。把智能体列表、技能列表、目录结构、执行规则细节从 CLAUDE.md 移除，只保留 **触发规则 + 变更历史**。智能体/技能列表由 `.claude/agents/`、`.claude/skills/` 以及编排器技能作为单一来源进行管理
- **删除 Phase 3/4 临时同步步骤** — 为减轻 CLAUDE.md 同步负担，移除 Phase 3/4 中的临时同步指令。最终指针注册只在 Phase 5-4 执行一次
- **核心原则 3 重定义** — "在 CLAUDE.md 注册 Harness 上下文" → "在 CLAUDE.md 注册 Harness 指针"
- **删除 CLAUDE.md vs 编排器角色分工表** — 指针策略简化后该表格已无必要

### Added

- **Phase 2-1：混合执行模式** — 在智能体团队 / 子智能体两种模式之外，新增按 Phase 切换模式的混合模式。明确常用组合（并行采集→共识汇总、团队生成→验证、Phase 间团队重组）
- **Phase 2-1 执行模式对比表** — 提供团队 / 子智能体 / 混合 三种模式的特性对比以及三步决策顺序
- **Phase 5-0 混合编排器模式** — 混合模式下要求每个 Phase 顶部显式标注执行模式
- **Phase 5-1 返回值数据传递** — 子智能体模式专用的数据传递策略（在原有消息/任务/文件三种之外新增"返回值"）
- **Phase 5-1 推荐组合（子模式 / 混合）** — 在团队模式之外，给出子模式与混合模式下的数据传递推荐组合

## [1.1.0] - 2026-04-05

### Added

- **Phase 0：现状审计** — 触发时先检查既有 Harness 状态，并按 新建 / 扩展 / 运维 三种分支路由
- **既有扩展 Phase 选择矩阵** — 按"新增智能体 / 新增技能 / 架构变更"分别给出所需 Phase 的决策表
- **Phase 3/4 CLAUDE.md 临时同步** — 智能体/技能创建后立即反映到 CLAUDE.md（具备会话中断耐受性）
- **Phase 5-4：CLAUDE.md Harness 上下文注册** — 记录智能体团队结构、技能列表、执行规则、目录结构、变更历史。包含 CLAUDE.md vs 编排器角色分工表
- **Phase 5-5：后续作业支持** — 编排器 description 中必含后续关键词；通过 Phase 0 上下文检查自动识别 初次 / 部分重跑 / 重新执行
- **Phase 5 编排器修改路径** — 既有扩展时不新建编排器，而是修改原有编排器的指南
- **Phase 7：Harness 演进机制** — 执行后收集反馈 → 按反馈类型映射修改对象 → 记录变更历史 → 自动触发演进
- **Phase 7-5：运维/维护工作流** — 现状审计 → 渐进式修改 → CLAUDE.md 同步 → 变更验证 四步流程
- **description 中新增运维/维护触发词** — "Harness 巡检"、"Harness 审计"、"Harness 现状"、"智能体/技能同步" 等关键词
- **产物检查清单强化** — 增加 CLAUDE.md 同步完成、变更历史记录、Phase 0 上下文检查 等条目
- 编排器模板新增 Phase 0（上下文检查） — 智能体团队/子智能体两种模式都适用
- 编排器 description 模板中纳入后续作业关键词模式

### Changed

- 核心原则由 2 条扩展到 4 条（新增 CLAUDE.md 注册、演进系统）
- **"演进日志" → "变更历史" 统一** — 把名称与 schema（4 列：日期 / 变更内容 / 对象 / 原因）在所有章节统一
- **Phase 1 Step 3** — 改为基于 Phase 0 审计结果做冲突分析（去重）
- **5-4 CLAUDE.md 模板代码块** — 修复嵌套渲染破损问题（3 个反引号 → 4 个反引号）
- **角色分工表扩展** — 新增 技能列表、目录结构、变更历史 三行
- **编排器模板** — 新增 Phase 0 上下文检查步骤与后续作业关键词指南

## [1.0.1] - 2026-03-28

### Changed

- 去除 SKILL.md ↔ references 之间的重复内容（330 行 → 285 行）
  - Phase 2-1：执行模式对比表/列表 → 核心原则 + 指向 agent-design-patterns.md
  - Phase 2-3：智能体分离基准列表 → 4 轴摘要 + 指向 agent-design-patterns.md
  - Phase 3：智能体定义模板代码块 → 列出必含章节 + 指向 references
  - Phase 5-2：错误处理 5 行表格 → 核心原则 + 指向 orchestrator-template.md

## [1.0.0] - 2026-03-27

### Added

- 基于 6-Phase 工作流的 Harness 构建元技能
- 6 种智能体架构模式（流水线 Pipeline、扇出/扇入 Fan-out/Fan-in、专家池 Expert Pool、生产者-审阅者 Producer-Reviewer、监督者 Supervisor、层级委派 Hierarchical Delegation）
- 支持 智能体团队 / 子智能体 两种执行模式
- 基于 Progressive Disclosure 的技能创建指南
- 编排器模板（智能体团队模式 + 子智能体模式）
- QA 智能体集成指南（基于真实项目 7 个缺陷案例）
- 技能测试/评估方法论（With-skill vs Without-skill 对比）
- 5 套实战团队配置示例（研究、小说、网漫、代码评审、迁移）
