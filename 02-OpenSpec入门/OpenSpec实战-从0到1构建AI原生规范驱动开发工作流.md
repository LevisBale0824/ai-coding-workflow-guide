# OpenSpec 实战：从 0 到 1 构建 AI 原生规范驱动开发工作流（以 Claude Code 为例）

> 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _69_ 篇，AI 编程最佳实战「2026」系列第 _12_ 篇
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的**技术实践者与开源布道者**！
>
> **Talk is cheap, let's explore。无界探索，有术而行。**

![OpenSpec 实战信息图封面](https://developer.qcloudimg.com/http-save/yehe-10642399/1b6c52e9a4b8bfa5fa6d34e0b3e1c834.png)

前面几篇文章我们深入解析了 OpenSpec 的架构设计、质量保障体系以及和 Superpowers 的对比。这篇文章进入实战模式，手把手教你从零搭建一个完整的 OpenSpec 工作流，以 Claude Code 为例。

### **1. 环境准备**

#### **前提条件**

- Node.js >= 20.19.0
- npm 或 pnpm 包管理器
- Claude Code CLI 已安装并登录
- 一个 Git 仓库（已有代码或空项目均可）

#### **安装 OpenSpec**

```
npm install -g @fission-ai/openspec
```

验证安装：

```
openspec --version
# 应输出 0.3.0 或更高版本
```

### **2. 项目初始化**

#### **Step 1: 进入项目目录**

```
cd your-project
```

#### **Step 2: 运行初始化命令**

```
openspec init
```

OpenSpec 会扫描项目中已有的 AI 编码工具，并生成配置文件。

交互式引导会询问：

1. **项目名称**：输入你的项目名
2. **技术栈**：选择或输入主要技术栈
3. **检测到的 AI 工具**：确认要集成哪些工具
4. **Schema 选择**：选择默认的 `spec-driven` 工作流

#### **Step 3: 查看生成的文件结构**

初始化完成后，项目根目录会生成以下文件：

```
your-project/
├── openspec/
│   ├── config.yaml           # 项目配置文件
│   └── schemas/              # 自定义 Schema（可选）
├── .claude/
│   └── commands/
│       └── opsx/
│           ├── propose.md    # 创建变更提案
│           ├── apply.md      # 实现任务
│           ├── continue.md   # 继续变更
│           ├── verify.md     # 验证实现
│           ├── archive.md    # 归档变更
│           └── ...           # 更多命令
└── ...（项目原有文件）
```

#### **Step 4: 配置项目上下文**

编辑 `openspec/config.yaml`，添加项目特定的信息：

```
# openspec/config.yaml
schema: spec-driven
context: |
  Tech stack: TypeScript, React, Node.js, PostgreSQL
  API style: RESTful, documented in docs/api.md
  Testing: Jest + React Testing Library
  Code style: ESLint + Prettier
rules:
  proposal:
    - Include rollback plan
    - Identify affected teams
  specs:
    - Use Given/When/Then format
    - Reference existing patterns before inventing new ones
  tasks:
    - Each task must specify file paths
    - Each task must include test files
```

### **3. 创建变更提案**

#### **Step 1: 使用 Claude Code 创建提案**

在 Claude Code 中输入：

```
/opsx:propose add-user-profile
```

Claude Code 会自动调用 OpenSpec 的指令系统，生成变更提案。

#### **Step 2: 审查提案内容**

生成的 `openspec/changes/add-user-profile/proposal.md`：

```
# 用户档案功能提案

## Why
当前系统缺少用户个人资料管理功能，用户无法维护个人信息。
需要添加用户档案功能，允许用户查看和编辑个人资料。

## What Changes
- 添加用户档案页面
- 添加个人资料编辑功能
- 添加头像上传功能

## Capabilities

### New Capabilities
- `user-profile`: 用户档案查看和编辑
- `avatar-upload`: 头像上传和管理

## Impact
- 前端：新增用户档案页面和编辑组件
- 后端：添加用户档案 API 端点
- 数据库：users 表添加 avatar_url 和 bio 字段
```

#### **Step 3: 审查 Delta 规范**

生成的 `openspec/changes/add-user-profile/specs/spec.md`：

```
## ADDED Requirements

### Requirement: User Profile Viewing
系统 SHALL 允许用户查看自己的个人资料，包括用户名、邮箱、头像和简介。

#### Scenario: View Own Profile
- **GIVEN** 用户已登录
- **WHEN** 用户访问个人资料页面
- **THEN** 显示用户名、邮箱、头像和简介

#### Scenario: View Other User Profile
- **GIVEN** 用户已登录
- **WHEN** 用户访问其他用户的资料页面
- **THEN** 显示用户名、头像和简介（不显示邮箱）

### Requirement: Profile Editing
系统 SHALL 允许用户编辑自己的简介信息。

#### Scenario: Edit Bio
- **GIVEN** 用户在自己的资料页面
- **WHEN** 用户修改简介并保存
- **THEN** 系统更新简介内容
- **AND** 显示保存成功提示

### Requirement: Avatar Upload
系统 SHALL 允许用户上传头像图片。

#### Scenario: Upload Avatar
- **GIVEN** 用户在资料编辑页面
- **WHEN** 用户选择并上传图片文件
- **THEN** 系统验证图片格式和大小
- **AND** 上传图片并更新头像 URL

#### Scenario: Invalid Image Format
- **GIVEN** 用户在资料编辑页面
- **WHEN** 用户上传非图片文件
- **THEN** 显示格式错误提示
```

#### **Step 4: 审查设计文档**

生成的 `openspec/changes/add-user-profile/design.md`：

```
# 技术设计：用户档案功能

## 架构

### 前端组件
- `UserProfilePage` - 用户档案页面容器
- `ProfileHeader` - 头像和基本信息展示
- `ProfileEditor` - 资料编辑表单
- `AvatarUploader` - 头像上传组件

### 后端 API
- `GET /api/users/:id/profile` - 获取用户资料
- `PUT /api/users/:id/profile` - 更新用户资料
- `POST /api/users/:id/avatar` - 上传头像

### 数据库变更
```sql
ALTER TABLE users ADD COLUMN avatar_url VARCHAR(255);
ALTER TABLE users ADD COLUMN bio TEXT;
```

## 文件变更列表
- 新增：`src/pages/UserProfilePage.tsx`
- 新增：`src/components/ProfileHeader.tsx`
- 新增：`src/components/ProfileEditor.tsx`
- 新增：`src/components/AvatarUploader.tsx`
- 修改：`src/services/api.ts`
- 修改：`migrations/001_add_profile_fields.sql`
```

#### **Step 5: 审查任务清单**

生成的 `openspec/changes/add-user-profile/tasks.md`：

```
# 实现任务：用户档案功能

## 任务列表

### 任务 1：数据库变更
- [ ] 1.1 创建数据库迁移文件
  - 文件：`migrations/001_add_profile_fields.sql`
  - 测试：`migrations/001_add_profile_fields.test.sql`

### 任务 2：后端 API
- [ ] 2.1 实现获取用户资料端点
  - 文件：`src/routes/profile.ts`
  - 测试：`src/routes/profile.test.ts`

- [ ] 2.2 实现更新用户资料端点
  - 文件：`src/routes/profile.ts`
  - 测试：`src/routes/profile.test.ts`

- [ ] 2.3 实现头像上传端点
  - 文件：`src/routes/profile.ts`
  - 测试：`src/routes/profile.test.ts`

### 任务 3：前端组件
- [ ] 3.1 创建 UserProfilePage 组件
  - 文件：`src/pages/UserProfilePage.tsx`
  - 测试：`src/pages/UserProfilePage.test.tsx`

- [ ] 3.2 创建 ProfileHeader 组件
  - 文件：`src/components/ProfileHeader.tsx`
  - 测试：`src/components/ProfileHeader.test.tsx`

- [ ] 3.3 创建 ProfileEditor 组件
  - 文件：`src/components/ProfileEditor.tsx`
  - 测试：`src/components/ProfileEditor.test.tsx`

- [ ] 3.4 创建 AvatarUploader 组件
  - 文件：`src/components/AvatarUploader.tsx`
  - 测试：`src/components/AvatarUploader.test.tsx`

### 任务 4：集成测试
- [ ] 4.1 端到端用户档案流程测试
  - 文件：`e2e/user-profile.spec.ts`

## 完成标准
- [ ] 所有任务完成
- [ ] 所有单元测试通过
- [ ] 端到端测试通过
- [ ] 代码审查通过
```

### **4. 实现任务**

#### **Step 1: 使用 Claude Code 实现任务**

在 Claude Code 中输入：

```
/opsx:apply
```

Claude Code 会读取 tasks.md，按顺序实现每个任务。

#### **Step 2: 监控实现进度**

实现过程中，Claude Code 会逐个完成任务并更新状态：

```
正在实现任务...
✓ 1.1 创建数据库迁移文件
✓ 2.1 实现获取用户资料端点
✓ 2.2 实现更新用户资料端点
✓ 2.3 实现头像上传端点
✓ 3.1 创建 UserProfilePage 组件
✓ 3.2 创建 ProfileHeader 组件
✓ 3.3 创建 ProfileEditor 组件
✓ 3.4 创建 AvatarUploader 组件
✓ 4.1 端到端用户档案流程测试

所有任务完成！
```

#### **Step 3: 验证实现**

在 Claude Code 中输入：

```
/opsx:verify
```

Claude Code 会检查：

1. 所有任务是否已完成
2. 代码是否满足 Delta 规范中的需求
3. 测试是否覆盖所有场景
4. 代码质量是否符合规则

### **5. 归档变更**

#### **Step 1: 执行归档**

在 Claude Code 中输入：

```
/opsx:archive
```

#### **Step 2: 查看归档结果**

```
已归档到 openspec/changes/archive/2026-04-05-add-user-profile/
Delta 规范已合并到主规范。
任务追踪文件已归档。
准备好下一个功能。
```

归档操作会：

1. 将 Delta Spec 合并到主规范文件
2. 将变更目录移动到 archive/ 子目录
3. 更新任务追踪状态

### **6. 常见操作**

#### **继续未完成的变更**

```
/opsx:continue add-user-profile
```

如果上次实现中断了，这个命令会恢复进度，从 tasks.md 中读取未完成的任务继续执行。

#### **列出所有活动变更**

```
openspec list
```

输出：

```
Active Changes:
  - add-user-profile (in progress)
  - fix-login-bug (proposal ready)
```

#### **验证规范文件**

```
openspec validate
```

检查所有规范文件的格式和语义正确性。

### **7. 完整工作流总结**

```
初始化 → 创建提案 → 编写规范 → 实现任务 → 验证 → 归档
   |           |          |          |         |        |
openspec   /opsx:     /opsx:     /opsx:   /opsx:   /opsx:
init       propose    continue   apply    verify   archive
```

每一步的输出都是下一步的输入，形成完整的工作流闭环。

### **总结**

这篇文章通过一个完整的实战案例，展示了如何从零搭建 OpenSpec 工作流。核心步骤：

1. **环境准备**：安装 OpenSpec 和 Claude Code
2. **项目初始化**：运行 `openspec init` 生成配置和命令
3. **创建变更**：用 `/opsx:propose` 创建提案、规范、设计和任务
4. **实现任务**：用 `/opsx:apply` 按任务清单实现代码
5. **验证实现**：用 `/opsx:verify` 检查实现质量
6. **归档变更**：用 `/opsx:archive` 合并规范并归档

这个工作流的核心价值在于：每一步都有明确的文档输出，AI 编码工具按照规范执行，需求变更通过 Delta Spec 管理，实现质量通过验证引擎保障。

如果你是第一次使用 OpenSpec，建议从一个简单的功能开始，按照这个流程走一遍，熟悉后再尝试更复杂的项目。

---

原始发表：2026-04-07 | 来源：腾讯云开发者社区

来源：https://cloud.tencent.com/developer/article/2654292
