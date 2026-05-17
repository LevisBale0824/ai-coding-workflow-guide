# openspec 最佳实战：不改 OpenSpec 源码，只改一段配置，代码质量从不可控到 80 分

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _108_ 篇，AI 编程最佳实战「2026」系列第 _33_
>
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
>
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的**技术实践者与开源布道者**！
>
> **Talk is cheap, let's explore。无界探索，有术而行。**

![封面图 - 信息图风格：任务粒度 = 代码质量杠杆](https://developer.qcloudimg.com/http-save/10642399/27b9e4ff9f7235d8fa4704bf63e72601.png)

_图 1：任务粒度——代码质量最大的杠杆_

> **说明**：本文内容基于 OpenSpec（Fission-AI/OpenSpec）v1.3.1 官方文档、笔者前两篇文章（《OpenSpec 最佳实战：4 步复盘 + 5 项升级》、《3 轮实测验证 5 个质量升级方向》）的实测数据，以及 Superpowers writing-plans skill 的公开设计文档分析整理而成。**文中的配置模板和参数建议仅供参考，实际效果请以你的业务数据和环境测试结果为准。**如果有实际使用经验，欢迎在评论区分享交流。

上篇文章用 3 轮 Lab 验证了 5 个质量升级方向。结论表贴出来，答案一目了然：**5 个方向都有效，但性价比天差地别。**

有些改动需要 fork schema、插入新工件、调依赖链，折腾一圈下来质量确实好了。但有个方向让我意外——**只改一段配置文本，不碰 OpenSpec 源码，不装外部工具，效果却覆盖了 80% 的质量问题**。

这个方向就是：**任务粒度控制**。

今天这篇是「OpenSpec 项目实战」系列第 1 期，定位方法论篇。先把为什么任务粒度是那个 20% 讲清楚，再说三步配置怎么做，最后画一张完整的日常工作流全景图。从第 2 期开始，会拿一个真实项目「shuge AI Toolbox」，用这套方法论从零搭建，逐期展开。

### 1. 问题根源：任务太粗，AI 就会自由发挥

3 轮 Lab 跑下来，有一个发现反复出现：**AI 在 apply 阶段夹带私货**。

Lab 1 裸跑的时候，AI 从 `todo-priority` 这个 change name 推断了全部需求。结果 tasks.md 里出现了一个没人提过的需求——ISO 8601 时间戳格式变更。这是个 breaking change，但 propose 阶段根本没人要求做这件事。

AI 做了什么？它**自行解释了需求**，然后**自行决定了实现范围**。

这个问题的根因不在 AI 的能力，而在我们给它的任务描述太粗了。看一个对比就明白。

粗粒度任务长这样：

```
- [ ] 1.1 实现用户注册接口
- [ ] 1.2 添加输入校验
- [ ] 1.3 处理异常情况
```

三个 task，每个都是一句话描述。AI 拿到这种 task 会做什么？它会**猜**——注册接口要不要发邮件？校验规则是什么？异常情况具体指哪些？猜完了就自己写，写完了你一看，不对。

细粒度任务长这样：

```
### 任务 1：邮箱格式校验

- [ ] 第 1 步：写失败测试
```typescript
test('无效邮箱格式返回 400', () => {
  const result = register({ email: 'abc' });
  expect(result.status).toBe(400);
});
```

- [ ] 第 2 步：运行测试——确认失败
  命令：`npx vitest run tests/auth.test.ts`
  预期：FAIL — function is not defined

- [ ] 第 3 步：写最小实现
  （完整代码）

- [ ] 第 4 步：运行测试——确认通过
  命令：`npx vitest run tests/auth.test.ts`
  预期：PASS
```

每个 step 附了完整代码、运行命令、预期输出。AI 拿到这种 task 还能发挥什么？**照着做就行了**。

![对比图：粗粒度任务 vs 细粒度任务的因果链](https://developer.qcloudimg.com/http-save/10642399/ff3fc9cfe16fc1049a5a9a9161dad814.png)

_图 2：任务粒度对比——粗任务给 AI 留了发挥空间，细任务把空间压缩到零_

因果链很清楚：

```
任务粗 → AI 自行解释 → 自行决定范围 → 夹带私货
任务细 + 附代码 → AI 只需执行 → 没有解释空间 → 夹不了私货
```

说到底，这不是 AI 能力的问题，是**指令质量**的问题。你给的指令有模糊空间，AI 就会填满这个空间。你把模糊空间压缩到零，AI 就只能照单执行。

### 2. 2/8 法则：任务粒度才是那个 20%

上篇文章验证了 5 个升级方向。如果从性价比角度重新排序，画面会完全不一样。

| | | | |
| --- | --- | --- | --- |
| **升级 tasks 的 instruction** | 改一段配置文本 | AI 生成细粒度 tasks → 源头消灭问题 | **极高** |
| 加代码审查 | 独立 subagent 或人工审查 | 发现已产生的问题 | 中 |
| 加归档前验证 | 开启 expanded workflow | 补抓遗漏 | 中 |
| 写更细的 rules | 调试 config.yaml rules | AI 遵循度不稳定 | 低 |
| 需求澄清 | Explore 多轮对话 | 效果好但耗时 | 中高 |

3 轮实测的结论支撑这个排序：

**Lab 2 加了 Rules + Explore**，产出质量主要来自 Explore 的需求澄清，rules 本身的效果很难单独量化。为什么？因为 rules 是文本约束，AI 有时候遵循有时候不遵循，你没法保证每次都生效。

**Lab 3 自定义 Schema 插入 Review 工件**，tasks.md 从 19 个增加到 22 个，新增的 3 个 task 都有明确的 review 建议来源。但整条链路需要 fork schema、写 review 模板、调整依赖关系，配置成本不低。

而升级 tasks 的 instruction 呢？只需要**改一个字段里的一段文字**。不改 OpenSpec 源码，不装外部工具，不调整 DAG 依赖链。效果是让 AI 在生成 tasks.md 的阶段就把任务拆到 2-5 分钟粒度，每个 step 附完整代码。

**20% 的改动，覆盖 80% 的质量提升。** 这就是 2/8 法则。

剩下的 80% 改动（审查 + 验证 + CI）能带来多少额外提升？大概 20%。它们是安全网，不是主力。

那这 20% 的改动具体怎么做？

### 3. 三步配置：从不可控到 80 分

三步配置的思路很简单：**给 AI 足够精确的上下文 → 给 AI 足够严格的执行指令 → 插一道自动检查**。每步只改一个文件。

#### 第 1 步：创建 config.yaml（上下文 + 规则）

这个文件给 AI 提供全局上下文和按工件分组的规则。放在项目根目录的 `openspec/config.yaml`：

```yaml
schema: with-review

context: |
  技术栈：TypeScript, Express, Vitest
  测试命令：npx vitest run
  所有新功能必须遵循 TDD 节奏——先写失败测试，再写实现代码

rules:
  specs:
    - 每个数据字段的变更，必须覆盖 null、空值、越界三种异常场景
    - Scenario 必须使用 #### 四级标题，否则归档时不生效
  design:
    - 涉及数据库 migration 的设计，必须包含回滚方案
  tasks:
    - 每个 task 必须包含完整的测试代码和实现代码
    - 每组 task 的第一步必须是写失败测试，最后一步必须是验证通过
  review:
    - 重点检查 tasks.md 的粒度是否达到 2-5 分钟一个 step
    - 检查是否有占位符（TBD、TODO、implement later）
```

几个要点：

- **`context` 字段注入所有工件**，告诉 AI 你的技术栈、测试命令、编码规范
- **`rules` 按 artifact ID 分组**，key 必须和 schema 里的 artifact id 完全一致
- rules 在 prompt 中以**约束条件**的形式传递，相当于给 AI 画红线

> 完整配置文件见附件：`articles/attachments/config.yaml`

#### 第 2 步：Fork Schema，升级 tasks 的 instruction

这是**核心改动**。

先 fork 一个现有的 schema：

```
openspec schema fork spec-driven with-review
```

> 注意：`openspec schema fork` 目前标记为 experimental，后续版本可能有变化。

然后在 `openspec/schemas/with-review/schema.yaml` 中，找到 `id: tasks` 的 artifact，把它的 `instruction` 字段替换成下面这段：

```yaml
instruction: |
  创建细粒度的实现计划。每个任务的工作量应在 2-5 分钟之间。

  每个任务必须遵循以下格式：
  - 涉及文件（精确路径）
  - 第 1 步：写失败测试（附完整测试代码）
  - 第 2 步：运行测试——确认失败（附命令 + 预期输出）
  - 第 3 步：写最小实现（附完整实现代码）
  - 第 4 步：运行测试——确认通过（附命令 + 预期输出）
  - 第 5 步：提交（附 git 命令）

  禁止事项（出现即视为计划不合格）：
  - TBD、TODO、implement later
  - 「添加适当的错误处理」（必须写出具体代码）
  - 「为以上代码写测试」（必须写出具体测试代码）
  - 只描述做什么但不展示怎么做
```

这段 instruction 做了两件事：

**第一，给 AI 一个严格的任务模板。** 每个 task 必须包含 5 个 step，从写失败测试到提交，每步附完整代码和命令。AI 没有发挥空间。

**第二，列了一堆禁止事项。** 这些都是 AI 常见的偷懒行为——写个 TBD 让你自己填，说一句「添加适当的错误处理」就算完成了。现在出现这些就直接判定计划不合格。

> 完整 instruction 内容见附件：`articles/attachments/tasks-instruction.yaml`

#### 第 3 步：插入 review 工件（配套措施）

在前两步的基础上，给 schema 加一个 `review` 工件，放在 design 和 tasks 之间：

```yaml
# schema.yaml 中的新增工件
- id: review
  generates: review.md
  template: review.md
  description: 五维审查 - 检查设计方案和 Spec 的完整性
  instruction: |
    从五个维度审查所有工件的完整性和质量。
    审查维度：
    1. 边界条件  2. 回滚方案  3. 测试覆盖
    4. 向后兼容  5. 任务粒度（最重要）
  requires: [proposal, specs, design]
```

然后把 tasks 的 `requires` 从 `[specs, design]` 改为 `[review]`。这样依赖链变成：

```
proposal → specs → design → review → tasks → APPLY
```

review 工件是个闸门：没有 review，tasks 就不会开始生成。审查结论里会明确标注每个维度的状态（✅ 通过 / ⚠️ 警告 / ❌ 失败），tasks.md 会参考 review 的建议来决定拆分方向。

坦率说，这一步的效果比前两步温和。3 轮实测发现，**自己审自己的审查结论偏宽松**——4 Pass + 1 Warning 看着不错，但换成人类审查可能更严格。不过作为一道结构化的安全网，它还是有价值的。

> 完整 review instruction 见附件：`articles/attachments/review-instruction.yaml`
> review 模板文件见附件：`articles/attachments/review-template.md`

![三步配置流程图：config.yaml → schema fork + instruction → review 工件](https://developer.qcloudimg.com/http-save/10642399/98c9f519d88c6229254907b16145a5f9.png)

_图 3：三步配置——从上下文注入到指令升级再到自动审查_

三步做完，日常开发的工作流长什么样？

### 4. 完整工作流：6 个 Phase 的日常开发

配置是一次性的，日常开发就是反复跑 6 个 Phase。每个 Phase 对应一条 OpenSpec 命令，AI 做完一步你确认一步。

#### Phase 1：需求澄清

可选但推荐。在 propose 之前，先把模糊地带想清楚：字段类型怎么选、边界条件有哪些、迁移策略是什么。Explore 会画出决策矩阵帮你理清思路。

3 轮实测的结论是：**Explore 阶段的需求澄清，是产出质量提升的主要来源**。Lab 2 的 rules 本身效果不太容易量化，但 Explore + propose 的组合比单独 propose 可靠得多——需求是你说的，不是 AI 猜的。

#### Phase 2：一键生成工件

propose 内部会自动创建 change 目录，然后按依赖顺序生成全部工件。用 `with-review` schema 的话，一次产出 5 个文件：proposal.md、specs/、design.md、review.md、tasks.md。

关键是 tasks.md——因为我们升级了 instruction，这里的每个 task 都是 2-5 分钟粒度、附完整代码和命令的。

#### Phase 3：人工检查（1-2 分钟）

这一步只需要扫一眼 review.md。重点看两个地方：

- **任务粒度**那个维度的状态：是 ✅ 还是 ⚠️？
- **整体评估**里对 tasks.md 的关键建议

如果 review 说 tasks.md 粒度不够或者有占位符，手动改一下 tasks.md 再往下走。OpenSpec 的工件都是 Markdown 文件，直接编辑就行。

#### Phase 4：按任务执行

AI 读取所有工件后，按 tasks.md 里的顺序逐个实现。因为每个 step 都有完整代码和命令，AI 只需要照着做——**没有发挥空间**。

#### Phase 5：一致性验证（安全网）

> 注意：verify 需要 expanded workflow 才能使用。

verify 检查实现是否匹配 spec 意图。本质是文本层面的一致性检查，不是运行验证。代码能不能跑，还得靠测试框架和 CI。但作为一道安全网，它能发现「spec 里写了要做 A，代码里做的是 B」这类问题。

你在项目中用过类似的 6 步工作流吗？有没有哪个 Phase 你觉得可以省掉？欢迎在评论区聊聊。

#### Phase 6：归档

将整个 change 目录归档。归档前建议扫一眼所有 task 的完成状态，确认没有遗漏。

### 5. 三层防线：不是所有防线都一样重要

三步配置搭好之后，日常开发中其实有三层防线在工作。但它们的重要性不在一个量级。

![三层防线示意图：源头控制（80%）→ 过程检查（15%）→ 收尾确认（5%）](https://developer.qcloudimg.com/http-save/10642399/1e2e7955a7877cdb404055ebe1e390a4.png)

_图 4：三层防线——80% 的质量来自源头控制_

**第一层：源头控制（tasks instruction 升级）→ 贡献 80% 的质量**

这是整篇文章的核心。通过升级 instruction，让 AI 在生成 tasks.md 的时候就把每个 step 拆到 2-5 分钟、附上完整代码。问题在源头就被消灭了，后面的防线只需要处理遗漏。

**第二层：过程检查（review + verify）→ 安全网**

review 工件在 design 和 tasks 之间插入一道结构化审查。verify 在 apply 后做一致性检查。这两道防线处理的是第一层漏掉的问题——边界条件遗漏、回滚方案缺失、spec 和代码不一致。

**第三层：收尾确认（archive 前的人工检查）→ 兜底**

扫一眼 review.md 的结论和 tasks.md 的完成状态，1-2 分钟的事。这是人的防线，处理前两层没覆盖到的边界情况。

三层防线的设计哲学：**第一层做重，第二层做轻，第三层做快。** 不要试图在每一层都做 100% 的检查——那是在每个环节都投入 80% 的精力，换来 20% 的边际提升。

### 6. 实战注意事项

坑点和技巧都是 3 轮 Lab 里踩过的，每条一句话。

**坑点：**

- **rules 的 key 必须和 artifact id 完全一致**。写成 `task` 而不是 `tasks`，规则就静默失效了，不会报错。
- **`openspec schema fork` 目前是 experimental 功能**，后续版本命令格式可能变化，升级时留意 changelog。
- **verify 不做运行验证**，只做文本一致性检查。代码能不能跑，得靠测试框架。
- **review 是自己审自己**，结论偏宽松。对质量有硬性要求的项目，review 适合初审，不宜终审。
- **context 窗口压力**。apply 阶段 AI 需要同时持有所有工件信息，复杂变更时建议先 `/clear` 再 apply。

**技巧：**

- 复杂变更（5 个文件以上）拆成多个独立 change，每个只做一件事。上下文压力小，AI 不容易遗忘。
- propose 不一定会问需求——change name 足够清晰时 AI 可能跳过确认直接生成。想控制需求就先跑一遍 Explore。
- 工件都是 Markdown 文件，随时可以手动改。propose 生成的 tasks 粒度不够？直接编辑 tasks.md，不用重跑 propose。
- instruction 里多写「禁止事项」比多写「建议事项」更有效。AI 对否定约束的遵循度高于模糊的正面指导。

### 下一期预告

这篇方法论讲完了为什么任务粒度是那个 20%，三步配置怎么做，日常工作流长什么样。

第 2 期开始实战。我会用这套方法论，从零开始搭建一个真实项目——**shuge AI Toolbox**。第 2 期的内容：

- 用 Explore 澄清项目需求，确定 MVP 范围
- 执行 propose 生成 5 个工件，逐个检查产出质量
- 重点看 tasks.md 的粒度是不是真的达到了 2-5 分钟一个 step

如果你也想跟着做，建议先把三步配置跑通：创建 config.yaml、fork schema、升级 instruction。具体配置文件下一期会放在公开的代码仓库作为附件，读者可以直接拿来改。

系列持续更新中，关注不迷路。

**好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！**

来源：https://cloud.tencent.com/developer/article/2667508
