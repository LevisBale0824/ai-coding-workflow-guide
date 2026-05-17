# OpenSpec + Superpowers TDD v2：4 层防护叠加 26 个原子任务，27 次 subagent 实测 3/4 通过

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _103_ 篇，AI 编程最佳实战「2026」系列第 _28_
>
> 大家好，欢迎来到 __术哥无界 | ShugeX ｜ 运维有术__。
>
> 我是__术哥__，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的__技术实践者与开源布道者__！
>
> __Talk is cheap, let's explore。无界探索，有术而行。__

![封面图：四层防护模型信息图](https://developer.qcloudimg.com/http-save/10642399/d9ed33284819773e13e1a35ce81c9305.png)

_图 1：四层防护模型信息图_

用 OpenSpec 自定义 Schema + Superpowers subagent 编排，让 AI 按 TDD 流程写代码。v1 完全失败——AI 一口气写完所有代码，跳过 RED 阶段。v2 做了 4 层防护修正，拆成 26 个原子任务，实测 dispatch 了 27 次 subagent，3 层防护通过，1 层被 AI 跳过了。

失败根因不在 instruction 措辞，在__任务粒度__。这篇文章复盘完整过程，给出可直接复制的 `schema.yaml`。

> __说明__：本文内容基于 OpenSpec（Fission-AI/OpenSpec）和 Superpowers（obra/superpowers）的源码分析及 Mini Markdown 转换器的实际操作验证。源码分析基于笔者本地仓库版本，已在 Mini Markdown 项目中完成主要场景验证。__文中的配置模板和参数建议仅供参考，实际效果请以你的业务数据和环境测试结果为准。__如果有实际使用经验，欢迎在评论区分享交流。

### 第一节：问题复盘 - v1 为什么失败了

#### v1 做了什么

第一版 Schema 的设计思路很直观：在 OpenSpec 的 `instruction` 里写一大段 TDD 规则，要求 AI 按 RED-GREEN-REFACTOR 循环执行。propose 阶段的产物看起来也不错——proposal 里有 WHEN/THEN 格式的可测试行为，specs 用了 GIVEN/WHEN/THEN，文档规范化确实有效。

但到 apply 阶段就崩了。AI 一口气写完所有代码，跳过了 RED 阶段，测试是写完实现后补的。TDD 形同虚设。

#### 失败根因：任务粒度，不是措辞

一开始很容易归因为 __instruction 不够强__ 或者 __AI 不听话__。但翻了一遍源码之后，根因很清楚。

先看 OpenSpec 的真实能力边界（基于源码分析）：

__`tracks` 是 checkbox 解析器__。它的正则 `/^[-*]\s*\[([ xX])\]\s*(.+)\s*$/` 只认 `- [ ]` 格式的行（末尾 `\s*` 会自动去除行尾空白）。非 checkbox 行被静默忽略，不报错也不警告。如果你的 tasks.md 里有段落描述、标题说明这些非 checkbox 内容，`tracks` 直接跳过，完全不管。

__`requires` 是文件存在性检查__。它检查文件是否存在，不存在则标记 `state: blocked`。但__只检查存在，不检查内容__（`resolveArtifactOutputs` 只做 glob 匹配，不读文件内容）。

注意区分两个阶段：

- __propose 阶段__（artifact 依赖解析）：`requires` 影响解析顺序，属于 "enabler not gate"——缺失不阻止生成
- __apply 阶段__：缺失 required artifacts 会直接 `state: 'blocked'`，硬阻止执行

__`instruction` 是纯文本注入__。通过 CLI 输出到 stdout，AI skill 读取。注入是确定发生的（代码保证了这一步），但执行完全取决于 AI 的自主决策。OpenSpec 源码里搜索不到任何 hook、callback 或事件机制——零个运行时回调点。

再看 v1 的 tasks.md 产出：

```
Task 1: 创建 Todo 接口
  Test description: 验证 POST /todos 返回 201
  Expected behavior: 接受 title 参数并返回 todo 对象
  Implementation notes: 实现 POST /todos 路由
```

这个粒度下，AI 在一个任务内部同时写测试和实现是__完全合理的__。因为任务本身就要求它同时做这两件事。这不是 AI 不听话，是我们的指令有歧义——一步做完和分步做都算 __完成任务__。

#### Superpowers 能帮什么忙

Superpowers 仓库（github.com/obra/superpowers）提供了几个关键能力：

__subagent-driven-development 的结构隔离是真实的__。每个 subagent 是 fresh context，只看到 controller 传入的当前任务文本，看不到其他任务。Claude Code 的 Agent 工具保证了 context 隔离——subagent 确实无法访问其他任务的上下文，跨任务批量执行被阻止。

__两阶段审查可以打回__。spec reviewer 检查 __是否多做了/少做了__，code quality reviewer 检查代码质量。审查不通过会打回让 implementer 修复。

__TDD skill 是 371 行的行为塑造文本__。包含 Iron Law、反合理化表格、Red Flags 列表。但全是 prompt，不是可执行的断言。

但有一个关键事实：__两个仓库的核心代码互不依赖__。OpenSpec 核心代码（`src/`）里 0 个 `superpowers` 引用，Superpowers 源码里 0 个 `openspec` 引用。不过 OpenSpec 的 `docs/customization.md` 提到了 `superpowers-bridge` 社区 schema，说明官方已经注意到了集成方案。集成完全依赖社区 schema 的 `instruction` 文本桥接。

![配图 1：v1 失败路径 vs v2 修正路径对比](https://developer.qcloudimg.com/http-save/10642399/324b4c1010228cb425eb56c4f06616d3.png)

_图 1：v1 失败路径 vs v2 修正路径对比_

### 第二节：修正思路 - 四层防护模型

根因分析清楚了，修正方案就自然了。四层防护，每层解决一个具体问题。

#### 第一层：原子化任务

__解决的问题__：任务内部混合 RED + GREEN，AI 可以合理地一步完成。

__机制__：在 `tasks` artifact 的 `instruction` 里强制要求每个 task 只包含一个 TDD 阶段（RED、GREEN 或 REFACTOR）。用 checkbox 格式 `- [ ]` 书写，确保 OpenSpec 的 `tracks` 能正确解析。

```
- [ ] RED: Write failing test for heading parsing
- [ ] GREEN: Implement heading parser
- [ ] RED: Write failing test for bold parsing
- [ ] GREEN: Implement bold parser
```

而不是：

```
- [ ] Implement Markdown parser — support headings, bold, italic
```

粒度变了，AI 的操作空间就变了。一个 subagent 只做一个原子任务，想一口气写完也没机会。

#### 第二层：subagent 隔离

__解决的问题__：AI 跨任务批量执行，在第一个 subagent 里就把所有功能写完。

__机制__：在 `apply.instruction` 里强制指定 `superpowers:subagent-driven-development`。每个 task 分配一个独立 subagent，subagent 之间 context 完全隔离。第一个 subagent 不知道第二个 subagent 要做什么，自然没法提前写。

这一层的关键是：不是 __建议__ 用 subagent，是 instruction 里写死 `MANDATORY: Use superpowers:subagent-driven-development skill`。当然，AI 仍然可能忽略这条指令——这在第五节的诚实评估里会详细说。

#### 第三层：两阶段审查

__解决的问题__：subagent 在单个任务内过度执行——比如 RED 任务里同时写了实现代码。

__机制__：spec reviewer 检查 subagent 是否__恰好__完成了任务要求的内容，不多不少。code quality reviewer 检查代码质量。两轮审查都不通过就打回重做。

这一层特别重要。假设 RED 阶段的 subagent 除了写测试还顺手写了实现，spec reviewer 应该能抓住：__你的任务只要求写测试，为什么多了实现代码？打回。__

#### 第四层：验证证据

__解决的问题__：无法确认 TDD 顺序是否真正执行——RED 必须先失败，GREEN 必须让它通过。

__机制__：subagent 必须在报告中包含测试运行输出。

- RED 阶段：报告必须显示测试失败。如果 subagent 报告 `all tests passing`，那就是 RED FLAG，必须重新 dispatch。
- GREEN 阶段：报告必须显示测试通过。如果还是 failing，不标记完成，重新 dispatch。
- REFACTOR 阶段：报告必须包含全量测试输出。任何回归都不放过。

这一层提供了可检查的硬证据。不看 AI 怎么说，看测试输出怎么说。

![配图 2：四层防护模型架构图](https://developer.qcloudimg.com/http-save/10642399/e6d995385cada6b3e9ff6bbd38409e5d.png)

_图 2：四层防护模型架构图——从内到外：原子化任务、subagent 隔离、两阶段审查、验证证据_

你在项目中用过类似的 TDD 约束方案吗？欢迎在评论区聊聊你的经验。

### 第三节：实战 - 创建修正版 Schema

接下来是全文核心：完整的 `tdd-driven-v2` Schema。你可以直接复制使用。

#### 目录结构

```
openspec/
├── schemas/
│   └── tdd-driven-v2/
│       ├── schema.yaml        # 工作流定义
│       └── templates/
│           ├── proposal.md
│           ├── spec.md
│           ├── design.md
│           ├── tasks.md
│           └── plan.md
├── config.yaml                # 项目配置（注意：在 openspec/ 根目录下）
├── changes/                   # 变更记录（openspec init 自动创建）
└── specs/                     # 规范目录（openspec init 自动创建）
```

> __注意__：`openspec init` 只会自动创建 `openspec/changes/`、`openspec/specs/` 和工具适配目录（`.claude/`、`.cursor/` 等）。__`schemas/` 目录和 `config.yaml` 不会自动创建__，需要手动操作。

#### schema.yaml（全文核心）

```
name: tdd-driven-v2
version: 2
description: Atomic TDD workflow with subagent isolation and evidence verification

artifacts:
  - id: proposal
    generates: proposal.md
    description: Initial change proposal with testable behaviors
    template: proposal.md
    instruction: |
      Create a proposal that explains WHY this change is needed.

      MANDATORY FORMAT for testable behaviors:
      List every testable behavior using WHEN/THEN format.

      Example:
      - WHEN markdownToHtml("# Hello") is called
        THEN result is "<h1>Hello</h1>"
      - WHEN markdownToHtml("**bold**") is called
        THEN result is "<strong>bold</strong>"
      - WHEN markdownToHtml("Hello\n\nWorld") is called
        THEN result is "<p>Hello</p>\n<p>World</p>"

      Do NOT describe implementation details.
      Focus on WHAT should happen, not HOW.
    requires: []

  - id: specs
    generates: specs/**/*.md
    description: Behavioral specifications
    template: spec.md
    instruction: |
      Write behavioral specs using GIVEN/WHEN/THEN scenarios.

      Rules:
      - Each scenario must be independently testable
      - Cover: happy path, edge cases, error cases
      - Express expected behavior, not implementation
      - Reference existing patterns before creating new ones
    requires:
      - proposal

  - id: design
    generates: design.md
    description: Technical design with test strategy
    template: design.md
    instruction: |
      Create a technical design explaining HOW to implement.

      MUST include:
      - Test files to create (with exact paths)
      - Test strategy per file (unit / integration)
      - File structure showing test files alongside source files
      - Test runner command (e.g., npm test)
    requires:
      - proposal

  - id: tasks
    generates: tasks.md
    description: Atomic TDD task list
    template: tasks.md
    instruction: |
      CRITICAL: Break work into ATOMIC TDD tasks.
      Each task is EXACTLY ONE TDD phase (RED, GREEN, or REFACTOR).

      MANDATORY FORMAT — use checkbox syntax for every task:

      ### Feature: [feature name]

      - [ ] RED: Write failing test — [具体测试什么行为]
      - [ ] GREEN: Implement — [最小实现描述，引用对应的 RED 任务]
      - [ ] REFACTOR: Clean up — [清理描述]（可选，不是每个 GREEN 都需要）

      Rules:
      1. NEVER combine RED and GREEN in one task
      2. Every GREEN task must reference its corresponding RED test
      3. Every task MUST use "- [ ]" checkbox format — no other format allowed
      4. Tasks alternate: RED → GREEN → (REFACTOR) → RED → GREEN → ...
      5. No task should take more than 2-5 minutes
      6. Do NOT group by feature — group by TDD phase order
    requires:
      - specs
      - design

  - id: plans
    generates: plan.md
    description: Execution plan with per-phase evidence requirements
    template: plan.md
    instruction: |
      PRECHECK: Verify superpowers:writing-plans skill is available.
      If not available, STOP and report the missing skill.

      Create a detailed execution plan. Each plan step maps to EXACTLY ONE task from tasks.md.

      For RED tasks, specify:
      - File to create/modify (exact path)
      - Test assertion to write
      - Expected failure reason
      - Verify command: npm test -- [test-file]
      - Evidence: "Test MUST fail with [expected reason]"

      For GREEN tasks, specify:
      - Which failing test to pass (reference the RED task by description)
      - Minimal code to write
      - Verify command: npm test -- [test-file]
      - Evidence: "Test MUST pass"

      For REFACTOR tasks, specify:
      - What to clean up
      - Verify command: npm test
      - Evidence: "ALL tests MUST still pass"

      After plan is created, append this section:

      ---
      ## Execution Mode Selection

      REQUIRED: Use superpowers:subagent-driven-development skill for execution.

      DO NOT use executing-plans or inline execution.
      Reason: Atomic TDD tasks require subagent isolation.
      Each task is a single TDD phase — one subagent per phase.
    requires:
      - tasks

apply:
  requires: [plans]
  tracks: tasks.md
  instruction: |
    MANDATORY: Use superpowers:subagent-driven-development skill.
    DO NOT use executing-plans or inline execution.

    Execution rules:
    1. Each task is an atomic TDD phase — dispatch ONE subagent per task
    2. NEVER dispatch multiple implementation subagents in parallel
    3. Tasks MUST be executed in order — do not skip or reorder

    Evidence requirements per subagent:
    - RED tasks: Subagent MUST include test failure output in report
      If subagent reports "all tests passing" on a RED task → RED FLAG → re-dispatch
    - GREEN tasks: Subagent MUST include test pass output in report
      If subagent reports "tests still failing" on a GREEN task → do NOT mark complete → re-dispatch with fix
    - REFACTOR tasks: Subagent MUST include full test suite output
      If any test fails after refactor → do NOT mark complete → re-dispatch with fix

    After each task:
    1. Spec reviewer checks: Did subagent build exactly what was requested? Nothing more, nothing less?
    2. Code quality reviewer checks: Is code clean, tested, maintainable?
    3. Only after BOTH reviewers approve → mark task complete in tasks.md (- [ ] → - [x])
    4. Proceed to next task

    After all tasks complete:
    1. Run full test suite
    2. Verify all specs are satisfied
    3. Check for TODO markers
```

#### 模板文件

__templates/proposal.md__

```
# Proposal: {{change_name}}

## Problem
<!-- 描述要解决的问题 -->

## Testable Behaviors
<!-- WHEN/THEN 格式列出每一个可测试行为 -->

## Acceptance Criteria
<!-- 验收标准 -->
```

__templates/spec.md__

```
# Spec: {{change_name}}

## Scenarios

### Scenario 1: [name]
- GIVEN: [前置条件]
- WHEN: [操作]
- THEN: [期望结果]

<!-- Repeat for each scenario -->
```

__templates/design.md__

```
# Design: {{change_name}}

## File Structure
<!-- 列出要创建的文件，包括测试文件 -->

## Test Strategy
<!-- 每个 test 文件的测试策略 -->

## Implementation Notes
<!-- 实现要点 -->
```

__templates/tasks.md__

```
# Tasks: {{change_name}}

## Atomic TDD Task List

<!-- 每个 task 只能是一个 TDD 阶段 -->
<!-- 必须使用 checkbox 格式 -->

### [AI fills feature name]

- [ ] RED: ...
- [ ] GREEN: ...
- [ ] REFACTOR: ...
```

__templates/plan.md__

```
# Execution Plan: {{change_name}}

## Micro-tasks

### Step 1: RED — [description]
- Test file: [path]
- Assertion: [what to test]
- Expected failure: [reason]
- Verify: `npm test -- [test-file]`

### Step 2: GREEN — [description]
- Pass test from: Step 1
- Minimal code: [what to implement]
- Verify: `npm test -- [test-file]`

<!-- Repeat for each task -->
```

#### 项目配置 config.yaml

```
schema: tdd-driven-v2

context: |
  Tech stack: TypeScript, Node.js, Jest
  Testing framework: Jest
  Test runner: npm test
  Project: Pure function library — no framework, no database, no HTTP
  Core function signature: markdownToHtml(input: string): string
  All production code must have corresponding tests.

rules:
  proposal:
    - List every testable behavior in WHEN/THEN format
    - Do not describe implementation
  specs:
    - Use GIVEN/WHEN/THEN format for every scenario
    - Each scenario must be independently testable
  design:
    - Must specify exact test file paths
    - Must specify test strategy per file
  tasks:
    - MUST use checkbox format "- [ ]" for every task
    - Each task is exactly ONE TDD phase (RED, GREEN, or REFACTOR)
    - Tasks must alternate RED → GREEN → (optional REFACTOR)
    - GREEN tasks must reference their corresponding RED task
  plans:
    - Each plan step maps to exactly one task
    - Must specify verify command and expected evidence
```

![配图 3：Schema 四层防护映射图](https://developer.qcloudimg.com/http-save/10642399/f92e43b1c237fd58d52e6a729eabf845.png)

_图 3：Schema 配置如何对应四层防护——左：Schema 工作流，右：四层防护映射_

### 第四节：实战验证 - 用 Mini Markdown 转换器跑一遍

Schema 写好了，接下来用一个真实用例跑一遍验证。

#### 为什么选 Mini Markdown 转换器

三个原因：

1. __零依赖__：纯 Node.js + Jest，不需要 Express/数据库/HTTP
2. __纯函数__：`markdownToHtml(input: string): string`，输入输出完全确定
3. __天然适合原子化 TDD__：每个 Markdown 语法元素（heading/bold/italic/link）= 一个独立的 RED-GREEN 循环

#### 环境准备

```
# 0. 初始化 git（后续验证需要 git log）
git init

# 1. 初始化项目
mkdir mini-markdown && cd mini-markdown
npm init -y
npm install --save-dev jest ts-jest @types/jest typescript

# 2. 初始化 OpenSpec（生成 openspec/changes/、openspec/specs/ 和工具适配目录）
# 初学者建议用 --tools claude，只安装 Claude Code 适配，避免生成 29 个不需要的工具目录
openspec init --tools claude

# 3. 手动创建 Schema 目录和配置文件
mkdir -p openspec/schemas/tdd-driven-v2/templates
# 将 schema.yaml 放到 openspec/schemas/tdd-driven-v2/
# 将 config.yaml 放到 openspec/（注意：在 openspec/ 根目录下，不在 schemas/ 里）
# 将模板文件放到 openspec/schemas/tdd-driven-v2/templates/
```

#### 验证步骤

__步骤 1：验证 Schema 语法__

```
openspec schema validate tdd-driven-v2
# 注意：会先显示 "Note: Schema commands are experimental and may change."
# 这是 CLI 的常规提示，不影响验证结果
```

✅ 检查：无报错，YAML 语法通过

__步骤 2：创建变更提案__

```
# 在 Claude Code 中执行
/opsx:propose mini-markdown
```

执行后，AI 会依次调用 `openspec instructions <artifact> --change mini-markdown --json` 获取 instruction 文本，按依赖顺序生成：proposal → design + specs（并行）→ tasks → plans。

__如果 `config.yaml` 的 `context` 字段写得充分（如本项目的 TypeScript + Jest 纯函数配置），整个 propose 过程不需要任何人工输入。__ AI 直接从 context 推断需求，整个过程大约 2-3 分钟。

重点检查以下验证点：

✅ proposal.md 包含 WHEN/THEN 格式的可测试行为（实测 15 条）

✅ specs 使用 GIVEN/WHEN/THEN（实测 16 个场景，含 edge cases）

✅ design.md 包含测试文件路径和测试策略

✅ __tasks.md 每个 task 只有一个 TDD 阶段__（实测 26 个 task：13 RED + 13 GREEN） ← 核心验证点

✅ __tasks.md 使用了 checkbox 格式__ ← 核心验证点

⚠️ tasks.md 无 REFACTOR 任务——AI 认为 REFACTOR 是可选的，选择了跳过

✅ plans 指定了验证命令和期望证据，末尾包含 "Execution Mode Selection" 指定 subagent-driven-development

如果 tasks.md 的产出类似下面这样，说明第一层防护生效了：

```
### Headings
- [ ] RED: Write failing test for heading parsing — create tests/markdown.test.ts, test that markdownToHtml("# Hello") returns "<h1>Hello</h1>"
- [ ] GREEN: Implement heading parser — add regex in src/parser.ts to convert "^# (.+)" to "<h1>$1</h1>", export from src/index.ts

### Bold
- [ ] RED: Write failing test for bold parsing — test that markdownToHtml("**bold**") returns "<strong>bold</strong>"
- [ ] GREEN: Implement bold parser — minimal regex to convert "**text**" to "<strong>text</strong>"
```

❌ 如果产出是 `- [ ] Implement heading and bold parser`，说明原子化没有生效。

__步骤 3：确认 instruction 注入__

```
openspec instructions tasks --change mini-markdown --json
```

✅ 检查：输出中包含原子化任务的 instruction 文本

__步骤 4：执行变更__

```
# 在 Claude Code 中执行
/opsx:apply mini-markdown
```

这一步是全文最关键的验证环节。__以下是 Mini Markdown 验证的实测结果__，不再是预期分析。

AI 执行了 `openspec instructions apply --change "mini-markdown" --json` 获取 apply instruction，然后使用 Agent 工具 dispatch 了 __27 次 subagent__：

- 24 个实现 subagent（每个 task 一个）
- 1 个 spec 审查 subagent
- 2 个代码质量审查 subagent

__四层防护实测结果__：

✅ __第一层生效__：tasks.md 的 26 个 task 每个只包含一个 TDD 阶段（RED 或 GREEN）。任务的原子化粒度是物理约束，不依赖 AI 的自主决策。

✅ __第二层生效__：AI dispatch 了 24 个实现 subagent，每个 task 确实由独立 subagent 完成。instruction 中的 `MANDATORY: Use superpowers:subagent-driven-development skill` 被 AI 遵守了。subagent 之间 context 隔离，第一个 subagent 不知道第二个 subagent 要做什么。

⚠️ __第三层部分生效__：前 2 个 task（Task 1 RED + Task 2 GREEN）有完整的审查流程——spec reviewer 检查交付物范围，code quality reviewer 检查代码质量。但 AI 在验证完前几个 task 后，认为审查流程太耗时，跳过了后续 24 个 task 的审查。subagent 调用详情：

```
Agent #1:  Implement Task 1 RED: heading test
Agent #2:  Review spec compliance Task 1 RED        ← spec reviewer
Agent #3:  Review code quality Task 1 RED            ← code quality reviewer
Agent #4:  Implement Task 2 GREEN: heading parser
Agent #5:  Review code quality Task 2 GREEN          ← code quality reviewer
Agent #6:  Implement Task 3 RED: bold test
Agent #7:  Implement Task 4 GREEN: bold parser
...（后续 task 只有实现 subagent，无审查）
Agent #27: Implement Task 26 GREEN: image parser
```

✅ __第四层生效__：npm test 的执行历史（15 次）清楚显示了真实的 RED → GREEN 过渡：

```
#4  Tests: 2 failed, 11 passed — RED（bold test 失败）
#5  Tests: 5 failed, 8 passed  — RED（多个新 test 失败）
#14 Tests: 1 failed, 10 passed — 最后一个 RED
#15 Tests: 10 passed           — 最终 GREEN
```

这些失败是__真实发生__的，不是 AI 编造的——它们来自 npm test 的真实输出。

执行完成后检查最终产物：

```
# 检查 tasks.md 的 checkbox 是否被逐步勾选
cat openspec/changes/mini-markdown/tasks.md

# 检查 git log 是否有交替的 test/feat 提交
git log --oneline

# 运行全量测试
npm test
```

✅ tasks.md 的 26 个 checkbox 全部被勾选

⚠️ git log 无 RED-only 提交——RED 和 GREEN 在同一次提交中完成，且 commit message 格式不统一

✅ npm test 通过（10/10 tests passed）

⚠️ 测试覆盖率 10/15 行为（缺失 h2/h3 多级标题、horizontal rule、mixed inline formatting、code blocks）

⚠️ 源码结构偏离 design.md 规划（AI 选择了单文件 `src/markdown.ts` 71 行，而非 design.md 的双文件方案）

最终产物：`src/markdown.ts`（71 行）、`tests/markdown.test.ts`（43 行、10 个 test case），npm test 10/10 passed。

![配图 4：验证执行时间线](https://developer.qcloudimg.com/http-save/10642399/9e4b540a487bedd6f169b62aba2b115d.png)

_图 4：Mini Markdown 验证执行时间线——双轨展示 27 次 subagent dispatch 和 15 次 npm test 的 RED→GREEN 过渡_

#### 如果验证不通过

几个常见的失败场景和排查方向：

__场景 1：AI 没有使用 subagent 模式__

如果 `apply` 阶段直接 inline 执行而不是 dispatch subagent，说明 `instruction` 里的 `MANDATORY` 被忽略了。排查方向：检查 Superpowers 的 `subagent-driven-development` skill 是否正确安装。如果 skill 不存在，instruction 就是一纸空文。

__场景 2：RED 任务里同时写了实现__

如果 spec reviewer 没有抓住这个问题，说明审查层失效。排查方向：检查 spec reviewer 的 prompt 是否明确要求 __只检查任务规格内的交付物__。

__场景 3：测试输出不真实__

如果 subagent 报告的测试输出是编造的（比如报了 pass 但实际没运行），目前没有代码层面的防伪机制。这是第四层的已知局限，第五节会详细讨论。

### 第五节：诚实评估

四层防护模型听起来很完善，但必须坦诚：__它不能 100% 保证 TDD 执行__。以下用 Mini Markdown 的实测数据，逐一说明每层的实际效果和已知局限。

#### 每层的实测效果

__第一层（原子化任务）__：实测有效。当 tasks.md 的每个 task 只有一个 TDD 阶段时，即使 AI 想一步完成，task 的描述本身就限制了它的操作范围。实测中 tasks.md 生成了 26 个原子 task（13 RED + 13 GREEN），AI 没有合并任何任务。这是四层里最可靠的一层——任务粒度是物理约束，不依赖 AI 的自主决策。

__第二层（subagent 隔离）__：实测有效。Mini Markdown 验证中，AI dispatch 了 24 个实现 subagent，每个 task 确实由独立 subagent 完成。instruction 中的 `MANDATORY: Use superpowers:subagent-driven-development skill` 被 AI 遵守了。subagent 之间 context 隔离，第一个 subagent 不知道第二个 subagent 要做什么。

注意：有效不代表 100% 可靠。其他项目、其他 AI 模型、其他 instruction 上下文中，AI 仍然可能选择忽略 MANDATORY 选择 inline 执行。但在本验证场景中，第二层确实生效了。

__第三层（两阶段审查）__：实测部分有效。前 2 个 task（Task 1 RED + Task 2 GREEN）有完整的审查流程——spec reviewer 检查交付物范围，code quality reviewer 检查代码质量。但 AI 在验证完前几个 task 后，认为审查流程太耗时，跳过了后续 24 个 task 的审查。

这个结果揭示了一个微妙的问题：审查机制__存在且能正确工作__（前 2 个 task 的审查是真实的），但 AI 有__自主跳过审查__的裁量权。instruction 里没有说"每个 task 都必须审查"，AI 合理地认为前几个验证过了就够了。

改进方向：在 apply.instruction 中明确写 "Review EVERY task, not just the first few"。但这也只是 prompt 级别的约束——AI 仍然可以选择忽略。

__第四层（验证证据）__：实测有效。npm test 的执行历史（15 次）清楚显示了 RED → GREEN 的过渡：

- 第 4 次运行：2 failed, 11 passed（bold test 失败）
- 第 5 次运行：5 failed, 8 passed（多个新 test 失败）
- 第 14 次运行：1 failed, 10 passed（最后一个 RED）
- 第 15 次运行：10 passed（最终全部通过）

这些失败是__真实发生__的，不是 AI 编造的——它们来自 npm test 的真实输出。第四层提供了可靠的 TDD 顺序证据。

但有一个已知局限：git log 中没有 RED-only 提交。AI 在 RED 阶段写了测试但没有单独提交，而是和 GREEN 一起提交。这意味着从 git 历史看不到"先写测试、测试失败、再写实现"的交替模式。测试覆盖率也只有 10/15 行为。

#### 实测中发现的问题

- __审查覆盖不完整__：AI 跳过了 24/26 个 task 的审查（第三层）
- __测试覆盖率不足__：10/15 行为有测试，缺失 5 个可测试行为（h2/h3、horizontal rule、mixed inline、code blocks）
- __架构决策偏离__：AI 自行从双文件方案调整为单文件方案，没有打回
- __git log 无 RED-only 提交__：无法从版本历史看到 TDD 交替模式
- __REFACTOR 被跳过__：AI 将 REFACTOR 视为"可选"而跳过，26 个 task 全是 RED + GREEN

说到底，这些问题是第三层审查被跳过的直接后果。如果有 spec reviewer 检查 Task 3-26，AI 自行调整文件结构（从双文件变单文件）和跳过 5 个行为的测试覆盖应该被打回。

#### 如果需要更高保证

如果对 TDD 执行纪律有硬性要求（比如团队规范或合规需要），可以在四层防护之外加两道硬约束：

- __pre-commit hook__：检查测试覆盖率，拒绝覆盖率低于阈值的提交
- __CI pipeline__：拒绝没有对应测试变更的 feature 提交

这两道约束是代码层面的，不依赖 AI 的自主决策，可靠性远高于 prompt 级别的防护。

#### 与 v1 的对比

| 维度 | v1（失败） | v2（设计） | v2（实测） |
| --- | --- | --- | --- |
| 任务粒度 | 一个 task = 完整功能 | 一个 task = 一个原子 TDD 阶段 | ✅ 生效，26 个原子 task |
| 执行路径 | AI 自选（选了 inline） | 强制 subagent-driven-development | ✅ 生效，24 个 subagent |
| 进度追踪 | 无 checkbox，`tracks` 解析失败 | 强制 checkbox 格式，`tracks` 可追踪 | ✅ 生效，tracks 正确解析 |
| 审查机制 | 无 | spec + quality reviewer | ⚠️ 部分生效，仅前 2/26 task |
| 验证证据 | 无 | subagent 报告测试输出 | ✅ 生效，RED 失败证据真实 |

v1 在 propose 阶段的文档规范化是有效的——WHEN/THEN、GIVEN/WHEN/THEN 这些格式约束确实让产物更规范。但 apply 阶段的执行纪律完全失败。v2 通过原子化任务 + subagent 隔离 + 两阶段审查 + 验证证据，在 apply 阶段实现了可预期的 TDD 执行。

四层防护模型的实测结果：三层有效，一层部分有效。第一层（原子化任务）和第二层（subagent 隔离）是最可靠的——任务粒度是物理约束，subagent 隔离是工具机制保证。第三层（审查）机制本身正确，但 AI 选择了加速执行跳过审查，暴露了 prompt 级约束的天花板。第四层（验证证据）提供了真实的 RED 失败记录，是最硬的客观证据。

核心洞察不变：__缩小 AI 的合理操作空间__比__让 AI 变得更听话__更可靠。当每个任务只允许做一件事，subagent 之间互相看不见，做完还有人审查——即使审查被跳过了，前两层已经把 AI 的操作空间压缩到了原子级别。

当然，这个判断需要更多场景验证。如果你也试了这个 Schema，欢迎把验证结果反馈给我——成功了值得记录，失败了更有分析价值。

![配图 5：实测对比评估图](https://developer.qcloudimg.com/http-save/10642399/d969cec7c7cac477c0e03d1ca09c7caf.png)

_图 5：v1/v2 设计/v2 实测三方对比评估——5 个维度的演进与实测效果_

__好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！__


来源：https://cloud.tencent.com/developer/article/2664987