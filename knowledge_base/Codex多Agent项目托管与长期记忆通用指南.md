# Codex 多 Agent 项目托管与长期记忆通用指南

> 一套可复制到任意软件、研究、产品或运维项目中的 Codex 长期托管方案，包含项目文件体系、多 Agent 管理、跨对话恢复、长期记忆、任务/决策/风险治理，以及可直接复制的完整提示词。

- 文档版本：v1.0
- 适用对象：使用 Codex 在同一个项目目录中进行多次、跨对话、长期开发或研究的个人与团队
- 核心原则：**聊天是临时工作区，项目文件才是长期记忆与事实来源**
- 推荐使用方式：先运行“初始化提示词”，之后每次新对话运行“日常启动提示词”或“指定任务提示词”

---

## 目录

1. [适用场景](#1-适用场景)
2. [核心设计原则](#2-核心设计原则)
3. [快速开始](#3-快速开始)
4. [通用变量与替换说明](#4-通用变量与替换说明)
5. [推荐项目文件体系](#5-推荐项目文件体系)
6. [文件职责与事实优先级](#6-文件职责与事实优先级)
7. [多 Agent 管理模型](#7-多-agent-管理模型)
8. [长期记忆模型](#8-长期记忆模型)
9. [任务与质量管理](#9-任务与质量管理)
10. [完整初始化提示词](#10-完整初始化提示词)
11. [日常开发启动提示词](#11-日常开发启动提示词)
12. [指定任务提示词](#12-指定任务提示词)
13. [上下文恢复提示词](#13-上下文恢复提示词)
14. [项目审计提示词](#14-项目审计提示词)
15. [架构决策提示词](#15-架构决策提示词)
16. [Bug 修复提示词](#16-bug-修复提示词)
17. [新功能开发提示词](#17-新功能开发提示词)
18. [发布准备提示词](#18-发布准备提示词)
19. [文档和记忆维护提示词](#19-文档和记忆维护提示词)
20. [暂停与交接提示词](#20-暂停与交接提示词)
21. [管理文件模板](#21-管理文件模板)
22. [常见错误与反模式](#22-常见错误与反模式)
23. [项目成熟度分级](#23-项目成熟度分级)
24. [推荐落地顺序](#24-推荐落地顺序)
25. [最终检查清单](#25-最终检查清单)

---

# 1. 适用场景

本指南适合以下情况：

- 同一个项目会在多个 Codex 对话中持续开发；
- 项目持续时间超过几天或几周；
- 项目包含代码、文档、测试、部署和研究等多个工作流；
- 希望使用多个子 Agent 并行工作；
- 担心新对话无法理解旧对话中的项目状态；
- 希望 Codex 可以自动选择下一项工作；
- 希望保留长期技术决策和失败经验；
- 希望项目暂停数周后仍能准确恢复；
- 希望不同 Codex 对话之间通过项目文件进行交接；
- 希望建立质量门、安全门、发布门和风险管理。

不适合的情况：

- 一次性、十分钟内完成的小任务；
- 不涉及文件或项目目录的简单问答；
- 完全不需要后续维护的临时脚本；
- 用户明确不希望创建管理文件的项目。

---

# 2. 核心设计原则

## 2.1 聊天不是长期记忆

Codex 对话中的上下文可能：

- 在新对话中不可见；
- 因为长度限制被压缩；
- 包含已经过时的判断；
- 与项目当前代码不一致；
- 缺少可审计的变更历史。

因此：

> 只有被写入项目管理文件并经过验证的信息，才属于项目正式记忆。

## 2.2 当前状态只能有一个事实来源

建议由：

```text
docs/project-management/PROJECT_STATE.md
```

作为当前项目状态的唯一来源。

不要同时在以下多个文件中记录不同版本的当前状态：

- README；
- ROADMAP；
- HANDOFF；
- SESSION_LOG；
- 聊天回复；
- Agent Ledger。

其他文件可以引用当前状态，但不应与 `PROJECT_STATE.md` 竞争。

## 2.3 稳定知识、当前状态和历史必须分离

```text
当前状态    → PROJECT_STATE.md
长期知识    → PROJECT_MEMORY.md
正式决策    → DECISIONS.md / ADR
任务状态    → BACKLOG.md
当前交接    → HANDOFF.md
历史记录    → SESSION_LOG.md
风险        → RISKS.md
质量要求    → QUALITY_GATES.md
```

## 2.4 代码和运行结果优先于文档

事实优先级建议：

1. 经过验证的代码、测试、数据库 Migration、配置和运行行为；
2. `PROJECT_STATE.md` 与 `BACKLOG.md`；
3. 已接受的 ADR 和 `AGENTS.md`；
4. `ROADMAP.md` 与 `QUALITY_GATES.md`；
5. `PROJECT_MEMORY.md`；
6. `HANDOFF.md`、`SESSION_LOG.md`；
7. 旧聊天和原始计划。

如果文档与代码冲突：

1. 验证代码和运行事实；
2. 确认哪个是最新正确行为；
3. 修正文档；
4. 不保留两个相互冲突的“当前事实”。

## 2.5 多 Agent 的关键不是数量，而是所有权

多 Agent 最重要的约束：

- 任务必须独立；
- 写入文件范围必须互斥；
- 主 Agent 负责关键路径；
- 主 Agent 负责最终整合和验证；
- 不允许多个 Agent 同时修改同一文件；
- 不允许主 Agent 重做已经委派的任务；
- 子 Agent 完成后必须交接并关闭。

## 2.6 管理系统必须小于项目本身

不要创建一个需要大量时间维护的“文档官僚系统”。

管理文件的目标是：

- 减少恢复时间；
- 减少重复工作；
- 减少冲突；
- 提高验证质量；
- 保存长期知识。

如果维护管理文件的时间长期超过实际开发时间的 10%～15%，应合并或精简文件。

---

# 3. 快速开始

## 3.1 第一次使用

1. 将项目放在一个独立目录中；
2. 确保 Codex 工作目录是该项目目录；
3. 替换本指南中的通用变量；
4. 将第 10 章“完整初始化提示词”复制到新的 Codex 对话；
5. 让 Codex 检查项目并创建管理体系；
6. 审查 Codex 创建的 `AGENTS.md` 和管理文件；
7. 确认 `PROJECT_STATE.md` 反映真实状态；
8. 之后每个新对话使用第 11 章“日常开发启动提示词”。

## 3.2 已经存在 AGENTS.md 的项目

初始化提示词必须要求 Codex：

- 首先阅读现有 `AGENTS.md`；
- 不覆盖其中用户环境、安全、Git 或部署规则；
- 将新项目规则与现有规则合并；
- 如果存在冲突，报告冲突而不是静默覆盖；
- 保持根 `AGENTS.md` 简洁，将详细内容放入管理目录。

## 3.3 最小版本

如果不希望创建完整体系，最低建议创建：

```text
AGENTS.md
docs/project-management/README.md
docs/project-management/PROJECT_STATE.md
docs/project-management/BACKLOG.md
docs/project-management/DECISIONS.md
docs/project-management/HANDOFF.md
```

长期项目推荐使用完整结构。

---

# 4. 通用变量与替换说明

使用提示词前替换以下占位符。

| 占位符 | 含义 | 示例 |
|---|---|---|
| `{{PROJECT_NAME}}` | 项目名称 | MySecureGateway |
| `{{PROJECT_PATH}}` | 项目绝对路径 | `/Users/me/Code/my-project` |
| `{{PROJECT_GOAL}}` | 一句话项目目标 | 构建一个本地优先的团队知识库 |
| `{{CURRENT_STAGE}}` | 已知当前阶段，不确定可写“请检查” | Stage 0 / MVP |
| `{{FOUNDATIONAL_DOCS}}` | 现有核心文档 | `README.md`、`docs/architecture.md` |
| `{{TECH_STACK}}` | 已知技术栈，不确定可留空 | Python、FastAPI、PostgreSQL |
| `{{CONSTRAINTS}}` | 项目特定约束 | 必须离线运行、禁止公共云 |
| `{{PRIMARY_VERTICAL_SLICE}}` | 首个端到端切片 | 用户注册→登录→创建第一条记录 |
| `{{QUALITY_REQUIREMENTS}}` | 特殊质量要求 | 安全关键逻辑覆盖率 95% |
| `{{USER_PREFERENCES}}` | 长期用户偏好 | 优先 Docker Compose、中文文档 |

如果暂时不知道变量内容，可以让 Codex从项目文件中发现，不要编造。

---

# 5. 推荐项目文件体系

```text
{{PROJECT_PATH}}/
├── AGENTS.md
├── README.md
├── docs/
│   ├── project-management/
│   │   ├── README.md
│   │   ├── PROJECT_STATE.md
│   │   ├── PROJECT_MEMORY.md
│   │   ├── ROADMAP.md
│   │   ├── BACKLOG.md
│   │   ├── AGENT_OPERATING_MODEL.md
│   │   ├── AGENT_LEDGER.md
│   │   ├── DECISIONS.md
│   │   ├── RISKS.md
│   │   ├── QUALITY_GATES.md
│   │   ├── HANDOFF.md
│   │   ├── SESSION_LOG.md
│   │   ├── archive/
│   │   └── session-logs/
│   ├── adr/
│   ├── architecture/
│   ├── operations/
│   └── security/
└── ...项目代码和配置...
```

## 5.1 何时合并文件

小型项目可以合并：

- `AGENT_OPERATING_MODEL.md` + `AGENT_LEDGER.md`；
- `RISKS.md` + `DECISIONS.md`；
- `PROJECT_MEMORY.md` + `PROJECT_STATE.md`，但必须清楚区分稳定知识和当前状态；
- `ROADMAP.md` + `BACKLOG.md`，但长期项目通常不推荐。

## 5.2 何时拆分文件

当以下文件超过约 500～800 行时考虑拆分：

- `DECISIONS.md` → 每项 ADR 独立文件；
- `SESSION_LOG.md` → 按月归档；
- `BACKLOG.md` → 按 Stage/Epic 归档；
- `PROJECT_MEMORY.md` → 按架构、产品、运维、安全拆分；
- `RISKS.md` → 关闭风险归档。

---

# 6. 文件职责与事实优先级

## 6.1 AGENTS.md

记录永久规则和阅读顺序。

应包含：

- 项目简介；
- 当前阶段的事实来源；
- 新任务阅读顺序；
- 永久规则；
- 禁止事项；
- 会话开始/进行/结束流程；
- 多 Agent 基本规则；
- 代码、测试、安全基线；
- Git 规则；
- 当前有效命令；
- 管理文件索引。

建议控制在 100～200 行。

不应包含：

- 每日状态；
- 大量任务列表；
- 完整架构设计；
- 大量历史记录；
- Secret 或环境凭据。

## 6.2 PROJECT_STATE.md

当前状态唯一来源。

必须回答：

- 当前日期；
- 当前 Stage/Milestone；
- 当前版本；
- 当前目标；
- 已完成；
- 进行中；
- 下一步；
- 阻塞；
- 当前分支和 Git 状态；
- 活跃 Agent；
- 环境和依赖；
- 最近验证结果；
- 当前主要风险。

它应该覆盖旧状态，而不是无限追加。

## 6.3 PROJECT_MEMORY.md

长期稳定记忆。

每条建议格式：

```markdown
### MEM-001：简短标题

- 类型：Fact / Constraint / Preference / Lesson / Validated Hypothesis
- 内容：
- 来源：
- 首次记录：YYYY-MM-DD
- 最近验证：YYYY-MM-DD
- 状态：Active / Superseded / Needs Review
- 复查条件：
```

不应记录：

- 当前正在做什么；
- 每日流水账；
- 未验证猜测；
- Secret；
- 个人隐私；
- 很快会变化的版本号，除非包含验证日期。

## 6.4 ROADMAP.md

记录阶段和 Gate，不记录每日工作。

每个阶段：

- 目标；
- 进入条件；
- 工作范围；
- 里程碑；
- 退出条件；
- 依赖；
- 当前进度；
- Go/No-Go 条件。

## 6.5 BACKLOG.md

任务的正式状态来源。

每项任务至少包含：

- Task ID；
- 优先级；
- 状态；
- Epic；
- 目标；
- 范围；
- Owner；
- 文件所有权；
- 依赖；
- 验收标准；
- 验证方法；
- 阻塞；
- 创建/更新时间。

## 6.6 AGENT_OPERATING_MODEL.md

记录：

- 主 Agent 责任；
- Agent 启用条件；
- 角色；
- 并发限制；
- 所有权；
- 交接；
- 验证；
- 冲突解决；
- Agent 关闭规则；
- 失败恢复。

## 6.7 AGENT_LEDGER.md

只记录活跃和最近完成的 Agent 工作。

不要把它当任务管理器。正式任务状态仍属于 `BACKLOG.md`。

## 6.8 DECISIONS.md

保存正式决策及其历史。

状态：

```text
PROPOSED
ACCEPTED
SUPERSEDED
DEPRECATED
REJECTED
```

不要静默重写 Accepted 决策；方向改变时新建 ADR 并引用旧决策。

## 6.9 RISKS.md

每项风险包含：

- ID；
- 描述；
- 概率；
- 影响；
- 等级；
- Owner；
- 触发信号；
- 缓解措施；
- 应急方案；
- 状态；
- 最近复查日期。

## 6.10 QUALITY_GATES.md

至少定义：

- Definition of Ready；
- Definition of Done；
- 代码门；
- 测试门；
- 安全门；
- 文档门；
- Migration 门；
- 发布门；
- 阶段门；
- 例外流程。

## 6.11 HANDOFF.md

最新交接，短小且可执行。

应包含：

- 本次目标；
- 完成内容；
- 修改文件；
- 验证；
- 未完成；
- 阻塞；
- 下一步精确行动；
- 避免重复事项；
- 可运行命令。

每次重要交接覆盖更新，历史放在 `SESSION_LOG.md`。

## 6.12 SESSION_LOG.md

短小的追加历史，不复制聊天。

每个记录：

- 日期；
- 目标；
- 输入；
- 结果；
- 重要决定；
- 验证；
- 阻塞/风险；
- 下一步；
- Commit/PR。

---

# 7. 多 Agent 管理模型

## 7.1 主 Agent

主 Agent 是项目协调者，负责：

- 理解整体任务；
- 识别关键路径；
- 决定是否需要子 Agent；
- 定义任务和验收；
- 分配互斥写入范围；
- 处理依赖；
- 审查结果；
- 集成改动；
- 运行最终验证；
- 更新管理文件；
- 关闭不再使用的 Agent。

主 Agent 不应把立即阻塞下一步的工作委派出去后空等。

## 7.2 子 Agent 的启用条件

适合启用：

- 多个问题彼此独立；
- 多个模块文件范围不重叠；
- 调研、文档、测试和实现可以并行；
- 子任务目标清楚且输出可验证；
- 并行能明显缩短关键路径。

不适合启用：

- 主 Agent 下一步立即依赖该结果；
- 任务高度耦合；
- 文件范围重叠；
- 需求尚未明确；
- 子任务太小，协调成本更高；
- 已有 Agent 正在处理同一问题。

## 7.3 推荐角色

按项目需要动态启用：

| 角色 | 典型职责 |
|---|---|
| Project Coordinator | 关键路径、任务、集成、状态、交接 |
| Product/Research Agent | 用户、竞品、需求、验证、指标 |
| Architecture Agent | 架构、ADR、接口、边界 |
| Security Agent | 威胁模型、权限、Secrets、安全测试 |
| Backend Agent | API、数据模型、业务逻辑 |
| Frontend Agent | UI、UX、状态管理、可访问性 |
| Integration Agent | 第三方 API、协议、Connector |
| QA Agent | Test Plan、E2E、回归、故障注入 |
| DevOps/SRE Agent | CI/CD、部署、监控、升级、恢复 |
| Documentation Agent | 文档一致性、Runbook、用户指南 |

角色不是永久常驻身份；同一个 Agent 可以在不同任务承担不同角色。

## 7.4 Agent 任务包

每次委派必须包含：

```text
Task ID：
角色：
目标：
为什么此任务可以独立并行：
明确范围：
禁止范围：
文件所有权：
依赖：
验收标准：
验证命令：
交接输出：
```

## 7.5 并发建议

- 单人或小项目：最多 2～3 个子 Agent；
- 中型项目：最多 3～4 个；
- 超过 4 个通常意味着拆分或集成成本过高；
- 如果多个 Agent 都需要修改同一核心文件，应改成顺序执行。

## 7.6 文件所有权

示例：

```text
Agent A：apps/api/**
Agent B：apps/console/**
Agent C：tests/security/** 和 docs/security/**
主 Agent：根配置、共享接口、最终集成
```

禁止：

```text
Agent A：修改所有后端文件
Agent B：也修改所有后端文件
```

如果所有权必须扩大：

1. 暂停修改；
2. 更新 Backlog/Agent Ledger；
3. 确认没有冲突；
4. 再继续。

## 7.7 子 Agent 完成输出

必须报告：

- 完成了什么；
- 修改了哪些文件；
- 运行了哪些验证；
- 验证结果；
- 哪些假设未验证；
- 剩余风险；
- 建议下一步；
- 是否可以关闭 Agent。

## 7.8 主 Agent 集成检查

主 Agent 必须：

- 查看实际文件，而非只看完成声明；
- 确认没有覆盖其他 Agent 改动；
- 运行集成测试；
- 检查接口契约；
- 检查文档和状态；
- 必要时修复集成问题；
- 完成后关闭 Agent。

---

# 8. 长期记忆模型

## 8.1 六层记忆

### 第一层：永久规则

文件：`AGENTS.md`

- 很少变化；
- 所有新对话必须遵守；
- 包含项目边界、禁止事项和工作流程。

### 第二层：当前状态

文件：`PROJECT_STATE.md`

- 每次重要任务更新；
- 唯一当前状态；
- 过时内容直接替换。

### 第三层：稳定知识

文件：`PROJECT_MEMORY.md`

- 经过验证；
- 未来仍然重要；
- 包含最近验证日期。

### 第四层：正式决策

文件：`DECISIONS.md` 或 `docs/adr/`

- 保存“为什么这样做”；
- 改变时不删除历史。

### 第五层：历史

文件：`SESSION_LOG.md`

- 简短追加；
- 不作为当前事实。

### 第六层：即时交接

文件：`HANDOFF.md`

- 最近一次工作结果；
- 下一对话快速恢复；
- 每次覆盖。

## 8.2 记忆分类

每条记忆标注：

- `FACT`：经过验证的事实；
- `CONSTRAINT`：不可忽略的约束；
- `PREFERENCE`：用户或团队偏好；
- `DECISION`：应链接 ADR；
- `LESSON`：失败或成功经验；
- `HYPOTHESIS`：待验证假设；
- `STALE`：可能过时，需复查。

## 8.3 记忆晋升规则

聊天中的内容不能直接永久化。

建议流程：

```text
聊天发现
→ 在任务中验证
→ 判断是否长期有价值
→ 写入对应 Owner 文件
→ 标记来源与日期
→ 定期复查
```

## 8.4 禁止进入长期记忆

- Password；
- API Token；
- Cookie；
- SSH Private Key；
- Secret；
- 原始客户隐私；
- 不必要的个人身份信息；
- 未验证的市场传闻；
- 大段聊天复制；
- 可从代码轻易发现的琐碎信息。

## 8.5 Memory Review

建议每月执行：

1. 查找重复条目；
2. 检查最近验证日期；
3. 标记过时内容；
4. 把正式决策移至 ADR；
5. 把当前状态移出 Memory；
6. 检查 Secret；
7. 合并相似经验；
8. 归档失效记忆；
9. 更新阅读索引。

---

# 9. 任务与质量管理

## 9.1 任务状态

```text
BACKLOG
READY
IN_PROGRESS
BLOCKED
REVIEW
DONE
CANCELLED
```

## 9.2 Definition of Ready

任务只有满足以下条件才能进入 READY：

- 目标明确；
- 范围明确；
- Owner 明确；
- 文件所有权明确；
- 依赖明确；
- 验收标准明确；
- 验证方法明确；
- 安全边界已考虑；
- 与其他活跃任务不冲突。

## 9.3 Definition of Done

任务只有满足以下条件才能 DONE：

- 功能或文档完成；
- 适用测试通过；
- 错误路径测试；
- 安全检查完成；
- 文档同步；
- Migration/兼容/回滚处理；
- 没有无主遗留问题；
- Backlog 更新；
- State/Handoff 更新；
- 记录验证命令和结果。

## 9.4 优先级

- `P0`：不完成无法安全运行、验证或发布；
- `P1`：当前阶段必须；
- `P2`：下一阶段；
- `P3`：探索。

## 9.5 WIP 限制

建议：

- 主关键路径同时最多 1 项；
- 每个 Agent 同时最多 1 个 IN_PROGRESS 任务；
- 全项目 P0/P1 并行不超过团队实际集成能力；
- 新任务开始前优先完成 REVIEW/BLOCKED 清理。

---

# 10. 完整初始化提示词

以下提示词可直接复制。使用前替换 `{{...}}`。

```text
你现在是 {{PROJECT_NAME}} 项目的长期项目托管负责人、技术负责人和多 Agent 协调者。

项目目录：

{{PROJECT_PATH}}

项目目标：

{{PROJECT_GOAL}}

已知技术栈：

{{TECH_STACK}}

项目特定约束：

{{CONSTRAINTS}}

现有核心文档：

{{FOUNDATIONAL_DOCS}}

首个应保护的端到端纵向切片：

{{PRIMARY_VERTICAL_SLICE}}

你的任务不是立即大规模开发功能，而是先为这个项目建立一套可以跨 Codex 对话、跨开发阶段长期运行的完整项目托管体系。

所有关键状态、长期记忆、架构决策、任务分配、风险和交接信息都必须保存到项目文件中，不能只存在于当前聊天上下文。

我明确授权你在存在多个互相独立的子任务时使用子 Agent 并行工作，但必须由主 Agent 统一规划、分配、审查、集成和关闭。

==================================================
一、目标
==================================================

建立一套支持以下场景的项目管理体系：

1. 新 Codex 对话能够快速恢复项目上下文；
2. 多个 Agent 并行但不重复、不冲突；
3. 每个 Agent 有明确职责、文件所有权和验收标准；
4. 架构决策、需求变化和失败经验能够长期保存；
5. 当前状态、下一步和阻塞始终明确；
6. 不依赖聊天线程作为唯一记忆；
7. 能管理代码、文档、测试、安全、部署和发布；
8. 项目暂停后仍能恢复；
9. 管理体系保持精简，不形成文档负担。

验收目标：

一个没有旧聊天上下文的新 Codex 对话，只阅读少量项目文件，就能在 5 分钟内回答：

- 项目是什么；
- 当前处于哪个阶段；
- 已完成什么；
- 正在进行什么；
- 下一步是什么；
- 哪些决策不能随意改变；
- 有哪些风险和阻塞；
- 哪些 Agent 或任务负责哪些模块；
- 如何运行、测试和验证项目。

==================================================
二、先检查现有项目
==================================================

开始前必须：

1. 查看当前目录结构；
2. 阅读现有 AGENTS.md；
3. 阅读 README、架构、路线、任务和其他核心文档；
4. 检查是否为 Git 仓库；
5. 运行 git status 和查看最近提交（如果存在）；
6. 检查现有代码、测试、配置和运行命令；
7. 检查是否已有项目管理、决策、风险、记忆或交接文件；
8. 不覆盖有价值的现有内容；
9. 识别重复、冲突和过时信息；
10. 给出当前项目成熟度评估。

如果信息可以从项目文件、Git 历史、代码、配置或运行环境中发现，不要先询问我；请自行检查并做合理判断。

如果现有 AGENTS.md 包含用户环境、Git 身份、SSH、安全或部署规则，必须保留并遵守，不得静默覆盖。

==================================================
三、建立管理文件体系
==================================================

优先建立：

docs/project-management/
├── README.md
├── PROJECT_STATE.md
├── PROJECT_MEMORY.md
├── ROADMAP.md
├── BACKLOG.md
├── AGENT_OPERATING_MODEL.md
├── AGENT_LEDGER.md
├── DECISIONS.md
├── RISKS.md
├── QUALITY_GATES.md
├── HANDOFF.md
└── SESSION_LOG.md

同时创建或更新根目录 AGENTS.md。

你可以根据项目规模合并文件，但必须：

- 说明合并理由；
- 保持职责清晰；
- 不让多个文件竞争当前事实；
- 不创建无内容的空壳文件；
- 不复制大型源文档；
- 使用链接引用详细材料。

==================================================
四、AGENTS.md 要求
==================================================

AGENTS.md 应保持简洁，尽量不超过 200 行，包含：

- 项目简介；
- 当前状态的事实来源；
- 每个新任务的强制阅读顺序；
- 永久规则；
- 禁止事项；
- 会话开始、进行和结束流程；
- 多 Agent 协作规则；
- 代码、测试和安全基线；
- Git 和变更管理规则；
- 当前有效命令；
- 管理文件索引。

频繁变化的当前状态必须放入 PROJECT_STATE.md，不要塞入 AGENTS.md。

==================================================
五、文件职责
==================================================

建立以下明确边界：

- PROJECT_STATE.md：唯一当前状态来源；
- PROJECT_MEMORY.md：经过验证的稳定长期知识；
- ROADMAP.md：阶段、Gate、里程碑和依赖；
- BACKLOG.md：任务定义、状态、Owner、验收和验证；
- AGENT_OPERATING_MODEL.md：Agent 角色、并发、所有权、交接和冲突规则；
- AGENT_LEDGER.md：活跃和最近完成的 Agent 分配；
- DECISIONS.md：正式 ADR 和决策历史；
- RISKS.md：风险、信号、缓解、应急和 Owner；
- QUALITY_GATES.md：Ready、Done、测试、安全和发布门；
- HANDOFF.md：最新简短交接；
- SESSION_LOG.md：简短追加历史，不是当前事实。

README.md 必须解释文件职责、优先级、更新频率和五分钟恢复流程。

==================================================
六、多 Agent 规则
==================================================

制定并保存完整协作规则：

1. 主 Agent 先识别关键路径；
2. 立即阻塞下一步的任务由主 Agent完成；
3. 只有独立任务才并行委派；
4. 每个 Agent 必须有 Task ID；
5. 每个 Agent 有明确、互斥的文件写入范围；
6. 不允许两个 Agent 同时修改同一文件组；
7. 不允许主 Agent重复执行已委派工作；
8. 子 Agent 不得撤销其他人的修改；
9. 子 Agent 必须报告修改文件、验证结果和剩余风险；
10. 主 Agent 必须审查实际文件并运行集成验证；
11. 完成或不再需要的 Agent 及时关闭；
12. 默认最大并发数不超过 3～4 个；
13. 所有正式任务状态仍由 BACKLOG.md 管理；
14. AGENT_LEDGER.md 只用于协调。

==================================================
七、长期记忆规则
==================================================

建立六层记忆：

1. AGENTS.md：永久规则；
2. PROJECT_STATE.md：当前状态；
3. PROJECT_MEMORY.md：稳定知识；
4. DECISIONS.md/ADR：正式决策；
5. SESSION_LOG.md：历史；
6. HANDOFF.md：最近交接。

要求：

- 事实、假设、决定、偏好和经验必须区分；
- 会变化的信息包含最近验证日期；
- 过时内容标记 Superseded/Stale，不静默删除决策历史；
- 聊天内容只有整理进正式文件后才是长期记忆；
- 禁止保存密码、Token、Secret、私钥和不必要的个人数据；
- 每月定义 Memory Review 和归档流程；
- SESSION_LOG 按月归档；
- HANDOFF 只保留最新交接；
- PROJECT_STATE 只保留当前状态。

==================================================
八、任务和质量管理
==================================================

任务字段至少包括：

- ID；
- 优先级；
- 状态；
- Epic；
- 目标；
- 范围；
- Owner；
- 文件所有权；
- 依赖；
- 验收标准；
- 验证方法；
- 阻塞项；
- 创建/更新时间。

状态：

BACKLOG → READY → IN_PROGRESS → REVIEW → DONE

另支持 BLOCKED 和 CANCELLED。

任务进入 READY 前必须满足 Definition of Ready；进入 DONE 前必须有实际验证证据。

建立 P0/P1/P2/P3 优先级和 WIP 限制。

QUALITY_GATES.md 至少定义：

- Definition of Ready；
- Definition of Done；
- 代码门；
- 测试门；
- 安全门；
- 文档门；
- Migration 门；
- 阶段门；
- 发布门；
- 例外流程。

特殊质量要求：

{{QUALITY_REQUIREMENTS}}

==================================================
九、会话生命周期
==================================================

定义并执行：

会话开始：

1. 读取 AGENTS.md；
2. 读取管理 README；
3. 读取 PROJECT_STATE；
4. 读取 HANDOFF；
5. 读取相关 BACKLOG、DECISIONS、RISKS；
6. 检查 Git；
7. 核对文档和代码；
8. 定义目标、验收、依赖和所有权；
9. 更新任务和当前状态。

会话进行中：

1. 保持 Backlog/Agent Ledger 准确；
2. 新决定进入 ADR；
3. 新风险进入 RISKS；
4. 稳定知识进入 Memory；
5. 代码、测试、安全和文档同步；
6. 子 Agent 输出必须审查；
7. 重要工作必须验证。

会话结束：

1. 运行适用测试；
2. 更新 BACKLOG；
3. 更新 PROJECT_STATE；
4. 更新 HANDOFF；
5. 必要时更新 MEMORY、DECISIONS 和 RISKS；
6. 追加 SESSION_LOG；
7. 清理临时文件；
8. 关闭不用的 Agent；
9. 输出修改、验证、风险和精确下一步。

==================================================
十、Git 与变更管理
==================================================

检查当前 Git 状态后建立适合项目的工作流。

默认建议：

- Trunk-Based Development；
- main 保持可运行；
- 短期功能分支；
- Focused Commit；
- Commit 前运行验证；
- 重要变更关联 Task ID 和 ADR；
- 禁止提交 Secret；
- 不自动改变用户 Git 身份；
- 不配置远程、不 Push、不 Force Push，除非得到明确授权；
- 尊重现有 AGENTS.md 中的 Git 规则。

==================================================
十一、上下文压缩和归档
==================================================

制定：

- SESSION_LOG 按月归档；
- DONE Backlog 按 Stage 归档；
- 已关闭 Agent 定期归档；
- PROJECT_MEMORY 定期去重；
- PROJECT_STATE 只保留当前状态；
- HANDOFF 每次覆盖；
- 大型研究材料只索引；
- 新对话优先读取 3～5 个核心文件；
- 管理维护成本过高时合并文件。

==================================================
十二、本次实际完成
==================================================

请实际：

1. 检查项目；
2. 建立或更新管理体系；
3. 创建/合并 AGENTS.md；
4. 创建管理 README；
5. 创建当前 PROJECT_STATE；
6. 创建稳定 PROJECT_MEMORY；
7. 从现有材料提取可执行 ROADMAP；
8. 创建初始 BACKLOG；
9. 创建 AGENT_OPERATING_MODEL 和 LEDGER；
10. 整理初始 DECISIONS；
11. 创建初始 RISKS；
12. 创建 QUALITY_GATES；
13. 创建第一份 HANDOFF；
14. 创建 SESSION_LOG；
15. 验证本地链接、代码围栏和文件职责；
16. 检查是否包含 Secret；
17. 检查文档之间是否矛盾；
18. 输出下一次对话可用的简短启动提示词。

不要因为路线中计划了某功能，就把它写成已经实现。

除非本次任务明确要求，否则不要同时开始大规模产品开发。

==================================================
十三、多 Agent 执行建议
==================================================

如果当前项目足够复杂，可以并行：

- Agent A：检查项目事实、阶段和路线；
- Agent B：设计 Agent Operating Model 和 Ledger；
- Agent C：设计长期记忆、状态、交接和归档；
- Agent D：设计 Backlog、决策、风险和质量门。

主 Agent 保留：

- 关键路径；
- AGENTS.md；
- PROJECT_STATE；
- 最终一致性检查；
- 文件整合；
- 最终验证。

所有 Agent 文件范围必须互斥。

==================================================
十四、最终输出
==================================================

完成后输出：

1. 管理体系概览；
2. 创建或修改的文件；
3. 每个文件用途；
4. 当前项目阶段；
5. 多 Agent 模型；
6. 长期记忆方式；
7. 初始 Backlog；
8. 当前风险和阻塞；
9. 验证命令和结果；
10. 下一次对话可复制的启动提示词。

不要只在聊天中描述方案，必须把正式内容写入项目文件。
```

---

# 11. 日常开发启动提示词

```text
继续托管并推进 {{PROJECT_NAME}} 项目。

项目目录：{{PROJECT_PATH}}

请严格执行根目录 AGENTS.md。

开始前必须：

1. 阅读 AGENTS.md；
2. 阅读 docs/project-management/README.md；
3. 阅读 PROJECT_STATE.md；
4. 阅读 HANDOFF.md；
5. 阅读与当前阶段相关的 BACKLOG、DECISIONS 和 RISKS；
6. 检查 Git 状态、当前分支和最近提交；
7. 核对文档状态是否与代码、测试和运行事实一致。

本次请根据 PROJECT_STATE 和 BACKLOG，自主选择当前最高优先级、满足 READY 且没有阻塞的任务推进。

我授权你在存在独立任务时使用子 Agent，但必须：

- 主 Agent 先识别关键路径；
- 不委派立即阻塞下一步的工作；
- 每个 Agent 有 Task ID、目标、互斥文件所有权、验收标准和验证方法；
- 默认最大并发不超过 3～4 个；
- 不重复其他 Agent 已做工作；
- 主 Agent 审查和整合；
- 完成后及时关闭 Agent。

执行期间：

- 保持 BACKLOG 和 AGENT_LEDGER 准确；
- 新决策进入 DECISIONS/ADR；
- 新风险进入 RISKS；
- 稳定知识进入 PROJECT_MEMORY；
- 不把重要信息只留在聊天；
- 代码、测试、安全和文档同步；
- 每个完成项必须有实际验证。

结束前：

- 运行适用测试；
- 更新 BACKLOG、PROJECT_STATE 和 HANDOFF；
- 必要时更新 MEMORY、DECISIONS 和 RISKS；
- 追加 SESSION_LOG；
- 清理临时文件并关闭不用的 Agent；
- 输出修改文件、验证结果、遗留风险和精确下一步。

除非存在无法从项目中发现的关键歧义，否则不要停止询问；请做合理、显式标注的假设并推进。
```

---

# 12. 指定任务提示词

```text
继续托管 {{PROJECT_NAME}}，并完成以下任务：

【任务】
{{TASK_DESCRIPTION}}

【期望结果】
{{EXPECTED_OUTCOME}}

【特殊约束】
{{TASK_CONSTRAINTS}}

开始前：

1. 阅读 AGENTS.md；
2. 按管理 README 恢复上下文；
3. 阅读 PROJECT_STATE 和 HANDOFF；
4. 检查任务是否已经存在于 BACKLOG；
5. 阅读相关 DECISIONS、RISKS、架构和代码；
6. 检查 Git 和并发文件所有权。

如果任务不在 BACKLOG：

- 创建正式 Task ID；
- 填写优先级、范围、Owner、文件所有权、依赖、验收和验证；
- 只有满足 Definition of Ready 才进入 READY/IN_PROGRESS。

执行要求：

- 先制定简短计划；
- 主 Agent 保留关键路径；
- 可以对独立子任务使用子 Agent；
- 子 Agent 文件所有权必须互斥；
- 不覆盖用户或其他 Agent 的修改；
- 完成代码、测试、安全分析和文档；
- 运行适用验证；
- 发现新决定、风险或稳定知识时写入对应文件。

任务只有在通过验收并记录验证证据后才能 DONE。

结束时更新：

- BACKLOG；
- PROJECT_STATE；
- HANDOFF；
- SESSION_LOG；
- 必要的 MEMORY、DECISIONS 和 RISKS。

最终报告：

- 完成内容；
- 修改文件；
- 验证命令和结果；
- 未验证假设；
- 剩余风险；
- 精确下一步。
```

---

# 13. 上下文恢复提示词

```text
请恢复 {{PROJECT_NAME}} 项目的完整当前上下文，暂时不要进行大规模功能开发。

项目目录：{{PROJECT_PATH}}

依次读取：

1. AGENTS.md；
2. docs/project-management/README.md；
3. PROJECT_STATE.md；
4. HANDOFF.md；
5. PROJECT_MEMORY.md；
6. BACKLOG.md；
7. DECISIONS.md；
8. RISKS.md；
9. Git 状态、当前分支和最近提交；
10. 与当前任务相关的代码、测试和配置。

检查：

- 文档之间是否一致；
- 文档与代码/测试/运行行为是否一致；
- 是否有已经完成但未更新的任务；
- 是否有 IN_PROGRESS 但没有 Owner 的任务；
- 是否存在过期记忆；
- 是否有活跃 Agent 或文件所有权冲突；
- 是否有应关闭、归档或重新安排的工作；
- 是否有未记录的决策或风险；
- 当前最合理的下一步是什么。

如果发现冲突，以经过验证的代码、测试和运行事实为准，并更新拥有该事实的管理文件。

不要把旧聊天当作事实来源。

最终输出：

- 当前阶段；
- 当前目标；
- 已完成；
- 正在进行；
- 阻塞；
- 活跃任务；
- 主要风险；
- 文档/代码不一致；
- 建议下一步；
- 建议投入的 Agent 角色。
```

---

# 14. 项目审计提示词

```text
对 {{PROJECT_NAME}} 执行一次完整项目健康审计。本次以检查和修正管理状态为主，不主动扩展产品范围。

必须阅读 AGENTS.md 和全部项目管理索引文件，并检查 Git、代码、测试、配置、依赖和运行命令。

审计维度：

1. 当前状态真实性；
2. Backlog 是否有无主、无验收、无验证任务；
3. 文档和代码是否冲突；
4. ADR 是否被实现绕过；
5. 风险是否有 Owner、信号和缓解；
6. 长期记忆是否重复、过时或混入当前状态；
7. Handoff 是否能让新对话恢复；
8. Agent Ledger 是否有遗留 Agent；
9. 文件所有权是否冲突；
10. 测试和质量门是否真实执行；
11. 是否存在 Secret、敏感数据或危险日志；
12. Git 工作树是否有来源不明或无关修改；
13. 依赖、Migration、部署和回滚是否可复现；
14. 是否存在范围膨胀；
15. 当前阶段是否真正满足进入/退出条件。

对每个问题给出：

- 严重级别；
- 证据；
- 影响；
- 修正建议；
- Owner；
- 建议 Task ID；
- 是否阻断当前阶段或发布。

允许修正明确的管理文件错误，但不要未经授权重构大量产品代码。

结束时更新 PROJECT_STATE、BACKLOG、RISKS、HANDOFF 和 SESSION_LOG，并输出审计摘要。
```

---

# 15. 架构决策提示词

```text
为 {{PROJECT_NAME}} 评估以下架构决策：

{{DECISION_TOPIC}}

不要直接修改实现。先：

1. 阅读 AGENTS.md、PROJECT_STATE、相关 ADR、ROADMAP、RISKS；
2. 检查当前实现和依赖；
3. 明确问题背景和不可违反的约束；
4. 区分事实、假设和偏好；
5. 至少比较 2～3 个可行替代方案；
6. 分析安全、维护、兼容、迁移、性能、成本和退出路径；
7. 说明是否需要技术 Spike；
8. 给出可证伪的选择标准。

如果证据足够，创建或更新 ADR，包含：

- 状态；
- 日期；
- 背景；
- 决策；
- 原因；
- 替代方案；
- 后果；
- 迁移/回滚；
- 重新评估条件；
- 相关任务和风险。

如果证据不足，不要假装形成 Accepted 决策；创建 Proposed ADR 和对应 Spike 任务。

结束时更新必要的 DECISIONS、BACKLOG、RISKS、PROJECT_STATE、HANDOFF 和 SESSION_LOG。
```

---

# 16. Bug 修复提示词

```text
修复 {{PROJECT_NAME}} 中的以下问题：

{{BUG_DESCRIPTION}}

要求：

1. 按 AGENTS.md 恢复上下文；
2. 在 BACKLOG 中确认或创建 Bug Task；
3. 先复现，不在未复现时猜测修复；
4. 记录实际与预期行为；
5. 确定根因，不只修复表面症状；
6. 检查同类代码路径；
7. 添加失败测试或回归测试；
8. 检查错误路径和安全影响；
9. 保持改动最小且聚焦；
10. 不顺手重构无关代码；
11. 运行适用测试、Lint、Type Check 和安全检查；
12. 更新文档、风险和长期经验（如适用）。

可以使用子 Agent 并行调查日志、测试和相邻模块，但不得让多个 Agent 同时修改同一代码范围。

任务 DONE 前必须记录：

- 复现步骤；
- 根因；
- 修改文件；
- 回归测试；
- 验证结果；
- 是否需要发布说明或安全公告。

结束时更新项目管理文件和交接。
```

---

# 17. 新功能开发提示词

```text
为 {{PROJECT_NAME}} 开发以下功能：

{{FEATURE_DESCRIPTION}}

先执行产品和技术门：

1. 读取当前 Stage、Roadmap 和 Backlog；
2. 确认此功能属于当前阶段，不是范围膨胀；
3. 确认有用户价值或明确项目需求；
4. 检查现有 ADR、接口和数据模型；
5. 定义非目标；
6. 定义验收标准；
7. 定义安全、隐私、兼容和回滚要求；
8. 将功能拆分为可验证的纵向切片；
9. 建立任务和互斥文件所有权；
10. 明确测试计划。

执行顺序：

- 先完成最小端到端切片；
- 再补充错误路径、安全和可观测性；
- 再扩展 UX、性能和配置；
- 不先建设大而全框架；
- 不提前实现未进入当前 Scope 的扩展。

子 Agent 只用于互相独立的模块；主 Agent 负责共享接口、关键路径和最终集成。

任务完成时必须：

- 验收通过；
- 单元/集成/E2E 适用测试通过；
- 安全检查通过；
- Migration/回滚已处理；
- Metrics/Logging/Tracing 已处理；
- 用户和运维文档已更新；
- 管理文件已更新。
```

---

# 18. 发布准备提示词

```text
为 {{PROJECT_NAME}} 的 {{RELEASE_VERSION}} 执行发布准备和 Go/No-Go 审查。

不要假设“测试过一些功能”就等于可发布。

检查：

1. 发布范围是否冻结并关联 Task/ADR；
2. 所有相关任务是否真正 DONE；
3. P0/P1 缺陷是否为 0；
4. Unit/Integration/E2E 是否通过；
5. 安全测试、依赖扫描和 Secret Scan 是否通过；
6. Migration、升级、回滚是否验证；
7. 备份是否实际恢复；
8. 默认配置是否安全；
9. 失败路径是否 fail-safe/fail-closed；
10. 版本、制品、源码 revision 是否唯一对应；
11. SBOM、校验和、签名是否适用；
12. Release Notes、已知限制和兼容矩阵是否完成；
13. Runbook、监控、告警和事件响应是否可执行；
14. 是否存在未接受的 Critical/High 风险；
15. 文档宣传是否超出验证事实。

每个 Gate 输出 PASS / FAIL / NOT APPLICABLE，并给出证据。

若任一不可豁免安全项失败，结论必须是 NO-GO。

允许创建发布阻塞任务，不要为了按期发布而降低标准。

结束时更新 QUALITY_GATES、BACKLOG、PROJECT_STATE、RISKS、HANDOFF 和 SESSION_LOG，并给出最终 Go/No-Go。
```

---

# 19. 文档和记忆维护提示词

```text
对 {{PROJECT_NAME}} 执行一次项目文档、长期记忆和上下文压缩维护。

本次不进行产品功能开发。

检查：

1. AGENTS.md 是否简洁且仍然有效；
2. PROJECT_STATE 是否只有当前事实；
3. PROJECT_MEMORY 是否有重复、过期或未经验证的内容；
4. DECISIONS 是否保留历史且链接有效；
5. BACKLOG 是否有过期 DONE/取消任务可归档；
6. AGENT_LEDGER 是否有未关闭 Agent；
7. HANDOFF 是否是最新交接；
8. SESSION_LOG 是否需要按月归档；
9. RISKS 是否需要复查；
10. README 索引和本地链接是否有效；
11. 大型内容是否被重复复制；
12. 是否包含 Secret 或隐私数据；
13. 新对话是否能在 5 分钟内恢复。

维护规则：

- 不删除仍有决策价值的历史；
- 过时决策标记 Superseded；
- 当前状态直接替换旧值；
- 历史移入 archive；
- 大文档用链接引用；
- 修正事实时记录来源和验证日期。

完成后输出维护前后变化、归档文件、发现问题和下一次复查时间。
```

---

# 20. 暂停与交接提示词

```text
准备暂停或将 {{PROJECT_NAME}} 交接给新的 Codex 对话/开发者。

请不要开始新功能，只做安全收尾和上下文落盘。

必须：

1. 检查 Git 和工作树；
2. 运行当前可运行的适用验证；
3. 将任务状态更新准确；
4. 不把未完成任务标记 DONE；
5. 更新 PROJECT_STATE；
6. 覆盖更新 HANDOFF；
7. 将稳定知识写入 PROJECT_MEMORY；
8. 将正式决定写入 DECISIONS；
9. 更新 RISKS；
10. 追加 SESSION_LOG；
11. 关闭不用的 Agent；
12. 清理临时文件；
13. 检查没有 Secret；
14. 列出未提交修改和原因；
15. 给出下一位 Owner 的精确第一步。

HANDOFF 必须包含：

- 当前阶段；
- 本次目标；
- 完成内容；
- 修改文件；
- 实际验证；
- 未完成；
- 阻塞；
- 避免重复事项；
- 下一步命令；
- 建议首先阅读的文件。

最终输出一份可复制的新对话启动提示词。
```

---

# 21. 管理文件模板

## 21.1 PROJECT_STATE.md 模板

```markdown
# Project State

> 本文件是当前项目状态的唯一事实来源。历史放入 SESSION_LOG.md。

- As of：YYYY-MM-DD（时区）
- Last updated：YYYY-MM-DD
- Lifecycle stage：
- Milestone：
- Version：
- Maturity：
- Current objective：

## Completed

- 

## In progress

- 

## Exact next actions

1. 
2. 

## Blockers and open inputs

- 

## Repository and branches

- Git：
- Current branch：
- Active branches：
- Remote：
- Working tree：

## Active agents

- 

## Environment and dependencies

- Runtime status：
- Supported versions：
- Available commands：

## Latest validation

- 日期：
- 命令：
- 结果：

## Current risks

- 参见 RISKS.md：
```

## 21.2 PROJECT_MEMORY.md 模板

```markdown
# Project Memory

> 只保存经过验证、未来仍然重要的稳定知识。不要保存当前任务和 Secret。

## MEM-001：标题

- 类型：FACT / CONSTRAINT / PREFERENCE / LESSON / VALIDATED_HYPOTHESIS
- 内容：
- 来源：
- 首次记录：YYYY-MM-DD
- 最近验证：YYYY-MM-DD
- 状态：Active
- 复查条件：
```

## 21.3 BACKLOG.md 模板

```markdown
# Backlog

## 状态

BACKLOG → READY → IN_PROGRESS → REVIEW → DONE

另支持 BLOCKED / CANCELLED。

| ID | Priority | Status | Epic | Goal | Scope | Owner | Write ownership | Dependencies | Acceptance | Verification | Blockers | Updated |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| PROJ-001 | P0 | READY | Example |  |  |  |  |  |  |  | None | YYYY-MM-DD |
```

## 21.4 AGENT_LEDGER.md 模板

```markdown
# Agent Ledger

| Agent ID | Role | Task ID | Goal | Write ownership | Started | Status | Dependencies | Output | Validation | Residual risks | Closed |
|---|---|---|---|---|---|---|---|---|---|---|---|
```

## 21.5 DECISIONS.md 模板

```markdown
# Decisions

## ADR-001：标题

- 状态：Proposed / Accepted / Superseded / Rejected
- 日期：YYYY-MM-DD
- 背景：
- 决策：
- 原因：
- 替代方案：
- 后果：
- 迁移/回滚：
- 重新评估条件：
- 相关任务：
- 相关风险：
```

## 21.6 RISKS.md 模板

```markdown
# Risk Register

## R-001：标题

- 描述：
- 概率：1～5
- 影响：1～5
- 等级：
- Owner：
- 触发信号：
- 缓解措施：
- 应急方案：
- 状态：Open / Mitigating / Materialized / Closed
- 最近复查：YYYY-MM-DD
- 关闭证据：
```

## 21.7 HANDOFF.md 模板

```markdown
# Handoff

- 日期：YYYY-MM-DD
- From：
- To：
- 当前阶段：

## Read first

1. AGENTS.md
2. docs/project-management/README.md
3. PROJECT_STATE.md
4. 本文件
5. 相关 BACKLOG/DECISIONS/RISKS

## 本次目标

- 

## 已完成

- 

## 修改文件

- 

## 验证

- 命令：
- 结果：

## 未完成和阻塞

- 

## 避免重复

- 

## 精确下一步

1. 

## 可直接运行命令

```bash
# commands
```
```

## 21.8 SESSION_LOG.md 模板

```markdown
# Session Log

## YYYY-MM

### YYYY-MM-DD — 主题

- Goal：
- Owner / write scope：
- Inputs：
- Completed：
- Decision/finding：
- Validation：
- Blockers/risks：
- Next：
- Commit/PR：
```

## 21.9 AGENT_OPERATING_MODEL.md 最小规则

```markdown
# Agent Operating Model

## 主 Agent

- 拥有关键路径、任务分配、集成和最终验证。

## 委派条件

- 任务独立；
- 文件范围互斥；
- 输出可验证；
- 并行有实际收益。

## 禁止

- 两个 Agent 修改同一文件；
- 主 Agent 重做委派任务；
- 无 Task ID 或验收的委派；
- Agent 完成后长期不关闭。

## Agent 交接

必须报告修改文件、验证、假设和风险。
```

## 21.10 QUALITY_GATES.md 最小模板

```markdown
# Quality Gates

## Definition of Ready

- [ ] 目标明确
- [ ] 范围明确
- [ ] Owner 明确
- [ ] 文件所有权明确
- [ ] 依赖明确
- [ ] 验收明确
- [ ] 验证明确
- [ ] 安全边界已考虑

## Definition of Done

- [ ] 实现或文档完成
- [ ] 适用测试通过
- [ ] 错误路径测试
- [ ] 安全检查
- [ ] 文档同步
- [ ] Migration/兼容/回滚处理
- [ ] 记录验证结果
- [ ] 更新 State/Backlog/Handoff

## 禁止发布

- P0/P1 未解决；
- 必需测试失败；
- Secret 泄漏；
- 数据损坏风险；
- 无回滚方案的破坏性 Migration；
- 文档宣称超出验证事实。
```

---

# 22. 常见错误与反模式

## 22.1 把聊天当作唯一记忆

问题：新对话无法恢复，结论不可审计。

修正：将稳定信息写入 Owner 文件。

## 22.2 创建很多文件但没有职责边界

问题：状态重复、内容冲突。

修正：README 说明每个文件拥有哪类事实。

## 22.3 ROADMAP 被当成实现事实

问题：“计划支持 OAuth”被误认为“已经支持 OAuth”。

修正：只有代码、测试、运行证据和 PROJECT_STATE 能证明实现。

## 22.4 HANDOFF 无限增长

问题：下一对话需要读取大量历史。

修正：HANDOFF 只保留最新交接，历史进入 SESSION_LOG。

## 22.5 PROJECT_MEMORY 变成流水账

问题：长期记忆失去价值。

修正：只保存稳定、经过验证、未来有用的知识。

## 22.6 启动过多 Agent

问题：协调和冲突成本超过收益。

修正：默认 3～4 个并发上限，只并行独立任务。

## 22.7 Agent 文件所有权重叠

问题：覆盖修改、合并冲突、责任不清。

修正：委派前写明互斥写入范围。

## 22.8 子 Agent 完成声明未经验证

问题：代码可能无法运行或违反架构。

修正：主 Agent 检查实际文件并运行集成验证。

## 22.9 没有 Definition of Ready

问题：Agent 在需求和验收不明确时开始实现。

修正：不满足 Ready 的任务不能进入 IN_PROGRESS。

## 22.10 没有停止条件

问题：项目不断堆功能但没有用户或技术验证。

修正：Roadmap 每个阶段设置 Go/No-Go 和停止条件。

## 22.11 在文档中保存 Secret

问题：泄漏到 Git、日志和长期上下文。

修正：只保存变量名、Secret Provider 和配置说明，不保存值。

## 22.12 每次新对话都重建管理体系

问题：重复劳动和状态冲突。

修正：AGENTS.md 明确“不要重建，先恢复现有体系”。

## 22.13 自动修改 Git 身份或推送远程

问题：可能使用错误身份或覆盖用户仓库。

修正：没有明确授权不改身份、不配置远程、不 Push、不 Force Push。

---

# 23. 项目成熟度分级

## Level 0：无托管

- 只有聊天和零散文件；
- 新对话无法恢复；
- 没有明确状态。

## Level 1：基础恢复

- 有 AGENTS.md；
- 有 PROJECT_STATE；
- 有 HANDOFF；
- 有基本 Backlog。

## Level 2：可持续开发

- 有 ADR；
- 有风险；
- 有质量门；
- 有测试和 Git 工作流；
- 新对话可在 5 分钟恢复。

## Level 3：多 Agent 可控

- 有 Operating Model；
- 有 Ledger；
- 文件所有权互斥；
- 委派和交接可审计；
- 主 Agent 能稳定整合。

## Level 4：发布可控

- 有 CI/CD；
- 有安全门；
- 有升级/回滚；
- 有发布门；
- 有运行和故障手册。

## Level 5：长期自治

- 项目可以在多个 Codex 对话间持续推进；
- Memory Review 和归档稳定；
- 任务、决策和风险状态可靠；
- 可自动选择下一项 READY 工作；
- 人类只需进行关键产品、风险和发布决策。

---

# 24. 推荐落地顺序

## 第一步：建立恢复链路

创建：

- AGENTS.md；
- README；
- PROJECT_STATE；
- HANDOFF。

## 第二步：建立任务和决策

创建：

- BACKLOG；
- ROADMAP；
- DECISIONS；
- RISKS。

## 第三步：建立质量和 Agent 管理

创建：

- QUALITY_GATES；
- AGENT_OPERATING_MODEL；
- AGENT_LEDGER。

## 第四步：建立长期记忆

创建：

- PROJECT_MEMORY；
- SESSION_LOG；
- 归档流程。

## 第五步：执行一次恢复演练

开启新的 Codex 对话，只使用日常启动提示词，检查是否能准确回答：

- 当前状态；
- 下一任务；
- 风险；
- 关键决策；
- 验证命令。

如果不能，修复管理文件，而不是把更多内容塞入提示词。

---

# 25. 最终检查清单

## 初始化完成检查

- [ ] 现有 AGENTS.md 已被保留或正确合并
- [ ] 根 AGENTS.md 足够简洁
- [ ] 管理 README 有五分钟恢复路径
- [ ] PROJECT_STATE 是唯一当前状态
- [ ] BACKLOG 有完整任务字段
- [ ] ROADMAP 区分计划和已实现事实
- [ ] DECISIONS 有决策状态和历史规则
- [ ] RISKS 有 Owner、信号和应急方案
- [ ] QUALITY_GATES 定义 Ready/Done
- [ ] AGENT_OPERATING_MODEL 定义所有权和并发
- [ ] AGENT_LEDGER 没有无主 Agent
- [ ] HANDOFF 足够短且下一步精确
- [ ] SESSION_LOG 不复制聊天
- [ ] PROJECT_MEMORY 不包含当前流水账
- [ ] 所有本地 Markdown 链接有效
- [ ] 代码围栏闭合
- [ ] 没有 Secret 或私钥文件
- [ ] Git 状态已经检查
- [ ] 没有未经授权 Push 或身份修改

## 每次会话结束检查

- [ ] 适用测试已运行
- [ ] Backlog 状态准确
- [ ] Project State 已更新
- [ ] Handoff 已更新
- [ ] Session Log 已追加
- [ ] 新决定已记录
- [ ] 新风险已记录
- [ ] 稳定知识已晋升
- [ ] 临时文件已清理
- [ ] Agent 已关闭或明确保留原因
- [ ] 最终报告有修改、验证、风险和下一步

## 每月检查

- [ ] Session Log 已归档
- [ ] Memory 已复查
- [ ] Risks 已复查
- [ ] DONE Backlog 已归档
- [ ] Agent Ledger 已清理
- [ ] AGENTS.md 仍然简洁
- [ ] 文档链接有效
- [ ] 新对话恢复时间不超过约 5 分钟

---

# 结语

一套有效的 Codex 项目托管体系不应试图让模型“记住所有聊天”，而应让项目目录本身成为可靠、可验证、可恢复的外部记忆。

最重要的三个文件通常是：

```text
AGENTS.md
PROJECT_STATE.md
HANDOFF.md
```

最重要的三个协作规则通常是：

```text
主 Agent 负责关键路径和集成
子 Agent 必须有互斥文件所有权
任务只有在实际验证后才能 DONE
```

最重要的三个长期治理原则通常是：

```text
当前状态只有一个事实来源
正式决策保留原因和历史
聊天结论只有写入项目文件后才成为项目记忆
```
