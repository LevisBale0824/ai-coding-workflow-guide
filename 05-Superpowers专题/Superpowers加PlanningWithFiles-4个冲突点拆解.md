# Superpowers + PlanningWithFiles：4 个冲突点拆解，别让两个好工具互相打架

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _105_ 篇，AI 编程最佳实战「2026」系列第 _30_ 篇
>
> 大家好，欢迎来到 __术哥无界 | ShugeX ｜ 运维有术__。
>
> 我是__术哥__，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的__技术实践者与开源布道者__！
>
> __Talk is cheap, let's explore。无界探索，有术而行。__

![封面图：Superpowers 和 PlanningWithFiles 双系统碰撞](https://developer.qcloudimg.com/http-save/10642399/49b87c5ca54774a6175ae333fb24aa85.png)

封面图：Superpowers 和 PlanningWithFiles 双系统碰撞

_图 1：Superpowers + PlanningWithFiles 冲突全景_

> __说明__：本文内容基于 Superpowers（obra/superpowers）和 PlanningWithFiles（OthmanAdi/planning-with-files）的源码分析整理而成，包括 README、SKILL.md、hooks 配置等第一手官方素材。__文中的配置模板和参数建议仅供参考，实际效果请以你的业务数据和环境测试结果为准。__如果有实际使用经验，欢迎在评论区分享交流。

群里有朋友问：__有没有 Superpowers + PlanningWithFiles 配合使用的文章？__

这个问题挺好。两个工具单独用都不错——PlanningWithFiles 给 AI 加了持久化记忆，Superpowers 给 AI 套了一套严格的开发流程。但一起装的时候，你会发现 agent 经常__精神分裂__：一个 hook 告诉它先读 plan，另一个 hook 告诉它先调 skill。

翻了一遍两个项目的源码之后，我得出一个结论：__这不是 bug，是结构性冲突__。两个工具的设计哲学不同，硬凑在一起用，注定互相干扰。

但也不是没法解。先说清楚问题在哪，再说怎么分工。

### 1. 拆解 PlanningWithFiles - 文件式工作记忆

PlanningWithFiles 的核心思路可以用一句话概括：__把文件系统当磁盘，把上下文窗口当 RAM__。

AI 的上下文窗口有容量限制，聊天记录一长就__忘事__。PlanningWithFiles 的解法是把任务计划、研究发现、操作进度全部写成 Markdown 文件存下来。新会话开始时，自动读取这些文件，瞬间__恢复记忆__。

#### 三文件模式

```
task_plan.md    → 追踪阶段和进度（核心文件）
findings.md     → 存储研究和发现（知识库）
progress.md     → 会话日志和测试结果（操作日记）
```

`task_plan.md` 是灵魂。它把任务拆成 3-7 个 Phase，每个 Phase 有状态标记（`pending` / `in_progress` / `complete`），还有 checklist 和决策记录。__这个文件解决的核心问题是：agent 在任何时候都知道自己在哪、该干嘛。__

#### Hook 注入机制

PlanningWithFiles 配了 4 个生命周期钩子：

|  |  |  |
| --- | --- | --- |
| UserPromptSubmit | 用户提交 prompt 时 | 注入 task_plan.md 内容到 agent 上下文 |
| PreToolUse | Write/Edit/Bash/Read 前 | 重读 task_plan.md 刷新目标 |
| PostToolUse | Write/Edit 后 | 提醒更新进度 |
| Stop | agent 尝试停止时 | 验证所有阶段是否完成 |

关键在 __PreToolUse__。每次 agent 调用工具之前，这个 hook 都会把 task_plan.md 的头 30-50 行塞进上下文。相当于在你每次动手之前，先贴一张便签提醒你__别忘了大目标__。

#### 几个值得关注的特性

- __Session Recovery__（v2.2.0+）：新会话自动恢复之前的进度，不需要重复解释
- __Hash Attestation__（v2.37.0）：SHA-256 哈希锁定 task_plan.md，防止内容被篡改或注入
- __并行计划__（v2.36.0）：`.planning/YYYY-MM-DD-<slug>/` 目录隔离，支持多个计划并行推进

![PlanningWithFiles 三文件工作流示意图](https://developer.qcloudimg.com/http-save/10642399/87e04de4eacca63cab309bfbeb1f6c39.png)

PlanningWithFiles 三文件工作流示意图

_图 2：PlanningWithFiles 三文件通过 4 个 Hook 与 Agent 上下文交互_

### 2. 拆解 Superpowers - 子代理驱动开发

Superpowers 的定位不一样。它是一个__完整的软件开发方法论框架__，把 AI 写代码这件事拆成了 7 步标准流程：

```
1. brainstorming      → 苏格拉底式提问，从粗糙想法提炼设计方案
2. using-git-worktrees → 创建隔离工作空间
3. writing-plans      → 将设计拆解为 bite-sized 任务（每步 2-5 分钟）
4. subagent-driven-development → 子代理执行计划
5. test-driven-development    → RED-GREEN-REFACTOR 循环
6. requesting-code-review     → 代码审查
7. finishing-a-development-branch → 分支收尾
```

核心隐喻很有意思：它把 AI agent 当成__一个热情但缺乏判断力的初级工程师__。所以每一步都需要明确的指令和严格的审查。

#### Plan 文件的角色

Superpowers 的 plan 文件路径是 `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`。

和 PlanningWithFiles 的 task_plan 完全不同。Superpowers 的 plan 是__实现蓝图__ - 里面是精确的文件路径、完整代码、测试用例、提交命令。每个任务粒度是 bite-sized（2-5 分钟），不允许 placeholder（TBD、TODO），每个任务自带 TDD 循环。

一句话区分：

- __Superpowers plan = 怎么把砖砌成墙__
- __PwF task_plan = 墙砌到哪了，出了什么问题__

#### Skill 触发机制

Superpowers 有个 `using-superpowers` skill，要求 agent 在任何行动前__先检查并调用对应的 skill__。这不是建议，是强制工作流。

它的审查机制是双阶段的：先是 spec compliance（规格合规），再是 code quality（代码质量）。写完代码不是终点，还得过两道审查关。

![Superpowers 7 步工作流全景图](https://developer.qcloudimg.com/http-save/10642399/8653fb5339ba6f46f2e3a3b4cc37e191.png)

Superpowers 7 步工作流全景图

_图 3：Superpowers 7 步工作流——从 brainstorming 到 branch 收尾_

### 3. 协同攻坚 - 为什么配合不好

这是文章的核心部分。我逐个拆解两个工具的冲突点。

#### 冲突 1：双 Hook 系统的 Context 竞争

两个工具都用 hook 往 agent 上下文里塞指令。

__Superpowers 的__ __`using-superpowers`__ 要求 agent 在任何行动前先检查 skill，然后按 skill 的指令行动。原文是：__follow skill exactly__。

__PlanningWithFiles 的 PreToolUse__ 在每次工具调用前注入 task_plan.md 的内容。原文是：__read plan before decisions__。

问题来了。agent 每次操作前会同时收到两套指令注入。后果：

1. __Context 膨胀__：两个系统的指令叠加，吃掉上下文窗口
2. __指令冲突__：一个说先调 skill，一个说先读 plan
3. __优先级混乱__：agent 不知道先听谁的

这不是偶尔出现的问题。PreToolUse 是__每次工具调用__都触发。Superpowers 的 skill 检查也是__每次行动前__都要做。两套指令以极高的频率叠加，agent 的行为就变得不可预测。

#### 冲突 2：双任务追踪系统不同步

两套独立的追踪系统，互不通信。

|  |  |  |
| --- | --- | --- |
| __文件位置__ | `docs/superpowers/plans/*.md` | 项目根目录 `task_plan.md` |
| __任务粒度__ | 极细（每步 2-5 分钟，含完整代码） | 粗（3-7 个 Phase，Phase 内含 checklist） |
| __状态机制__ | TodoWrite（易失性，session 结束就没了） | Phase status（持久化，写死在文件里） |

__同一个任务在两个文件里各有一份，状态永远对不上。__

更麻烦的是 Stop hook。PlanningWithFiles 的 Stop hook 会在 agent 尝试停止时验证 task_plan.md 的完成度。但任务的实际执行进度在 Superpowers 的 plan 文件里。PwF 检查自己的 task_plan 说 Phase 2 还没完成，但 Superpowers 已经把 Phase 2 拆成了 15 个 bite-sized task 并且全做完了。

#### 冲突 3：Plan 文件角色冲突

前面说过，两个 plan 文件解决的问题不一样：

- __Superpowers plan__ 面向初级工程师，告诉它每一步怎么写代码
- __PwF task_plan__ 面向注意力管理，记录你在哪、该干嘛

当两者同时存在时，agent 需要在两个源文件之间来回切换。Superpowers 说按 plan 写代码，PwF 说先看 task_plan 确认你在哪。两套系统争夺 agent 的注意力。

#### 冲突 4：writing-plans 不会以 PwF task_plan 为骨架

这个冲突是最根本的。

Superpowers 的 `writing-plans` skill 有严格的格式要求：

- 每个任务必须包含__精确的文件路径、完整代码、验证步骤__
- 任务粒度是 bite-sized（2-5 分钟）
- __不允许 placeholder__（TBD、TODO 等）
- 每个任务有完整的 TDD 循环

PwF 的 task_plan 组织逻辑要宽松得多：

- Phase 级别的 checklist
- 不要求精确代码
- 更关注你在哪而不是你怎么做

__writing-plans 不会以 PwF 的 task_plan 为骨架__ - 它有自己的结构化标准。PwF 的 Phase checklist 太粗，不满足 writing-plans 的输入要求。

结果就是：两套 plan 文件__平行存在，永远不交叉__。你写你的蓝图，我记我的进度，各干各的。

#### 冲突全景图

![4 个冲突点对比图](https://developer.qcloudimg.com/http-save/10642399/9a83025843e3a18f6445f1abf5abfc83.png)

4 个冲突点对比图

_图 4：4 个冲突点严重程度与影响范围_

|  |  |  |  |
| --- | --- | --- | --- |
| 双 Hook Context 竞争 | 两个系统都高频注入指令 | agent 行为不可预测 | 🔴 高 |
| 双追踪不同步 | 两套独立状态系统 | 进度永远对不上 | 🟠 中高 |
| Plan 角色冲突 | 两个文件解决不同问题 | agent 注意力分散 | 🟡 中 |
| writing-plans 不兼容 | 格式标准完全不同 | 两套 plan 永远平行 | 🔴 高 |

### 4. 落地方案 - 分工协同，不做强行整合

说了这么多问题，解决方案的思路其实很简单：__不试图合并两个系统，让各自做擅长的事__。

核心思路：__PwF 管全局进度，Superpowers 管实现细节__。

#### 具体做法

__1. PwF 管全局__

`task_plan.md` 只记录 Phase 级进度、关键决策、错误日志。不写具体代码，不写实现细节。

__2. Superpowers 管实现__

plan 文件记录精确的实现步骤、完整代码、测试用例。不管进度追踪，只管怎么做。

__3. 桥接规则__

在 `task_plan.md` 的每个 Phase 中引用 Superpowers plan 文件路径。agent 通过这个路径知道这个 Phase 对应的实现细节在哪。

__4. 状态同步__

Superpowers 每完成一个 task，手动（或通过自定义 hook）更新 `task_plan.md` 对应 Phase 的状态。

#### task_plan.md 示例

下面是一个桥接版的 `task_plan.md` 结构：

```
# Task Plan: 用户认证模块

## Phase 1: 数据库模型设计 [in_progress]
> 🔗 Superpowers Plan: docs/superpowers/plans/2026-05-07-user-auth.md#task-1-3

- [x] 设计 User 表结构
- [x] 设计 Session 表结构
- [ ] 编写迁移脚本

**决策记录**: 使用 UUID 作为主键，避免 ID 枚举攻击

## Phase 2: 认证 API 实现 [pending]
> 🔗 Superpowers Plan: docs/superpowers/plans/2026-05-07-user-auth.md#task-4-8

- [ ] POST /auth/register
- [ ] POST /auth/login
- [ ] POST /auth/logout
- [ ] JWT Token 签发与验证

## Phase 3: 前端集成 [pending]
> 🔗 Superpowers Plan: docs/superpowers/plans/2026-05-07-user-auth.md#task-9-12

- [ ] 登录页面
- [ ] 注册页面
- [ ] Token 存储（localStorage）

## Errors Encountered
| Error | Context | Resolution |
|-------|---------|------------|
| - | - | - |
```

注意每个 Phase 下面的 `🔗 Superpowers Plan` 行。这是桥接的关键 - agent 知道这个 Phase 的实现细节在 Superpowers plan 的哪些 task 里。

#### Superpowers plan 桥接示例

Superpowers plan 文件本身不需要改动。它保持原有的精确格式：

```
# Plan: 用户认证模块

## Task 1: 创建 User 模型
File: `src/models/user.py`
Duration: ~3 minutes

` ` `python
import uuid
from datetime import datetime
from sqlalchemy import Column, String, DateTime
from src.db.base import Base

class User(Base):
    __tablename__ = "users"
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    email = Column(String, unique=True, nullable=False, index=True)
    password_hash = Column(String, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
` ` `

Verify: `pytest tests/models/test_user.py -v`

## Task 2: 创建 Session 模型
...
```

#### 工作流全貌

```
1. brainstorming 阶段 → 正常使用 Superpowers 的苏格拉底式提问
2. brainstorming 完成 → 手动创建 PwF 的 task_plan.md（Phase 级别）
3. writing-plans 阶段 → 正常使用 Superpowers 生成精确 plan
4. writing-plans 完成 → 在 task_plan.md 每个 Phase 中写入 plan 路径引用
5. 执行阶段 → Superpowers 的 skill 驱动实现，PwF 的 hook 驱动进度提醒
6. 每完成一个 Superpowers task → 更新 task_plan.md 对应 Phase 的 checklist
7. session 结束 → PwF 的 Stop hook 验证整体完成度
8. session 恢复 → PwF 的 hook 自动注入进度，Superpowers 读取 plan 继续
```

![协同方案架构图](https://developer.qcloudimg.com/http-save/10642399/76b9e6376bff7f910e7af6a1d6d4b804.png)

协同方案架构图

_图 5：PwF 和 Superpowers 桥接架构——Plan 路径引用打通双系统_

#### 注意事项

说实话，这个方案不是银弹。有几个坑需要知道：

- __同步靠自律__：Superpowers 完成一个 task 后，agent 不一定记得去更新 task_plan.md。需要在 hook 或 skill 中加入提醒
- __Context 仍然会膨胀__：PwF 的 PreToolUse hook 照常注入，Superpowers 的 skill 照常加载。但至少指令不会冲突了 - 一个管进度，一个管实现
- __PwF 的 Stop hook 可能误判__：它只看 task_plan.md 的完成度。如果同步没做好，会在不该停的时候叫停。建议在 task_plan.md 中加入备注，标记哪些 Phase 是跨 session 的

你在项目中同时用过两个以上的 Claude Code 插件吗？遇到过类似的冲突吗？欢迎在评论区聊聊。

### 总结

Superpowers 和 PlanningWithFiles 不是互斥的，是互补的。但__互补的前提是分工明确__。

PlanningWithFiles 擅长管理全局进度和跨 session 记忆，Superpowers 擅长管理代码质量和实现流程。让 PwF 做项目经理，让 Superpowers 做技术负责人，通过 plan 路径引用做桥接，是目前成本低于收益的协同方式。

不建议强行整合。两个工具的设计哲学不同，硬要统一格式或合并 hook，只会制造更多问题。接受两个系统平行存在这个事实，把精力放在桥接和同步上，才是务实的做法。

说到底，__好的工具组合不是 1+1=2，而是各管各的，互不添乱__。

你身边有同事也在用 AI 编程助手做开发？转发给他们看看这个拆解。关于插件协同，你踩过什么坑？评论区聊聊。

__好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！__


来源：https://cloud.tencent.com/developer/article/2666142