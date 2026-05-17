# OpenSpec 0.2.0 正式发布：Delta 规范核心功能、全新工作流命令及重要变更一览

> 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _70_ 篇，AI 编程最佳实战「2026」系列第 _13_ 篇
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的**技术实践者与开源布道者**！
>
> **Talk is cheap, let's explore。无界探索，有术而行。**

OpenSpec 0.2.0 来了！这次是个大版本更新，带来了 Delta 规范的核心功能、全新的工作流命令、双层 Schema 定制以及 22+ AI 工具集成。

> 补充说明：OpenSpec 0.2.0 已升级为 0.3.0，本文以 0.2.0 版本为基础讲解核心功能和架构，实际使用时请参考最新版本。

![OpenSpec 0.2.0 发布信息图](https://developer.qcloudimg.com/http-save/yehe-10642399/7e393ce4e9df0b1e1aa6cd4db93dbda9.png)

### **1. 版本升级概览：从 0.1.0 到 0.2.0**

#### **为什么需要 0.2.0？**

OpenSpec 0.1.0 是第一个正式版本，它引入了规范驱动开发的基本框架：结构化 Markdown 规范、Delta Spec 机制、验证引擎。但 0.1.0 存在几个明显的不足：

- **没有工作流命令**：创建变更、生成文档、归档等操作需要手动执行，流程不够顺畅
- **Schema 定制能力有限**：只支持包内 Schema，无法在项目或用户级别定制工作流
- **AI 工具集成不够完善**：只支持少数几个 AI 编码工具的命令生成
- **CLI 命令分散**：命令设计不够系统化，缺少状态管理和进度追踪

0.2.0 针对这些痛点进行了全面升级。

#### **核心变化一览**

| 维度 | 0.1.0 | 0.2.0 |
| --- | --- | --- |
| 工作流命令 | 无 | `init`/`new`/`apply`/`archive` 全套 |
| Schema 定制 | 包内 Schema | 三层 Schema 解析（项目/用户/包） |
| AI 工具集成 | 少数工具 | 22+ 工具适配器 |
| CLI 设计 | 基础命令 | 完整的命令体系 + `--json` 输出 |
| Delta Spec | 基础支持 | 完整的操作语义和合并引擎 |
| 验证引擎 | 基础验证 | 多层验证策略 |

### **2. Delta 规范核心功能**

#### **Delta Spec 操作语义**

0.2.0 完善了 Delta Spec 的操作语义，支持四种操作类型：

**ADDED** - 新增行为：

```
## ADDED Requirements

### Requirement: Two-Factor Authentication
系统 MUST 支持 TOTP 双因素认证。

#### Scenario: 2FA Enrollment
- **GIVEN** 用户在设置中启用 2FA
- **WHEN** 用户配置认证器应用
- **THEN** 系统显示 QR 码供扫描
```

**MODIFIED** - 修改行为：

```
## MODIFIED Requirements

### Requirement: Session Expiration
系统 MUST 在 15 分钟无活动后过期会话。
```

**REMOVED** - 弃用行为：

```
## REMOVED Requirements

### Requirement: Legacy Export
**Reason**: 被新的导出系统替代
**Migration**: 使用新端点 /api/v2/export
```

**RENAMED** - 重命名：

```
## RENAMED Requirements

### Requirement: Password Recovery → Password Reset
保留原有需求内容，仅更新标题名称。
```

#### **Delta 合并引擎**

0.2.0 引入了完整的 Delta 合并引擎（Specs Apply），在归档时将 delta specs 合并到主规范。

合并算法的关键特性：

1. **操作顺序保证**：RENAMED -> REMOVED -> MODIFIED -> ADDED
2. **重复检测**：节内不允许重复需求
3. **冲突检测**：跨节不允许语义冲突
4. **原始顺序保持**：合并后的需求保持主规范的原始排列

### **3. 全新工作流命令**

#### **init 命令**

```
openspec init
```

初始化 OpenSpec 项目，扫描已有的 AI 编码工具并生成配置文件。交互式引导，支持：

- 检测已安装的 AI 编码工具
- 生成 `openspec/config.yaml`
- 为检测到的工具生成命令文件

#### **new 命令**

```
openspec new change <name>
```

创建新的变更提案，生成变更目录结构：

```
openspec/changes/<name>/
├── proposal.md
├── specs/
│   └── spec.md
├── design.md
└── tasks.md
```

#### **apply 命令**

```
openspec apply <change-name>
```

实现变更，按 tasks.md 中的任务清单逐步执行。

#### **archive 命令**

```
openspec archive <change-name>
```

归档已完成的变更，执行 Delta 合并，更新主规范文件。

#### **辅助命令**

```
openspec list                    # 列出所有活动变更
openspec status --change <name>  # 查看变更状态
openspec validate                # 验证规范文件
openspec instructions <artifact> # 获取工件生成指令
```

所有命令支持 `--json` 输出模式，方便 AI Agent 和其他工具集成。

### **4. 双层 Schema 定制**

0.2.0 引入了三层 Schema 解析机制：

1. **项目级 Schema**：`<project>/openspec/schemas/<name>/schema.yaml`
2. **用户级 Schema**：`~/.local/share/openspec/schemas/<name>/schema.yaml`
3. **包级 Schema**：`<package>/schemas/<name>/schema.yaml`

解析优先级从高到低：项目级 > 用户级 > 包级。

#### **Schema 管理命令**

```
openspec schema list             # 列出所有可用 Schema
openspec schema init <name>      # 创建新 Schema
openspec schema fork <from> <to> # 复刻现有 Schema
openspec schema validate <name>  # 验证 Schema 配置
openspec schema which <name>     # 查看 Schema 来源
```

### **5. 22+ AI 工具集成**

0.2.0 扩展了 AI 工具适配器，支持 22+ 主流 AI 编码工具：

| 工具 | 适配器 ID | 命令路径 |
| --- | --- | --- |
| Claude Code | claude | `.claude/commands/opsx/...` |
| Cursor | cursor | `.cursor/commands/opsx/...` |
| Windsurf | windsurf | `.windsurf/workflows/...` |
| GitHub Copilot | github-copilot | `.github/prompts/...` |
| Codex | codex | `~/.codex/prompts/...` |
| Gemini CLI | gemini | `.gemini/commands/opsx/...` |
| RooCode | roocode | `.roo/commands/opsx/...` |
| Cline | cline | `.cline/commands/opsx/...` |
| Kilo Code | kilocode | `.kilocode/commands/opsx/...` |
| Kiro | kiro | `.kiro/commands/opsx/...` |
| Trae | trae | `.trae/commands/opsx/...` |
| Augment | augment | `.augment/commands/opsx/...` |
| Aider | aider | `.aider/commands/opsx/...` |
| Copilot Workspace | copilot-workspace | `.github/prompts/...` |
| Amazon Q | amazon-q | `.amazonq/commands/opsx/...` |
| Zed | zed | `.zed/commands/opsx/...` |
| OpenCode | opencode | `.opencode/commands/opsx/...` |
| Continue | continue | `.continue/commands/opsx/...` |
| Amp | amp | `.amp/commands/opsx/...` |
| Goose | goose | `.goose/commands/opsx/...` |
| Cody | cody | `.cody/commands/opsx/...` |
| Jules | jules | `.jules/commands/opsx/...` |

每个工具有独立的适配器，处理文件路径和格式差异。

### **6. 破坏性变更**

#### **命令变更**

- `openspec propose` 已移除，替换为 `openspec new change`
- `openspec ff` 已移除，替换为 `openspec new change --quick`
- 配置文件格式有变化，需要重新运行 `openspec init`

#### **配置迁移**

0.1.0 的配置文件需要手动迁移到 0.2.0 格式：

```
# 0.1.0 格式
schema: spec-driven
context: "Tech stack: TypeScript"

# 0.2.0 格式
schema: spec-driven
context: |
  Tech stack: TypeScript
  Testing: Jest
rules:
  proposal:
    - Include rollback plan
```

#### **目录结构变更**

```
# 0.1.0
openspec/
├── config.yaml
├── proposals/
└── specs/

# 0.2.0
openspec/
├── config.yaml
├── changes/          # 替代 proposals/
│   └── <name>/
│       ├── proposal.md
│       ├── specs/
│       ├── design.md
│       └── tasks.md
└── schemas/          # 新增：自定义 Schema
```

### **7. 升级指南**

#### **从 0.1.0 升级到 0.2.0**

1. 更新包：

```
npm update -g @fission-ai/openspec
```

2. 重新初始化：

```
openspec init
```

3. 迁移配置：手动更新 `openspec/config.yaml` 格式

4. 迁移目录：将 `proposals/` 下的内容移动到 `changes/` 目录

5. 验证安装：

```
openspec --version
openspec validate
```

### **8. 总结**

OpenSpec 0.2.0 是一个里程碑版本，它从基础框架进化为一个完整的工作流系统：

**Delta 规范**：完整的操作语义（ADDED/MODIFIED/REMOVED/RENAMED）和合并引擎，让规范演进变得可追溯。

**工作流命令**：`init`/`new`/`apply`/`archive` 全套命令，覆盖变更管理的完整生命周期。

**Schema 定制**：三层解析机制（项目/用户/包），支持自定义工作流。

**AI 工具集成**：22+ 工具适配器，让 OpenSpec 在各种 AI 编码环境中无缝工作。

如果你已经在使用 0.1.0，建议按照升级指南迁移到 0.2.0。如果你是第一次接触 OpenSpec，直接从 0.2.0 开始，体验完整的规范驱动开发工作流。

---

原始发表：2026-04-09 | 来源：腾讯云开发者社区

来源：https://cloud.tencent.com/developer/article/2661648
