# OpenSpec 最佳实战：3 条命令 + Delta Specs，让 AI 编程告别"口头约定"

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _89_ 篇，AI 编程最佳实战「2026」系列第 _21_ 篇
>
> 大家好，欢迎来到 __术哥无界 | ShugeX ｜ 运维有术__。
>
> 我是__术哥__，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的__技术实践者与开源布道者__！
>
> __Talk is cheap, let's explore。无界探索，有术而行。__

![Image 1: 封面图](https://developer.qcloudimg.com/http-save/10642399/5d2922569b7eb837a2b0d5a7825c88b6.png)

封面图

_图 1：OpenSpec 核心工作流概览_

跟 AI 编程助手说"帮我加个暗黑模式"，它给你整出一套完整的主题系统，还顺带重构了 CSS 架构。你说不是这个意思，它又说好的我改改，改完变成了另一番模样。

来回几轮对话，原始需求早就淹没在聊天记录里了。更头疼的是，隔天重新开一个会话，AI 对之前做了什么毫无记忆——你得从零再解释一遍。

这种体验太普遍了。问题不在于 AI 不够聪明，而在于需求和实现之间__缺了一层规范__。双方没有在写代码之前先对齐"要建什么"，全靠口头约定，自然跑偏。

OpenSpec 就是干这事的。它用一套轻量级规范框架，让 AI 在动手写代码前先把需求、设计、任务全部落地成文件。核心机制叫 __Delta Specs__——只记录变更的部分，不重复描述已有行为。

下面从安装到实战，完整走一遍。

### 1. OpenSpec 是什么

OpenSpec 是一个轻量级规范框架（Spec Framework），帮开发者和 AI 编程助手在写代码之前先对齐"要建什么"。核心理念用四句话概括：

```
→ 灵活不僵化（fluid not rigid）
→ 迭代不瀑布（iterative not waterfall）
→ 简单不复杂（easy not complex）
→ 适配存量项目（built for brownfield）
```

说人话：别搞瀑布流那套重度流程，用轻量方式把需求记下来，随时能改，而且对已有项目特别友好。

它跟市面上的同类工具相比，差异挺明显的：

|  |  |  |  |
| --- | --- | --- | --- |
| 重量 | 轻量 | 重量级，阶段门控严格 | 中等 |
| 灵活性 | 高，可自由迭代 | 低，严格阶段门控 | 低，锁定 IDE |
| 工具支持 | 20+ AI 工具 | GitHub 生态 | 仅 Kiro IDE |

OpenSpec 目前支持 Claude Code、Cursor、Windsurf、GitHub Copilot、Cline、Kiro、Gemini CLI 等 20 多款 AI 编程工具。不管你用哪个 IDE，基本都能接上。

### 2. 安装与初始化

#### 你需要准备

- Node.js 20.19.0 或更高版本
- 一个现有的项目目录

#### 安装与初始化

```
# 全局安装
npm install -g @fission-ai/openspec@latest

# 进入项目目录并初始化
cd your-project
openspec init
```

初始化过程会问你几个问题：用哪些 AI 工具、选择哪种工作流 profile。如果你赶时间，可以直接用非交互模式：

```
# 指定 Claude Code 和 Cursor
openspec init --tools claude,cursor

# 或者支持所有工具
openspec init --tools all
```

初始化完成后，项目里会多出 `openspec/` 目录，同时在对应的 AI 工具配置目录下生成技能文件（比如 `.claude/skills/`）。

#### 目录结构

![Image 2: 目录结构图](https://developer.qcloudimg.com/http-save/10642399/f77ec4e368f36ab4e6310a649f67c72b.png)

目录结构图

_图 2：OpenSpec 项目目录结构_

```
openspec/
├── specs/              # 规范的真实来源（Source of Truth）
│   └── <domain>/
│       └── spec.md
├── changes/            # 提议的修改（每个修改一个文件夹）
│   └── <change-name>/
│       ├── proposal.md
│       ├── design.md
│       ├── tasks.md
│       └── specs/      # Delta Specs（变更内容）
│           └── <domain>/
│               └── spec.md
└── config.yaml         # 项目配置（可选）
```

两个核心目录的含义：

- __`specs/`__ — 系统行为规范的真实来源（Source of Truth），按领域组织（比如 `auth/`、`payments/`、`ui/`）
- __`changes/`__ — 待处理的修改，每个修改独占一个文件夹，互不干扰

这个结构的精髓在于__隔离__。每个变更都是独立的文件夹，多个变更可以并行推进。比如你同时在加暗黑模式和修登录 Bug，两个变更文件夹并存，互不影响。

### 3. 工作流：3 条命令走天下

![Image 3: 工作流程图](https://developer.qcloudimg.com/http-save/10642399/ed5c7780e9d535e759f1a2132dc8ebd3.png)

工作流程图

_图 3：OpenSpec 3 条命令工作流_

OpenSpec 默认的 `core` profile 只需要记住 3 条命令：

```
/opsx:propose ──► /opsx:apply ──► /opsx:archive
```

|  |  |
| --- | --- |
| `/opsx:propose` | 创建变更 + 生成所有规划制品（proposal、specs、design、tasks） |
| `/opsx:apply` | 执行任务，写代码 |
| `/opsx:archive` | 归档变更，合并 Delta Specs 到主规范 |

就这三步，没有阶段门控，没有强制流程。做完 `propose` 之后发现设计不对？直接改 `design.md`，然后继续 `apply`。不需要重新来过。

这是 OpenSpec 和传统规范框架的关键区别：__动作（Actions）取代阶段（Phases）__。你不是"在规划阶段"或"在实现阶段"，而是随时可以执行任何动作。

如果你需要更细粒度的控制（比如一个一个制品地创建，或者归档前做验证），可以用 `openspec config profile` 切换到扩展模式，解锁 `/opsx:new`、`/opsx:continue`、`/opsx:ff`、`/opsx:verify`、`/opsx:bulk-archive` 等命令。

不过对大多数人来说，core profile 的三条命令足够用了。

### 4. Delta Specs：OpenSpec 的核心设计

这一节是整篇文章的重点。Delta Specs 是 OpenSpec 区别于其他规范工具的核心设计，理解了它就理解了 OpenSpec 的全部价值。

#### 一句话说清楚 Delta Specs 是什么？

Delta 直译就是"增量"或"差异"。在 OpenSpec 里，__Delta Specs 就是只记录"这次改了什么"的规范文件__。你的项目有一份主规范（`specs/`），描述系统当前的行为；每次提一个变更，不需要重写整份规范，只需要写一个 Delta 文件，标明哪些行为新增了、哪些修改了、哪些废弃了。等变更做完归档时，这个 Delta 会自动合并回主规范。

打个比方：主规范像是一本__产品说明书__，Delta Specs 就是这份说明书的__勘误表__。你不需要每次改版都重印整本说明书，只需要发一张勘误页，写清楚"第 3 章新增了 XX""第 5 章的 YY 改成了 ZZ""附录里的 ZZ 已删除"。等下一版印刷时（归档），把这些勘误合并进去就行。

你可能会问：主规范是哪来的？答案很简单：__归档归出来的__。

`openspec init` 之后，`specs/` 目录是空的，你的项目还没有任何规范。当你用 `/opsx:propose` 提出第一个变更（比如添加暗黑模式），AI 会在变更文件夹里生成一个 Delta Spec，描述这次新增了哪些行为。等你执行 `/opsx:archive` 归档时，这个 Delta 就被合并进 `specs/`，变成主规范的一部分。

再提第二个变更（比如添加 2FA），AI 会基于已有的主规范写新的 Delta——主规范里已经记录了暗黑模式的行为，新的 Delta 只需要描述 2FA 相关的变更。归档后，主规范又丰富了一层。

用时间线看就是这样的：

```
init        → specs/ 为空
propose #1  → 生成 Delta（暗黑模式的需求）
archive #1  → Delta 合并进 specs/，主规范诞生
propose #2  → 基于主规范，生成新 Delta（2FA 的需求）
archive #2  → 新 Delta 合并，主规范继续丰富
...         → 循环往复，主规范越来越完整
```

所以主规范不是一开始就要写好的大而全文档，而是__随着每次变更归档，逐步积累起来的__。这一点对存量项目特别友好——不需要先花时间把现有系统的行为全部补齐再开始用，直接从第一个变更起步就行。

#### 为什么是 Delta 而不是全量规范

传统做法是维护一份完整的行为规范，每次改动都要更新全文。问题很明显：只改了一个登录超时时间，你得把整个认证规范重新审一遍才能确认改了哪里。

Delta Specs 反过来——__只描述变更的部分__。

一个 Delta Spec 包含三个 Section：

|  |  |  |
| --- | --- | --- |
| `## ADDED Requirements` | 新增的行为 | 追加到主规范 |
| `## MODIFIED Requirements` | 修改的行为 | 替换已有需求 |
| `## REMOVED Requirements` | 废弃的行为 | 从主规范中删除 |

#### 完整的 Delta Specs 示例

来看一个实际的例子：__给认证系统添加两步验证（2FA），同时调整会话超时时间，并废弃"记住我"功能：__

```
# Delta for Auth

## ADDED Requirements

### Requirement: Two-Factor Authentication
The system MUST support TOTP-based two-factor authentication.

#### Scenario: 2FA enrollment
- GIVEN a user without 2FA enabled
- WHEN the user enables 2FA in settings
- THEN a QR code is displayed for authenticator app setup
- AND the user must verify with a code before activation

#### Scenario: 2FA login
- GIVEN a user with 2FA enabled
- WHEN the user submits valid credentials
- THEN an OTP challenge is presented
- AND login completes only after valid OTP

## MODIFIED Requirements

### Requirement: Session Expiration
The system MUST expire sessions after 15 minutes of inactivity.
(Previously: 30 minutes)

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 15 minutes pass without activity
- THEN the session is invalidated

## REMOVED Requirements

### Requirement: Remember Me
(Deprecated in favor of 2FA. Users should re-authenticate each session.)
```

![Image 4: Delta Specs 概念图](https://developer.qcloudimg.com/http-save/10642399/bf7769447ba049ece8ac4e30939f0660.png)

Delta Specs 概念图

_图 4：Delta Specs 三种变更类型及其合并机制_

注意几个细节：

- `MODIFIED Requirements` 里括号标注了 `(Previously: 30 minutes)`——原来的值一目了然
- `REMOVED Requirements` 里说明了废弃原因——不是凭空删掉，而是有上下文
- 每个 Requirement 下面用 `Scenario` 给出具体的 Given/When/Then 测试场景

审查者只需要看这个 Delta 文件，不用翻出主规范做对比。这对团队协作的效率提升是实打实的。

#### Spec 格式的关键元素

|  |  |
| --- | --- |
| `## Purpose` | 规范领域的高层描述 |
| `### Requirement:` | 系统必须具备的特定行为 |
| `#### Scenario:` | 需求的具体示例（Given/When/Then 格式） |
| SHALL/MUST/SHOULD | RFC 2119 关键词，表示需求强度 |

其中 RFC 2119 关键词的含义：

- __MUST/SHALL__ — 绝对要求，没有例外
- __SHOULD__ — 推荐做法，允许特殊情况
- __MAY__ — 可选项

一个重要原则：__规范是行为契约，不是实施计划__。好的规范写"系统应该做什么"，不写"用什么类名、调什么库函数"。如果一个细节的实现方式可以改变而不影响外部行为，那它就不属于 Spec，而是属于 `design.md`。

#### 为什么 Delta 更适合存量项目

翻了一下官方文档，它列了四个理由：

1. __清晰性__——只看变更，不需要在大脑里做 diff
2. __冲突避免__——多个变更可以触及同一规范文件，只要修改不同需求就不会冲突
3. __审查效率__——审查者只看变更，不看未改的上下文
4. __存量适配__——大部分工作是修改已有行为，Delta 让修改变成第一等公民

第四点特别关键。现实中的大部分开发工作不是从零建新系统，而是在已有项目上改东西。Delta 天然适配这种场景——不需要先维护一份完整的系统规范再开始工作，你可以从零开始，每次变更往规范里添砖加瓦。

### 5. 实战：给项目添加暗黑模式

接下来用一个完整的例子走一遍。假设你有一个 React 项目，现在要给它加暗黑模式。

#### Step 1：Propose — 创建变更计划

在 Claude Code（或你用的 AI 工具）中输入：

```
/opsx:propose add-dark-mode
```

AI 会自动创建 `openspec/changes/add-dark-mode/` 目录，并生成所有规划制品：

```
Created openspec/changes/add-dark-mode/
 ✓ proposal.md — why we're doing this, what's changing
 ✓ specs/       — requirements and scenarios
 ✓ design.md    — technical approach
 ✓ tasks.md     — implementation checklist
 Ready for implementation!
```

这一步生成了四个文件，各有各的职责：

- __proposal.md__ — 变更意图和范围（为什么做、做什么、不做什么）
- __specs/__ — Delta Specs，描述暗黑模式对 UI 规范的新增需求
- __design.md__ — 技术方案（用 React Context 还是 Redux、CSS 变量还是 CSS-in-JS 等）
- __tasks.md__ — 实施清单，带编号的复选框

以 `design.md` 为例，AI 可能会生成这样的技术方案：

```
# Design: Add Dark Mode

## Technical Approach
Theme state managed via React Context to avoid prop drilling.
CSS custom properties enable runtime switching without class toggling.

## Architecture Decisions

### Decision: Context over Redux
Using React Context for theme state because:
- Simple binary state (light/dark)
- No complex state transitions
- Avoids adding Redux dependency

### Decision: CSS Custom Properties
Using CSS variables instead of CSS-in-JS because:
- Works with existing stylesheet
- No runtime overhead
- Browser-native solution
```

> 如果你对自动生成的方案不满意，直接编辑这些文件就行。比如你想用 Tailwind 的 `dark:` 前缀而不是 CSS 变量，改完 `design.md` 再继续。OpenSpec 的理念是灵活优先，没有任何阶段限制。

#### Step 2：Apply — 实现任务

确认方案没问题后，执行：

AI 会按 `tasks.md` 里的清单逐项实现：

```
Implementing tasks...
 ✓ 1.1 Add theme context provider
 ✓ 1.2 Create toggle component
 ✓ 2.1 Add CSS variables
 ✓ 2.2 Wire up localStorage
 All tasks complete!
```

实现过程中如果发现问题——比如方案里说用 CSS 变量，但你发现项目已经全局用了 Tailwind——可以直接修改 `design.md` 和 `tasks.md`，然后继续 `apply`。不需要从头来过。

这也是 OPSX 工作流和旧版工作流的本质区别：旧版是线性的，一旦进入实现阶段就很难回头调整规划；OPSX 是流式的，随时可以修改任何制品然后继续。

#### Step 3：Archive — 归档变更

实现完成后，执行：

```
Archived to openspec/changes/archive/2025-01-23-add-dark-mode/
Specs updated. Ready for the next feature.
```

归档做了三件事：

1. __合并 Delta__——把 `add-dark-mode` 里的 Delta Specs 合并到 `openspec/specs/` 的主规范中
2. __移动文件夹__——整个变更目录移到 `changes/archive/`，带日期前缀
3. __保留上下文__——所有制品在归档中保持完整，随时可以回头查看

归档后的目录结构：

```
openspec/
├── specs/
│   └── ui/
│       └── spec.md              ← 已包含暗黑模式的需求
└── changes/
    └── archive/
        └── 2025-01-23-add-dark-mode/   ← 完整历史保留
            ├── proposal.md
            ├── design.md
            ├── tasks.md
            └── specs/ui/spec.md
```

活跃变更目录（`changes/`）只显示进行中的工作，已完成的全移到归档里。需要回顾历史时，打开 `changes/archive/` 就能看到完整的决策记录——不只是改了什么，还包括为什么改、怎么改的。

下次你（或 AI）打开项目，`specs/ui/spec.md` 里已经包含了暗黑模式的行为描述。如果再提一个新变更（比如"添加自定义主题"），新的 Delta 会基于这份更新后的主规范来写。

这就是 OpenSpec 的__良性循环__：Specs 描述当前行为 → Changes 提出修改（以 Delta 形式） → 实现让修改落地 → Archive 合并 Delta 到 Specs → Specs 变成新的真实来源 → 下一个 Change 基于更新后的 Specs 继续。

### 6. 归档机制深入理解

![Image 5: 归档前后对比图](https://developer.qcloudimg.com/http-save/10642399/f38fdd38f8ea5890241be93def4327c5.png)

归档前后对比图

_图 5：归档操作前后的目录结构变化_

归档是 OpenSpec 工作流的关键收尾动作，值得单独展开说一下。

#### 归档时 Delta 的合并逻辑

每个 Delta Section 的处理方式不同：

- __ADDED__ → 直接追加到主规范的 Requirements 部分
- __MODIFIED__ → 找到主规范中同名 Requirement，整体替换
- __REMOVED__ → 找到主规范中同名 Requirement，删除

这个合并逻辑很朴素，但够用。关键是 Requirement 的名称要一致——如果你在 Delta 里把 `Session Expiration` 写成了 `Session Timeout`，合并时就匹配不上。

#### 并行变更的冲突处理

如果多个变更同时修改同一规范文件怎么办？只要修改的是不同 Requirement，就不会冲突。比如 `add-2fa` 和 `fix-session-bug` 都涉及 `auth/spec.md`，但前者新增 2FA 需求，后者修改会话超时时间——归档时互不干扰。

如果你开启了扩展模式，还可以用 `/opsx:bulk-archive` 一次性归档多个变更，它会自动检测并处理冲突。

#### CLI 手动归档

除了在 AI 工具里用 `/opsx:archive`，你也可以在终端手动操作：

```
# 交互式归档（会提示确认）
openspec archive add-dark-mode

# 跳过确认（适合 CI 脚本）
openspec archive add-dark-mode --yes

# 不影响规范的变更（如 CI 配置修改）
openspec archive update-ci-config --skip-specs
```

### 7. CLI 命令速查

日常用得最多的命令都在这里了。所有命令都支持 `--json` 输出，方便在 CI 脚本中使用。

#### 项目管理

|  |  |
| --- | --- |
| `openspec init` | 初始化项目 |
| `openspec update` | 升级 CLI 后更新指令文件 |
| `openspec list` | 列出活跃变更 |
| `openspec list --specs` | 列出所有规范 |
| `openspec show <name>` | 查看变更详情 |
| `openspec validate` | 验证变更和规范的格式 |
| `openspec view` | 交互式仪表盘（终端 UI） |
| `openspec status --change <name>` | 查看某个变更的制品进度 |

#### Schema 管理

|  |  |
| --- | --- |
| `openspec schemas` | 列出可用的 Schema |
| `openspec schema init <name>` | 从零创建自定义 Schema |
| `openspec schema fork <source> <name>` | 从已有 Schema 派生 |
| `openspec schema validate <name>` | 验证 Schema 结构 |

#### 归档操作

|  |  |
| --- | --- |
| `openspec archive <name>` | 归档指定变更 |
| `openspec archive <name> --yes` | 跳过确认直接归档 |
| `openspec archive <name> --skip-specs` | 不影响规范的变更 |

查看某个变更的制品完成进度：

```
openspec status --change add-dark-mode
```

输出示例：

```
Change: add-dark-mode
Schema: spec-driven
Progress: 2/4 artifacts complete

[x] proposal
[ ] design
[x] specs
[-] tasks (blocked by: design)
```

`[x]` 表示已完成，`[ ]` 表示可以创建（依赖已满足），`[-]` 表示被阻塞（依赖未满足）。这个状态图可以帮助你快速了解某个变更卡在哪一步。

你在项目中用过类似的规范管理方案吗？欢迎在评论区聊聊你的经验。

### 8. 进阶：自定义 Schema

OpenSpec 默认使用 `spec-driven` Schema，制品依赖关系形成一张有向无环图（DAG）：

```
              proposal
             (root node)
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
     specs               design
  (requires:          (requires:
   proposal)           proposal)
       │                   │
       └─────────┬─────────┘
                 ▼
              tasks
          (requires:
          specs, design)
```

`proposal` 是根节点，`specs` 和 `design` 都依赖它（但两者互不依赖，可以并行创建），`tasks` 同时依赖 `specs` 和 `design`。

#### 创建自定义 Schema

如果默认 Schema 不适合你的工作流，可以自定义。比如你的团队习惯先做调研再出方案：

```
# 从零创建（交互式）
openspec schema init research-first

# 或者从已有 Schema 派生
openspec schema fork spec-driven research-first
```

创建出来的 Schema 目录结构：

```
openspec/schemas/research-first/
├── schema.yaml
└── templates/
    ├── research.md
    ├── proposal.md
    └── tasks.md
```

在 `schema.yaml` 里定义制品和依赖关系：

```
name: research-first
artifacts:
  - id: research
    generates: research.md
    requires: []           # 调研没有前置依赖

  - id: proposal
    generates: proposal.md
    requires: [research]   # 方案依赖调研结果

  - id: tasks
    generates: tasks.md
    requires: [proposal]   # 跳过 specs 和 design，直接出任务
```

对应的依赖图变成了：`research → proposal → tasks`，比默认 Schema 少了 specs 和 design 两个制品，更适合快速迭代的场景。

官方文档里有一句话说得挺好：__"依赖是赋能者，不是门控。"__ 它们展示的是"什么条件下可以创建某个制品"，而不是"你必须按这个顺序做"。想跳过某个制品？没问题，直接跳。

验证自定义 Schema 的结构：

```
openspec schema validate research-first
```

查看 Schema 的来源（用于调试优先级问题）：

```
openspec schema which research-first
```

Schema 的解析优先级从高到低：项目目录 `openspec/schemas/` > 用户全局 `~/.local/share/openspec/schemas/` > 内置 Schema。

### 9. 项目配置

`openspec/config.yaml` 是可选的，但建议配上。它能让 AI 在生成制品时自动注入项目上下文：

```
# openspec/config.yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js
  API conventions: RESTful, JSON responses
  Testing: Vitest for unit tests, Playwright for e2e

rules:
  proposal:
    - Include rollback plan
    - Identify affected teams
  specs:
    - Use Given/When/Then format for scenarios
  design:
    - Include sequence diagrams for complex flows
```

三个字段的作用：

- __`schema`__ — 新变更的默认 Schema
- __`context`__ — 注入到所有制品的生成指令中，帮 AI 理解项目的技术栈和约定
- __`rules`__ — 按制品 ID 匹配注入特定规则

这样 AI 生成 proposal 时就知道要包含回滚计划，生成 specs 时就知道用 Given/When/Then 格式，生成 design 时就知道复杂流程要画序列图。

Schema 的选择也有优先级：CLI 参数 `--schema` > 变更目录下的 `.openspec.yaml` > `config.yaml` > 默认值。

### 10. 最佳实践与常见问题

#### 命名建议

好的变更命名让 `openspec list` 一目了然：

```
推荐：                         避免：
add-dark-mode                  feature-1
fix-login-redirect             update
optimize-product-query         changes
implement-2fa                  wip
```

用动词开头，表达具体意图，用短横线连接。

#### 变更的粒度

一个逻辑工作单元一个变更。如果你在做"加功能 X 并且重构 Y"，考虑拆成两个变更。这样做的好处：更容易审查、归档历史更清晰、可以独立发布、回滚也更简单。

#### 什么时候该新建变更，什么时候该修改现有变更

官方给了一个简单的判断标准：

|  |  |
| --- | --- |
| 同一个意图，只是实现方式调整 | 修改现有变更 |
| 范围缩小（先发 MVP） | 修改现有变更 |
| 发现代码结构和预想不同 | 修改现有变更 |
| 意图根本变了 | 新建变更 |
| 范围膨胀到完全不同的工作 | 新建变更 |
| 原来的变更可以独立完成 | 先归档，再新建 |

一句话总结：__修改保留上下文，新建提供清晰度__。选择修改当思考历史有价值时，选择新建当重新开始比打补丁更清晰时。

#### 常见问题

__Q：`openspec init` 之后 AI 工具没有识别到命令？__

A：确认 `openspec init` 时选择了正确的工具。也可以手动运行 `openspec update` 重新生成技能文件。如果用的是 Claude Code，检查 `.claude/skills/` 目录下是否有 `openspec-*` 相关文件。

__Q：Delta Specs 里的 Requirement 名称不一致怎么办？__

A：归档合并时靠 Requirement 名称匹配。名称不一致会导致 MODIFIED 和 REMOVED 匹配失败。建议在写 Delta 时参考主规范中已有的 Requirement 名称，保持一致。

__Q：多个变更同时修改同一个 Requirement 怎么办？__

A：这种情况下后归档的变更会覆盖先归档的。建议避免两个变更同时修改同一个 Requirement，或者用 `/opsx:bulk-archive` 让它自动检测冲突。

### 总结

回过头看，OpenSpec 解决的问题其实很朴素：AI 编程助手很强，但如果需求只存在于聊天记录里，结果就是不可控的。加一层轻量级规范，把需求、设计、任务全部落地成文件，AI 就有了明确的行动指南。

Delta Specs 是这套框架的灵魂——只记录变更，不做全量描述。这让它在存量项目上特别好使，因为日常工作的大部分时间不是在建新东西，而是在改已有行为。

三个命令记住就行：`/opsx:propose`（规划）、`/opsx:apply`（实现）、`/opsx:archive`（归档）。流程简单，但解决的问题是实在的。

如果你的团队也在用 AI 编程工具，建议收藏这篇，下次遇到"AI 写的代码跟我想的不一样"的时候翻出来看看。

> __必须说明的一点__：文中的实战案例（添加暗黑模式的 propose → apply → archive 全流程）是基于 OpenSpec 官方文档推导的完整流程演示，__不是实际跑通后的终端实录__。对话输出、制品内容、目录结构变化均为根据文档描述还原的示意，旨在展示工作流的全貌而非复现精确的 AI 输出。
>
> __建议__：动手之前，先装好 OpenSpec，用自己项目里的一个小改动（比如加个按钮、改个文案）完整跑一遍 propose → apply → archive，感受一下每个制品长什么样。小改动跑通了，再往复杂的变更上用，上手会顺畅很多。

__好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！__

来源：https://cloud.tencent.com/developer/article/2660452
