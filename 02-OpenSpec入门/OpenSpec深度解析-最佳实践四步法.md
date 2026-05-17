# OpenSpec 深度解析：最佳实践四步法，让 AI 编码从"黑盒"变"白盒"的规范引擎

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _65_ 篇，AI 编程最佳实战「2026」系列第 _9_ 篇
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的**技术实践者与开源布道者**！
>
> **Talk is cheap, let's explore。无界探索，有术而行。**

![OpenSpec 信息图封面](https://developer.qcloudimg.com/http-save/yehe-10642399/ffe20476f6e83b97a4bea5e65fee4802.png)

用 AI 编程助手写代码时，你可能遇到过这些问题：需求散落在聊天记录里，退出会话就找不到了；每次生成的代码都不一样，难以复现和追溯；团队协作时，其他人不知道 AI 到底改了什么。

OpenSpec 用一套文件化的规范系统解决了这些问题。它在编码之前先建立明确的需求文档，让 AI 按照规范输出可预测的结果。更重要的是，这套系统支持 Claude Code、Cursor、Windsurf 等 22+ AI 编程工具，完全不锁定特定工具。

### **1. OpenSpec 是什么**

OpenSpec 是一个 **AI 原生的规范驱动开发（Spec-Driven Development）系统**。它的核心思路是：在写代码之前，先让 AI 和人达成"要构建什么"的共识。

GitHub 上这个项目有 34,900+ Stars，用 TypeScript 写成（98.7% 代码），支持 Node.js 20.19.0+。它通过三个机制让 AI 编码变得可控：

**第一，文件化需求**。需求不再是聊天记录中的几句话，而是存在项目里的 Markdown 文件。这些文件有明确的结构和格式，AI 可以读取和更新。

**第二，Delta Spec 机制**。改需求时不用重写整个规范文件，只需要描述"什么在变化"——新增、修改、删除或重命名需求。这让规范演进变得清晰可追溯。

**第三，三层验证**。从格式到语义到业务逻辑，OpenSpec 在多个层面验证规范的正确性，确保 AI 理解的和你想表达的是同一回事。

### **2. 底层架构：三层设计**

![OpenSpec 三层架构图](https://developer.qcloudimg.com/http-save/yehe-10642399/e24298f5f9e91f6cec74f0b41e5307c9.png)

OpenSpec 的架构分成三层，每层有明确的职责边界：

#### **CLI 层（命令入口）**

这一层用 Commander.js 实现，提供 15+ 个终端命令。常见的有：

- `openspec init` - 初始化项目，扫描已有 AI 工具并生成配置
- `openspec list` - 列出所有活动的变更提案
- `openspec validate` - 验证规范文件格式
- `openspec archive` - 归档已完成的变更

所有命令支持 `--json` 输出，可以作为其他工具的后端服务。

#### **核心引擎层**

这是 OpenSpec 的技术核心，包含多个独立模块：

**验证引擎**：检查规范文件的格式、语义和业务规则。比如，每个需求必须包含 SHALL 或 MUST 关键字，每个场景必须用 GIVEN-WHEN-THEN 格式。

**Artifact Graph（工件依赖图）**：用有向无环图（DAG）管理文档之间的依赖关系。proposal.md 依赖什么，design.md 又依赖什么，通过拓扑排序确定生成顺序。

**解析器系统**：把 Markdown 文件转换成结构化的 TypeScript 对象。栈式算法解析标题层级，支持嵌套的 Section 树。

**Specs Apply（规范合并引擎）**：归档时将 delta specs 合并到主规范。操作顺序是 RENAMED → REMOVED → MODIFIED → ADDED，确保替换逻辑正确。

#### **Schema 层**

这一层用 YAML 定义工作流。一个 Schema 描述了需要生成哪些文档、文档之间的依赖关系、每个文档的模板和指令。

```
# spec-driven schema (默认工作流)
artifacts:
  - id: proposal
    generates: proposal.md
    requires: []

  - id: specs
    generates: "specs/**/*.md"
    requires: [proposal]

  - id: design
    generates: design.md
    requires: [proposal]

  - id: tasks
    generates: tasks.md
    requires: [specs, design]
```

这种设计让工作流完全可配置。你可以定义自己的 Schema，不需要修改代码。

### **3. 规范定义格式：Spec/Delta Spec/Change/Schema**

OpenSpec 定义了四种核心文件格式，每种都有明确的用途。

#### **Spec（规范文件）**

规范文件用结构化 Markdown 描述系统应该做什么。格式如下：

```
# Authentication Specification

## Purpose
描述系统的认证机制，确保用户身份验证安全可靠。

## Requirements

### Requirement: User Login
系统 SHALL 允许用户使用用户名和密码登录。

#### Scenario: Successful Login
- **GIVEN** 用户输入有效的用户名和密码
- **WHEN** 用户点击登录按钮
- **THEN** 系统返回认证 token
- **AND** 重定向到用户首页

#### Scenario: Failed Login
- **GIVEN** 用户输入错误的密码
- **WHEN** 用户点击登录按钮
- **THEN** 显示错误提示
- **AND** 不返回 token
```

层级语义很清晰：H1 是能力标题，`## Purpose` 描述领域，`## Requirements` 是需求容器，`### Requirement:` 是单个行为需求，`#### Scenario:` 是验证场景。

每个需求必须包含 SHALL 或 MUST 关键字（来自 RFC 2119 规范性强度），每个场景用 GIVEN-WHEN-THEN 格式确保可测试。

#### **Delta Spec（增量规范）**

这是 OpenSpec 的核心创新。改需求时，不是重写整个规范，而是描述变化：

```
## ADDED Requirements

### Requirement: Two-Factor Authentication
系统 MUST 支持 TOTP 双因素认证。

#### Scenario: 2FA Enrollment
- **GIVEN** 用户在设置中启用 2FA
- **WHEN** 用户配置认证器应用
- **THEN** 系统显示 QR 码供扫描

## MODIFIED Requirements

### Requirement: Session Expiration
系统 MUST 在 15 分钟无活动后过期会话。

## REMOVED Requirements

### Requirement: Legacy Export
**Reason**: 被新的导出系统替代
**Migration**: 使用新端点 /api/v2/export
```

![Delta Spec 操作机制图](https://developer.qcloudimg.com/http-save/yehe-10642399/ad80d78ba7db6a0e4fe28832ea7d5129.png)

四种操作语义：

| 操作 | 语义 | 行为 |
| --- | --- | --- |
| ADDED | 新增行为 | 追加到主规范 |
| MODIFIED | 修改行为 | 替换已有需求块 |
| REMOVED | 弃用行为 | 从主规范中删除 |
| RENAMED | 重命名 | 保留内容，更新名称 |

#### **Change（变更提案）**

变更提案描述为什么要改、改什么：

```
## Why
现有会话管理存在安全隐患，需要增加双因素认证。

## What Changes
- 添加 TOTP 双因素认证
- 缩短会话过期时间
- 移除旧的导出功能

## Capabilities
### New Capabilities
- `two-factor-auth`: TOTP 双因素认证支持

### Modified Capabilities
- `session-management`: 会话过期时间从 30 分钟改为 15 分钟

## Impact
- 后端：添加 OTP 生成和验证端点
- 前端：创建 OTP 输入组件
- 数据库：users 表添加 otp_secret 列
```

#### **Schema（工作流定义）**

Schema 定义了整个工作流需要生成哪些文档、依赖关系是什么：

```
name: spec-driven
version: 1
description: Default OpenSpec workflow

artifacts:
  - id: proposal
    generates: proposal.md
    description: Initial proposal document
    template: proposal.md
    requires: []

  - id: specs
    generates: "specs/**/*.md"
    requires: [proposal]

  - id: design
    generates: design.md
    requires: [proposal]

  - id: tasks
    generates: tasks.md
    requires: [specs, design]

apply:
  requires: [tasks]
  tracks: tasks.md
```

Artifact Graph 引擎会根据 `requires` 字段计算拓扑排序，确定文档生成顺序。

### **4. 验证引擎与类型系统**

![验证引擎三层结构图](https://developer.qcloudimg.com/http-save/yehe-10642399/7c3d406d97de20b15ceb40a896a473eb.png)

OpenSpec 的质量保障通过验证引擎和类型系统实现。

#### **验证引擎**

验证引擎提供四个核心验证方法：

**`validateSpec(filePath)`** - 验证规范文件

- 用 MarkdownParser 解析为结构化 Spec 对象
- 通过 Zod SpecSchema 进行类型安全验证
- 应用业务规则（Purpose 最少 50 字符，每个需求最多 500 字符等）

**`validateChange(filePath)`** - 验证变更提案

- 解析 Why/What Changes 章节
- 从 What Changes 或 specs/ 目录提取 deltas
- 验证 delta 完整性和语义正确性

**`validateChangeDeltaSpecs(changeDir)`** - 验证 delta 规范文件

- 遍历 specs/ 下的所有 spec.md 文件
- 使用 parseDeltaSpec() 解析 ADDED/MODIFIED/REMOVED/RENAMED 块
- 检查每个 ADDED/MODIFIED 需求必须包含 SHALL 或 MUST
- 检查每个 ADDED/MODIFIED 需求必须至少有一个场景
- 检查节内不允许重复、跨节不允许冲突

**`validateSpecContent(specName, content)`** - 验证重建后的规范内容

- 用于归档前验证合并后的主规范

验证阈值常量：

| 常量 | 值 | 说明 |
| --- | --- | --- |
| MIN_PURPOSE_LENGTH | 50 | Purpose 最小字符数 |
| MAX_REQUIREMENT_TEXT_LENGTH | 500 | 需求文本最大字符数 |
| MIN_WHY_SECTION_LENGTH | 50 | Why 最小字符数 |
| MAX_WHY_SECTION_LENGTH | 1000 | Why 最大字符数 |
| MAX_DELTAS_PER_CHANGE | 10 | 单个变更最大 delta 数 |

#### **类型系统（Zod Schemas）**

OpenSpec 用 Zod v4 作为运行时类型验证引擎，定义了四层 schema。

**基础层**：

```
const ScenarioSchema = z.object({
  rawText: z.string().min(1),
});

const RequirementSchema = z.object({
  text: z.string().min(1).refine(
    (text) => text.includes('SHALL') || text.includes('MUST'),
    'Requirement must contain SHALL or MUST keyword'
  ),
  scenarios: z.array(ScenarioSchema).min(1),
});
```

**规范层**：

```
const SpecSchema = z.object({
  name: z.string().min(1),
  overview: z.string().min(1),
  requirements: z.array(RequirementSchema).min(1),
  metadata: z.object({
    version: z.string().default('1.0.0'),
    format: z.literal('openspec'),
    sourcePath: z.string().optional(),
  }).optional(),
});
```

**变更层**：

```
const DeltaOperationType = z.enum(['ADDED', 'MODIFIED', 'REMOVED', 'RENAMED']);

const DeltaSchema = z.object({
  spec: z.string().min(1),
  operation: DeltaOperationType,
  description: z.string().min(1),
  requirement: RequirementSchema.optional(),
  requirements: z.array(RequirementSchema).optional(),
  rename: z.object({ from: z.string(), to: z.string() }).optional(),
});

const ChangeSchema = z.object({
  name: z.string().min(1),
  why: z.string().min(50).max(1000),
  whatChanges: z.string().min(1),
  deltas: z.array(DeltaSchema).min(1).max(10),
  metadata: z.object({
    version: z.string().default('1.0.0'),
    format: z.literal('openspec-change'),
    sourcePath: z.string().optional(),
  }).optional(),
});
```

**配置层**：

```
const ProjectConfigSchema = z.object({
  schema: z.string().min(1),
  context: z.string().optional(),              // 50KB 硬限制
  rules: z.record(z.string(), z.array(z.string())).optional(),
});
```

所有 TypeScript 类型通过 `z.infer<>` 从 Zod schema 推导，确保编译时类型和运行时验证的一致性。

### **5. 开发流程实战：Core Profile 和 Extended Profile**

OpenSpec 提供两种工作流：Core Profile（快速路径）和 Extended Profile（细粒度控制）。

#### **Core Profile（快速路径）**

适合新手和小型功能，三个命令搞定：

**Step 1: 创建变更提案**

```
你：/opsx:propose add-dark-mode

AI：已创建 openspec/changes/add-dark-mode/
    ✓ proposal.md — 为什么要做，改什么
    ✓ specs/      — 需求和场景
    ✓ design.md   — 技术方案
    ✓ tasks.md    — 实现清单
    准备好实现了！运行 /opsx:apply
```

生成的 tasks.md：

```
# 实现任务：深色模式

## 任务列表

### 任务 1：组件开发
- [ ] 1.1 创建主题上下文提供者
  - 文件：`src/context/ThemeContext.tsx`
  - 测试：`src/context/ThemeContext.test.tsx`

- [ ] 1.2 创建切换组件
  - 文件：`src/components/ThemeToggle.tsx`
  - 测试：`src/components/ThemeToggle.test.tsx`

### 任务 2：样式实现
- [ ] 2.1 添加 CSS 变量
  - 文件：`src/styles/variables.css`

- [ ] 2.2 连接 localStorage
  - 文件：`src/hooks/useTheme.ts`

## 完成标准
- [ ] 所有任务完成
- [ ] 测试通过
- [ ] 代码审查通过
```

**Step 2: 实现**

```
你：/opsx:apply

AI：正在实现任务...
    ✓ 1.1 添加主题上下文提供者
    ✓ 1.2 创建切换组件
    ✓ 2.1 添加 CSS 变量
    ✓ 2.2 连接 localStorage
    所有任务完成！
```

**Step 3: 归档**

```
你：/opsx:archive

AI：已归档到 openspec/changes/archive/2026-03-27-add-dark-mode/
    规范已更新。准备好下一个功能。
```

#### **Extended Profile（细粒度控制）**

适合大型功能和团队协作，支持更多阶段：

```
/opsx:explore    ← 探索想法
/opsx:new        ← 创建变更脚手架
/opsx:continue   ← 继续未完成的变更
/opsx:ff         ← 快速创建所有规划文档
/opsx:apply      ← 实现任务
/opsx:verify     ← 验证实现是否符合规范
/opsx:sync       ← 同步规范与代码状态
/opsx:archive    ← 归档
```

**探索阶段**：

```
你：/opsx:explore 如何实现深色模式

AI：让我调查一下你当前的 UI 设置...
    [分析代码库]

    我看到你的应用使用 CSS-in-JS。
    有三个主要选项：
    1. CSS 变量 + 主题切换
    2. 主题提供商模式
    3. 本地存储持久化

    哪个方向你感兴趣？
```

**验证阶段**：

```
你：/opsx:verify

AI：验证实现是否符合文档...
    ✓ 检查 tasks.md 中的任务
    ✓ 验证代码实现
    ✓ 检查测试覆盖

    验证通过！所有任务正确实现。
```

Extended Profile 的优势在于每个阶段都可以审查和迭代，适合团队协作场景。

![Extended Profile 工作流图](https://developer.qcloudimg.com/http-save/yehe-10642399/1e3627f49aeab08b7fcfd8c6fb3e99c0.png)

### **6. 核心模块解析**

#### **Artifact Graph（工件依赖图引擎）**

这是 OPSX 工作流的核心引擎，用有向无环图（DAG）管理工件依赖关系。

**拓扑排序算法**：

使用 Kahn's 算法计算构建顺序：

1. 初始化所有工件的入度（依赖数量）
2. 建立反向邻接表（谁依赖谁）
3. 从入度为 0 的根节点开始
4. 每处理一个节点，将其依赖者的入度减 1
5. 新就绪的节点排序后加入队列
6. 确定性排序保证：同一层级的节点按字母序排列

**状态检测**：

对每个工件，检查其生成的文件是否存在。支持 glob 模式（如 `specs/**/*.md`），使用 fast-glob 进行跨平台文件匹配。

#### **命令生成系统**

这是 OpenSpec 跨 AI 工具支持的关键系统，使用注册表 + 适配器模式。

**22 个适配器**：

| 工具 | ID | 路径 |
| --- | --- | --- |
| Claude Code | claude | .claude/commands/opsx/... |
| Cursor | cursor | .cursor/commands/opsx/... |
| Windsurf | windsurf | .windsurf/workflows/... |
| GitHub Copilot | github-copilot | .github/prompts/... |
| Codex | codex | ~/.codex/prompts/...（全局） |
| ... | ... | ... |

每个工具有独立的格式化策略。Claude Code 适配器示例：

```
const claudeAdapter: ToolCommandAdapter = {
  toolId: 'claude',
  getFilePath(commandId: string): string {
    return path.join('.claude', 'commands', 'opsx', `${commandId}.md`);
  },
  formatFile(content: CommandContent): string {
    return `---
name: ${escapeYamlValue(content.name)}
description: ${escapeYamlValue(content.description)}
category: ${escapeYamlValue(content.category)}
tags: ${formatTagsArray(content.tags)}
---

${content.body}
`;
  },
};
```

#### **Specs Apply（规范合并引擎）**

这是归档流程的核心——将 delta specs 合并到主规范。

**合并算法**：

```
async function buildUpdatedSpec(update, changeName): Promise<{rebuilt, counts}> {
  // 1. 解析 delta 规范
  const plan = parseDeltaSpec(changeContent);

  // 2. 预验证：检查节内重复和跨节冲突
  validateNoDuplicates(plan);
  validateNoCrossSectionConflicts(plan);

  // 3. 加载或创建目标规范
  let targetContent = await loadOrCreateTarget();

  // 4. 提取 Requirements 区域并建立名称→块映射
  const parts = extractRequirementsSection(targetContent);
  const nameToBlock = new Map(parts.bodyBlocks);

  // 5. 按顺序应用操作：RENAMED → REMOVED → MODIFIED → ADDED
  applyRenamed(plan.renamed, nameToBlock);
  applyRemoved(plan.removed, nameToBlock);
  applyModified(plan.modified, nameToBlock);
  applyAdded(plan.added, nameToBlock);

  // 6. 重组规范文件（保持原始顺序）
  const rebuilt = recompose(parts, nameToBlock);

  return { rebuilt, counts };
}
```

操作顺序的选择理由：

- RENAMED 先执行 — MODIFIED 需要引用新名称
- REMOVED 在 MODIFIED 之前 — 避免修改后删除
- MODIFIED 在 ADDED 之前 — 确保替换正确

### **7. 关键设计模式**

OpenSpec 使用了多种设计模式来实现灵活性和扩展性。

#### **注册表 + 适配器模式**

命令生成系统用注册表管理 22 个适配器：

```
class CommandAdapterRegistry {
  static register(adapter: ToolCommandAdapter): void
  static get(toolId: string): ToolCommandAdapter | undefined
  static getAll(): ToolCommandAdapter[]
  static has(toolId: string): boolean
}
```

这种设计让添加新工具支持变得简单——只需要实现适配器接口并注册。

#### **策略模式**

验证引擎使用策略模式实现可扩展的验证规则：

```
const validateSpecRules = [
  checkPurposeLength,
  checkRequirementTextLength,
  checkScenariosFormat,
  // ... 可以添加更多规则
];
```

#### **三级缓存模式**

Schema 解析器使用三级解析优先级：

1. **项目级** — `<projectRoot>/openspec/schemas/<name>/schema.yaml`
2. **用户级** — `$XDG_DATA_HOME/openspec/schemas/<name>/schema.yaml`
3. **包级** — `<package>/schemas/<name>/schema.yaml`

这种设计让用户可以在不同层级覆盖默认配置。

#### **构建器模式**

`generateInstructions` 函数用构建器模式分层组装指令：

```
function generateInstructions(context, artifactId, projectRoot): ArtifactInstructions {
  // 1. 加载基础信息
  // 2. 注入项目上下文（来自 config.yaml）
  // 3. 注入工件规则（来自 config.yaml 的 rules 字段）
  // 4. 注入模板内容（来自 schema 的 templates/ 目录）
  // 5. 返回完整的指令对象
}
```

### **8. 最佳实践**

#### **规格编写**

**聚焦 What 而非 How**：描述系统应该做什么，而不是如何实现。

不好的规范：

```
需要实现一个用户登录功能，用 JWT 实现，存到 localStorage 里。
```

好的规范：

```
### Requirement: User Login
用户应能够使用用户名和密码登录系统。

#### Scenario: Successful Login
- **GIVEN** 用户输入有效的用户名和密码
- **WHEN** 用户点击登录按钮
- **THEN** 系统返回认证 token
- **AND** 重定向到用户首页
```

#### **工作流选择**

**Core Profile 适合**：

- 小型功能
- 快速迭代
- 个人项目
- 熟悉 OpenSpec 的团队

**Extended Profile 适合**：

- 大型功能
- 需要逐步审查
- 团队协作
- 新手学习

#### **上下文管理**

**保持上下文清洁**：

- 开始实现前清除上下文
- 定期总结长对话
- 使用文档而非聊天记录作为参考

**恢复进度的技巧**：

```
/opsx:continue [change-name]
```

自动恢复要做的事情，从 tasks.md 中读取进度。

#### **变更管理**

1. **保持变更聚焦** - 每个变更应是一个逻辑单元
2. **清晰命名** - 如 `add-dark-mode`，避免 `feature-1`
3. **及时归档** - 完成的变更及时归档，保持工作区整洁
4. **迭代完善** - 初版规格不必完美，可随迭代补充

### **总结**

OpenSpec 通过文件化规范、Delta Spec 机制和三层验证，让 AI 编码从"黑盒"变成"白盒"。它的核心价值在于：

**可预测性**：规范先行，AI 按照明确的需求输出代码，减少返工。

**可追溯性**：所有需求都存在项目里，退出会话也能找到历史决策。

**工具自由**：支持 22+ AI 工具，完全不锁定特定工具。

**渐进式采用**：不需要一次性迁移所有项目，可以逐步引入规范。

如果你正在用 AI 编程助手，并且遇到需求混乱、输出不可控的问题，OpenSpec 值得一试。从 Core Profile 开始，在项目中运行 `openspec init`，试试用 `/opsx:propose` 创建第一个变更提案。

---

原始发表：2026-03-28 | 来源：腾讯云开发者社区

来源：https://cloud.tencent.com/developer/article/2649108
