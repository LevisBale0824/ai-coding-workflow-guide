# Superpowers 最佳实战：标准开发 7 步法，从需求到交付工程级代码的闭环工作流深度指南

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _81_ 篇，AI 编程最佳实战「2026」系列第 _15_ 篇
>
> 大家好，欢迎来到 __术哥无界 | ShugeX ｜ 运维有术__。
>
> 我是__术哥__，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的__技术实践者与开源布道者__！
>
> __Talk is cheap, let's explore。无界探索，有术而行。__

![Image 1: Superpowers 全景信息图](images/d195c18c0de492370923f66c14a72a2f.png)

Superpowers 全景信息图

_图 1：Superpowers 7 阶段闭环工作流全景_

进阶导读：如果你已经知道 Superpowers 是什么，但不确定它在真实项目中如何端到端运行 — 这篇文章就是为你准备的。本文以一个__优惠券核销 API__ 为实战主线，完整走一遍从需求到交付的 7 个阶段。假设你有 AI 编程工具的使用经验，基础概念不再解释。

### 一、为什么你需要工程纪律而不是 AI 聊天

大多数开发者用 AI 编程工具的方式，本质上和让实习生直接上手写代码没有区别：给一个需求描述，拿到一段代码，检查一下能用就合并。

问题在哪？__没有设计，没有测试，没有审查。__ 你拿到的是一段"能跑"的代码，不是"靠谱"的代码。

Superpowers 的核心哲学用源码里的一句话概括：

> __"AI 不是用来写代码的，而是用来执行严格工程标准的。"__

原始表述更犀利：__"Skills are not prose — they are code that shapes agent behavior"__ — Skills 不是散文式的建议，而是代码级的精确指令，塑造 Agent 的行为。

这不是语义上的区别。Superpowers 的 14 个 Skills 是__强制执行__的。Agent 启动时，一个 session-start-hook 直接注入：

```
<EXTREMELY_IMPORTANT>
You have Superpowers.
RIGHT NOW, go read: @skills/getting-started/SKILL.md
</EXTREMELY_IMPORTANT>
```

Agent 学到的第一条规矩：__如果你有 skill 来做某事，你必须使用它来做那项活动。__ 不是建议，不是可选，是必须。

这组数据很能说明问题：GitHub 147,000+ Stars，发布半年就从 0 增长到热榜头部。Superpowers 不是在解决__AI 能不能写代码__的问题（AI 已经能写了），而是在解决__AI 写的代码能不能信__的问题。

### 二、14 个 Skills 构成的闭环工作流

![Image 2: Superpowers 三层架构图](images/f3a709a9fe478145816c5ea8ea1cd2e3.png)

Superpowers 三层架构图

_图 2：Superpowers 三层架构 — Agent 集成层 / 工作流层 / Skills 层_

Superpowers 的架构分三层：

|  |  |  |
| --- | --- | --- |
| Agent 集成层 | 对接编码工具 | Claude Code、Cursor、Codex、OpenCode、Gemini CLI、Copilot CLI |
| 工作流层 | 强制流程控制 | brainstorming → worktrees → plans → subagent → TDD → review → finish |
| Skills 层 | 行为定义 | 14 个独立 SKILL.md 文件，可发现、可调用、可组合 |

14 个 Skills 按职责分成四类：

|  |  |  |
| --- | --- | --- |
| __测试__ | test-driven-development, verification-before-completion | 铁律级 TDD + 完成验证门禁 |
| __调试__ | systematic-debugging | 4 阶段根因分析 |
| __协作__ | brainstorming, writing-plans, executing-plans, subagent-driven-development, dispatching-parallel-agents, requesting-code-review, receiving-code-review, using-git-worktrees, finishing-a-development-branch | 需求到交付全链路 |
| __元技能__ | writing-skills, using-superpowers | 自定义 Skills + 入门指南 |

完整闭环严格线性执行：

```
brainstorming → using-git-worktrees → writing-plans →
  subagent-driven-development →
    test-driven-development (每个任务) →
      requesting-code-review (每个任务) →
        finishing-a-development-branch
```

每个阶段的触发是自动的。当你试图跳过某个阶段，Agent 会拒绝并引导你回到正确的流程。

下面用一个真实的电商场景走完全部 7 个阶段。

### 三、实战：优惠券核销 API — 从需求到交付

场景描述：实现一个优惠券核销接口，需要处理三件事： __核销状态转换、过期/已用检查、并发安全。__

#### 3.1 Brainstorming — 让 AI 替你问出你没想过的问题

brainstorming skill 有一个 __HARD-GATE 规则__：

> "Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it."

设计没批准之前，一行代码都不准写。

Agent 的提问是苏格拉底式的：__一次一个问题，不贪多。__面对"优惠券核销"这个需求，真实对话是这样的：

```
Agent: "优惠券有几种类型？满减、折扣、还是固定金额？"

你: "满减和折扣两种。"

Agent: "一张券能用几次？一次性还是可多次使用？"

你: "一次性。"

Agent: "并发场景怎么处理？两张订单同时用一张券，期望行为是什么？"

你: "只有一个能成功。"

Agent: "核销失败的券，状态应该恢复为可用，还是标记为异常？"
```

注意第 3 个问题。很多人在提需求时根本不会想到并发，但 brainstorming skill 会__强制__把它捞出来。

Agent 还会特别警惕一个反模式：

> "This Is Too Simple To Need A Design" — 每个项目都需要设计。'Simple' 项目是未审查假设导致浪费工作最多的地方。

问完问题后，Agent 提出 2-3 种方案，附带权衡分析：

|  |  |  |  |
| --- | --- | --- | --- |
| Redis 分布式锁 | 强一致 | 性能好 | 引入新依赖 |
| 数据库乐观锁 | 最终一致 | 实现简单 | 高并发下重试多 |
| 数据库悲观锁 | 强一致 | 稳妥 | 锁粒度粗，性能差 |

你选定方案后，Agent 将设计写入文档，执行规格自查（检查占位符、矛盾、歧义），然后请你审批。审批通过，进入下一步。

#### 3.2 Git Worktrees — 隔离你的实验场

__为什么不能在主分支上开发？__

writing-plans skill 里有句话很直白：假设执行者是 "an enthusiastic junior engineer with poor taste, no judgement, no project context" — 一个热情但没有判断力、没有项目上下文的初级工程师。这种人直接在 main 上改代码，回滚成本极高。

using-git-worktrees skill 的执行流程：

1. __选择目录__：按优先级 `.worktrees/` → `worktrees/` → CLAUDE.md 配置 → 询问
2. __安全验证__：确认目录在 `.gitignore` 中（避免污染主仓库）
3. __创建 worktree__：隔离的文件系统，共享同一个 Git 仓库
4. __项目初始化__：自动检测项目类型，执行依赖安装
5. __基线验证__：运行测试，确认从干净状态开始

```
# Agent 自动执行的命令序列
git worktree add .worktrees/coupon-redeem feature/coupon-redeem
cd .worktrees/coupon-redeem
npm install
npm test
# 确认所有测试通过，基线干净
```

关键点：worktree 和主工作区文件系统完全隔离，但共享同一个 Git 仓库。你可以在主分支继续其他工作，两边互不干扰。

#### 3.3 Writing Plans — 把大象装进冰箱需要几步

writing-plans skill 的核心假设：

> "Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know."

任务粒度是 __2-5 分钟一个 action__。优惠券核销 API 的计划拆解如下：

```
Task 1: 定义 CouponRedeemRequest DTO
  Step 1: 写测试 — 验证 DTO 字段校验规则
  Step 2: 运行测试，确认失败（RED）
  Step 3: 实现 DTO — couponId、orderId、userId
  Step 4: 运行测试，确认通过（GREEN）
  Step 5: 提交

Task 2: 核销状态校验
  Step 1: 写测试 — 过期券拒绝核销
  Step 2: 运行测试，确认失败（RED）
  Step 3: 实现 validateCoupon() 方法
  Step 4: 运行测试，确认通过（GREEN）
  Step 5: 写测试 — 已使用的券拒绝核销
  Step 6: 运行测试，确认失败（RED）
  Step 7: 扩展 validateCoupon() 处理已用状态
  Step 8: 运行测试，确认通过（GREEN）
  Step 9: 提交

Task 3: 并发控制 — 乐观锁实现
  Step 1: 写测试 — 并发核销同一张券只有一个成功
  Step 2: 运行测试，确认失败（RED）
  Step 3: Coupon 表增加 version 字段
  Step 4: 实现乐观锁扣减逻辑
  Step 5: 运行测试，确认通过（GREEN）
  Step 6: 提交
```

三个必须遵守的规矩：

- __没有 TBD__：计划里不能出现 "implement later"、"add appropriate error handling" 这种废话。每个步骤必须给出具体的代码逻辑。
- __禁止引用__：不能写 "Similar to Task 1"。因为子代理可能不按顺序读取任务，必须重复写出完整代码。
- __自查三要素__：规格覆盖（每个需求有对应任务？）、占位符扫描（有没有含糊描述？）、类型一致性（跨任务的方法签名是否一致？）。

#### 3.4 编码实现 — 子代理按计划逐任务执行

subagent-driven-development 是 Superpowers 的核心执行引擎。

原理：__每个任务派遣一个全新的子代理，子代理不继承主会话的任何上下文。__ 主 Agent 把精力放在协调工作上，子代理只拿到当前任务的完整描述和相关代码。

核心公式：

> "Fresh subagent per task + two-stage review = high quality, fast iteration"

这不是一个子代理干完所有活。每个任务都是一个新的 subagent，从零开始，没有历史包袱。源码里说得很清楚：

> "You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task."

在优惠券核销场景中，执行过程如下：

```
主 Agent: "读取计划文件，提取全部 3 个 Task。创建 TodoWrite 跟踪。"

Task 1: 定义 CouponRedeemRequest DTO
  → 派遣 implementer subagent（快速/廉价模型即可）
  → subagent 按计划执行 Step 1-5
  → subagent 报告状态: DONE

Task 2: 核销状态校验
  → 派遣 implementer subagent
  → subagent 按计划执行 Step 1-9
  → subagent 报告状态: DONE

Task 3: 并发控制 — 乐观锁实现
  → 派遣 implementer subagent
  → subagent 报告状态: DONE_WITH_CONCERNS
    "完成了，但发现 Coupon 表缺少 version 字段的索引，高并发下可能性能较差"
  → 主 Agent 评估：关注点合理，记录，继续
```

每个 subagent 在执行过程中，TDD 铁律被强制激活 — 这就是下一步要讲的。

模型选择有讲究：

|  |  |  |
| --- | --- | --- |
| 机械实现（定义 DTO、写校验） | 快速/廉价模型 | 1-2 个文件，规格完整 |
| 集成判断（多文件协调） | 标准模型 | 跨模块依赖 |
| 架构/审查 | 最强模型 | 需要设计判断 |

一个关键原则：__不要用旗舰级模型去定义 DTO。__ 源码里明确说了，"Use the least powerful model that can handle each role to conserve cost and increase speed."

#### 3.5 TDD 闭环 — 没有测试就不准写代码

![Image 3: TDD 循环流程图](images/d0aa5cc03555fd7577354e11bd7ffa41.png)

TDD 循环流程图

_图 3：RED-GREEN-REFACTOR 循环 — TDD 铁律可视化_

test-driven-development skill 在每个任务的执行过程中自动激活。它的 __Iron Law（铁律）__：

> "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"
>
> "If you didn't watch the test fail, you don't know if it tests the right thing."

铁律的执行非常暴力。如果 Agent 发现你在写测试之前就写了生产代码：

> "Delete it. Start over. Don't keep it as 'reference'. Don't 'adapt' it while writing tests. Don't look at it. Delete means delete."

以 Task 3"并发核销"为例，subagent 的执行过程严格遵循 RED-GREEN-REFACTOR：

__RED — 写失败测试：__

```
// Task 3 Step 1: 写测试 — 并发核销同一张券只有一个成功
describe('CouponService.redeem - 并发安全', () => {
  it('同一张券并发核销，只有一个成功', async () => {
    const coupon = await Coupon.create({
      id: 'cpn_001',
      type: 'FIXED_DISCOUNT',
      value: 10,
      status: 'ACTIVE',
      version: 0,
    });

    const [result1, result2] = await Promise.allSettled([
      CouponService.redeem({ couponId: 'cpn_001', orderId: 'ord_001' }),
      CouponService.redeem({ couponId: 'cpn_001', orderId: 'ord_002' }),
    ]);

    const successCount = [result1, result2].filter(
      r => r.status === 'fulfilled'
    ).length;
    expect(successCount).toBe(1);
  });
});
```

运行测试。断言失败 — 不是编译错误，是断言失败。RED 阶段确认。

__GREEN — 写最少代码让测试通过：__

```
// Task 3 Step 4: 乐观锁扣减逻辑
async redeem(req: CouponRedeemRequest): Promise<RedeemResult> {
  // 原子更新：只更新 status=ACTIVE 且 version 匹配的记录
  const affected = await db.query(
    `UPDATE coupons
     SET status = 'USED', version = version + 1, used_at = NOW()
     WHERE id = ? AND status = 'ACTIVE' AND version = ?`,
    [req.couponId, currentVersion]
  );

  if (affected === 0) {
    throw new CouponAlreadyRedeemedError(req.couponId);
  }

  return { success: true, couponId: req.couponId };
}
```

运行测试，通过。GREEN 阶段完成。

__REFACTOR — 在测试保护下重构：__ 消除重复、改善命名、提取辅助函数。但只能改结构，不能加行为。

三个阶段的顺序不能乱，每个阶段的验证不能跳。源码里的"借口反驳表"更是一针见血：

|  |  |
| --- | --- |
| "太简单不需要测试" | 简单代码也会出错，测试只需 30 秒 |
| "我之后写测试" | 之后通过的测试什么也证明不了 |
| "删掉 X 小时的工作太浪费" | 沉没成本谬误，保留不可信的代码才是技术债 |
| "TDD 太教条了" | TDD 才是实用主义：提交前发现 bug 比之后调试快 |

> "Thinking 'skip TDD just this once'? Stop. That's rationalization."

#### 3.6 代码审查 — 每个任务完成后强制审查

requesting-code-review skill 的原则只有六个字：

> "Review early, review often."

源码明确规定了三个__强制审查时机__：

- 每个任务在 subagent-driven-development 中完成后
- 完成重大功能后
- 合并到 main 分支前

审查由 code-reviewer subagent 独立执行，它只拿到变更的 diff 和规格文档 — __不看你的思考过程，只看你的工作成果。__

在优惠券核销场景中，Task 3 完成后的审查流程：

```
# 主 Agent 获取 git 变更范围
BASE_SHA=$(git log --oneline | grep "Task 2" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

# 派遣 code-reviewer subagent
# 传入: WHAT_WAS_IMPLEMENTED / PLAN_OR_REQUIREMENTS / BASE_SHA / HEAD_SHA
```

code-reviewer 返回分级结果：

```
Review Result:

Critical (必须立即修复):
  - 并发失败时返回 HTTP 500，规格要求 HTTP 409
    → 修复状态码映射，新增 CONFLICT 处理

Important (继续前修复):
  - 魔法数字：affected === 0 的 0 应提取为常量
  - 缺少核销失败的日志记录

Minor (稍后处理):
  - redeem 方法可以提取为独立的 CouponRedeemService

Assessment: 修复 Critical 和 Important 后可继续
```

审查的核心价值：__Critical 问题阻塞进度，Important 问题在继续前必须修复，Minor 问题记下稍后处理。__

如果审查发现问题，implementer subagent 修复后，重新派遣 code-reviewer 审查 — 直到通过。源码的 Red Flags 写得很明白：

- 不能因为"看起来简单"就跳过审查
- 不能忽略 Critical 问题
- 不能带着未修复的 Important 问题继续推进

#### 3.7 分支完成 — 验证、合并、交付

finishing-a-development-branch 是闭环的最后一步。

所有任务执行完毕，所有审查通过后，Agent 执行最终验证：

```
# Agent 自动执行的验证序列
npm test            # 全量测试
npm run lint        # 代码风格检查
npm run typecheck   # 类型检查
# 读取输出，确认 0 failures, 0 errors
```

verification-before-completion skill 有一句刻薄但精准的话：

> "Claiming work is complete without verification is dishonesty, not efficiency."

__"没验证就宣称工作完成了，这不是高效，这是不诚实。"__

它定义了严格的 Gate Function：

```
IDENTIFY → 找到所有需要验证的东西
RUN      → 执行验证命令
READ     → 读取完整输出
VERIFY   → 确认结果符合预期
→ 只有通过以上四步，才能宣称"完成"
```

> "Skip any step = lying, not verifying."

验证全部通过后，Agent 呈现四个选项：

|  |  |
| --- | --- |
| __Merge__ | 直接合并到主分支 |
| __PR__ | 创建 Pull Request，走团队审查流程 |
| __Keep__ | 保留分支，暂不合并 |
| __Discard__ | 丢弃变更 |

你做出选择后，Agent 自动清理 worktree，回到主工作区。整个 7 步闭环完成。

### 四、效果对比：有纪律 vs 无纪律

![Image 4: 有纪律 vs 无纪律对比图](images/90981087678e7ccf44b2cbf6b1721337.png)

有纪律 vs 无纪律对比图

_图 4：7 维度对比 — 有 Superpowers 工程纪律 vs 无纪律开发_

社区里有个真实的反馈（来源：CSDN 用户评价）：

> "一个跨 15 个文件的功能，Claude Code 写到一半开始乱改之前的代码，最后不得不手动回滚 git 重来。用了 Superpowers 框架，同样的任务，Claude Code 自主跑了两个小时没偏离计划。"

两种模式的对比：

|  |  |  |
| --- | --- | --- |
| 设计阶段 | 跳过，直接写代码 | 强制 brainstorming，逐个提问 |
| 开发环境 | 主分支直接改 | worktree 隔离 |
| 任务粒度 | "实现优惠券功能" | 2-5 分钟原子 action |
| 测试策略 | 写完补测试 | RED-GREEN-REFACTOR 铁律 |
| 质量保障 | 人工检查 | 双阶段自动审查 + 验证门禁 |
| 代码审查 | 可选 | 每个任务完成后强制 |
| 并发安全 | 凭感觉写 | 先写并发失败测试，再写修复代码 |

核心差异不在快慢，在于__可控性__。Superpowers 把 AI 的输出从"不确定"变成了"可预期"。

社区里还有一句评价很到位：__"Superpowers 值得研究的地方不是'更强'，而是'更稳'。"__

### 五、最佳实践与常见陷阱

#### 实战技巧

1. __不要跳过 brainstorming__。即使需求看起来很简单。源码原文："'Simple' 项目是未审查假设导致浪费工作最多的地方。"
2. __计划要写给零上下文的人看__。子代理不继承主会话上下文。计划里写了 "TBD"，子代理就真的会 TBD。
3. __YAGNI 要狠__。brainstorming skill 原则：__"YAGNI ruthlessly — Remove unnecessary features from all designs"__。GREEN 阶段只写最少代码让测试通过，不要"顺便"加功能。
4. __用对模型__。机械任务用便宜模型，审查用强模型。不要用旗舰级模型去定义 DTO。

#### 常见陷阱

|  |  |  |
| --- | --- | --- |
| 跳过 brainstorming 直接写代码 | 需求理解偏差，返工 | HARD-GATE 拦截，必须先设计 |
| 计划中出现 "TODO" | 子代理产出不可预期的代码 | 每个步骤给出具体代码 |
| 先写代码再补测试 | 测试只是"恰好通过"的装饰 | 删除代码，从测试开始 |
| 跳过 spec review 直接看代码质量 | 功能遗漏或多余 | 两阶段审查缺一不可 |
| 并行派遣多个 implementer | 代码冲突 | 串行执行，一次一个任务 |

### 总结

Superpowers 的核心洞察：__AI 编程的问题不是 AI 不够聪明，而是缺少纪律。__

147,000+ Stars 的爆发式增长说明一件事 — 很多开发者在 AI 编程工具上踩过同样的坑：代码能跑但不可靠，功能实现了但没测试，改了 A 坏了 B。

14 个 Skills，7 个阶段闭环，每个环节都有硬性约束。从 brainstorming 的 HARD-GATE，到 TDD 的 Iron Law，再到 finishing-a-development-branch 的验证交付 — 这不是建议，是纪律。

说到底，Superpowers 的全部价值浓缩在源码里那句话就够了：

> __"Skills are not prose — they are code that shapes agent behavior."__

工程纪律才是真正的超能力。

如果你觉得这篇文章有帮助，建议收藏备用。下次遇到 AI 编程产出不稳定的问题，翻出来对照着检查，看看缺了哪个阶段。

__相关资源__

Superpowers GitHub 仓库：https://github.com/obra/superpowers

Superpowers 中文增强版：https://github.com/jnMetaCode/superpowers-zh

Jesse Vincent 首发博文：https://blog.fsck.com/2025/10/09/superpowers/

__好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！__

来源：https://cloud.tencent.com/developer/article/2658943
