# OpenSpec定制化全攻略：让AI开发工作流精准适配你的团队

AI编程时代，最头疼的莫过于"工具不合用"——通用的开发流程套不上团队的独特节奏，AI生成的内容总需要反复调整才能贴合技术栈和规范。而OpenSpec的定制化能力，正是为解决这个痛点而来！它提供了三个层级的定制方案，从基础配置到深度自定义工作流，既能满足普通团队的快速适配需求，也能支撑资深开发者的个性化探索，让这份"AI开发的量体单"精准贴合你的团队开发习惯。

### **为什么要做OpenSpec定制化？**

作为AI时代规范驱动开发的核心工具，OpenSpec的核心价值是让人和AI基于同一套标准工作。但不同团队的技术栈不同、开发流程各异：有的团队主打快速迭代，不需要复杂的设计文档；有的团队注重严谨性，需要增加专属的评审环节；还有的团队有固定的技术规范，希望AI能直接遵循。

如果直接使用默认配置，难免会出现"水土不服"：要么重复输入相同的命令参数，要么AI生成的内容不符合团队约定，反而增加沟通和返工成本。而定制化后的OpenSpec，能让AI精准理解团队的技术语境、遵循专属规则，让规范驱动开发的效率发挥到极致。

### **三个定制层级，适配所有团队需求**

OpenSpec的定制化并非"一刀切"，而是分为**项目配置、自定义模式、全局覆盖**三个层级，从易到难、从局部到全局，你可以根据团队规模和需求灵活选择，不用再为"用不上的功能"浪费精力。

#### **层级1：项目配置（Project Config）—— 多数团队的首选，5分钟快速适配**

这是最基础也最实用的定制方式，通过一个`openspec/config.yaml`文件就能搞定，不用修改复杂的工作流，适合90%的开发团队。核心作用是**设置默认值、注入团队语境、添加专属规则**，让你告别重复操作，让AI"天生懂你团队的规矩"。

##### **三步完成基础配置**

1. **快速初始化**：执行`openspec init`，跟着交互式引导就能生成配置文件，小白也能上手；
2. **手动配置更灵活**：直接创建`openspec/config.yaml`，按如下完整示例填写即可，核心分三部分：

```
# openspec/config.yaml
schema: spec-driven
context: |
  Tech stack: TypeScript, React, Node.js, PostgreSQL
  API style: RESTful, documented in docs/api.md
  Testing: Jest + React Testing Library
  We value backwards compatibility for all public APIs
rules:
  proposal:
    - Include rollback plan
    - Identify affected teams
  specs:
    - Use Given/When/Then format
    - Reference existing patterns before inventing new ones
```

- **默认模式**：设置`schema: spec-driven`，后续创建变更时不用每次加`--schema`参数，少敲代码更高效；
- **团队语境**：写明技术栈、API风格、测试工具等，AI生成内容时会直接参考，比如指定`TypeScript+React+PostgreSQL`，AI就不会生成Python相关代码；
- **专属规则**：为提案、规范等不同文档设置规则，比如要求提案必须包含回滚方案、规范必须用Given/When/Then格式。

3. **立即生效**：配置完成后，后续所有`openspec`命令都会自动读取这些设置，无需额外操作。

##### **配置后有多香？**

- 简化命令：之前创建变更需要敲`openspec new change my-feature --schema spec-driven`，现在只需`openspec new change my-feature`；
- AI精准生成：AI生成任何文档时，会自动将语境和规则注入生成提示，比如生成提案时会主动包含回滚方案和受影响团队，不用再人工提醒修改。

#### **层级2：自定义模式（Custom Schemas）—— 为独特流程打造专属工作流**

如果项目配置满足不了需求，比如你的团队有专属的开发流程（如先调研再提案、必须加评审环节），就可以用自定义模式，打造完全贴合团队的工作流。自定义模式的文件都放在项目的`openspec/schemas/`目录下，和代码一起版本控制，团队成员能同步使用，不怕规则不一致。

##### **两种创建方式，新手老手都适配**

###### **复刻现有模式（推荐新手）：快速改造，不用从零开始**

这是最高效的方式，直接复刻OpenSpec的内置模式，再根据需求修改，执行一条命令就能搞定：

```
openspec schema fork spec-driven my-workflow
```

这条命令会把默认的`spec-driven`模式完整复制到`openspec/schemas/my-workflow/`，生成如下文件结构，你只需按需修改，不用从头写起：

```
openspec/schemas/my-workflow/
├── schema.yaml           # 工作流定义核心文件
└── templates/
    ├── proposal.md       # 提案文档模板
    ├── spec.md           # 规范文档模板
    ├── design.md         # 设计文档模板
    └── tasks.md          # 任务清单模板
```

###### **从零创建模式（适合老手）：完全自定义，无任何束缚**

如果想打造全新的工作流，比如极简的"快速迭代模式"，可以用`openspec schema init`命令：

- 交互式创建：`openspec schema init research-first`，跟着引导设置工作流名称、描述、包含的文档；
- 非交互式创建：直接指定参数，一步到位，示例如下：

```
openspec schema init rapid \
  --description "Rapid iteration workflow" \
  --artifacts "proposal,tasks" \
  --default
```

##### **核心：读懂schema.yaml，掌握工作流定义**

自定义模式的核心是`schema.yaml`文件，它定义了工作流包含哪些文档、文档之间的依赖关系、AI生成文档的指令等，以下是完整的示例配置，关键字段简单易懂：

```
# openspec/schemas/my-workflow/schema.yaml
name: my-workflow
version: 1
description: My team's custom workflow

artifacts:
  - id: proposal
    generates: proposal.md
    description: Initial proposal document
    template: proposal.md
    instruction: |
      Create a proposal that explains WHY this change is needed.
      Focus on the problem, not the solution.
    requires: []

  - id: design
    generates: design.md
    description: Technical design
    template: design.md
    instruction: |
      Create a design document explaining HOW to implement.
    requires:
      - proposal    # 必须先有提案，才能创建设计文档

  - id: tasks
    generates: tasks.md
    description: Implementation checklist
    template: tasks.md
    requires:
      - design

apply:
  requires: [tasks]
  tracks: tasks.md
```

关键字段说明：

- `id`：文档的唯一标识，用于命令和规则配置；
- `generates`：生成的输出文件名，支持通配符如`specs/**/*.md`；
- `template`：关联`templates/`目录下的模板文件；
- `instruction`：给AI的专属生成指令，定义生成要求；
- `requires`：文档依赖项，定义工作流执行顺序。

##### **模板定制：让AI生成的文档格式完全符合要求**

除了工作流，你还能修改`templates/`目录下的markdown模板，以下是`proposal.md`的模板示例，可按需添加章节和引导：

```
<!-- templates/proposal.md -->
## Why
<!-- Explain the motivation for this change. What problem does this solve? -->

## What Changes
<!-- Describe what will change. Be specific about new capabilities or modifications. -->

## Impact
<!-- Affected code, APIs, dependencies, systems -->
```

模板中可包含：需要AI填充的章节标题、给AI的引导式HTML注释、预期格式的示例，AI会严格按照模板结构生成内容，保证团队文档格式统一。

##### **必做：验证自定义模式，避免踩坑**

使用前一定要执行验证命令，检查配置是否存在问题，示例如下：

```
openspec schema validate my-workflow
```

工具会自动检查：`schema.yaml`语法是否正确、所有关联模板是否存在、有无循环依赖、Artifact ID是否有效，有问题会直接给出提示，避免后续使用出故障。

##### **如何使用自定义模式？**

- 临时使用：创建变更时通过参数指定，示例：

```
openspec new change feature --schema my-workflow
```

- 设为默认：在项目配置文件中设置，后续所有操作自动使用，示例：

```
# openspec/config.yaml
schema: my-workflow
```

#### **层级3：全局覆盖（Global Overrides）—— 资深开发者的专属，跨项目共享模式**

这是最高阶的定制方式，适合资深开发者（Power users），可以将自定义模式共享到所有项目中，不用在每个项目里重复创建。

只需将自定义的模式文件放到用户级目录`~/.local/share/openspec/schemas/`，所有本地项目就能直接调用该模式，实现跨项目的工作流共享。

小提醒：虽然全局模式很方便，但更推荐使用项目级模式（`openspec/schemas/`），因为它能和项目代码一起版本控制，团队成员拉取代码后就能同步使用，避免"本地能用，同事用不了"的问题。

### **实用示例+小技巧，玩转OpenSpec定制化**

#### **技巧1：调试模式解析，确认当前使用的模式**

不确定当前项目用的是默认模式、项目自定义模式还是全局模式？用以下命令快速查询：

- 查看指定模式的来源和路径，示例：

```
openspec schema which my-workflow
```

输出示例：
```
Schema: my-workflow
Source: project
Path: /path/to/project/openspec/schemas/my-workflow
```

- 列出所有可用模式，查看完整清单：

```
openspec schema which --all
```

#### **示例1：打造极简快速迭代工作流**

适合小功能开发、BUG修复的极简流程，仅保留提案和任务，去掉复杂的设计、规范环节，完整配置如下：

```
# openspec/schemas/rapid/schema.yaml
name: rapid
version: 1
description: Fast iteration with minimal overhead

artifacts:
  - id: proposal
    generates: proposal.md
    description: Quick proposal
    template: proposal.md
    instruction: |
      Create a brief proposal for this change.
      Focus on what and why, skip detailed specs.
    requires: []

  - id: tasks
    generates: tasks.md
    description: Implementation checklist
    template: tasks.md
    requires: [proposal]

apply:
  requires: [tasks]
  tracks: tasks.md
```

#### **示例2：为默认流程添加评审环节**

适合对代码质量要求高的团队，在默认的`spec-driven`模式基础上，增加预开发评审环节，步骤+配置如下：

1. 先复刻默认模式：

```
openspec schema fork spec-driven with-review
```

2. 编辑`openspec/schemas/with-review/schema.yaml`，添加`review`artifact，并修改`tasks`的依赖：

```
# 新增评审环节
- id: review
  generates: review.md
  description: Pre-implementation review checklist
  template: review.md
  instruction: |
    Create a review checklist based on the design.
    Include security, performance, and testing considerations.
  requires:
    - design

# 修改原有tasks依赖，增加review
- id: tasks
  # ... 保留原有配置 ...
  requires:
    - specs
    - design
    - review    # 现在必须完成评审，才能创建任务
```

3. 在`templates/`目录下新建`review.md`模板，定义评审文档的结构。

### **定制化的核心价值：让规范为团队服务，而非束缚**

很多人觉得"规范=束缚"，但OpenSpec的定制化恰恰打破了这个认知——它不是让团队去适配工具，而是让工具主动适配团队的开发习惯、技术栈和流程特点。

定制后的OpenSpec，能让：

- **普通开发**：不用反复沟通规范，AI生成的内容直接可用，减少返工；
- **团队负责人**：将团队的开发规则沉淀为配置，新人快速上手，避免"人走知识失"；
- **资深开发者**：摆脱通用流程的限制，打造高效的个性化工作流，提升开发效率。

无论是几人的小团队，还是跨部门的大团队，无论是快速迭代的创业项目，还是要求严谨的企业级项目，OpenSpec的定制化能力都能让规范驱动开发的价值最大化，让AI真正成为团队的高效助手，而非"需要反复调教的工具"。

### **最后总结**

OpenSpec的定制化没有门槛，从5分钟搞定的项目配置，到按需打造的自定义工作流，再到跨项目共享的全局覆盖，你可以根据团队需求"按需解锁"。核心记住这几点：

1. 多数团队直接用**项目配置**，快速注入语境和规则，性价比最高；
2. 有独特流程用**自定义模式**，复刻现有模式改造比从零创建更高效；
3. 自定义模式后一定要执行**验证命令**，避免语法和依赖问题；
4. 优先使用**项目级模式**，方便团队同步和版本控制。

现在就打开你的项目，执行`openspec init`开始定制吧，让OpenSpec成为专属于你团队的AI开发"专属助手"！

---

原始发表：2026-05-12 | 来源：腾讯云开发者社区

来源：https://cloud.tencent.com/developer/article/2667733
