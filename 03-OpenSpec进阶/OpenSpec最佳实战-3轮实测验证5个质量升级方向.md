# OpenSpec 最佳实战：3 轮实测验证 5 个质量升级方向，结论一目了然

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _107_ 篇，AI 编程最佳实战「2026」系列第 _32_
>
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
>
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的**技术实践者与开源布道者**！
>
> **Talk is cheap, let's explore。无界探索，有术而行。**

![封面图：三轮实验对比信息图](https://developer.qcloudimg.com/http-save/10642399/75bc4a02643fee8d40066a45a10e5884.png)

_图 1：三轮实验全貌——从裸跑到结构化门控_

上篇文章：《OpenSpec 实战复盘：4 步流程 + 5 项升级，让 AI 编程从能跑到跑对》，复盘 OpenSpec 四步法的时候，我提了 5 个质量升级方向，末尾留了一句话：**还没有用真实项目验证过**。

这篇就来兑现承诺。

验证方式很简单：用同一个功能需求 - 给已有 TODO API 加任务优先级（priority 字段支持 Low / Medium / High），跑三轮实验。Lab 1 裸跑基线对照，看原始产出；Lab 2 加 Rules + Validate + Explore，看低成本方案改善多少；Lab 3 自定义 Schema 插入 Review 工件，看中等成本方案能做到什么程度。

三轮跑完，5 个升级方向逐一对标。

> **说明**：本文内容基于 OpenSpec（Fission-AI/OpenSpec）源码和笔者本地实际操作验证。三轮 Lab 均在同一 TODO API 项目中完成，主要场景已实际跑通。**文中的配置模板和参数建议仅供参考，实际效果请以你的业务数据和环境测试结果为准。**OpenSpec 仍在快速迭代，部分命令（如 schema fork）标记为 experimental，后续版本可能有变化。如果有实际使用经验，欢迎在评论区分享交流。

### 你将看到什么

- ✅ 裸跑 propose → apply → archive 的真实产出长什么样
- ✅ 加 Rules + Explore 后 Spec 质量有多大改善
- ✅ 自定义 Schema 插入 Review 工件的完整操作
- ✅ 5 个升级方向的逐一对标结论表

### 环境准备

#### 你需要准备

- OpenSpec CLI 已安装（`openspec --version` 能正常输出）
- 一个现有的 TODO API 项目（或任何你想加功能的项目）
- Claude Code / Cursor 等支持 OpenSpec skills 的 AI 编程工具

#### 预计时间

⏱️ 跟着做三轮实验大约需要 30-40 分钟

#### 难度等级

⭐⭐ 中级 - 需要了解 OpenSpec 四步法的基本概念

### Lab 1：裸跑 propose → apply → archive（基线对照）

这一轮不加任何质量措施，纯粹走一遍 `/opsx:propose` → `/opsx:apply` → `/opsx:archive`，看看原始产出长什么样。

说白了这个 Lab 的目的不是做对，而是**看看不做额外约束时 AI 会产出什么**。

#### Propose：一键生成工件

在 AI 编程工具中执行：

不需要手动先执行 `openspec new change` - propose skill 内部会自动创建 change 目录。如果你已经手动创建了同名 change，propose 会检测到并问你：**Continue existing** 还是 **Recreate**？选 Recreate 就行。

AI 会从 change name 推断需求（本例中 `todo-priority` 被推断为"加 priority 字段"），然后按 schema 定义的依赖顺序生成四个工件。实际输出类似这样：

```
**Change created: `todo-priority`**

Location: `openspec/changes/todo-priority/`

Artifacts created:
- **proposal.md** — defines the problem and motivation for adding priority support
- **design.md** — technical decisions including default priority, strict enum validation, sort order, and timestamp format change
- **specs/todo-priority/spec.md** — detailed requirements with scenarios covering normal cases, null/empty/invalid priority values
- **tasks.md** — 6 implementation task groups with paired test tasks

All artifacts complete! Ready for implementation.

Run `/opsx:apply` to start implementing.
```

四个工件分别是：`proposal.md`（做什么、为什么）、`specs/todo-priority/spec.md`（行为规范）、`design.md`（怎么做）、`tasks.md`（实现步骤）。

> ⚠️ **注意**：propose 不一定会问你需求 - 如果 change name 足够清晰，AI 可能跳过需求确认直接生成。如果你想让 AI 按你的想法来做，可以在执行 propose 之前先通过 `/opsx:explor`e 澄清需求（Lab 2 会演示）。

#### Apply：执行任务

AI 读取所有工件后，按 tasks.md 里的顺序逐个实现。每个 task 完成后会把 `- [ ]` 改成 `- [x]`。

有个真实细节值得注意：AI 第一次运行测试时有 1 个没通过（`should reject undefined priority type`），然后自动修复了代码，第二次全部通过。这个 RED-GREEN 循环是 AI 自发完成的：

```
# 第一次运行 - 1 个失败
⎯⎯⎯⎯⎯⎯⎯ Failed Tests 1 ⎯⎯⎯⎯⎯⎯⎯
FAIL  src/store.test.ts > priority validation > should reject undefined priority type
 Test Files  1 failed (1)
      Tests  1 failed | 17 passed (18)

# 修复后运行 - 全部通过
 Test Files  1 passed (1)
      Tests  18 passed (18)
```

实现完成后的完整汇报：

```
## Implementation Complete

**Change:** todo-priority
**Schema:** spec-driven
**Progress:** 15/15 tasks complete ✓

### Implemented This Session

**Files Created:**
- `src/types.ts` — Todo interface with Priority type
- `src/store.ts` — Store with priority validation and sorting
- `src/routes.ts` — Express routes with priority support
- `src/store.test.ts` — 18 unit tests

**Test Results:** 18 passed

All tasks complete! You can archive this change with `/opsx:archive`.
```

注意：`/opsx:apply` 不支持 `--task` 参数指定单个任务，它是按顺序遍历所有 task 的。

#### Archive：归档

AI 会将整个 change 目录移动到 `openspec/changes/archive/` 下，自动以日期命名（如 `2026-05-09-todo-priority/`）。归档过程是自动完成的，不需要手动选择或确认。

```
## Archive Complete

**Change:** todo-priority
**Schema:** spec-driven
**Archived to:** `openspec/changes/archive/2026-05-09-todo-priority/`
**Specs:** No delta specs (new capability only)

All artifacts complete. All tasks complete.
```

> ⚠️ **注意**：当前 archive 实现是直接移动目录，不会将 delta specs 同步到 `openspec/specs/` 主目录。如果你的后续变更需要读取已有 specs，可能需要手动处理。

#### Lab 1 产出诊断

裸跑完之后，我检查了一下产出，结果比预期好一些 - 但也暴露了更根本的问题：

| | | |
| --- | --- | --- |
| Spec 有没有覆盖边界条件 | ✅ 碰巧覆盖了 | null、空字符串、无效值都有场景 - 但 AI 是从 change name 推断的，不是你告诉它的 |
| tasks 粒度 | ⚠️ 还行 | 拆成了 6 组 15 个 task - 但 AI 自行加了 ISO 8601 时间戳的 breaking change，这个需求没人提过 |
| 有没有测试任务 | ✅ 碰巧配对了 | 每组都有测试 task - 但这是 AI 的自觉行为，没有结构约束保证 |
| 有没有回滚方案 | ⚠️ 有但不完整 | design.md 有 Migration Plan - 但没提具体回滚 SQL |

总结：**四步法保证流程完整性，但不保证需求对齐**。这次 propose 没有问用户需求 - AI 从 `todo-priority` 这个名字猜了全部设计，碰巧猜得不错。但如果你要做的功能名字不够直白（比如叫 `user-experience-upgrade`），或者你对边界条件有特定要求，裸跑就可能跑偏。

这就是下面两轮 Lab 要解决的问题 - 不是产出质量差，而是**产出不可控**。

### Lab 2：加 Rules + Validate + Explore（低成本方案）

Lab 1 暴露的问题，不需要改 OpenSpec 的 Schema 也能部分解决。具体做法：propose 前先用 Explore 澄清需求，创建 config.yaml 写入 rules，propose 后用 validate 检查产出。

这一轮对标上篇的**升级 1（Spec Review）**和**升级 4（TDD 思路）**。

#### Step 1：Explore 澄清需求

裸跑时 AI 从 change name 推断了需求，碰巧还行，但需求毕竟是 AI 猜的。这次先用 Explore 模式把模糊地带想清楚：

在对话中抛出你的核心问题：

> 我要给 Todo API 加 priority 字段（Low / Medium / High）。帮我想想：
> 1. 这个字段的可选值会不会扩展？需不需要做成 enum 还是 string？
> 2. priority 为 null 时怎么处理？默认值策略是什么？
> 3. 有没有排序需求？按 priority 排序时 Low / Medium / High 的权重怎么定？
> 4. 现有数据迁移策略：已有任务的 priority 怎么设？

AI 会画出决策矩阵帮你理清思路：

```
┌─────────────────────────────────────────────────────────────┐
│                    Priority 设计决策                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. 类型选择                                                │
│   ┌──────────────┐         ┌──────────────┐                │
│   │    Enum      │◀───────▶│    String     │                │
│   │  type-safe   │         │   flexible    │                │
│   └──────────────┘         └──────────────┘                │
│                    ▼                                        │
│   2. Null 处理                                               │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │ NOT NULL     │  │ NULL = Medium│  │ NULL = Low   │     │
│   │ 强制有值      │  │ 隐式默认值   │  │ 显式缺失     │     │
│   └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│   3. 排序权重                                                │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │ HIGH→MED→LOW │  │ LOW→MED→HIGH │  │ MED→HIGH→LOW │     │
│   │ 紧急优先      │  │ 后台任务优先  │  │ 折中方案     │     │
│   └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│   4. 迁移策略                                                │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │ 全部设为Medium│  │ 用户手动设置  │  │ 脚本批量迁移 │     │
│   └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

然后 AI 会反问四个澄清问题。我的回答是：

> 三个固定值基本够用，2 我不确定你来分析决定，3 最紧急的在前，4 一次性迁移

AI 分析了 null 语义后给出推荐方案：

```
## 推荐方案
- 类型：Enum（固定三个值）
- 数据库：NOT NULL，默认值：'medium'
- 排序权重：HIGH → MEDIUM → LOW
- 迁移：一次性，SQL UPDATE SET priority='medium'
```

我确认后，AI 总结了全部决策，并建议退出 explore 进入 propose。

Explore 的价值不是生成代码，而是**在写 Spec 之前把模糊地带想清楚**。三轮对话下来，priority 的类型、默认值、排序、迁移策略全部明确了 - 这些都是 Lab 1 里 AI 自己猜的。

#### Step 2：创建 config.yaml 写入 Rules

OpenSpec 的 rules 机制很有意思：它不是追加到 instruction 后面的文本，而是**独立的约束字段**，单独传递给 AI。rules 在 prompt 中以 **Do NOT include in output** 和 **Apply as constraints** 的指令出现，相当于给 AI 画了一道红线。

创建配置文件：

然后在 `openspec/config.yaml` 中写入：

```
rules:
  specs:
    - 每个数据字段的变更，必须覆盖 null、空值、越界三种异常场景
    - Scenario 必须使用 #### 四级标题，否则会被静默忽略
  design:
    - 涉及数据库 migration 的设计，必须包含回滚方案
  tasks:
    - 所有实现任务必须配对对应的测试任务，测试任务写在实现任务正下方
    - 单个 task 不超过 30 分钟工作量
```

几个关键点：

- **rules 按 artifact ID 分组**，key 是工件名（specs、design、tasks），value 是规则列表。不是平铺的列表格式
- **位置固定**：放在项目根目录的 `openspec/config.yaml` 下
- **key 必须和 schema 中的 artifact id 对应**：默认 schema 有 proposal、specs、design、tasks 四个工件

#### Step 3：Propose 生成工件

```
# 重新创建变更
openspec new change todo-priority-v2
```

然后执行 propose：

有意思的是，Lab 2 的 propose 也没有问用户需求 - 但这次不是"没问"，而是 Explore 阶段已经澄清了需求，AI 记住了上下文。这说明 explore + propose 的组合比单独 propose 更可靠：**需求是用户说的，不是 AI 猜的**。

实际输出：

```
**Change created: `todo-priority-v2`**

Location: `openspec/changes/todo-priority-v2/`

Artifacts created:
- **proposal.md** — defines three-level priority enum, sorting, validation
- **design.md** — enum type, NOT NULL default 'medium', migration plan
- **specs/todo-priority/spec.md** — 4 requirements with scenarios (normal, null, empty, invalid cases)
- **tasks.md** — 6 task groups with paired test tasks (15 total implementation + test tasks)

All artifacts complete! Ready for implementation.

Run `/opsx:apply` to start implementing.
```

这时 AI 生成 Spec 时会自动把 rules 作为约束条件。

#### Step 4：Validate 检查产出

```
openspec validate todo-priority-v2 --type change
```

这个命令检查 delta spec 的结构合规性 - 比如 specs 目录下是否有 `## ADDED Requirements` 等 headers、Scenario 是否使用了 `####` 四级标题。

> ⚠️ **注意**：`--type change` 参数是必需的 - `openspec validate` 不能直接识别 change 名称。你也可以用 `openspec validate --changes` 批量验证所有 change。

通过验证后你会看到：

```
Change 'todo-priority-v2' is valid
```

如果 delta spec 格式有问题（比如缺少 Scenario 块），validate 会报错误：

```
Change 'todo-priority-v2' has issues
✗ [ERROR] file: Change must have at least one delta...
```

validate 检查的是**结构合规性**，不是业务逻辑正确性。它不会告诉你 Spec 里忘了处理 priority 为空的场景，也不会检查 tasks.md 里有没有测试任务 - 这些需要 rules 和 review 工件来约束。

#### Lab 2 产出对比

| | | | |
| --- | --- | --- | --- |
| 异常场景覆盖 | ✅ AI 自行覆盖 | ✅ 覆盖更完整（多了 null=400） | 轻微 |
| 回滚方案 | ⚠️ 有但不完整 | ✅ 有 Rollback + migration plan | 明显 |
| 测试任务 | ✅ 每组配对 | ✅ 每组配对 | 无差异 |
| tasks 粒度 | 6 组 15 个 | 6 组 19 个 | 更细 |
| 迁移策略 | ⚠️ 提到但不完整 | ✅ 专门 Data Migration 任务组 | 明显 |
| 需求来源 | ❌ AI 从 change name 推断 | ✅ 基于用户多轮确认 | **关键差异** |

核心发现：Lab 2 产出质量的提升**主要来自 Explore 阶段的多轮需求澄清**，而不是 rules 本身。Rules 确实被传递了，但效果很难单独量化 - 因为 Lab 1 的基线已经不低了。Lab 2 真正的价值在于：**需求是用户说的，不是 AI 猜的**。

但有一个问题 rules 解决不了：**tasks 粒度的控制不够精确**。rules 写了"不超过 30 分钟工作量"，但 AI 对 30 分钟的估算本身就不可靠。更好的方式是在 Schema 层面定义 task 的结构约束。

这就是 Lab 3 要解决的问题。

### Lab 3：自定义 Schema + Review 工件（中等成本方案）

如果说 Lab 2 是在不改 OpenSpec 结构的前提下打补丁，Lab 3 就是**直接改骨架**。

具体做法：fork 一个现有的 spec-driven schema，在 design 和 tasks 之间插入一个 review 工件，让 AI 在生成 tasks 之前先做一轮质量审查。

这一轮对标**升级 2（原子任务 + 检查点）**和**升级 5（质量门控）**。

#### Step 1：查看可用 Schema

实际输出：

```
Available schemas:

  spec-driven
    Default OpenSpec workflow - proposal → specs → design → tasks
    Artifacts: proposal → specs → design → tasks
```

默认只有 `spec-driven` 这一个 schema。

#### Step 2：Fork Schema

```
openspec schema fork spec-driven with-review
```

实际输出：

```
Note: Schema commands are experimental and may change.
- Forking 'spec-driven' to 'with-review'...
✔ Forked 'spec-driven' to 'with-review'

Source: /opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven (package)
Destination: /Users/shuge/.../todo-api/openspec/schemas/with-review

You can now customize the schema at:
  /Users/shuge/.../todo-api/openspec/schemas/with-review
```

注意第一行 `Schema commands are experimental` - 这个功能可能还会变。这会在本地创建一个新的 schema 叫 `with-review`，基于 `spec-driven` 的全部配置。

#### Step 3：修改 Schema，插入 Review 工件

Fork 完成后，找到 schema 配置文件（通常在 `openspec/schemas/with-review/` 目录下），修改 artifacts 定义。

核心改动：在 design 和 tasks 之间插入一个 `review` 工件。

Schema 配置的关键部分（示意）：

```
artifacts:
  - id: proposal
    generates: proposal.md
    template: proposal.md
    description: 变更提案 - 做什么、为什么做
    requires: []

  - id: specs
    generates: specs/**/*.md
    template: spec.md
    description: 行为规范 - 系统应该做什么
    requires: [proposal]

  - id: design
    generates: design.md
    template: design.md
    description: 技术设计 - 怎么做
    requires: [proposal]

  # ⬇️ 新增的 review 工件
  - id: review
    generates: review.md
    template: review.md   # ← 需要在 templates/ 目录下创建此文件
    description: 五维审查 - 检查设计方案和 Spec 的完整性
    instruction: |
      从五个维度审查所有工件的完整性：
      1. 边界条件：Spec 是否覆盖 null、空值、越界、并发等异常场景
      2. 回滚方案：数据库变更是否包含回滚策略
      3. 测试覆盖：Design 是否明确了需要测试的场景和用例
      4. 向后兼容：是否分析了现有接口和数据的兼容性
      5. 任务粒度：初步判断 tasks 应该怎么拆分才合理
    requires: [proposal, specs, design]

  - id: tasks
    generates: tasks.md
    template: tasks.md
    description: 实现任务列表
    requires: [review]   # ← 从 [specs, design] 改为 [review]
```

几个要点：

- **每个 artifact 需要 `template` 字段**：指向 `templates/` 目录下的模板文件。新增的 review 工件需要在 `openspec/schemas/with-review/templates/` 下创建 `review.md` 模板文件（可以是一个空文件或简单模板）
- `description` 字段保留 - 实际 schema.yaml 中确实有这个字段，它为每个 artifact 提供简短描述
- review 依赖 proposal + specs + design 三个工件，确保审查时所有前置工件都已就绪
- tasks 的 `requires` 从默认的 `[specs, design]` 改为 `[review]`，形成 `proposal → specs → design → review → tasks` 的依赖链
- 没有在 schema 中写死 `tracks` - 进度追踪由 `apply` 阶段负责

![自定义 Schema 架构对比图](https://developer.qcloudimg.com/http-save/10642399/b3b04ef1711379d59e080dca22eabd7f.png)

_图 2：默认 Schema vs 自定义 Schema——review 工件插入 design 和 tasks 之间_

#### Step 4：用新 Schema 创建变更

```
openspec new change todo-priority-v3 --schema with-review
```

实际输出：

```
- Creating change 'todo-priority-v3' with schema 'with-review'...
✔ Created change 'todo-priority-v3' at openspec/changes/todo-priority-v3/ (schema: with-review)
```

输出中的 `with schema 'with-review'` 确认使用了自定义 schema。

#### Step 5：Propose 并观察

这次 propose 会按依赖顺序生成五个工件：`proposal` → `specs` 和 `design`（两者可并行）→ `review` → `tasks`。实际输出：

```
**Change created: `todo-priority-v3`**

Location: `openspec/changes/todo-priority-v3/`
Schema: `with-review` (includes review artifact)

Artifacts created:
- **proposal.md** — same content as v2, defines priority enum with three values
- **design.md** — same content as v2, documents decisions and migration plan
- **specs/todo-priority/spec.md** — same content as v2, 4 requirements with scenarios
- **review.md** — quality review across 5 dimensions (boundary conditions, rollback, test coverage, backward compatibility, task granularity)
- **tasks.md** — 6 task groups with paired test tasks (20 total tasks, includes review recommendations)

All artifacts complete! Ready for implementation.
```

关键观察点：

1. **review.md 确实生成了**：输出中有 `Schema: with-review (includes review artifact)` 确认使用了自定义 schema
2. **review 的审查结论**：打开 review.md 看看五个维度的审查结果，完整内容如下：

```
# Quality Review

## 1. Boundary Conditions

**Status**: ✅ Pass

**Findings**:
- Specs cover null priority (rejected with 400)
- Specs cover empty string priority (rejected with 400)
- Specs cover invalid value "urgent" (rejected with 400)
- Specs cover non-existent todo PATCH (404)
- Scenarios for: valid priority creation, default priority on create, sorted listing, sorted vs default order

**Recommendations**:
None — all three error cases plus normal flow are covered.

## 2. Rollback Plan

**Status**: ✅ Pass

**Findings**:
- Design explicitly states "No rollback needed for priority field" because it's additive
- Migration is one-time and idempotent (UPDATE to 'medium' is safe to re-run)
- Migration plan includes rollback strategy for breaking changes
- Default 'medium' ensures graceful behavior even if migration partially runs

**Recommendations**:
None — rollback plan is complete for this change.

## 3. Test Coverage

**Status**: ⚠️ Warning

**Findings**:
- Specs identify test scenarios for each requirement
- Scenarios map to test cases (18 tests implemented in v1)
- Tasks.md requires paired test tasks (rule from config)

**Recommendations**:
- Ensure tasks.md includes explicit test tasks for each requirement
- Verify 18 tests cover all spec scenarios

## 4. Backward Compatibility

**Status**: ✅ Pass

**Findings**:
- Priority field is additive — existing GET responses unchanged
- POST/PATCH validation is new — only affects new writes
- Migration sets existing todos to 'medium' default
- No breaking changes to existing API contracts except timestamp format (already covered in separate change)

**Recommendations**:
None — migration addresses backward compatibility.

## 5. Task Granularity

**Status**: ✅ Pass

**Findings**:
- Requirements are clear and can be broken into small tasks
- Config rule: single task ≤ 30 minutes
- 15 tasks in todo-priority-v2, each specific and verifiable

**Recommendations**:
None — task breakdown is adequate.

---

## Overall Assessment

All five dimensions pass. The change is well-scoped with clear requirements and migration plan.

**Key recommendations for tasks.md**:
- Include explicit test tasks paired with each implementation task
- Verify sorting edge case (empty list returns empty array)
- Verify PATCH with only priority update (not touching other fields)
```

5 个维度中 **4 Pass + 1 Warning**，没有 Fail。说实话，自己审自己的审查结论整体偏宽松 - 毕竟是同一 AI 生成了 specs/design 又审查自己的产出。审查机制存在且能工作，但 AI 倾向于给自己的产出打高分。

1. **tasks.md 受了 review 影响**：从 Lab 2 的 19 个 task 增加到了 Lab 3 的 22 个 - 新增的 3 个 task 都有明确的 review 建议来源：

| | | | |
| --- | --- | --- | --- |
| 新增 4.5 排序空列表测试 | ❌ 无 | ✅ 4.5 Add unit tests for sorting empty list | review 建议 "Verify sorting edge case (empty list)" |
| 拆分 POST 测试 | 1 个（5.4） | 3 个（5.4 valid + 5.5 invalid + 5.6 400） | review 建议 "explicit test tasks" |
| 拆分 PATCH 测试 | 2 个（5.5 + 5.6） | 4 个（5.7 valid + 5.8 invalid + 5.9 404） | review 建议 "explicit test tasks" |

这是 review 工件"影响下游"的量化的证据。

#### Lab 3 产出对比

| | | | |
| --- | --- | --- | --- |
| 异常场景覆盖 | ✅ 碰巧覆盖 | ✅ 覆盖更完整 | ✅ Review 确认 |
| 回滚方案 | ⚠️ 不完整 | ✅ 有 Rollback | ✅ Review 确认 |
| 测试覆盖 | ✅ 碰巧配对 | ✅ 每组配对 | ✅ Review 建议拆分 |
| tasks 粒度 | 6 组 15 个 | 6 组 19 个 | 6 组 22 个 |
| **审查检查点** | ❌ 无 | ❌ 无 | ✅ design→review→tasks |
| **质量门控** | ❌ 无 | ⚠️ 仅结构检查 | ✅ Review 阻塞 tasks |
| **需求来源** | ❌ AI 推断 | ✅ 用户确认 | ✅ 用户确认 |

Lab 3 的核心优势：review 工件在 design 和 tasks 之间插了一个**结构化的检查点**。这不是依赖 AI 的自觉性（请记得检查边界条件），而是通过 schema 的依赖关系**强制执行** - 没有 review，tasks 就不会开始生成。

![三轮实验工作流程对比图](https://developer.qcloudimg.com/http-save/10642399/b794ec7d39711f02ab615e3cc6b6f941.png)

_图 3：三轮实验工作流对比——从裸跑到结构化门控的递进_

### 5 个升级方向的对标结论

三轮 Lab 跑完，回到上篇提出的 5 个升级方向，逐一对标。

#### 结论表

| | | | | |
| --- | --- | --- | --- | --- |
| 1 | Propose 后加 Spec Review | ✅ **有效** | `openspec validate` 做结构检查；Lab 3 的 review 工件做内容审查 | validate + review 工件组合 |
| 2 | Apply 拆原子任务 + 检查点 | ✅ **有效** | 自定义 Schema 的 `requires` 依赖链实现检查点；`instruction` 引导任务拆分 | 自定义 Schema + instruction |
| 3 | Verify 加入运行验证 | ⚠️ **部分有效** | `openspec validate` 只做文本/结构检查，不做运行验证 | 需要外部 CI + 测试框架 |
| 4 | 引入 TDD 思路 | ✅ **有效** | config.yaml 的 rules 强制测试任务配对 | Rules + review 工件双保险 |
| 5 | Archive 前设质量门控 | ✅ **有效** | 自定义 Schema 的依赖链 + review 工件充当门控 | review 作为 tasks 的前置依赖 |

![五维验证结论矩阵图](https://developer.qcloudimg.com/http-save/10642399/0a64121410ee8edab711bdf87f4cf360.png)

_图 4：5 个升级方向验证结论——4 个有效，1 个需要外部工具_

#### 逐条展开

**升级 1（Spec Review）**：Lab 2 的 `openspec validate` 能检查结构合规性，Lab 3 的 review 工件能做内容层面的五维审查。两者组合效果不错。成本也不高 - validate 是现成命令，review 工件只需要 fork 一个 schema。

**升级 2（原子任务 + 检查点）**：Lab 3 验证了在 schema 中用 `requires` 依赖链插入检查点是可行的。review 工件在 design 和 tasks 之间起到了**闸门**作用。但任务拆分的原子性仍然依赖 instruction 的描述质量 - OpenSpec 没有内置每个 task 必须 ≤ N 行代码这种硬约束。

**升级 3（运行验证）**：这是 5 个方向里**OpenSpec 原生能力覆盖不到的那个**。

说直白点：`openspec validate` 检查的是**tasks.md 有没有按 schema 格式写**，不是**代码跑起来会不会报错**。真正的运行验证 - 单元测试、集成测试、端到端测试 - 需要你在 tasks.md 里把测试写成强制性任务，然后通过 CI 管道执行。

OpenSpec 能做的是：确保测试任务被写进了 tasks.md（通过 rules + review），但**测试能不能跑通**这件事，OpenSpec 管不了。

**升级 4（TDD 思路）**：Lab 2 的 rules 强制**实现任务必须配对测试任务**，效果立竿见影。但更进一步的 TDD 红绿循环（先写失败测试 → 实现代码 → 测试通过 → 重构），OpenSpec 的 schema 里没有对应的阶段定义。如果你需要严格 TDD，得在自定义 schema 的 instruction 里明确写出**每个 task 必须是且仅是一个 TDD 阶段**。

**升级 5（质量门控）**：Lab 3 的 review 工件就是一个简易的质量门控。它通过 schema 的 `requires` 依赖确保 tasks 不会跳过审查直接生成。这比在 config.yaml 里写一条 rule 说记得检查靠谱得多 - 因为 rule 是文本约束，AI 可能忽略；但 `requires` 是结构约束，不满足就生成不了。

你在项目中用过类似的方案吗？是更喜欢 Lab 2 的轻量规则，还是 Lab 3 的结构性检查？欢迎在评论区聊聊。

### 三个 Lab 的选型建议

根据你的项目规模和团队情况，选择合适的方案。

| | | |
| --- | --- | --- |
| 个人项目、快速迭代 | Lab 2（Rules + Validate） | 配置简单，5 分钟搞定 |
| 团队项目、需要代码审查 | Lab 3（自定义 Schema） | 结构化门控，质量有保障 |
| 生产环境、高可靠性要求 | Lab 3 + 外部 CI | Schema 门控 + 运行验证，双保险 |
| 学习 OpenSpec、刚开始用 | Lab 1（裸跑） | 先熟悉流程，再逐步加质量措施 |

一个务实建议：别一上来就搞 Lab 3。先用 Lab 1 跑几轮熟悉流程，觉得哪里不够再加对应的措施。Rules 和 Validate 是性价比很高的起步选择。

### 诚实评估：OpenSpec 的能力边界

三轮 Lab 跑下来，我对 OpenSpec 质量工作流的判断是：

**能做到的**：通过 rules 约束 AI 行为、通过 validate 检查结构合规、通过自定义 Schema 插入结构化检查点。这三件事覆盖了 5 个升级方向中的 4 个。

**做不到的**：运行验证。代码能不能跑、测试过不过、性能达不达标，这些需要测试框架和 CI 管道。OpenSpec 定位在**文档对齐和流程编排**，不是运行时质量保障。

还有一个 Lab 3 暴露但容易被忽略的问题：**自己审自己的审查结论整体偏宽松**。review 工件由同一个 AI 生成，它审查的是自己写的 specs 和 design。4 Pass + 1 Warning 的结果看起来不错，但换成人类审查可能更严格。如果你的项目对质量有硬性要求，review 工件适合作为**初审**，不宜作为**终审**。

说到底，OpenSpec 的质量工作流需要配合外部工具一起用。它解决的是**AI 写出来的方案和你的意图是否对齐**，不是**AI 写出来的代码能不能跑**。

这两个问题不一样，解决方案也不一样。

建议收藏这篇，下次用 OpenSpec 做项目时翻出来对照着选方案。如果你的同事也在用 OpenSpec，转发给他看看这三个 Lab 的对比。

**好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！**

来源：https://cloud.tencent.com/developer/article/2667507
