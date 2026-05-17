# OpenSpec 项目实战（二） | 工具注册中心：从骨架到模块化架构

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _111_ 篇，OpenSpec 项目实战「2026」系列第 _2_ 篇
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的**技术实践者与开源布道者**！
> **Talk is cheap, let's explore。无界探索，有术而行。**

![封面图 - 工具注册中心架构示意](https://developer.qcloudimg.com/http-save/10642399/f3277ff28046abdd915a4c6630f05cc2.png)

> **说明**：本文内容基于 OpenSpec（Fission-AI/OpenSpec）v1.3.1 和 React 19 + TypeScript + Vite 的实际操作记录整理而成，所有命令和代码均在 shuge AI Toolbox 项目中实际验证。

## 1. 从骨架到架构

第 1 期做完，shuge AI Toolbox 有了一个能跑的项目骨架 - `npm run dev` 启动，首页显示项目名称，404 页面正常跳转。但骨架是空的：`catalog.ts` 里的 `tools` 数组只有一行 `const tools: ToolManifest[] = [];`，路由写死了两条静态规则，首页永远显示"暂无工具"。

这期要做的事情叫**工具注册中心**（change name: `tool-registry`）。一句话需求：catalog 驱动路由生成、Layout 共享布局、首页按分类展示工具、未实现的工具显示占位页。

做完这期，后续加工具只需两步：在 `catalog.ts` 注册 + 在 `modules/` 下实现组件。不用改路由配置，不用改首页布局，不用改任何已有代码。

完整流程和第 1 期一致：

```
Explore  →  Propose  →  Apply  →  Verify  →  Archive
   ↓           ↓          ↓         ↓          ↓
  澄清       生成       按任务      验证       归档
  需求       5 工件      执行       检查      change
```

## 2. Explore：澄清注册中心需求

第 1 期的 `project-init` 只需要回答"用 React 还是 Vue"、"目录怎么分" - Explore 两三轮就搞定了。`tool-registry` 不一样。虽然实际只经过一轮讨论就理清了关键决策点，但它涉及接口设计、分类策略、路由结构、布局方案、占位页逻辑等多个决策点。

### 关键决策点

**ToolManifest 接口：要不要加 stage 字段？**

AI 的回答很明确：**加**。理由是蓝图感很重要 - 用户看到的是"完整平台"而不是"还在做一半"。stage 的值域和路由行为对应关系：

| stage | 首页展示 | 路由行为 |
| --- | --- | --- |
| active | ✅ | → 实际组件 |
| beta | ✅ | → 实际组件（或限制） |
| planned | ✅ | → 占位页 |

**分类策略：怎么给 AI 工具分类？**

最终采用了原方案 - **文本处理 / 数据转换 / 开发工具 / 内容创作**。

**路由结构：catalog 驱动还是静态配置？**

AI 确认 catalog 驱动路由可行，提醒注意动态 import 的边界情况。

**Layout：需不需要共享布局？**

AI 同意共享布局是刚需，Home 也要包 Layout。

## 3. Propose：5 个工件产出

Explore 结束后执行 `/opsx:propose`，AI 按 `with-review` schema 的依赖顺序生成 5 个工件：`proposal.md` → `design.md` → `specs/` → `review.md` → `tasks.md`。

| Artifact | Path | 描述 |
| --- | --- | --- |
| **proposal.md** | `openspec/changes/tool-registry/proposal.md` | Why + What Changes + Capabilities |
| **design.md** | `openspec/changes/tool-registry/design.md` | 6 个技术决策，含 alternatives considered |
| **specs** | `openspec/changes/tool-registry/specs/**/*.md` | 3 个 capability |
| **review.md** | `openspec/changes/tool-registry/review.md` | 五维度审查 |
| **tasks.md** | `openspec/changes/tool-registry/tasks.md` | 8 个任务组，~38 个 step |

### design.md：核心设计决策

**ToolManifest 扩展字段：** 在第 1 期的 5 个字段（`id`、`name`、`route`、`category`、`description`）基础上，新增 `stage` 字段。

```typescript
export interface ToolManifest {
  id: string;
  name: string;
  route: string;
  category: string;
  description: string;
  stage: 'active' | 'beta' | 'planned';
}

const tools: ToolManifest[] = [
  {
    id: 'text-summary',
    name: '文本摘要',
    route: '/tools/text-summary',
    category: '文本处理',
    description: '快速提取长文本的核心观点',
    stage: 'active',
  },
  // ...更多工具
];
```

## 4. Apply：按任务实现

执行 `/opsx:apply tool-registry`，AI 读取所有工件，按 tasks.md 顺序逐个实现。

### 文件变更概览

Apply 完成后，本期新增/修改了 16 个文件：

| 操作 | 文件路径 | 说明 |
| --- | --- | --- |
| 修改 | `src/tool-registry/catalog.ts` | 扩展接口 + 填充工具数据 |
| 新增 | `src/tool-registry/catalog.test.ts` | catalog 查询函数测试 |
| 新增 | `src/layout/Layout.tsx` | 共享布局组件 |
| 新增 | `src/layout/Layout.test.tsx` | Layout 测试 |
| 新增 | `src/layout/TopNav.tsx` | 顶部导航组件 |
| 新增 | `src/layout/TopNav.test.tsx` | TopNav 测试 |
| 修改 | `src/router/index.tsx` | catalog 驱动动态路由 |
| 新增 | `src/router/index.test.tsx` | 路由测试 |
| 修改 | `src/app/views/Home.tsx` | 按分类展示工具卡片 |
| 新增 | `src/app/views/Home.test.tsx` | Home 测试 |
| 新增 | `src/app/views/PlaceholderPage.tsx` | planned 工具占位页 |
| 新增 | `src/app/views/PlaceholderPage.test.tsx` | 占位页测试 |
| 新增 | `scripts/validate-catalog.ts` | 构建时校验脚本 |
| 修改 | `vite.config.ts` | 添加 Vitest 配置 + 路径别名 |
| 修改 | `tsconfig.app.json` | 添加 paths 别名 |
| 新增 | `src/test-setup.ts` | 测试 setup 文件 |

### 关键实现

**router/index.tsx - 从静态到动态：**

```typescript
import { lazy, Suspense } from 'react';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { getTools } from '../tool-registry/catalog';
import Layout from '../layout/Layout';
import Home from '../app/views/Home';
import NotFound from '../app/views/NotFound';
import PlaceholderPage from '../app/views/PlaceholderPage';

const tools = getTools();

const toolRoutes = tools.map((tool) => ({
  path: `/tools/${tool.id}`,
  element: tool.stage === 'planned' ? (
    <PlaceholderPage tool={tool} />
  ) : (
    <Suspense fallback={<div className="p-4">加载中...</div>}>
      <LazyTool tool={tool} />
    </Suspense>
  ),
}));

function LazyTool({ tool }: { tool: (typeof tools)[number] }) {
  const ToolComponent = lazy(() => import(`../modules/${tool.id}/index.tsx`));
  return <ToolComponent />;
}

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout><Home /></Layout>,
  },
  ...toolRoutes,
  {
    path: '*',
    element: <Layout><NotFound /></Layout>,
  },
]);
```

**Layout 组件：**

```tsx
// src/layout/Layout.tsx
import TopNav from './TopNav';

interface LayoutProps {
  children: React.ReactNode;
}

export default function Layout({ children }: LayoutProps) {
  return (
    <div className="min-h-screen flex flex-col">
      <TopNav />
      <main className="flex-1 px-6 py-4">
        {children}
      </main>
    </div>
  );
}
```

## 5. Verify：一致性检查

OpenSpec 默认安装**不含 verify 步骤**。这期补上了。

启用 verify 需要三步：

```bash
# 第一步：切换到 custom profile
openspec config set profile custom

# 第二步：编辑 ~/.config/openspec/config.json
# 在 workflows 数组中加 "verify"

# 第三步：更新插件
openspec update
```

### 第一次 verify：发现问题

执行 `/opsx:verify`，发现了一个 CRITICAL 和几个 WARNING：

- CRITICAL：task 7.5 的 git commit 没执行
- 3 处编译错误（import 路径、配置问题）

### 第二次 verify：全部通过

修复后再次执行 `/opsx:verify`：

```
| Dimension    | Status                        |
|--------------|-------------------------------|
| Completeness | 38/38 tasks ✓                 |
| Correctness  | 9/9 requirements covered      |
| Coherence    | All design decisions followed |
```

### 构建验证

```bash
npm run build
# ✓ 31 modules transformed
# ✓ built in 121ms
```

## 6. Archive：归档

```
Archive Complete
Change: tool-registry
Schema: with-review
Archived to: openspec/changes/archive/2026-05-14-tool-registry/
```

## 7. 回顾：本期学到了什么

### 和第 1 期的复杂度对比

| 维度 | 第 1 期（project-init） | 第 2 期（tool-registry） |
| --- | --- | --- |
| Explore 轮次 | 2-3 轮 | 2 轮 |
| 决策点数量 | 4-5 个 | 7-8 个 |
| tasks 任务组 | 8 组 33 个子任务 | 8 组 38 个子任务 |
| 涉及文件变更 | 约 10 个文件 | 16 个文件（含 6 个测试文件） |
| apply 执行时间 | 约 20 分钟 | 约 30-40 分钟 |

### 核心收获

5 个工件把"为什么做"、"做什么"、"怎么做"、"做得好不好"、"按什么顺序做"全部记录下来了。三个月后回来看，每个设计决策都能在 `design.md` 里找到理由。

verify 的实战价值：第一次 verify 发现了 CRITICAL 问题和 3 处编译错误。如果不 verify，这些错误会直接进入 archive。

**源头控制是核心防线。** Explore 把决策理清，design 把方案定准，tasks 自然就细了。verify 是安全网，但不是主力。

## 8. 下一期预告

第 3 期做工具市场（change name: `tool-market`）。

shuge AI Toolbox 项目代码地址：https://github.com/shuge-x/shuge-ai-toolbox

---

来源：https://cloud.tencent.com/developer/article/2669087
