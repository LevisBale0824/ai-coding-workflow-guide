# AI 编程工作流培训分享方案

> 基于文章合集「术哥无界 | ShugeX | 运维有术」AI 编程最佳实战 2026 系列，共 28 篇
>
> 整理时间：2026-05-17

---

## 一、培训概述

### 培训目标

通过 4 次递进式分享，让开发团队掌握 AI 编程工作流的完整工具链，从"让 AI 写代码"进化到"用流程控制 AI 写出可靠代码"。

### 核心命题

**如何让 AI 编程从"碰运气"变成"可控流程"？**

### 受众

- 全员参加，主要受众为开发人员
- 无需前置知识，但需要基本的命令行操作能力

### 节奏安排

| 次序 | 主题 | 时长 | 建议间隔 |
|------|------|------|----------|
| 分享 1 | AI 编程工作流入门 — 工具定位 + OpenSpec 跑通 | 60 min | — |
| 分享 2 | OpenSpec 进阶 — 增量开发 + 质量控制 | 60 min | 1 周后 |
| 分享 3 | Superpowers 实战 — 7 步工作流跑通完整项目 | 60 min | 1 周后 |
| 分享 4 | 组合实战 — OpenSpec + Superpowers 联合工作流 | 60 min | 1 周后 |

### 依赖关系

```
分享 1（OpenSpec 入门：3 条命令跑通）
  ↓
分享 2（OpenSpec 进阶：增量开发 + 质量控制）
  ↓
分享 3（Superpowers 单独：7 步工作流完整实战）
  ↓
分享 4（组合：OpenSpec + Superpowers 联合 + TDD v1/v2 实测）
```

每次间隔 1 周，给开发者时间消化和练手。

### 每次分享的标准结构

- **20% 理论**：讲清概念和为什么
- **60% 现场演示**：Live Coding，跟做
- **20% 踩坑经验**：真实翻车案例和解决方案

---

## 二、分享 1：AI 编程工作流入门 — 工具定位 + OpenSpec 3 条命令跑通

### 基本信息

| 项目 | 内容 |
|------|------|
| 时长 | 60 分钟 |
| 目标 | 所有人理解工具全景；开发者在会上跑通第一个 OpenSpec 流程 |
| 前置要求 | 无 |
| 素材范围 | 文章 1、2、3、7 |

### 环节拆解

#### 环节 1：痛点共鸣（5 分钟）

**目标**：让所有人感受到"AI 写代码会翻车"不是个别现象

**内容要点**：

1. AI 编程的 3 个典型翻车场景：
   - **跑偏**：AI 自作主张加了没要求的功能
   - **遗漏**：AI 跳过了边界条件、错误处理
   - **风格不统一**：不同会话产出的代码风格不一致

2. 核心观点：问题不在"AI 不够强"，在于"人类给指令太随意"

**素材**：文章 12《OpenSpec 最佳实战：4 步复盘 5 项升级》中的"能跑 ≠ 跑对"论述

> 参考原文：`03-OpenSpec进阶/OpenSpec最佳实战-4步复盘5项升级.md`

---

#### 环节 2：工具全景（10 分钟）

**目标**：3 分钟讲清每个工具定位，3 分钟讲清它们的关系

**内容要点**：

1. 三个工具一句话定位：

   | 工具 | 一句话定位 | Star 数 | 核心能力 |
   |------|-----------|---------|----------|
   | Spec-Kit | GitHub 官方，可执行规格生成代码 | 82.5K | 7 阶段 spec → code |
   | OpenSpec | 轻量级规格管理层 | 34.5K+ | 3 条命令管理 spec 全生命周期 |
   | Superpowers | 技能驱动的 Agent 行为纪律 | 115K+ | 14 个 Skills 约束 AI 执行过程 |

2. 关键区别：它们不是竞争关系，解决的是不同层的问题：
   - **OpenSpec** 管"写什么"（规格层）— 把模糊需求变成结构化规格文件
   - **Superpowers** 管"怎么做"（执行层）— 用流程纪律约束 AI 的编码行为
   - 两者互补，可以组合使用

**素材**：
- 文章 1《AI 编程工作流选型：Spec-Kit、OpenSpec、Superpowers 深度对比》

  > 参考原文：`01-选型与对比/AI编程工作流选型-SpecKit-OpenSpec-Superpowers深度对比.md`

- 文章 2《OpenSpec vs Superpowers：2 套 AI 编码工作流怎么选？》

  > 参考原文：`01-选型与对比/OpenSpec-vs-Superpowers-2套AI编码工作流怎么选.md`

---

#### 环节 3：环境搭建（5 分钟）

**目标**：所有人安装好 OpenSpec，准备好跟做

**操作步骤**：

```bash
# 前置：确认 Node.js 版本
node -v   # 需要 >= 20.19.0

# 安装 OpenSpec
npm install -g openspec

# 验证安装
openspec --version

# 在 Claude Code 中安装 OpenSpec 适配器
# 进入项目目录后执行
openspec init
```

**注意事项**：
- 提前确认网络环境，npm install 可能需要配置镜像
- 建议提前准备一个空项目目录用于演练

**素材**：文章 5《OpenSpec 实战：从 0 到 1 构建AI 原生规范驱动开发工作流》

> 参考原文：`02-OpenSpec入门/OpenSpec实战-从0到1构建AI原生规范驱动开发工作流.md`

---

#### 环节 4：Live Demo — 3 条命令跑通全流程（30 分钟）

**目标**：现场用 Todo API 项目走完 propose → apply → archive 完整循环

**演示项目**：Todo API（一个简单的待办事项管理接口）

**Step 1：启动 propose（10 分钟）**

```bash
# 在项目目录中执行
/opsx:propose "创建一个 Todo REST API，支持增删改查"
```

**演示重点**：
- propose 做了什么：AI 根据需求自动生成 4 个制品
  - `proposal.md`：需求提案（做什么、为什么、范围）
  - `specs.md`：功能规格（GIVEN/WHEN/THEN 格式）
  - `design.md`：技术设计（架构、数据模型）
  - `tasks.md`：任务清单（具体实施步骤）
- 强调：**先审批规格，再写代码**

**Step 2：审批 + apply（10 分钟）**

```bash
# 审阅 4 个制品，确认无误后执行
/opsx:apply
```

**演示重点**：
- AI 按照 tasks.md 中的任务清单逐个实现
- 每个任务完成后可以查看代码变更
- 对比"裸跑"（直接让 AI 写代码）vs "走 OpenSpec" 的输出质量差异

**Step 3：验证 + archive（10 分钟）**

```bash
# 验证功能是否正确
# 运行测试或手动验证

# 确认后归档
/opsx:archive
```

**演示重点**：
- archive 做了什么：将 Delta Specs 合并回主规格文件，清理临时文件
- 归档后的项目状态：规格文件是最新且完整的

**素材**：文章 7《OpenSpec 最佳实战：3 条命令跑通规范驱动开发全流程》

> 参考原文：`02-OpenSpec入门/OpenSpec最佳实战-3条命令跑通规范驱动开发全流程.md`

---

#### 环节 5：回顾总结（10 分钟）

**内容要点**：

1. 3 条命令速查：

   | 命令 | 做什么 | 产生什么 |
   |------|--------|----------|
   | `propose` | 根据需求生成规格制品 | proposal / specs / design / tasks |
   | `apply` | 按规格实现代码 | 生产代码 |
   | `archive` | 归档规格，合并回主线 | 更新后的主规格文件 |

2. 4 个制品的含义：

   | 制品 | 回答的问题 |
   |------|-----------|
   | proposal | 做什么？为什么做？ |
   | specs | 功能怎么验证？（GIVEN/WHEN/THEN） |
   | design | 技术上怎么实现？ |
   | tasks | 具体分几步做？ |

3. 核心理念：**先写规格再写代码，质量始于规格而非测试**

**素材**：文章 8《OpenSpec 最佳实战：3 个命令 4 个制品 4 步闭环》

> 参考原文：`02-OpenSpec入门/OpenSpec最佳实战-3个命令4个制品4步闭环.md`

---

### 关键知识点

1. OpenSpec 的三层架构：CLI 层 → 核心引擎层 → Schema 层
2. DAG 依赖解析：制品之间有依赖关系，OpenSpec 自动按依赖顺序生成
3. 三层校验：格式校验 → 语义校验 → 逻辑校验

> 参考：文章 3 `02-OpenSpec入门/OpenSpec深度解析-最佳实践四步法.md`

### 踩坑提醒

- propose 时需求描述要具体，太模糊会导致规格跑偏
- apply 前务必审阅 tasks.md，任务粒度决定代码质量
- archive 不可逆，确认功能验证通过后再操作

### 带走物

- OpenSpec 安装命令速查卡
- 3 条命令速查卡（命令 → 做什么 → 产生什么）
- 4 个制品模板示例（从 Demo 项目中复制）

---

## 三、分享 2：OpenSpec 进阶 — 增量开发、定制配置、质量提升

### 基本信息

| 项目 | 内容 |
|------|------|
| 时长 | 60 分钟 |
| 目标 | 掌握 Delta Specs 增量开发；学会用配置文件控制 AI 输出质量 |
| 前置要求 | 完成分享 1，能独立跑通 propose → apply → archive |
| 素材范围 | 文章 8、9、10、12、13、14、15 |

### 环节拆解

#### 环节 1：回顾 + 延伸（10 分钟）

**目标**：回顾 3 条命令，引出"已有项目要加新功能怎么办"的问题

**内容要点**：

1. 快速回顾：propose → apply → archive，3 条命令 4 个制品
2. 提出问题：
   - 上次的 Todo API 已经有了，现在要加一个"优先级"功能，怎么办？
   - 全部规格重写？太浪费。只改一部分？怎么管理变更？
3. 引出 Delta Specs：只记录变更部分，不重写全部规格

**素材**：
- 文章 7（回顾）：`02-OpenSpec入门/OpenSpec最佳实战-3条命令跑通规范驱动开发全流程.md`
- 文章 8（延伸）：`02-OpenSpec入门/OpenSpec最佳实战-3个命令4个制品4步闭环.md`

---

#### 环节 2：Live Demo — 增量开发（20 分钟）

**目标**：用 Delta Specs 给 Todo API 加"优先级"功能

**演示场景**：在已有的 Todo API 基础上，新增任务优先级字段（high / medium / low）

**Step 1：发起增量 propose（5 分钟）**

```bash
/opsx:propose "给 Todo 增加优先级功能，支持 high/medium/low 三级"
```

**演示重点**：
- Delta Specs 自动识别：哪些是新增（ADDED），哪些是修改（MODIFIED）
- 与首次 propose 的区别：这次只有变更部分，不是全量生成

**Step 2：查看 Delta 标记（5 分钟）**

```
ADDED:     priority 字段定义
MODIFIED:  Todo 数据模型（新增 priority 属性）
MODIFIED:  POST /todos 接口（新增 priority 参数）
ADDED:     GET /todos?priority=high 按优先级筛选
```

**演示重点**：
- 4 种 Delta 操作类型：`ADDED` / `MODIFIED` / `REMOVED` / `RENAMED`
- Delta 只记录差异，不重复已有规格

**Step 3：apply + archive（10 分钟）**

```bash
# 审阅 Delta Specs 后执行
/opsx:apply

# 验证新功能后归档
/opsx:archive
```

**演示重点**：
- apply 时 AI 只修改涉及的部分，不动已有功能
- archive 后 Delta 合并回主规格文件，规格重新变为完整一致的状态

**素材**：文章 8《OpenSpec 最佳实战：3 个命令 4 个制品 4 步闭环》

> 参考原文：`02-OpenSpec入门/OpenSpec最佳实战-3个命令4个制品4步闭环.md`

---

#### 环节 3：Live Demo — 质量优化（15 分钟）

**目标**：不改 OpenSpec 源码，只改 config.yaml，就能显著提升 AI 输出质量

**核心结论**：任务粒度是唯一最高 ROI 的质量控制手段，20% 的改动覆盖 80% 的质量提升

**演示步骤**：

**Step 1：展示"粗粒度任务"的问题（5 分钟）**

```markdown
# 粗粒度 tasks.md 示例（不好）
- [ ] 实现用户注册功能
- [ ] 实现用户登录功能
```

**问题**：AI 在一个任务内把注册、验证、错误处理、数据库操作全做了，无法控制质量

**Step 2：改为"细粒度任务"（5 分钟）**

```markdown
# 细粒度 tasks.md 示例（好）
- [ ] 创建 User 数据模型，包含 username/email/password_hash 字段
- [ ] 实现 POST /api/register 路由，接收 username/email/password
- [ ] 添加密码 bcrypt 加密逻辑
- [ ] 实现注册参数校验（用户名不为空、邮箱格式、密码长度）
- [ ] 添加重复用户名检测，返回 409 状态码
```

**关键**：每个任务控制在 2-5 分钟可完成的粒度

**Step 3：通过 config.yaml 控制 tasks 生成质量（5 分钟）**

```yaml
# config.yaml 中的关键配置
tasks:
  instruction: |
    将每个任务拆分为 2-5 分钟可完成的原子任务。
    每个任务只做一件事。
    明确写出输入、输出、验证条件。
```

**演示重点**：
- 对比改动前后，propose 生成的 tasks.md 粒度差异
- 强调：**这是成本最低、效果最显著的质量优化手段**

**素材**：文章 14《OpenSpec 最佳实战：不改源码只改配置提升代码质量》

> 参考原文：`03-OpenSpec进阶/OpenSpec最佳实战-不改源码只改配置提升代码质量.md`

---

#### 环节 4：Core vs Expanded 模式（10 分钟）

**目标**：知道什么时候用 Core（3 条命令），什么时候切 Expanded（7 条命令）

**内容要点**：

1. 两种模式对比：

   | 维度 | Core 模式 | Expanded 模式 |
   |------|-----------|---------------|
   | 命令数 | 3 条（propose/apply/archive） | 7 条额外命令 |
   | 特点 | 自动挡，一步到位 | 手动挡，逐步控制 |
   | 适合场景 | 需求明确、变更简单 | 需求复杂、需要逐步审阅 |

2. Expanded 模式新增的 7 条命令：

   | 命令 | 做什么 |
   |------|--------|
   | `new` | 创建空白的 spec scaffold |
   | `continue` | 逐个填充制品 |
   | `ff` | 快速推进到下一个待填充制品 |
   | `verify` | 三维校验（格式/语义/逻辑） |
   | `bulk-archive` | 批量归档 |
   | `onboard` | 生成项目引导教程 |

3. 选型建议：**先用 Core 跑通，遇到质量控制需求再切 Expanded**

**素材**：
- 文章 9：`03-OpenSpec进阶/OpenSpec工作流全解析-选对模式开发效率翻倍.md`
- 文章 15：`03-OpenSpec进阶/OpenSpec进阶-从Core到Expanded-7个命令解锁全部工作流.md`

---

#### 环节 5：团队定制化（5 分钟）

**目标**：知道如何让 OpenSpec 适配自己的团队

**内容要点**：

1. 3 层定制路径：

   | 层级 | 方式 | 耗时 | 覆盖率 |
   |------|------|------|--------|
   | 第 1 层 | 修改 config.yaml | 5 分钟 | 90% 的团队 |
   | 第 2 层 | 自定义 Schema 模式 | 30 分钟 | 特殊需求 |
   | 第 3 层 | 全局覆写 | 1 小时+ | 完全定制 |

2. 建议：**从第 1 层开始，90% 的团队只改 config.yaml 就够了**

**素材**：文章 10《OpenSpec 定制化全攻略：让 AI 开发工作流适配团队》

> 参考原文：`03-OpenSpec进阶/OpenSpec定制化全攻略-让AI开发工作流适配团队.md`

---

### 关键知识点

1. **Delta Specs 的 4 种操作类型**：ADDED / MODIFIED / REMOVED / RENAMED
2. **任务粒度是唯一最高杠杆的质量控制**：粗粒度任务让 AI "自由发挥"，细粒度原子任务（2-5 分钟）是真正的约束手段
3. **质量提升的 5 个方向**（按 ROI 排序）：
   - 任务粒度控制（最高 ROI）
   - Rules 注入
   - Validate 校验
   - Explore 探索
   - 自定义 Schema + Review 制品

> 参考：文章 12、13 的 4 步复盘 + 3 轮实测
> `03-OpenSpec进阶/OpenSpec最佳实战-4步复盘5项升级.md`
> `03-OpenSpec进阶/OpenSpec最佳实战-3轮实测验证5个质量升级方向.md`

### 踩坑提醒

- Delta Specs 适用于增量开发，全新项目仍用标准 propose
- archive 操作会合并 Delta，确认无误后再执行
- config.yaml 的 tasks instruction 是文本提示，不是硬约束 — 如果 AI 不遵守，需要进一步拆细任务
- 不要一上来就用 Expanded 模式，先用 Core 跑通再升级

### 带走物

- 一份优化过的 config.yaml 模板（包含 tasks 粒度控制配置）
- Delta Specs 4 种操作类型速查卡
- Core vs Expanded 模式选择决策树

---

## 四、分享 3：Superpowers 实战 — 7 步工作流跑通完整项目

### 基本信息

| 项目 | 内容 |
|------|------|
| 时长 | 60 分钟 |
| 目标 | 独立跑通 Superpowers 7 步流程；理解每个阶段在干什么 |
| 前置要求 | 完成分享 1、2（理解 OpenSpec 的定位即可，不要求精通） |
| 素材范围 | 文章 19、20、21、22 |

### 环节拆解

#### 环节 1：Superpowers 是什么（5 分钟）

**目标**：一句话讲清定位和核心理念

**内容要点**：

1. 一句话定位：**不是让 AI 更聪明，而是让 AI 更守规矩**
2. 核心理念：**Process over Prompt** — 与其花时间调教提示词，不如让 AI 走完固定的开发流程
3. 关键数据：
   - 14 个 Skills
   - 7 步工作流
   - 3 条铁律
   - 支持 6 个平台（Claude Code / Cursor / Codex / OpenCode / Gemini CLI / Copilot CLI）

**素材**：文章 19《Superpowers 实战指南：7 步流程 + 14 个技能 + 3 条铁律》

> 参考原文：`05-Superpowers专题/Superpowers实战指南-7步流程14个技能3条铁律.md`

---

#### 环节 2：机制解析（10 分钟）

**目标**：理解 Superpowers 的强制执行机制 — 不是建议，是纪律

**内容要点**：

1. 7 步工作流全景：

   | 阶段 | 英文名 | 做什么 |
   |------|--------|--------|
   | 1 | brainstorming | 苏格拉底式提问，澄清需求 |
   | 2 | using-git-worktrees | 创建隔离工作空间 |
   | 3 | writing-plans | 拆成 2-5 分钟小任务 |
   | 4 | subagent-driven-development | 每个任务派独立子代理 |
   | 5 | test-driven-development | 先写失败测试，再写实现 |
   | 6 | requesting-code-review | 自动代码审查 |
   | 7 | finishing-a-development-branch | 验证 + 合并 + 清理 |

2. 强制执行机制：

   ```
   Agent 启动 → session-start-hook 注入：

   <EXTREMELY_IMPORTANT>
   You have Superpowers.
   RIGHT NOW, go read: @skills/getting-started/SKILL.md
   </EXTREMELY_IMPORTANT>
   ```

   Agent 学到的第一条规矩：**如果你有 skill 来做某事，你必须使用它。不是建议，不是可选，是必须。**

3. Skills 的本质：**不是散文式的建议，而是代码级的精确指令，塑造 Agent 行为**（"Skills are not prose — they are code that shapes agent behavior"）

4. subagent 隔离：
   - 每个子代理是 fresh context，只看到当前任务
   - 两阶段审查：spec reviewer（检查是否多做/少做）+ code quality reviewer（检查代码质量）
   - 审查不通过可打回

**素材**：文章 20《Superpowers 最佳实战：标准开发 7 步法闭环工作流》

> 参考原文：`05-Superpowers专题/Superpowers最佳实战-标准开发7步法闭环工作流.md`

---

#### 环节 3：Live Demo — 7 阶段完整实战（30 分钟）

**目标**：用真实业务场景走完 Superpowers 全部 7 个阶段

**演示项目**：电商系统 SKU 商品库存扣减（Python + FastAPI + SQLAlchemy）

**演示步骤**：

**阶段 1：brainstorming — 需求澄清（3 分钟）**

- Superpowers 不会直接写代码，先退后一步问你真正想做什么
- 通过苏格拉底式对话提炼需求，输出设计文档
- **演示点**：让听众看到 AI 是如何"追问"而非"直接开写"的

**阶段 2：using-git-worktrees — 工作空间隔离（2 分钟）**

- 自动创建独立的 Git Worktree
- 验证测试基线
- **演示点**：隔离环境，不影响主分支

**阶段 3：writing-plans — 任务拆分（5 分钟）**

- 把需求拆成 2-5 分钟的小任务
- 每个任务有完整的代码和验证步骤
- **演示点**：任务拆分的粒度控制是质量关键

**阶段 4：subagent-driven-development — 子代理执行（10 分钟）**

- 每个任务派一个独立子代理
- subagent 只看到当前任务文本，context 隔离
- 两阶段审查：先查规格合规性，再查代码质量
- **演示点**：对比直接让 AI 写 vs 走 subagent 流程的输出差异

**阶段 5：test-driven-development — TDD 循环（5 分钟）**

- RED：先写失败测试
- GREEN：写最小实现让测试通过
- REFACTOR：重构代码
- **演示点**：强制 TDD 循环，AI 不能跳步

**阶段 6：requesting-code-review — 代码审查（3 分钟）**

- 自动审查代码质量和规格合规性
- Critical 问题阻断进度
- **演示点**：审查报告长什么样，什么问题会被拦截

**阶段 7：finishing-a-development-branch — 收尾（2 分钟）**

- 验证所有测试通过
- 合并 / PR / 清理
- **演示点**：完整的交付闭环

**素材**：文章 21《Superpowers 最佳实战：7 阶段工作流交付电商 SKU 库存逻辑》

> 参考原文：`05-Superpowers专题/Superpowers最佳实战-7阶段工作流交付电商SKU库存逻辑.md`

---

#### 环节 4：踩坑 — 工具组合冲突（10 分钟）

**目标**：了解 Superpowers 与其他工具混装时的结构性冲突

**内容要点**：

1. **案例**：Superpowers + PlanningWithFiles 一起安装后，Agent 会"精神分裂"

2. **4 个冲突点**：

   | 冲突点 | 具体表现 |
   |--------|----------|
   | Hook 优先级冲突 | 两个工具都用 session-start-hook，抢占 Agent 启动流程 |
   | 工作记忆冲突 | PlanningWithFiles 的文件记忆 vs Superpowers 的 Skill 指令，Agent 不知道听谁的 |
   | 任务计划冲突 | 两个工具各自有一套任务管理方式，互不兼容 |
   | 生命周期冲突 | 任务推进的节奏不同，导致 Agent 在错误时机触发错误行为 |

3. **核心结论**：这不是 bug，是结构性冲突。两个工具的设计哲学不同，硬凑在一起注定互相干扰

4. **解决思路**：明确分工 — 一个管计划，一个管执行，不要重叠

**素材**：文章 22《Superpowers + PlanningWithFiles：4 个冲突点拆解》

> 参考原文：`05-Superpowers专题/Superpowers加PlanningWithFiles-4个冲突点拆解.md`

---

#### 环节 5：对比收尾（5 分钟）

**目标**：总结 Superpowers 定位，为下次组合分享铺垫

**内容要点**：

| 维度 | OpenSpec | Superpowers |
|------|----------|-------------|
| 解决什么 | "写什么" — 规格管理 | "怎么做" — 执行纪律 |
| 核心产物 | 结构化 Markdown 规格文件 | 按流程执行的生产代码 |
| 约束方式 | 硬约束（文件存在性、格式校验） | 软约束（prompt 注入行为塑造） |
| 适用阶段 | 需求定义、设计规划 | 编码实现、测试审查 |

**预告**：下次分享会把两个工具串起来，跑通"定义需求 → TDD 开发 → 验证交付"的完整链路。

---

### 关键知识点

1. **Superpowers 的 3 条铁律**：
   - 有 Skill 可用时，必须使用 Skill
   - 每个阶段完成前，不进入下一阶段
   - 代码审查的 Critical 问题必须修复
2. **subagent 隔离是真实的**：每个子代理是 fresh context，无法跨任务批量执行
3. **Skills 是行为塑造代码，不是散文建议**：强制触发，不是可选执行

### 踩坑提醒

- Superpowers 的 TDD 阶段，AI 可能"假装做了 RED 阶段"（这个问题在分享 4 会深入讨论）
- 不要把 Superpowers 和其他行为管理工具（如 PlanningWithFiles）同时安装，会产生结构性冲突
- 7 步流程比直接写代码慢，但前期多花的时间会从后期返工中省回来

### 带走物

- Superpowers 7 步流程速查卡（阶段 → 英文名 → 做什么 → 关键产出）
- 安装配置 Checklist
- 3 条铁律卡片

---

## 五、分享 4：组合实战 — OpenSpec + Superpowers 联合工作流

### 基本信息

| 项目 | 内容 |
|------|------|
| 时长 | 60 分钟 |
| 目标 | 两个工具串起来跑通完整链路；了解组合方案的实际效果和已知问题 |
| 前置要求 | 完成分享 1、2、3 |
| 素材范围 | 文章 23、24、25、26、27、28 |

### 环节拆解

#### 环节 1：联合流程设计（10 分钟）

**目标**：讲清两个工具如何协作，各自的职责边界

**内容要点**：

1. 联合工作流设计：

   ```
   OpenSpec（定义意图）          Superpowers（执行交付）
   ┌─────────────────┐         ┌─────────────────────┐
   │ propose          │         │ brainstorming        │
   │  → 生成 4 个制品  │ ──────→ │ writing-plans        │
   │ apply            │         │ subagent-development │
   │  → AI 按规格实现  │         │ TDD                  │
   │ archive          │         │ code-review          │
   │  → 归档规格      │         │ finish-branch        │
   └─────────────────┘         └─────────────────────┘
   ```

2. 架构师角色重构：
   - **传统角色**：写代码 + 审代码 + 改代码
   - **新角色**：只做两件事 — 定义意图（OpenSpec propose）+ 审批方案（review artifacts）
   - OpenSpec + Superpowers 处理其余：规格生成、任务拆分、TDD 编码、代码审查

3. 实证数据：30 分钟交付一个看板管理系统（CRUD + DB + 单元测试）

**素材**：文章 28《架构师手册：OpenSpec + Superpowers 打造一人极简开发工作流》

> 参考原文：`06-OpenSpec-Superpowers协作/架构师手册-OpenSpec加Superpowers打造一人极简开发工作流.md`

---

#### 环节 2：Live Demo — 联合实战（20 分钟）

**目标**：用用户认证模块跑通 6 步联合流程

**演示项目**：用户认证模块（注册/登录/JWT，Python + FastAPI + SQLite + pytest）

**演示步骤**：

**Step 1-2：OpenSpec 阶段 — 锁定设计意图（8 分钟）**

```bash
# Step 1: 生成规格
/opsx:propose "实现用户认证模块，支持注册、登录、JWT token"

# Step 2: 审阅 4 个制品，确认设计意图正确后执行
/opsx:apply
```

**演示重点**：
- OpenSpec 产出的 specs.md 中的 GIVEN/WHEN/THEN 格式 — 这些是可以直接映射为测试用例的
- tasks.md 中的任务拆分 — 这些是 Superpowers subagent 的输入

**Step 3-5：Superpowers 阶段 — TDD 交付（10 分钟）**

```
# Step 3: Superpowers 接手，按 tasks.md 的任务清单逐个执行
# Step 4: subagent 逐任务开发，每个任务走 RED → GREEN → REFACTOR
# Step 5: 代码审查 + 分支收尾
```

**演示重点**：
- 哪些环节衔接顺畅：OpenSpec 的 tasks 输出 → Superpowers 的 subagent 输入
- 哪些环节需要手动干预（为下个环节铺垫）

**Step 6：归档（2 分钟）**

```bash
/opsx:archive
```

**素材**：文章 25《OpenSpec + Superpowers：6 步实现 AI 规格 TDD 开发》

> 参考原文：`06-OpenSpec-Superpowers协作/OpenSpec加Superpowers-6步实现AI规格TDD开发.md`

---

#### 环节 3：衔接断点实录（10 分钟）

**目标**：诚实呈现组合方案的当前状态 — 不是完美的，但可用

**内容要点**：

1. 3 个衔接点测试结果：

   | 衔接点 | 状态 | 说明 |
   |--------|------|------|
   | OpenSpec tasks → Superpowers plan | ⚠️ 部分通过 | 格式需手动对齐 |
   | OpenSpec specs → Superpowers TDD | ❌ 断裂 | AI 不自动将 specs 映射为测试 |
   | Superpowers review → OpenSpec archive | ❌ 未触发 | 两个工具无联动机制 |

2. 2 个断点需要手动干预：
   - **断点 1**：OpenSpec 的 specs 格式和 Superpowers 的 plan 格式不完全匹配，需要手动桥接
   - **断点 2**：Superpowers 审查完成后不会自动触发 OpenSpec archive，需要手动执行

3. 核心结论：**自动无缝串联目前做不到，但手动干预成本可控**

**素材**：文章 24《OpenSpec + Superpowers 协作实战：3 个衔接点断了 2 个》

> 参考原文：`06-OpenSpec-Superpowers协作/OpenSpec加Superpowers协作实战-3个衔接点断了2个.md`

---

#### 环节 4：TDD 强制执行实验 — v1 失败到 v2 改进（15 分钟）

**目标**：这是整个培训中最有深度的实战案例 — 展示如何真正让 AI 按 TDD 流程写代码

**背景**：用 OpenSpec 自定义 Schema + Superpowers subagent 编排，让 AI 按 TDD 流程写代码。这是一个经过实测验证的方案。

**v1：完全失败（5 分钟）**

1. v1 的设计思路：在 OpenSpec 的 `instruction` 里写一大段 TDD 规则，要求 AI 按 RED-GREEN-REFACTOR 循环执行

2. propose 阶段看起来不错：proposal 里有 WHEN/THEN 格式，specs 用了 GIVEN/WHEN/THEN

3. 但 apply 阶段崩了：**AI 一口气写完所有代码，跳过 RED 阶段，测试是写完实现后补的**

4. 失败根因分析：

   | OpenSpec 能力 | 实际行为 | 约束力 |
   |---------------|----------|--------|
   | `tracks` | checkbox 解析器，只认 `- [ ]` 格式 | 仅追踪进度 |
   | `requires` | 文件存在性检查，不检查内容 | 仅阻止缺失 |
   | `instruction` | 纯文本注入到 stdout | **零运行时约束** |

5. 关键发现：**根因不在 instruction 措辞，在任务粒度**。v1 的一个任务包含"创建接口 + 写测试 + 实现功能"，AI 一步做完是"合理"的。

**v2：四层防护（8 分钟）**

1. 修正方案：四层防护，每层解决一个具体问题

   | 防护层 | 解决的问题 | 机制 | 实测结果 |
   |--------|-----------|------|----------|
   | 第 1 层：原子化任务 | 任务内混合 RED + GREEN | 每个任务只做一件事（写测试 OR 写实现） | ✅ 通过 |
   | 第 2 层：Schema 注入 TDD 规则 | instruction 是纯文本，AI 可忽略 | 在 schema.yaml 的多个位置注入 TDD 约束 | ✅ 通过 |
   | 第 3 层：subagent context 隔离 | AI 可能跨任务批量执行 | Superpowers 的 subagent 隔离机制 | ✅ 通过 |
   | 第 4 层：审查打回 | AI 可能产出不合格代码 | spec reviewer + code quality reviewer | ❌ 被跳过 |

2. 实测数据：
   - 26 个原子任务
   - 27 次 subagent dispatch
   - 3/4 防护层通过
   - 1 层被 AI 跳过（审查环节）

3. 核心结论：
   - **instruction 是文本注入，不是可执行约束** — OpenSpec 源码里搜索不到任何 hook、callback 或运行时回调点
   - **真正有效的是原子化任务拆分** — 把任务拆到 2-5 分钟粒度，AI 没有空间"自由发挥"
   - **两个工具的核心代码互不依赖** — OpenSpec 源码里 0 个 superpowers 引用，Superpowers 源码里 0 个 openspec 引用。集成完全依赖社区 schema 的 instruction 文本桥接

**总结对比（2 分钟）**

| 维度 | v1 | v2 |
|------|----|----|
| 任务数 | 粗粒度，混合测试和实现 | 26 个原子任务 |
| TDD 执行 | 完全失败，AI 跳过 RED | 3/4 防护通过 |
| 关键改进 | — | 原子化任务拆分（最高 ROI） |

**素材**：
- 文章 26：`06-OpenSpec-Superpowers协作/OpenSpec加Superpowers-TDD-v2-4层防护叠加实测.md`
- 文章 27：`06-OpenSpec-Superpowers协作/用OpenSpec-Schema把Superpowers-TDD焊进编码工作流.md`

---

#### 环节 5：经验总结（5 分钟）

**目标**：提炼贯穿 4 次分享的核心结论

**内容要点**：

1. **3 个核心结论**：

   | 结论 | 出处 | 含义 |
   |------|------|------|
   | 质量始于规格，而非测试 | 文章 4 | OpenSpec 的核心理念 — 先有可验证的规格，再写代码 |
   | 任务粒度是唯一最高杠杆 | 文章 14、26 | 用 20% 的改动覆盖 80% 的质量提升 |
   | 先跑通再完美，3 条命令就够 | 文章 7 | 降低上手门槛，不要一上来就用 Expanded 模式 |

2. **工具组合的诚实评估**：
   - 目前组合方案不完美（3/4 通过率，2/3 衔接点断裂）
   - 但已经是从"碰运气"到"可控"的巨大飞跃
   - 已知问题都有明确的 workaround（原子任务 + 手动桥接）

**素材**：文章 23《OpenSpec + Superpowers：2 个工具 4 步流程》

> 参考原文：`06-OpenSpec-Superpowers协作/OpenSpec加Superpowers-2个工具4步流程.md`

---

### 关键知识点

1. **OpenSpec 的 `instruction` 是纯文本注入，不是可执行约束** — 这是理解组合方案局限性的关键
2. **原子化任务拆分是 TDD 强制执行的根本手段** — 不是 instruction 措辞，不是 Schema 设计，是任务粒度
3. **两个工具的核心代码互不依赖** — 集成完全依赖 instruction 文本桥接，这决定了组合方案的天花板
4. **OpenSpec 在 propose 阶段有效，在 apply 阶段约束力有限** — 规划工具，不是执行强制器

### 踩坑提醒

- v1 的错误不要重复犯：不要指望在 instruction 里写 TDD 规则就能强制执行
- 衔接断点是已知的：OpenSpec specs → Superpowers TDD 的映射需要手动处理
- 不要过度优化 Schema：第 1 层（原子化任务）的 ROI 远高于其他 3 层

### 带走物

- OpenSpec + Superpowers 联合工作流 Checklist（含已知断点和手动干预点标注）
- 优化版 schema.yaml 模板（文章 27 提供，可直接复制使用）
- TDD 强制执行四层防护速查卡
- v1 vs v2 对比表

---

## 六、附录

### A. 贯穿 4 次分享的 3 个金句

1. **"质量始于规格，而非测试"** — 文章 4，OpenSpec 的核心理念
2. **"任务粒度是唯一的最高杠杆"** — 文章 14、26，用 20% 改动覆盖 80% 质量提升
3. **"先完成再完美，3 条命令就够"** — 文章 7，降低上手门槛

### B. 全部素材索引

#### 分享 1 使用

| 编号 | 文章 | 文件路径 |
|------|------|----------|
| 1 | AI 编程工作流选型：Spec-Kit、OpenSpec、Superpowers 深度对比 | `01-选型与对比/AI编程工作流选型-SpecKit-OpenSpec-Superpowers深度对比.md` |
| 2 | OpenSpec vs Superpowers：2 套 AI 编码工作流怎么选 | `01-选型与对比/OpenSpec-vs-Superpowers-2套AI编码工作流怎么选.md` |
| 3 | OpenSpec 深度解析：最佳实践四步法 | `02-OpenSpec入门/OpenSpec深度解析-最佳实践四步法.md` |
| 5 | OpenSpec 实战：从 0 到 1 构建 AI 原生规范驱动开发工作流 | `02-OpenSpec入门/OpenSpec实战-从0到1构建AI原生规范驱动开发工作流.md` |
| 7 | OpenSpec 最佳实战：3 条命令跑通规范驱动开发全流程 | `02-OpenSpec入门/OpenSpec最佳实战-3条命令跑通规范驱动开发全流程.md` |
| 8 | OpenSpec 最佳实战：3 个命令 4 个制品 4 步闭环 | `02-OpenSpec入门/OpenSpec最佳实战-3个命令4个制品4步闭环.md` |

#### 分享 2 使用

| 编号 | 文章 | 文件路径 |
|------|------|----------|
| 8 | OpenSpec 最佳实战：3 个命令 4 个制品 4 步闭环 | `02-OpenSpec入门/OpenSpec最佳实战-3个命令4个制品4步闭环.md` |
| 9 | OpenSpec 工作流全解析：选对模式开发效率翻倍 | `03-OpenSpec进阶/OpenSpec工作流全解析-选对模式开发效率翻倍.md` |
| 10 | OpenSpec 定制化全攻略：让 AI 开发工作流适配团队 | `03-OpenSpec进阶/OpenSpec定制化全攻略-让AI开发工作流适配团队.md` |
| 12 | OpenSpec 最佳实战：4 步复盘 5 项升级 | `03-OpenSpec进阶/OpenSpec最佳实战-4步复盘5项升级.md` |
| 13 | OpenSpec 最佳实战：3 轮实测验证 5 个质量升级方向 | `03-OpenSpec进阶/OpenSpec最佳实战-3轮实测验证5个质量升级方向.md` |
| 14 | OpenSpec 最佳实战：不改源码只改配置提升代码质量 | `03-OpenSpec进阶/OpenSpec最佳实战-不改源码只改配置提升代码质量.md` |
| 15 | OpenSpec 进阶：从 Core 到 Expanded 7 个命令 | `03-OpenSpec进阶/OpenSpec进阶-从Core到Expanded-7个命令解锁全部工作流.md` |

#### 分享 3 使用

| 编号 | 文章 | 文件路径 |
|------|------|----------|
| 19 | Superpowers 实战指南：7 步流程 + 14 个技能 + 3 条铁律 | `05-Superpowers专题/Superpowers实战指南-7步流程14个技能3条铁律.md` |
| 20 | Superpowers 最佳实战：标准开发 7 步法闭环工作流 | `05-Superpowers专题/Superpowers最佳实战-标准开发7步法闭环工作流.md` |
| 21 | Superpowers 最佳实战：7 阶段工作流交付电商 SKU 库存逻辑 | `05-Superpowers专题/Superpowers最佳实战-7阶段工作流交付电商SKU库存逻辑.md` |
| 22 | Superpowers + PlanningWithFiles：4 个冲突点拆解 | `05-Superpowers专题/Superpowers加PlanningWithFiles-4个冲突点拆解.md` |

#### 分享 4 使用

| 编号 | 文章 | 文件路径 |
|------|------|----------|
| 23 | OpenSpec + Superpowers：2 个工具 4 步流程 | `06-OpenSpec-Superpowers协作/OpenSpec加Superpowers-2个工具4步流程.md` |
| 24 | OpenSpec + Superpowers 协作实战：3 个衔接点断了 2 个 | `06-OpenSpec-Superpowers协作/OpenSpec加Superpowers协作实战-3个衔接点断了2个.md` |
| 25 | OpenSpec + Superpowers：6 步实现 AI 规格 TDD 开发 | `06-OpenSpec-Superpowers协作/OpenSpec加Superpowers-6步实现AI规格TDD开发.md` |
| 26 | OpenSpec + Superpowers TDD v2：4 层防护叠加实测 | `06-OpenSpec-Superpowers协作/OpenSpec加Superpowers-TDD-v2-4层防护叠加实测.md` |
| 27 | 用 OpenSpec Schema 把 Superpowers TDD 焊进编码工作流 | `06-OpenSpec-Superpowers协作/用OpenSpec-Schema把Superpowers-TDD焊进编码工作流.md` |
| 28 | 架构师手册：OpenSpec + Superpowers 打造一人极简开发工作流 | `06-OpenSpec-Superpowers协作/架构师手册-OpenSpec加Superpowers打造一人极简开发工作流.md` |

### C. 全部带走物清单

#### 分享 1

- [ ] OpenSpec 安装命令速查卡
- [ ] 3 条命令速查卡（命令 → 做什么 → 产生什么）
- [ ] 4 个制品模板示例

#### 分享 2

- [ ] 优化版 config.yaml 模板（含 tasks 粒度控制配置）
- [ ] Delta Specs 4 种操作类型速查卡
- [ ] Core vs Expanded 模式选择决策树

#### 分享 3

- [ ] Superpowers 7 步流程速查卡
- [ ] 安装配置 Checklist
- [ ] 3 条铁律卡片

#### 分享 4

- [ ] 联合工作流 Checklist（含已知断点和手动干预点标注）
- [ ] 优化版 schema.yaml 模板
- [ ] TDD 四层防护速查卡
- [ ] v1 vs v2 对比表

### D. 未使用的文章

以下文章未在 4 次分享中直接使用，可作为延伸阅读或 Q&A 参考资料：

| 编号 | 文章 | 文件路径 | 推荐用途 |
|------|------|----------|----------|
| 4 | OpenSpec 深度解析：AI 编程质量实战 5 个关键阶段 | `02-OpenSpec入门/OpenSpec深度解析-AI编程质量实战5个关键阶段.md` | 分享 1 延伸：质量保障框架 |
| 6 | OpenSpec 0.2.0 正式发布：Delta 规范核心功能 | `02-OpenSpec入门/OpenSpec-0.2.0正式发布-Delta规范核心功能.md` | 分享 2 延伸：版本功能详解 |
| 11 | OpenSpec 最佳实战：3 条命令 + Delta Specs | `03-OpenSpec进阶/OpenSpec最佳实战-3条命令加Delta-Specs.md` | 分享 2 延伸：Delta Specs 深入 |
| 16 | OpenSpec propose 扩展命令：4 步开启 6 个扩展命令 | `03-OpenSpec进阶/OpenSpec-propose扩展命令-4步开启6个扩展命令.md` | 分享 2 延伸：Expanded 模式实操 |
| 17 | OpenSpec 项目实战（一）：从零搭建项目骨架 | `04-OpenSpec项目实战/OpenSpec项目实战一-从零搭建项目骨架.md` | 分享 2 延伸：真实项目实录 |
| 18 | OpenSpec 项目实战（二）：工具注册中心 | `04-OpenSpec项目实战/OpenSpec项目实战二-工具注册中心.md` | 分享 2 延伸：模块化架构 |

---

*整理时间：2026-05-17*
