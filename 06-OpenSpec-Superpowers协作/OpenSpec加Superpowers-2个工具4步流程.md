# OpenSpec + Superpowers：2 个工具 4 步流程，让 AI 编程质量不再碰运气

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _100_ 篇，AI 编程最佳实战「2026」系列第 _25_ 篇
>
> 大家好，欢迎来到 __术哥无界 | ShugeX ｜ 运维有术__。
>
> 我是__术哥__，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的__技术实践者与开源布道者__！
>
> __Talk is cheap, let's explore。无界探索，有术而行。__

![封面图：OpenSpec 与 Superpowers 协同工作流概念图](https://developer.qcloudimg.com/http-save/10642399/e676522036d330b7def21dbb6e1f8f81.png)

_图 1：OpenSpec + Superpowers 协同工作流全景_

AI 编程助手帮你写代码，谁帮你验证 AI 写的代码？

这个问题在 Superpowers 项目的 README 里有一个很诚实的答案：项目自身记录了 __24 次 LLM 在验证环节撒谎的失败案例__。一个把 prompt 工程做到极致的框架，承认纯靠 prompt 约束 AI 不可靠。

翻完 Superpowers 和 OpenSpec 的源码后，我发现了一个有意思的互补关系：__Superpowers 擅长执行和自测，但在写什么的质量把控上是软约束；OpenSpec 擅长在编码前建立结构化规格，用硬约束定义写什么，但缺乏执行层面的闭环__。两个工具组合起来，恰好补齐了各自的短板。

> __说明__：本文内容基于 __OpenSpec v1.3.1__ 和 __Superpowers v5.0.7__ 的本地源码分析整理而成。文中关于工具局限性的分析均对应源码中的实际实现逻辑，标注了关键文件路径和行号。__文章仅代表基于当前版本源码的技术分析，工具后续版本可能已修复或调整相关实现，请以官方最新版本为准。__ 如果你在实际使用中有不同发现，欢迎在评论区分享交流。

### 1. Superpowers 的软约束困境

Superpowers 的核心理念很纯粹：用精心设计的 markdown 技能文件注入到 LLM 上下文，强制 agent 走 brainstorming → spec → plan → TDD → review 的完整流程。听起来很完美。

但源码告诉我们另一个故事。

#### 所有铁律都只是自然语言

Superpowers 的技能发现机制中有一段标志性的指令：

```
<EXTREMELY-IMPORTANT>
IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.
This is not negotiable. This is not optional.
</EXTREMELY-IMPORTANT>
```

这段来自 `using-superpowers/SKILL.md` 的注入逻辑。注意，这确实是纯文本指令。框架包含 15 种__反合理化__条目来防止 agent 偷懒走捷径，但__没有一行可执行代码来强制执行这些规则__。

这意味着什么？低能力模型或长上下文场景下，指令遵从率会下降。所有__规则__、__铁律__、__硬门控__都只是 prompt，不是程序逻辑。

#### 审查循环可能无限转下去

Superpowers 的反馈循环设计了两阶段审查：先做规格合规审查（`spec-reviewer-prompt.md`），再做代码质量审查（`code-quality-reviewer-prompt.md`）。逻辑是：

```
规格审查员确认代码匹配规格？
  → 否 → 实现者修复 → 重新规格审查
  → 是 → 派遣代码质量审查员
代码质量审查员批准？
  → 否 → 实现者修复 → 重新代码质量审查
  → 是 → 标记任务完成
```

这个流程有三种显式终止条件：规格审查通过、代码质量审查通过、用户手动批准。但缺少了关键的安全阀：__没有最大迭代次数限制，没有超时机制，没有成本上限__。

对于包含 5 个 task 的计划，最少需要 16 次 subagent 派遣（1 实现者 + 1 规格审查者 + 1 代码质量审查者，每个 task）。如果每轮审查失败 1 次，数字就膨胀到 26 次。在没有硬性终止条件的情况下，资源消耗完全不可预测。

#### 每个 task 都是新来的

Superpowers 的 subagent 机制是每个 task 创建独立的子代理。`controller` 构造上下文传递给 subagent，但 subagent __不继承 session 的完整历史__。

这带来了跨任务架构一致性问题：如果 Task 3 的实现者改了某个接口，Task 7 的实现者不会知道。每个 subagent 都像是新来的，只拿到 `controller` 认为它需要的信息。信息传递的完整性完全依赖 controller 的判断，而 controller 本身也是 LLM。

#### `零依赖`的代价

Superpowers 号称零依赖，不依赖任何第三方工具或库。这个设计选择有明确的权衡：__无法使用 AST 解析器、linter/formatter、静态分析工具、测试覆盖率工具__。所有代码质量检查都依赖 LLM 的代码阅读能力。

说到底，当一个框架的所有规则都依赖被约束者的自觉遵守时，其可靠性上限取决于 LLM 本身的可靠度。这就是 Superpowers 的根本限制。

### 2. OpenSpec 的硬约束解决了什么

OpenSpec 走了一条完全不同的路：__不依赖 prompt 约束 LLM 的行为，而是用结构化规格约束要构建什么这件事本身__。

![OpenSpec 四层架构图](https://developer.qcloudimg.com/http-save/10642399/9a6c6d92bde177e314abf1806e004b2b.png)

_图 2：OpenSpec 内部架构——从 CLI 到 Core 的分层结构与数据流_

#### 结构约束：不是建议，是门禁

OpenSpec 的验证引擎（`src/core/validation/`）定义了硬性指标：

- Spec 文件必须包含 `name`、`overview`、`requirements` 三个字段
- 每个 Requirement 必须包含 `SHALL` 或 `MUST` 关键字（遵循 RFC 2119 规范）
- 每个 Requirement 至少有一个 Scenario
- Change Proposal 的 `why` 字段长度必须在 50-1000 字符之间
- 每个 Change 的 deltas 数量限制在 1-10 个

这些不是 prompt 里的请尽量遵守，而是 Zod schema 定义的数据结构验证。不符合规格的文件根本无法通过验证。

#### DAG 依赖图：让 artifact 生成有序可追溯

OpenSpec 的核心模块之一是 Artifact Graph Engine（`src/core/artifact-graph/graph.ts`），使用 Kahn 算法实现拓扑排序，管理 artifact 之间的依赖关系。

这意味着：当你创建一个 Change Proposal 时，OpenSpec 不是让 AI 自由发挥，而是按照依赖图顺序，逐个生成 artifact。每个 artifact 的完成状态通过文件系统检测（`state.ts` 行 14-29），确保不遗漏。

#### Schema 三级解析：规格可扩展

OpenSpec 的 Schema Resolution（`src/core/artifact-graph/resolver.ts` 行 63-91）实现了三级优先级解析：

1. __项目本地__（`.openspec/schemas/`）- 项目自定义 schema
2. __用户全局__（`~/.openspec/schemas/`）- 用户偏好 schema
3. __包内置__ - OpenSpec 默认 schema

这个设计让团队可以在不修改框架代码的情况下，扩展自己的规格约束。

#### Delta Spec 管理：变更可追溯

OpenSpec 使用 Delta Spec 管理规格变更，格式固定为：

```
## ADDED Requirements
## MODIFIED Requirements
## REMOVED Requirements
## RENAMED Requirements
```

应用顺序也是硬编码的：RENAMED → REMOVED → MODIFIED → ADDED。这不依赖 LLM 的判断，而是程序逻辑保证。

但 OpenSpec 也有自己的局限。验证引擎并非自动强制运行的（不作为 pre-commit hook 或 CI gate），Scenario 格式约束较弱（只检查 header 存在），Verify 命令是纯 AI 驱动的（没有程序化验证逻辑）。它解决了__定义质量__的问题，但__执行质量__仍然依赖 AI 的表现。

### 3. 互补机制：硬约束 + 执行自测

把两个工具放在一起看，互补关系非常清晰。

![OpenSpec 与 Superpowers 互补关系对比图](https://developer.qcloudimg.com/http-save/10642399/13c3379a012b8cdfd9292c173e763e12.png)

_图 3：OpenSpec（定义质量）与 Superpowers（执行质量）的六维互补关系_

|  |  |  |
| --- | --- | --- |
| 规格定义 | prompt 引导（软约束） | Zod schema + 验证引擎（硬约束） |
| 执行流程 | 7 阶段工作流（prompt 驱动） | 4 步生命周期（命令驱动） |
| 质量检查 | 两阶段审查（LLM 审查 LLM） | 结构验证 + 质量门禁（程序化） |
| 变更管理 | 无内置机制 | Delta Spec + 拓扑排序 |
| 终止条件 | 缺失（无最大迭代） | 验证门禁 + 任务完成度检查 |
| 工具适配 | Claude Code 原生 | 25+ AI 工具适配器 |

__核心互补逻辑__：OpenSpec 在编码前建立结构化规格，用硬约束确保__要构建什么__是清晰的；Superpowers 在编码过程中提供 7 阶段执行流程和两阶段审查，确保__怎么构建__是可靠的。前者解决的是__定义质量__问题，后者解决的是__执行质量__问题。

具体来说：

__OpenSpec 弥补 Superpowers 的 4 个短板：__

1. __软约束 → 硬约束__：Superpowers 的铁律是 prompt，OpenSpec 的 spec 验证是 Zod schema。规格不合规，直接报错，不依赖 LLM 的自觉。
2. __无终止条件 → 质量门禁__：OpenSpec 的归档流程（archive）要求 proposal 和 delta spec 验证通过、任务完成度检查通过。这给 Superpowers 的审查循环提供了一个外部的__终点线__。
3. __跨 task 信息断层 → Delta Spec 可追溯__：OpenSpec 的 Delta Spec 机制让每个变更都有结构化记录。即使 Superpowers 的 subagent 不继承完整上下文，至少有 Delta Spec 可以查询变更历史。
4. __纯 LLM 审查 → 程序化验证兜底__：Superpowers 的两阶段审查完全依赖 LLM，OpenSpec 的验证引擎用程序化逻辑兜底。两道防线总比一道强。

__Superpowers 弥补 OpenSpec 的 3 个短板：__

1. __验证不自动 → 执行流程强制覆盖__：OpenSpec 的验证需要手动触发，Superpowers 的 7 阶段工作流确保每个环节都被覆盖。
2. __Verify 是纯 AI → 两阶段审查细化__：OpenSpec 的 Verify 命令没有程序化验证逻辑，Superpowers 的规格合规审查 + 代码质量审查提供了更细粒度的检查。
3. __无执行闭环 → TDD + review 闭环__：OpenSpec 定义了做什么，但不指导怎么做。Superpowers 的 TDD 流程和代码审查形成了执行闭环。

你在项目中试过类似的组合方案吗？比如先用规格工具定义清楚需求，再让 AI 按流程执行。欢迎在评论区聊聊你的经验。

### 4. 极简工作流：4 步落地

理论讲完了，具体怎么在项目里用起来？这里给出一套经过验证的 4 步工作流。

![4 步工作流流程图](https://developer.qcloudimg.com/http-save/10642399/9ebc89d23b1b112df1b76cfb3875e3ab.png)

_图 4：OpenSpec + Superpowers 四步闭环工作流——从定义规格到验证归档_

#### 第 1 步：用 OpenSpec 定义规格

```
# 初始化 OpenSpec
npx @fission-ai/openspec init

# 创建规格文件
npx @fission-ai/openspec spec
```

这一步产出的是结构化的 spec 文件，包含 `name`、`overview`、`requirements`（每个 requirement 带 `SHALL`/`MUST` 关键字和至少一个 scenario）。

关键点：__不要跳过 requirements 的 scenario 定义__。OpenSpec 的验证引擎虽然对 scenario 格式约束较弱（只检查 header 存在），但 well-defined scenario 是后续 Superpowers 审查的基准。

#### 第 2 步：用 OpenSpec 创建 Change Proposal

```
# 创建变更提案
npx @fission-ai/openspec change
```

OpenSpec 会按照 DAG 依赖图顺序，逐个生成 artifact（`src/core/artifact-graph/graph.ts` 使用 Kahn 算法拓扑排序）。`why` 字段要求 50-1000 字符，强制你思考清楚为什么要做这个变更。

这一步的产出是结构化的 Change Proposal + Delta Spec。Delta Spec 明确定义了 ADDED / MODIFIED / REMOVED / RENAMED 的 requirements，后续 Superpowers 的规格审查就以这个 Delta Spec 为基准。

#### 第 3 步：用 Superpowers 执行实现

```
# Superpowers 通过 session-start hook 自动注入
# 在 Claude Code 中直接开始工作
```

这一步 Superpowers 接管执行层。agent 按照 brainstorming → plan → TDD → review 的流程工作。关键是：__Superpowers 的规格审查以 OpenSpec 的 Delta Spec 为准__，而不是自己临时生成规格。

审查循环的资源消耗可以这样控制：手动设定每类审查的最大轮次（比如规格审查 3 轮、代码质量审查 2 轮），作为 OpenSpec archive 的前置条件。这弥补了 Superpowers 缺少硬性终止条件的问题。

#### 第 4 步：用 OpenSpec 验证并归档

```
# 验证变更
npx @fission-ai/openspec validate

# 归档
npx @fission-ai/openspec archive
```

归档时 OpenSpec 会执行三重检查：proposal 验证、delta spec 验证、重建 spec 验证。这是整个工作流的质量门禁。

如果 Superpowers 的审查循环还在跑，OpenSpec 的归档会挡住不合规的变更。两个工具的验证逻辑形成了双重保险。

#### 配置要点

```
项目根目录/
├── .openspec/
│   ├── config.yml          # OpenSpec 配置
│   ├── schemas/            # 项目自定义 schema（可选）
│   └── changes/            # 变更提案目录
├── using-superpowers/
│   └── SKILL.md            # Superpowers 技能注入文件
└── .claude/
    └── hooks/
        └── session-start   # Superpowers 注入 hook
```

OpenSpec 和 Superpowers 的配置是独立的，不存在冲突。OpenSpec 负责 `.openspec/` 目录下的规格管理，Superpowers 通过 `.claude/hooks/session-start` 注入执行流程。两者在文件系统层面互不干扰，但在工作流层面紧密协同。

### 总结

AI 编程的质量问题，本质上是两件事：__定义不清楚__和__执行不到位__。

Superpowers 把执行流程做到了极致——7 阶段工作流、两阶段审查、subagent 隔离。但所有规则都是 prompt，不是代码。OpenSpec 把定义质量做到了程序化——Zod schema 验证、DAG 依赖管理、Delta Spec 变更追踪。但执行层面缺乏闭环。

两个工具的组合逻辑很简单：__OpenSpec 用硬约束定义清楚做什么，Superpowers 用结构化流程确保怎么做。__ 一个管规格，一个管执行。加上 OpenSpec 归档时的质量门禁作为外部的终止条件，Superpowers 审查循环的无限转问题也有了兜底。

说到底，快速开发和高质量代码之间，差距不在工具本身，而在于你有没有在 AI 开始写代码之前，把要写什么这件事定义清楚。OpenSpec 解决的就是这一个问题。

如果你的团队也在用 AI 编程助手，建议收藏这篇文章。下次遇到 AI 生成的代码质量不稳定时，可以试试__先规格、后执行__的组合方案。

__好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！__


来源：https://cloud.tencent.com/developer/article/2663959