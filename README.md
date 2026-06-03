<p align="center">
  <img src="harness_banner.png" alt="Harness Banner" width="600">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Version-1.2.0-brightgreen.svg" alt="Version">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License"></a>
  <img src="https://img.shields.io/badge/Claude_Code-Plugin-purple.svg" alt="Claude Code Plugin">
  <img src="https://img.shields.io/badge/Patterns-6_Architectures-orange.svg" alt="6 Architecture Patterns">
  <img src="https://img.shields.io/badge/Mode-Agent_Teams-green.svg" alt="Agent Teams">
  <a href="https://github.com/revfactory/harness/stargazers"><img src="https://img.shields.io/github/stars/revfactory/harness?style=social" alt="GitHub Stars"></a>
</p>

<p align="center">
  <a href="#category--where-harness-sits"><img src="https://img.shields.io/badge/Layer-L3%20Meta--Factory-orange" alt="Layer"></a>
  <a href="#category--where-harness-sits"><img src="https://img.shields.io/badge/Sub--layer-Team--Architecture%20Factory-teal" alt="Sub-layer"></a>
  <a href="#"><img src="https://img.shields.io/badge/README-CN%20%7C%20EN%20%7C%20KO%20%7C%20JA-lightgrey" alt="i18n"></a>
</p>

# Harness — 面向 Claude Code 的团队架构工厂

**中文** | [English](README_EN.md) | [한국어](README_KO.md) | [日本語](README_JA.md)

> **Harness 是面向 Claude Code 的团队架构工厂。** 在 Claude Code 中输入 **"build a harness for this project"**（英文）、**"하네스 구성해줘"**（韩文）或 **"ハーネスを構成して"**（日文），插件就会根据你提供的领域描述，从六种预定义的团队架构模式中挑选合适的方案，生成相应的智能体团队及其所用的技能。

> 📌 **中文版说明**
>
> 本文档为社区中文翻译版本。原项目地址：https://github.com/revfactory/harness
> 如有翻译错误或建议，欢迎提 Issue。

## Overview（总览）

Harness 借助 Claude Code 的智能体团队系统，将复杂任务拆解为由多个专业化智能体协同组成的团队。只需说一声 "build a harness for this project"，它就会根据你的领域自动生成智能体定义（`.claude/agents/`）和技能（`.claude/skills/`）。

## Category — Harness 的定位

Harness 位于 Claude Code 生态系统的 **L3 元工厂（Meta-Factory）** 层——它本身不直接充当某个 Harness，而是用来生成其他 Harness 的层级。在 L3 内部，我们选择了特定的子层：**团队架构工厂（Team-Architecture Factory）**。

| 层级 | 作用 | 邻近项目 |
|-------|--------------|---------------------------|
| **L3 — 元工厂 / 团队架构工厂**（本项目） | 领域描述 → 智能体团队 + 技能，通过 6 种预定义团队模式生成 | — |
| L3 — 元工厂 / 运行时配置工厂 | 确定性的、可复现的运行时配置 | [coleam00/Archon](https://github.com/coleam00/Archon) |
| L3 — 元工厂 / Codex 运行时移植 | 同一概念，Codex 运行时 | [SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness) |
| L2 — 跨 Harness 工作流 | 在多个 Harness 间统一技能/规则/钩子 | [affaan-m/ECC](https://github.com/affaan-m/everything-claude-code) |

> Archon 生成确定性的运行时配置，Harness 则生成团队架构（流水线（Pipeline）、扇出/扇入（Fan-out/Fan-in）、专家池（Expert Pool）、生产者-审阅者（Producer-Reviewer）、监督者（Supervisor）、层级委派（Hierarchical Delegation））以及智能体所用的技能。二者处于同一 L3 的不同子层。需要运行时确定性时选 Archon，需要团队架构时选 Harness，也可以组合使用。

## Star History（Star 历史）

<a href="https://www.star-history.com/?repos=revfactory%2Fharness&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=revfactory/harness&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=revfactory/harness&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=revfactory/harness&type=date&legend=top-left" />
 </picture>
</a>


## Key Features（核心特性）

- **智能体团队设计** — 6 种架构模式：流水线（Pipeline）、扇出/扇入（Fan-out/Fan-in）、专家池（Expert Pool）、生产者-审阅者（Producer-Reviewer）、监督者（Supervisor）、层级委派（Hierarchical Delegation）
- **技能生成** — 自动生成技能，采用渐进式披露（Progressive Disclosure）以高效管理上下文
- **编排** — 智能体间的数据传递、错误处理和团队协调协议
- **验证** — 触发词校验、空跑测试，以及"启用技能 vs 不启用技能"的对比测试


## Workflow（工作流）

```
Phase 1: Domain Analysis
    ↓
Phase 2: Team Architecture Design (Agent Teams vs Subagents)
    ↓
Phase 3: Agent Definition Generation (.claude/agents/)
    ↓
Phase 4: Skill Generation (.claude/skills/)
    ↓
Phase 5: Integration & Orchestration
    ↓
Phase 6: Validation & Testing
```

## Installation（安装）

### 通过 Marketplace

#### 添加 Marketplace
```shell
/plugin marketplace add revfactory/harness
```

#### 安装插件
```shell
/plugin install harness@harness-marketplace
```

### 直接作为全局技能安装

```shell
# Copy the skills directory to ~/.claude/skills/harness/
cp -r skills/harness ~/.claude/skills/harness
```

## Plugin Structure（插件结构）

```
harness/
├── .claude-plugin/
│   └── plugin.json                 # Plugin manifest
├── skills/
│   └── harness/
│       ├── SKILL.md                # Main skill definition (6-Phase workflow)
│       └── references/
│           ├── agent-design-patterns.md   # 6 architectural patterns
│           ├── orchestrator-template.md   # Team/subagent orchestrator templates
│           ├── team-examples.md           # 5 real-world team configurations
│           ├── skill-writing-guide.md     # Skill authoring guide
│           ├── skill-testing-guide.md     # Testing & evaluation methodology
│           └── qa-agent-guide.md          # QA agent integration guide
└── README.md
```

## Usage（用法）

在 Claude Code 中通过以下触发词调用：

```
Build a harness for this project
Design an agent team for this domain
Set up a harness
```

### 执行模式（Execution Modes）

| 模式 | 说明 | 适用场景 |
|------|-------------|-----------------|
| **智能体团队**（Agent Teams，默认） | TeamCreate + SendMessage + TaskCreate | 需要协作的 2+ 智能体 |
| **子智能体（Subagents）** | 直接调用 Agent 工具 | 一次性任务，无需智能体间通信 |

<p align="center">
  <img src="harness_team.png" alt="Harness Agent Team" width="500">
</p>

### 架构模式（Architecture Patterns）

| 模式 | 说明 |
|---------|-------------|
| 流水线（Pipeline） | 顺序依赖的任务 |
| 扇出/扇入（Fan-out/Fan-in） | 并行独立的任务 |
| 专家池（Expert Pool） | 依据上下文的选择性调用 |
| 生产者-审阅者（Producer-Reviewer） | 先生成，再做质量审阅 |
| 监督者（Supervisor） | 由中心智能体动态分配任务 |
| 层级委派（Hierarchical Delegation） | 自上而下的递归委派 |

## Output（产物）

Harness 生成的文件：

```
your-project/
├── .claude/
│   ├── agents/          # Agent definition files
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── qa.md
│   └── skills/          # Skill files
│       ├── analyze/
│       │   └── SKILL.md
│       └── build/
│           ├── SKILL.md
│           └── references/
```

## Use Cases — 试试这些触发提示

安装 Harness 后，把以下任意一条提示复制到 Claude Code 中即可：

**Deep Research（深度研究）**
```
Build a harness for deep research. I need an agent team that can investigate
any topic from multiple angles — web search, academic sources, community
sentiment — then cross-validate findings and produce a comprehensive report.
```

**Website Development（网站开发）**
```
Build a harness for full-stack website development. The team should handle
design, frontend (React/Next.js), backend (API), and QA testing in a
coordinated pipeline from wireframe to deployment.
```

**Webtoon / Comic Production（漫画/网络漫画制作）**
```
Build a harness for webtoon episode production. I need agents for story
writing, character design prompts, panel layout planning, and dialogue
editing. They should review each other's work for style consistency.
```

**YouTube Content Planning（YouTube 内容规划）**
```
Build a harness for YouTube content creation. The team should research
trending topics, write scripts, optimize titles/tags for SEO, and plan
thumbnail concepts — all coordinated by a supervisor agent.
```

**Code Review & Refactoring（代码评审与重构）**
```
Build a harness for comprehensive code review. I want parallel agents
checking architecture, security vulnerabilities, performance bottlenecks,
and code style — then merging all findings into a single report.
```

**Technical Documentation（技术文档）**
```
Build a harness that generates API documentation from this codebase.
Agents should analyze endpoints, write descriptions, generate usage
examples, and review for completeness.
```

**Data Pipeline Design（数据流水线设计）**
```
Build a harness for designing data pipelines. I need agents for schema
design, ETL logic, data validation rules, and monitoring setup that
delegate sub-tasks hierarchically.
```

**Marketing Campaign（营销活动）**
```
Build a harness for marketing campaign creation. The team should research
the target market, write ad copy, design visual concepts, and set up
A/B test plans with iterative quality review.
```

## Coexistence — Harness 与邻近项目

Harness 在 Claude Code / 智能体框架生态中并非孤品。下面这些仓库位于相邻的层级；每个项目都按"X 是……，Harness 是……"的对照形式描述，方便你挑选最契合需求的一个，或将多个组合使用。

| 仓库 | 它们的定位 | 与 Harness 的关系 |
|------|----------------|-------------------------|
| [coleam00/Archon](https://github.com/coleam00/Archon) | "harness builder" — 确定性的、可复现的运行时配置 | **同一 L3 的相邻子层。** Archon 是运行时配置工厂，Harness 是团队架构工厂。需要运行时确定性时选 Archon，需要团队架构时选 Harness，也可以组合使用。 |
| [SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness) | 同一概念的 Codex 移植版 | **同一 L3，不同运行时。** 在 Claude Code 上用 Harness，在 Codex 上用 meta-harness。 |
| [affaan-m/ECC](https://github.com/affaan-m/everything-claude-code) | "Agent harness performance & workflow layer"（建立在现有 Harness 之上） | **不同层级。** ECC 是跨 Harness 的标准化层，Harness 是生成 Harness 的工厂，可以串行组合。 |
| [wshobson/agents](https://github.com/wshobson/agents) | 子智能体 / 技能目录（182 个智能体，149 项技能） | **工厂 ↔ 零件供应。** wshobson 是可供挑选的目录，Harness 设计团队。可把 wshobson 的条目作为零件，吸纳进 Harness 生成的团队中。 |
| [LangGraph](https://langchain-ai.github.io/langgraph/) | 状态图编排，与 LLM 解耦 | **不同路线。** LangGraph 面向长时间运行、可状态恢复的编排；Harness 面向 Claude-Code 原生、快速的团队设计。 |

## Built with Harness（基于 Harness 构建）

### Harness 100

**[revfactory/harness-100](https://github.com/revfactory/harness-100)** — 100 个生产就绪的智能体团队 Harness，覆盖 10 个领域，同时提供英文和韩文版本（合计 200 个包）。每个 Harness 都包含 4-5 个专业化智能体、一个编排器技能以及领域相关的技能——全部由本插件生成。共 1,808 个 Markdown 文件，覆盖内容创作、软件开发、数据/AI、商业战略、教育、法律、健康等领域。

### Research: A/B Testing Harness Effectiveness（研究：A/B 测试 Harness 的有效性）

**[revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)** — 在 15 个软件工程任务上进行的对照实验，用于衡量结构化预配置对 LLM 代码智能体输出质量的影响。

| 指标 | 不使用 Harness | 使用 Harness | 提升 |
|--------|:-:|:-:|:-:|
| 平均质量分 | 49.5 | 79.3 | **+60%** |
| 胜率 | — | — | **100%**（15/15） |
| 输出方差 | — | — | **-32%** |

关键发现：效果随任务复杂度提升而扩大——任务越难，提升越大（基础级 +23.8，进阶级 +29.6，专家级 +36.2）。

**统一表述：** +60% 平均质量（49.5 → 79.3），15/15 胜率，−32% 方差（n=15，作者自测 A/B，第三方复现待补）。

> 完整论文：*Hwang, M. (2026). Harness: Structured Pre-Configuration for Enhancing LLM Code Agent Output Quality.*

## Requirements（环境要求）

- [启用智能体团队（Agent Teams）](https://code.claude.com/docs/en/agent-teams)：`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## FAQ（常见问题）

<details>
<summary><b>Q1. "+60%" 是不是言过其实？</b></summary>

**A.** +60% 这一数字来自**作者自测的 A/B 实验（n=15，15 个任务，测量于姊妹仓库 `claude-code-harness`）**。本仓库中每次引用这个数字时，都会在同一句话里加上"n=15、作者自测、第三方复现待补"的披露。在做采用决策时，我们建议先开展 2–4 周的内部试点，自行测量具体数据。

**证据：**
- 作者 A/B：[revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)
- 论文：*Hwang, M. (2026). Harness: Structured Pre-Configuration for Enhancing LLM Code Agent Output Quality*
</details>

<details>
<summary><b>Q2. 为什么叫"harness factory"而不是"harness builder"？这不和 Archon 撞车了吗？</b></summary>

**A.** Archon 生成确定性的运行时配置——它是一个**运行时配置工厂（Runtime-Configuration Factory）**。Harness 则生成智能体团队架构（团队结构、消息协议、审阅关卡）——它是一个**团队架构工厂（Team-Architecture Factory）**。二者是**同一 L3 元工厂的相邻子层**，服务不同需求。需要运行时确定性时选 Archon，需要团队架构模式时选 Harness，也可以组合使用（用 Harness 设计架构 → 用 Archon 部署运行时）。

**证据：**
- Archon 自述：[clawfit docs/reference-levels.md](https://github.com/hongsw/clawfit/blob/main/docs/reference-levels.md)
- 子层声明：见上文 **Category — Harness 的定位** 一节
- Archon 仓库：[github.com/coleam00/Archon](https://github.com/coleam00/Archon)
</details>

<details>
<summary><b>Q3. "只支持 Claude Code"是不是太窄了？Gemini/Codex 怎么办？</b></summary>

**A.** 目前官方运行时仅支持 Claude Code。同一概念的 Codex 移植版——[SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness)——已经开源，Codex 团队可以从那里起步。Harness 选择了"Claude-Code 原生、深入"而非"多运行时、浅尝辄止"；与兄弟仓库（meta-harness、harness-init、OpenRig）的跨运行时协作已在路线图上。

**证据：**
- Codex 移植版：[github.com/SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness)
- 跨运行时脚手架：[github.com/Gizele1/harness-init](https://github.com/Gizele1/harness-init)
</details>

## License（许可证）

Apache 2.0
