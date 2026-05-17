# 分享 1：AI 编程工作流入门 — 工具定位 + OpenSpec 3 条命令跑通 PPT 大纲

> 主题：从“让 AI 写代码”到“用规格控制 AI 写代码”
>
> 时长：60 分钟
>
> 目标：让全员理解 AI 编程工作流工具全景；让开发者现场跑通第一个 OpenSpec 流程
>
> 适用对象：全员参加，重点面向开发人员
>
> 前置要求：基本命令行操作能力，无需 OpenSpec 经验

---

## 一、素材来源

### 主要素材

| 编号 | 文件 | 用途 |
|---|---|---|
| 1 | `01-选型与对比/AI编程工作流选型-SpecKit-OpenSpec-Superpowers深度对比.md` | 工具全景、三类工具定位、选型决策 |
| 2 | `01-选型与对比/OpenSpec-vs-Superpowers-2套AI编码工作流怎么选.md` | OpenSpec 与 Superpowers 分层关系、场景选择 |
| 3 | `02-OpenSpec入门/OpenSpec深度解析-最佳实践四步法.md` | OpenSpec 架构、Delta Spec、DAG、验证引擎 |
| 5 | `02-OpenSpec入门/OpenSpec实战-从0到1构建AI原生规范驱动开发工作流.md` | 环境准备、初始化、配置文件、命令结构 |
| 7 | `02-OpenSpec入门/OpenSpec最佳实战-3条命令跑通规范驱动开发全流程.md` | Todo API 实战主线：propose → apply → archive |
| 8 | `02-OpenSpec入门/OpenSpec最佳实战-3个命令4个制品4步闭环.md` | 3 命令、4 制品、闭环总结 |
| 12 | `03-OpenSpec进阶/OpenSpec最佳实战-4步复盘5项升级.md` | 痛点引入：“能跑 ≠ 跑对”、流程完整不等于代码正确 |

### 建议使用的图片素材与识别内容

| 图片路径 | 图片内容识别 | 建议用途 |
|---|---|---|
| `01-选型与对比/images/dba6c2e20e3ea3726e672a49899445e4.png` | Spec-Kit、OpenSpec、Superpowers 三款工具深度对比卡片，包含核心范式、技术栈、适用场景、技术选型建议 | 工具全景页 |
| `01-选型与对比/images/96d1d9312721d357cfbfea899a5889f6.png` | 三种工具架构对比：Spec-Kit 模板引擎，OpenSpec 活跃变更/持久规范/归档，Superpowers 技能库/钩子系统/代理集成 | 架构差异页 |
| `01-选型与对比/images/5b689f686590cea63f339284f4a822d2.png` | 三种工作流范式对比：阶段门控式、流畅迭代式、技能触发式 | 工作流对比页 |
| `01-选型与对比/images/4f46eb29ee7ac112d98280ba3e86119d.png` | 技术选型决策树：大型企业项目选 Spec-Kit、快速迭代选 OpenSpec、质量优先选 Superpowers等 | 工具选择页 |
| `01-选型与对比/images/3f0c1342fb67235c5c477c18e93a5f3a.png` | OpenSpec vs Superpowers 信息图，左侧讲 OpenSpec 增量规格、DAG、验证引擎、22+ 工具；右侧讲 Superpowers 技能、TDD、子 Agent、反合理化 | OpenSpec 与 Superpowers 分层页 |
| `01-选型与对比/images/e983a8559b8b170d52ed0d8f59ae0ccf.png` | OpenSpec 核心架构图：DAG 工件依赖图、增量规格系统、验证引擎三块 | OpenSpec 原理页 |
| `01-选型与对比/images/d4da871d19a6d3b68e0bc0563615cf31.png` | 项目类型决策图：大型企业项目、中型团队协作、个人小项目对应 OpenSpec/Superpowers/组合方案 | 选型总结页 |
| `02-OpenSpec入门/images/ffe20476f6e83b97a4bea5e65fee4802.png` | OpenSpec 深度解析总览信息图，包含核心问题、三大核心机制、三层架构、Core/Extended 工作流、Delta Spec 四种操作 | OpenSpec 介绍页 |
| `02-OpenSpec入门/images/e24298f5f9e91f6cec74f0b41e5307c9.png` | OpenSpec 三层架构：CLI 层、核心引擎层、Schema 层，以及命令输入/文件输出关系 | 架构讲解页 |
| `02-OpenSpec入门/images/ad80d78ba7db6a0e4fe28832ea7d5129.png` | 增量规格操作机制与合并流程：ADDED/MODIFIED/REMOVED/RENAMED → 解析 → 预验证 → 加载 → 应用 → 重建 → 验证 | Delta Spec 讲解页 |
| `02-OpenSpec入门/images/7c3d406d97de20b15ceb40a896a473eb.png` | 验证引擎三层结构：格式验证、语义验证、业务规则验证 | 质量保障页 |
| `02-OpenSpec入门/images/1b6c52e9a4b8bfa5fa6d34e0b3e1c834.png` | OpenSpec Logo 封面图 | 开场或章节过渡页 |
| `02-OpenSpec入门/images/b84a3a67865747930b6a865c1890eb00.png` | OpenSpec 实战流程图：/opsx:propose → /opsx:apply → /opsx:archive，含 4 制品、任务执行、Delta Specs | Live Demo 总览页 |
| `02-OpenSpec入门/images/0bf8c5ae5d9fe0ff396fd55e83b1db0e.png` | 初始化后目录结构：openspec/changes/archive/specs 与 .claude/commands/opsx、skills 对照 | 环境搭建页 |
| `02-OpenSpec入门/images/fd0f3bde04b39a1e2b043937222c10b4.png` | /opsx:propose 生成 4 个制品关系图：proposal、specs、design、tasks | propose 讲解页 |
| `03-OpenSpec进阶/images/983c1cf083593a886fb10b4984a42177.png` | 四个工件 DAG 依赖关系：proposal → specs/design → tasks → 实施阶段 | 工件依赖页 |

---

## 二、整场分享结构

| 时间 | 模块 | 目标 | 形式 |
|---:|---|---|---|
| 0-5 min | 痛点共鸣 | 让听众意识到 AI 编程翻车是流程问题，不是单纯模型问题 | 讲解 + 互动提问 |
| 5-15 min | 工具全景 | 讲清 Spec-Kit、OpenSpec、Superpowers 的定位和关系 | 对比图 + 决策树 |
| 15-20 min | 环境搭建 | 让开发者完成 OpenSpec 安装与初始化准备 | 命令演示 |
| 20-50 min | Live Demo | 用 Todo API 跑通 propose → apply → archive | 现场演示 |
| 50-60 min | 回顾总结 | 固化 3 条命令、4 个制品、3 个踩坑点 | 速查表 + Q&A |

---

## 三、PPT 大纲与每页内容

## Slide 1：标题页

**标题**：AI 编程工作流入门：工具定位 + OpenSpec 3 条命令跑通

**副标题**：从“AI 自由发挥”到“规格驱动开发”

**页面内容**：

- 本次分享解决一个核心问题：如何让 AI 编程从“碰运气”变成“可控流程”？
- 关键词：AI 编程工作流、OpenSpec、规格驱动开发、propose/apply/archive
- 时长：60 分钟

**讲解要点**：

今天不讲“怎么写一个更长的 prompt”，而是讲“怎么建立一套流程，让 AI 在流程里工作”。

**建议配图**：

- `02-OpenSpec入门/images/1b6c52e9a4b8bfa5fa6d34e0b3e1c834.png`

---

## Slide 2：今天的目标

**标题**：这 60 分钟你会带走什么？

**页面内容**：

1. 理解 3 类 AI 编程工作流工具的定位
2. 搞清楚 OpenSpec 解决的核心问题
3. 现场跑通第一个 OpenSpec 流程
4. 掌握 3 条命令和 4 个制品
5. 知道新手最容易踩的坑

**讲解要点**：

本次不是理论分享，而是“入门 + 可复制实战”。分享结束后，开发同学应该能在自己的项目里完成一次最小闭环。

---

## Slide 3：痛点共鸣：AI 写代码为什么会翻车？

**标题**：AI 写得很快，但经常不是你想要的

**页面内容**：

常见翻车场景：

- **跑偏**：让它加登录，它顺手设计 OAuth2 + JWT + 微服务
- **遗漏**：正常路径能跑，边界条件、错误处理全漏
- **风格不统一**：同一个项目，不同会话产出不同风格
- **难追溯**：需求只存在聊天记录里，会话一清就没了
- **难复现**：同样需求下次再问，输出又变了

**讲解要点**：

AI 编程最大的问题不是“写不出来”，而是“写出来的东西不可控”。

**互动问题**：

你遇到过 AI 自作主张加功能、重构无关代码、或者测试全挂的情况吗？

---

## Slide 4：核心观点：能跑 ≠ 跑对

**标题**：流程走完了，不代表代码就是对的

**页面内容**：

来自 OpenSpec 深度复盘的结论：

- 流程完整性 ≠ 代码正确性
- 文档对齐 ≠ 运行验证
- AI 写完 ≠ 业务可用
- Verify 通过 ≠ 没有运行时问题

**本次入门先解决第一层问题**：

> 先把“需求对齐”做好，再谈“代码质量闭环”。

**讲解要点**：

OpenSpec 的第一价值不是替代测试，而是让 AI 在写代码前先和人对齐“到底要做什么”。

**建议配图**：

- 可选：`03-OpenSpec进阶/images/dcd770fd10b88118265c267b1a99140d.png`
- 可选：`03-OpenSpec进阶/images/983c1cf083593a886fb10b4984a42177.png`

---

## Slide 5：问题根因：缺少结构化工作流约束

**标题**：问题不在 AI 不够强，而在人类指令太随意

**页面内容**：

传统 AI 编程方式：

```text
一句模糊需求 → AI 自由发挥 → 代码输出 → 人工返工
```

更可控的方式：

```text
明确需求 → 结构化规格 → 审批方案 → 按任务实现 → 归档沉淀
```

**关键转变**：

- 从聊天记录到项目文件
- 从一次性 prompt 到可审阅制品
- 从黑盒生成到过程可追踪

**讲解要点**：

OpenSpec 的目标是把 AI 编程从“对话驱动”变成“规格驱动”。

---

## Slide 6：工具全景：三个热门方案分别管什么？

**标题**：Spec-Kit、OpenSpec、Superpowers 不是一回事

**页面内容**：

| 工具 | 一句话定位 | 解决的问题 |
|---|---|---|
| Spec-Kit | 规范可执行化 | 让规格直接驱动代码生成 |
| OpenSpec | 轻量规格管理层 | 管“写什么”，把需求变成结构化制品 |
| Superpowers | 技能驱动的行为纪律 | 管“怎么做”，约束 AI 执行过程 |

**讲解要点**：

很多工具都叫“规范驱动”，但底层哲学不同。Spec-Kit 偏重“规格生成代码”，OpenSpec 偏重“变更与规格管理”，Superpowers 偏重“AI 行为纪律”。

**建议配图**：

- `01-选型与对比/images/dba6c2e20e3ea3726e672a49899445e4.png`

---

## Slide 7：三种工作流范式对比

**标题**：阶段门控、流畅迭代、技能触发

**页面内容**：

| 范式 | 代表工具 | 工作方式 | 适合场景 |
|---|---|---|---|
| 阶段门控式 | Spec-Kit | constitution → specify → plan → tasks → implement | 大型项目、强流程管控 |
| 流畅迭代式 | OpenSpec | propose → apply → archive | 快速接入、增量变更、团队共享规格 |
| 技能触发式 | Superpowers | brainstorming → TDD → review → finish | 质量优先、TDD、行为约束 |

**讲解要点**：

本次选择 OpenSpec 作为入门，是因为它上手成本低：先用 3 条命令跑通，再逐步增强质量控制。

**建议配图**：

- `01-选型与对比/images/5b689f686590cea63f339284f4a822d2.png`

---

## Slide 8：OpenSpec 与 Superpowers 的关系

**标题**：一个管“写什么”，一个管“怎么做”

**页面内容**：

| 维度 | OpenSpec | Superpowers |
|---|---|---|
| 关注层次 | 规格层 | 执行层 |
| 核心问题 | AI 不知道该做什么 | AI 不知道该怎么守规矩地做 |
| 核心产物 | proposal/specs/design/tasks | skills、TDD、review、subagent 流程 |
| 约束方式 | 文件化规格 + Delta 变更 | prompt/skill 行为约束 |

**讲解要点**：

二者不是竞品关系。OpenSpec 先让 AI 明确“要做什么”，Superpowers 后续可以约束 AI “按什么纪律做”。今天先把 OpenSpec 入门跑通。

**建议配图**：

- `01-选型与对比/images/3f0c1342fb67235c5c477c18e93a5f3a.png`

---

## Slide 9：OpenSpec 是什么？

**标题**：AI 原生的轻量级规范驱动开发框架

**页面内容**：

OpenSpec 的核心理念：

> Agree before you build — 先对齐，再构建。

它做了三件事：

1. **文件化需求**：需求不再散落在聊天里，而是进入项目文件
2. **增量规格**：每次变更只描述变化部分，支持 ADDED/MODIFIED/REMOVED/RENAMED
3. **工作流闭环**：propose → apply → archive，让规格持续沉淀

**讲解要点**：

OpenSpec 不要求你一次性写完完整规格，而是让项目规格随着每次变更逐步长出来。

**建议配图**：

- `02-OpenSpec入门/images/ffe20476f6e83b97a4bea5e65fee4802.png`

---

## Slide 10：OpenSpec 的底层结构

**标题**：三层架构：CLI 层 → 核心引擎层 → Schema 层

**页面内容**：

| 层级 | 作用 | 典型内容 |
|---|---|---|
| CLI 层 | 命令入口 | init、list、validate、archive 等 |
| 核心引擎层 | 核心能力 | 验证引擎、DAG、Markdown 解析、Specs 合并 |
| Schema 层 | 工作流定义 | artifact、requires、template、apply 配置 |

**讲解要点**：

OpenSpec 本质上是“命令 + 文件 + 工作流规则”。它把 AI 要做的事情拆成一组可读、可改、可追踪的 Markdown 制品。

**建议配图**：

- `02-OpenSpec入门/images/e24298f5f9e91f6cec74f0b41e5307c9.png`

---

## Slide 11：OpenSpec 的核心流程

**标题**：3 条命令跑通一个完整闭环

**页面内容**：

```text
/opsx:propose  →  /opsx:apply  →  /opsx:archive
生成规格制品       按任务实现代码       归档变更，沉淀规格
```

| 命令 | 做什么 | 产生什么 |
|---|---|---|
| `/opsx:propose` | 根据需求生成变更提案 | proposal/specs/design/tasks |
| `/opsx:apply` | 按 tasks.md 执行实现 | 代码变更 |
| `/opsx:archive` | 归档完成的变更 | archive 记录、更新后的规格 |

**讲解要点**：

初学者不要一上来追求复杂模式。先把这 3 条命令跑通，就已经完成了从“裸 prompt”到“规格驱动”的第一步。

**建议配图**：

- `02-OpenSpec入门/images/b84a3a67865747930b6a865c1890eb00.png`

---

## Slide 12：环境准备

**标题**：开始前需要准备什么？

**页面内容**：

前置要求：

- Node.js >= 20.19.0
- npm 或 pnpm
- Claude Code / Cursor / Windsurf 等 AI 编程助手
- 一个空项目目录或 Git 仓库
- 稳定网络连接

安装命令：

```bash
npm install -g @fission-ai/openspec@latest
openspec --version
```

**讲解要点**：

注意 Node.js 版本要求较高。如果安装失败，第一优先检查 Node 版本，其次检查 npm 网络或镜像配置。

---

## Slide 13：初始化项目

**标题**：用 openspec init 接入项目

**页面内容**：

```bash
mkdir todo-api && cd todo-api
openspec init --tools claude
```

也可以支持全部工具：

```bash
openspec init --tools all
```

初始化后生成两类内容：

1. `openspec/`：管理规格与变更
2. `.claude/commands/opsx/` 与 `.claude/skills/`：让 Claude Code 获得 OpenSpec 命令与技能

**讲解要点**：

`openspec/` 是规格目录，`.claude/` 是 Claude Code 适配目录。OpenSpec 的跨工具支持就是通过不同 AI 工具的适配器生成对应命令文件。

**建议配图**：

- `02-OpenSpec入门/images/0bf8c5ae5d9fe0ff396fd55e83b1db0e.png`

---

## Slide 14：初始化后的目录结构

**标题**：两个目录，各司其职

**页面内容**：

```text
openspec/
├── changes/
│   └── archive/
└── specs/

.claude/
├── commands/opsx/
│   ├── propose.md
│   ├── apply.md
│   ├── archive.md
│   └── explore.md
└── skills/
    ├── openspec-propose-change/
    ├── openspec-apply-change/
    ├── openspec-archive-change/
    └── openspec-explore/
```

**讲解要点**：

`changes/` 是活跃变更队列，`archive/` 是完成后的历史记录，`specs/` 是项目长期沉淀的主规范。

---

## Slide 15：Live Demo 场景说明

**标题**：演示项目：Todo REST API

**页面内容**：

需求：创建一个 Todo REST API，支持增删改查。

接口范围：

- `GET /todos`：获取所有待办事项
- `POST /todos`：创建待办事项
- `PUT /todos/:id`：更新待办事项
- `DELETE /todos/:id`：删除待办事项

**演示目标**：

不是展示复杂业务，而是展示完整工作流闭环。

**讲解要点**：

Todo API 足够简单，适合让大家把注意力放在 OpenSpec 工作流本身，而不是业务复杂度上。

---

## Slide 16：Step 1 — propose：先生成规格制品

**标题**：/opsx:propose：先把需求说清楚

**页面内容**：

```bash
/opsx:propose "创建一个 Todo REST API，支持增删改查"
```

propose 会生成 4 个制品：

| 制品 | 回答的问题 |
|---|---|
| `proposal.md` | 为什么做？做什么？不做什么？ |
| `specs/` | 系统行为如何验证？ |
| `design.md` | 技术上怎么实现？ |
| `tasks.md` | 具体分几步做？ |

**讲解要点**：

不要急着写代码。OpenSpec 的第一步是把需求、设计、任务都落成文件，让人可以审阅。

**建议配图**：

- `02-OpenSpec入门/images/fd0f3bde04b39a1e2b043937222c10b4.png`

---

## Slide 17：4 个制品之间的关系

**标题**：从“为什么”到“怎么做”的完整链路

**页面内容**：

```text
proposal.md
  ├── specs/      定义需求和验收场景
  └── design.md   定义技术方案
        ↓
      tasks.md    拆成可执行任务
        ↓
      code        按任务实现
```

**关键关系**：

- specs 和 design 都依赖 proposal
- tasks 同时依赖 specs 和 design
- apply 阶段依赖 tasks

**讲解要点**：

这就是 DAG 工件依赖。它防止 AI 在没有需求和设计的情况下直接拆任务、写代码。

**建议配图**：

- `03-OpenSpec进阶/images/983c1cf083593a886fb10b4984a42177.png`

---

## Slide 18：审阅 proposal.md

**标题**：proposal.md：确认“为什么做”和“做什么”

**页面内容**：

重点看 4 件事：

1. 背景和动机是否准确
2. 变更范围是否过大
3. 是否包含不该做的内容
4. 影响范围是否清楚

**示例检查问题**：

- 它有没有把 Todo API 扩展成用户系统？
- 它有没有加认证、数据库、部署等本次不需要的内容？
- 它有没有明确 CRUD 的边界？

**讲解要点**：

proposal 是防止 AI 跑偏的第一道关。看到多做的内容，要在这里删掉。

---

## Slide 19：审阅 specs：把需求变成可验证场景

**标题**：specs：用 GIVEN/WHEN/THEN 定义行为

**页面内容**：

典型格式：

```markdown
### Requirement: Create Todo
The system SHALL allow creating a todo item.

#### Scenario: Create todo successfully
- GIVEN a valid todo title
- WHEN a POST request is sent to /todos
- THEN the system returns HTTP 201
- AND the response contains the created todo
```

**审阅重点**：

- 有没有覆盖正常路径？
- 有没有覆盖错误路径？
- 是否用了 SHALL/MUST 等规范关键字？
- 场景是否可测试？

**讲解要点**：

规格不是散文，它应该能直接指导测试。写不出 GIVEN/WHEN/THEN 的需求，通常还不够清楚。

---

## Slide 20：Delta Specs 是什么？

**标题**：需求变化只描述“差异”，不用重写全文

**页面内容**：

Delta Specs 的 4 种操作：

| 操作 | 含义 | 示例 |
|---|---|---|
| ADDED | 新增需求 | 新增 Todo CRUD API |
| MODIFIED | 修改需求 | 创建 Todo 时新增 priority 字段 |
| REMOVED | 删除需求 | 删除旧接口 |
| RENAMED | 重命名需求 | 重命名能力名称 |

**本次首次变更**：通常主要是 ADDED。

**讲解要点**：

Delta Specs 对已有项目特别重要。你不需要让 AI 重读和重写全部规格，只要让它关注这次变化。

**建议配图**：

- `02-OpenSpec入门/images/ad80d78ba7db6a0e4fe28832ea7d5129.png`

---

## Slide 21：审阅 design.md

**标题**：design.md：确认技术方案，不让 AI 自作主张

**页面内容**：

重点看：

- 技术栈是否符合项目现状
- 数据模型是否合理
- API 设计是否符合团队习惯
- 错误码和响应格式是否明确
- 是否引入了不必要的复杂度

**示例提醒**：

如果只是演示 Todo API，不要让 AI 自动引入：

- OAuth2
- 微服务
- Docker/K8s
- 复杂数据库迁移
- 过度抽象架构

**讲解要点**：

design.md 是防止 AI 过度设计的关键。技术方案越简单，越适合第一次 Demo。

---

## Slide 22：审阅 tasks.md

**标题**：tasks.md：任务粒度决定实现质量

**页面内容**：

好的 tasks.md 应该：

- 每个任务只做一件事
- 有明确文件范围
- 有明确验证方式
- 任务顺序符合依赖关系

不好的任务：

```markdown
- [ ] 实现 Todo API
```

更好的任务：

```markdown
- [ ] 创建 Todo 数据模型
- [ ] 实现 POST /todos
- [ ] 实现 GET /todos
- [ ] 实现 PUT /todos/:id
- [ ] 实现 DELETE /todos/:id
- [ ] 添加基础测试或手动验证命令
```

**讲解要点**：

tasks.md 是 apply 阶段的执行脚本。任务越粗，AI 自由发挥空间越大；任务越清楚，输出越可控。

---

## Slide 23：Step 2 — apply：按任务清单实现

**标题**：/opsx:apply：让 AI 按规格动手

**页面内容**：

```bash
/opsx:apply
```

apply 会做什么：

1. 读取 proposal/specs/design/tasks
2. 按 tasks.md 顺序逐项实现
3. 完成后更新任务状态
4. 输出变更摘要

**观察点**：

- AI 是否只做了 tasks.md 中的任务？
- 是否引入了无关技术？
- 是否覆盖了 specs 中的关键场景？

**讲解要点**：

apply 不是魔法。它的质量高度依赖前面 4 个制品的质量，尤其是 tasks.md。

---

## Slide 24：Step 3 — 验证功能

**标题**：别只相信 AI 说完成，要验证

**页面内容**：

建议至少做 3 类验证：

1. 看 tasks.md 是否全部完成
2. 运行测试或手动 curl 验证接口
3. 检查是否符合 specs 场景

示例：

```bash
# 创建 todo
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy milk"}'

# 查询 todos
curl http://localhost:3000/todos
```

**讲解要点**：

OpenSpec 能帮助你把需求说清楚，但不能替代真实运行验证。AI 说完成，只是一个信号，不是证据。

---

## Slide 25：Step 4 — archive：归档变更，沉淀规格

**标题**：/opsx:archive：把一次变更变成项目记忆

**页面内容**：

```bash
/opsx:archive
```

archive 做两件事：

1. 将变更目录移动到 `openspec/changes/archive/`
2. 将 Delta Specs 合并或沉淀到主规范中

归档前：

```text
openspec/changes/add-todo-crud/
```

归档后：

```text
openspec/changes/archive/YYYY-MM-DD-add-todo-crud/
```

**讲解要点**：

archive 的意义是让项目规格持续积累。下一次 AI 读取项目时，不再只靠聊天上下文，而是可以读取项目中的规范文件。

---

## Slide 26：完整流程回顾

**标题**：3 条命令，4 个制品，1 个闭环

**页面内容**：

```text
需求输入
  ↓
/opsx:propose
  ↓
proposal.md + specs/ + design.md + tasks.md
  ↓
人工审阅与调整
  ↓
/opsx:apply
  ↓
代码实现 + 验证
  ↓
/opsx:archive
  ↓
规格沉淀，进入下一轮迭代
```

**讲解要点**：

这个闭环的核心不是“命令少”，而是“每一步都有可审阅、可修改、可追踪的产物”。

---

## Slide 27：三条命令速查卡

**标题**：新手只需要先记住这 3 条

**页面内容**：

| 命令 | 使用时机 | 结果 |
|---|---|---|
| `/opsx:propose "需求描述"` | 想开始一个新变更 | 生成 4 个规划制品 |
| `/opsx:apply` | 审阅制品后开始实现 | AI 按任务写代码 |
| `/opsx:archive` | 验证通过后收尾 | 归档变更并沉淀规格 |

**补充命令**：

- `/opsx:explore`：需求不清楚时，先探索，不写代码

**讲解要点**：

如果需求还不清楚，不要直接 propose，更不要直接 apply。先 explore，把问题讲清楚。

---

## Slide 28：4 个制品速查卡

**标题**：每个制品负责回答一个问题

**页面内容**：

| 制品 | 回答的问题 | 审阅重点 |
|---|---|---|
| `proposal.md` | 为什么做？做什么？ | 范围是否跑偏 |
| `specs/` | 行为如何验证？ | 场景是否完整 |
| `design.md` | 技术上怎么做？ | 是否过度设计 |
| `tasks.md` | 分几步实现？ | 粒度是否足够细 |

**讲解要点**：

OpenSpec 的制品都是普通 Markdown，不是生成后就不能改。发现问题直接改，改完再 apply。

---

## Slide 29：新手最容易踩的 5 个坑

**标题**：第一次用 OpenSpec，注意这 5 件事

**页面内容**：

1. **需求描述太模糊**：导致 proposal 和 specs 跑偏
2. **不审阅就 apply**：等于让 AI 自由发挥
3. **tasks.md 太粗**：一个任务里混太多事情
4. **不验证就 archive**：把问题沉淀进项目历史
5. **长会话不清理上下文**：AI 可能遗忘早期决策

**建议做法**：

- propose 后先审阅 4 个制品
- apply 前必要时手动修改 tasks.md
- archive 前至少做一次运行验证

**讲解要点**：

OpenSpec 的关键不是“自动化越多越好”，而是“在正确的位置让人类介入审阅”。

---

## Slide 30：本次分享结论

**标题**：AI 编程质量，始于规格

**页面内容**：

3 个核心结论：

1. **AI 编程的核心问题不是写不快，而是不可控**
2. **OpenSpec 先解决“写什么”的问题，再让 AI 写代码**
3. **入门只需要 3 条命令：propose → apply → archive**

一句话总结：

> 先写规格，再写代码；先对齐意图，再交给 AI 实现。

**讲解要点**：

今天的目标不是让大家一次性掌握所有 OpenSpec 能力，而是建立一个最小可用的 AI 编程工作流。

---

## Slide 31：课后练习

**标题**：回去后请完成一个最小闭环

**页面内容**：

练习任务：

1. 新建一个空项目
2. 安装并初始化 OpenSpec
3. 用 `/opsx:propose` 生成一个小功能的 4 个制品
4. 手动审阅并修改 tasks.md
5. 执行 `/opsx:apply`
6. 验证功能后 `/opsx:archive`

推荐练习功能：

- Todo API CRUD
- 简单用户注册接口
- 文件上传接口
- 前端计数器组件

**讲解要点**：

第一次练习不要选复杂业务。目标是熟悉流程，而不是挑战 AI 的极限。

---

## Slide 32：Q&A

**标题**：讨论与答疑

**可引导的问题**：

- OpenSpec 适合所有项目吗？
- 如果 propose 生成的文档不准确怎么办？
- OpenSpec 能不能替代测试？
- 什么时候需要 Superpowers？
- 什么时候要从 Core 模式切到 Expanded 模式？

---

## 四、讲师演示脚本

### 演示前准备

```bash
node -v
npm install -g @fission-ai/openspec@latest
openspec --version
mkdir todo-api && cd todo-api
openspec init --tools claude
```

### 演示命令 1：propose

```bash
/opsx:propose "创建一个 Todo REST API，支持增删改查，包括 GET /todos、POST /todos、PUT /todos/:id、DELETE /todos/:id"
```

讲师重点：

- 展示生成目录
- 打开 `proposal.md` 看范围
- 打开 `specs/` 看 GIVEN/WHEN/THEN
- 打开 `design.md` 看技术方案
- 打开 `tasks.md` 看任务拆分

### 演示命令 2：apply

```bash
/opsx:apply
```

讲师重点：

- 观察 AI 是否按 tasks.md 执行
- 展示代码变更
- 对照 specs 看功能是否覆盖

### 演示命令 3：验证

根据实际生成技术栈选择：

```bash
npm test
```

或手动验证：

```bash
curl http://localhost:3000/todos
```

讲师重点：

- 强调“验证不是可选项”
- AI 说完成不等于真的完成

### 演示命令 4：archive

```bash
/opsx:archive
```

讲师重点：

- 展示 `openspec/changes/archive/`
- 解释归档是为了沉淀项目规格

---

## 五、建议讲法节奏

### 0-5 分钟：痛点

讲得快，重点引发共鸣。不要陷入工具细节。

### 5-15 分钟：工具全景

只讲定位，不讲安装细节。让听众知道为什么本次选 OpenSpec 作为入门工具。

### 15-20 分钟：环境搭建

命令必须简洁，提前准备好网络和 Node 环境，避免现场安装拖慢节奏。

### 20-50 分钟：Live Demo

核心时间给 Demo。每一步都强调“生成了什么文件，人应该审什么”。

### 50-60 分钟：总结

回到 3 条命令、4 个制品、5 个坑。确保听众带走可操作清单。

---

## 六、一页版速查

```text
AI 编程翻车原因：缺少结构化工作流约束

OpenSpec 核心理念：Agree before you build

3 条命令：
1. /opsx:propose  生成 proposal/specs/design/tasks
2. /opsx:apply    按 tasks.md 实现代码
3. /opsx:archive  归档变更，沉淀规格

4 个制品：
1. proposal.md    为什么做、做什么
2. specs/         行为需求与验收场景
3. design.md      技术方案与关键决策
4. tasks.md       实施任务清单

5 个坑：
1. 需求太模糊
2. 不审阅就 apply
3. tasks 太粗
4. 不验证就 archive
5. 长会话上下文污染
```

---

## 七、建议后续带走物

1. OpenSpec 安装命令速查卡
2. propose/apply/archive 三命令速查卡
3. 4 个制品审阅清单
4. Todo API Demo 产物样例
5. 新手踩坑 Checklist
