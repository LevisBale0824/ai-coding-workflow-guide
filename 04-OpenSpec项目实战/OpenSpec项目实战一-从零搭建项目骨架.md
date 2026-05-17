# OpenSpec 项目实战（一） | 从零搭建项目骨架：OpenSpec 工作流跑通全流程实录

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _109_ 篇，OpenSpec 最佳实战「2026」系列第 _1_ 篇
>
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
>
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的**技术实践者与开源布道者**！

## 1. 从方法论到实战

上期讲完方法论——2/8 法则、三步配置、6 个 Phase 的日常工作流。结论很明确：**改 tasks 的 instruction 这一段配置文本，投入 20%，覆盖 80% 的质量提升。**

但方法论终归是纸上谈兵。tasks instruction 升级之后，AI 生成的 tasks.md 真的能达到 2-5 分钟粒度吗？propose 一次产出 5 个工件，每个都是什么样子？apply 阶段 AI 还会夹带私货吗？

这期用一个真实项目把这些问题回答掉。项目名叫 **shuge AI Toolbox**——一个 AI 工具集合平台，纯前端，技术栈是 React 19 + TypeScript + Vite + Tailwind CSS 4。change name 叫 `project-init`，做的事情很简单：搭项目骨架、配好 OpenSpec 工作流、提交 GitHub。

完整流程：

```
Explore  →  Propose  →  Review  →  Apply   →  验证    →  Archive
  ↓           ↓          ↓         ↓           ↓          ↓
 澄清       生成       人工检查   按任务       浏览器     归档
 需求       5 工件     1-2 分钟   实现        确认       change
```

## 2. 探索：用 Explore 澄清需求

### project-init 也需要 Explore？

项目初始化看起来是 `npm create vite@latest` 一行命令的事，有什么好探索的？

实际上有几个关键决策如果不提前想清楚，propose 阶段 AI 会自行做主：

- 技术选型：React 还是 Vue？用什么 UI 库？需不需要后端？
- 目录结构：用默认的 `src/components/` 平铺结构，还是参考 Creator-Toolbox 的模块化架构？
- MVP 边界：第 2 期做到什么程度就停下？

### Explore 是多轮交互，不是一次输出

**第 1 轮：你说需求**

> 项目名：shuge AI Toolbox
> 定位：AI 工具集合平台，前端项目，工具数据暂时硬编码
> 参考：Creator-Toolbox 的模块化架构
> 技术偏好：React 19 + TypeScript + Vite + Tailwind CSS
> MVP 目标：项目骨架能跑 + OpenSpec 工作流配置好 + 提交 GitHub

**第 2 轮：AI 追问细节**

AI 问了 4 个深挖问题：模块化方式、工具数据存储、骨架范围、GitHub 配置。

**第 3 轮：AI 追加设计决策**

关于 components/ 目录位置等问题。

### Explore 最终输出的决策结论

**技术栈确认：**

| Layer | Choice |
| --- | --- |
| Framework | React 19 + TypeScript (strict) |
| Build | Vite |
| Styling | Tailwind CSS |
| Routing | React Router（动态，registry 驱动） |
| Data | TS 硬编码数组 + ToolManifest 接口抽象 |

**目录结构设计：**

```
src/
  ├── app/
  │   ├── layouts/
  │   └── views/
  │       ├── Home.tsx
  │       └── NotFound.tsx
  ├── modules/               # .gitkeep — Phase 3
  ├── router/
  │   └── index.ts
  ├── lib/                   # .gitkeep
  └── tool-registry/
      └── catalog.ts
```

## 3. 提议：propose 生成 5 个工件

执行 `/opsx:propose`，AI 按依赖顺序逐个生成 5 个工件：

| Artifact | Path | Status |
|----------|------|--------|
| proposal | `proposal.md` | ✅ |
| design | `design.md` | ✅ |
| specs | `specs/project-scaffold/spec.md` | ✅ |
| review | `review.md` | ✅ |
| tasks | `tasks.md` | ✅ |

### specs/：7 个 Requirement

```
### Requirement: 项目使用 Vite + React + TypeScript 脚手架初始化
### Requirement: 目录结构包含 app/views、modules、router、lib、tool-registry
### Requirement: 首页路由 / 显示项目名称 "shuge AI Toolbox"
### Requirement: 404 fallback 路由显示页面不存在提示
### Requirement: tool-registry 包含 ToolManifest 接口和 getTools 查询函数
### Requirement: OpenSpec 工作流配置完成（config.yaml + schema）
### Requirement: Git 仓库初始化并推送到 GitHub
```

### tasks.md：8 个任务组、33 个子任务

```
- [ ] 1. 项目脚手架初始化（8 steps）
- [ ] 2. 目录结构创建（6 steps）
- [ ] 3. 工具注册中心接口（1 step）
- [ ] 4. 路由配置（5 steps）
- [ ] 5. 页面组件创建和清理（5 steps）
- [ ] 6. OpenSpec 工作流配置（3 steps）
- [ ] 7. Git 初始化和 GitHub 推送（4 steps）
- [ ] 8. 验证（1 step）
```

## 4. 执行：apply 按任务实现

### 关键实现片段

**工具注册中心 `src/tool-registry/catalog.ts`：**

```typescript
export interface ToolManifest {
  id: string;
  name: string;
  description: string;
  route: string;
  category: string;
}

const tools: ToolManifest[] = [];

export function getTools(): ToolManifest[] {
  return tools;
}
```

**路由配置 `src/router/index.tsx`：**

```typescript
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import Home from '../app/views/Home';
import NotFound from '../app/views/NotFound';

const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '*', element: <NotFound /> },
]);

export default function Router() {
  return <RouterProvider router={router} />;
}
```

### OpenSpec 配置步骤

1. 初始化：`openspec init --tools claude`
2. 手动创建 `openspec/config.yaml`
3. Fork Schema：`openspec schema fork spec-driven with-review`
4. 手动添加 review 工件到 schema.yaml

### 踩坑记录

**踩坑 1：Vite 初始化目录非空** — 建议先 `npx create-vite` 再 `openspec init`

**踩坑 2：GitHub 推送需要预先配置 `gh auth login`**

## 5. 浏览器验证

确认三件事：
- 首页显示 `shuge AI Toolbox` ✅
- 首页显示"暂无工具" ✅
- 访问不存在的路径显示 404 提示 ✅

## 6. 归档：archive

```
## Archive Complete
**Change:** project-init
**Archived to:** `openspec/changes/archive/2026-05-11-project-init/`
```

## 7. 回顾

- Explore 确实有用，把决策提前理清
- propose 一次生成 5 个工件效率很高
- tasks.md 粒度实测通过：8 组 33 个子任务
- 初始化顺序：先 Vite 再 OpenSpec
- GitHub CLI 配一次受用全程

## 8. 下一期预告

第 3 期做工具注册中心（change name: `tool-registry`）。

shuge AI Toolbox 项目代码地址：https://github.com/shuge-x/shuge-ai-toolbox

---

来源：https://cloud.tencent.com/developer/article/2669086
