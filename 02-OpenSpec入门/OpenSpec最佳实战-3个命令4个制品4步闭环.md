# OpenSpec 最佳实战：3 个命令，4 个制品，4 步闭环一个增量功能开发需求（实战版）

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 _94_ 篇，AI 编程最佳实战「2026」系列第 _23_ 篇
>
> 大家好，欢迎来到 __术哥无界 | ShugeX ｜ 运维有术__。
>
> 我是__术哥__，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、AIOps、Milvus 向量数据库的__技术实践者与开源布道者__！
>
> __Talk is cheap, let's explore。无界探索，有术而行。__

![Image 1: OpenSpec 增量开发工作流概览](https://developer.qcloudimg.com/http-save/10642399/f71230dab37eb048e0b4ef7b61e60027.png)

OpenSpec 增量开发工作流概览

_图 1：OpenSpec 增量开发工作流概览_

项目做了一半，产品经理说要加个__任务优先级__功能。改数据库、改接口、改验证逻辑……牵一发动全身，改漏一个地方就出 bug。

这是开发者的日常。真实项目从来不是"从零开始"的，而是在已有代码上不断堆叠新需求。但大多数开发工具只擅长处理 greenfield（全新项目），对 brownfield（已有项目）的增量场景支持很弱。

上一篇OpenSpec 最佳实战：3 条命令跑通规范驱动开发全流程（实战版）带你从零走通了 OpenSpec 的核心流程。这次换个场景——在已有的 todo-api 项目上，用 OpenSpec 增量添加__任务优先级__功能。你会发现，增量场景才是 OpenSpec 真正的用武之地。

### 你将完成什么

- ✅ 在已有项目上用 `/opsx:propose` 创建增量变更提案
- ✅ 理解 Delta Specs 中 __ADDED__ 和 __MODIFIED__ 的区别
- ✅ 用 `/opsx:apply` 让 AI 按任务清单实现代码
- ✅ 用 `/opsx:archive` 归档变更，Delta Specs 合并到主规范

#### 你需要准备

- 已完成第一篇教程的 todo-api 项目（包含 CRUD 功能）
- OpenSpec CLI 已安装（`npm install -g @fission-ai/openspec@latest`）
- Claude Code 或其他支持 OpenSpec 的 AI 编程助手
- Node.js >= 20.19.0

#### 预计时间

⏱️ 完成本实战大约需要 20 分钟

#### 难度等级

⭐⭐ 进阶级 - 需要先完成第一篇教程

### 快速回顾：第一篇完成了什么

上一篇教程里，我们用 3 条命令从零搭建了 todo-api：

1. `openspec init --tools claude` — 初始化 OpenSpec
2. `/opsx:propose` → `/opsx:apply` → `/opsx:archive` — 完成了 CRUD 变更

现在项目结构是这样的：

```
todo-api/
├── src/
│   ├── todo.js        # Todo 数据模型
│   ├── store.js       # 内存存储
│   ├── app.js         # HTTP 服务
│   └── test.js        # 集成测试
└── openspec/
    ├── specs/
    │   └── todo-crud-api/
    │       └── spec.md          # 主规范（Source of Truth）
    └── changes/
        └── archive/
            └── 2026-04-23-add-todo-crud/  # 已归档的 CRUD 变更
```

注：实际项目中还会有 `.claude/` 目录（AI 编程助手配置）和各 `.openspec.yaml` 文件，此处省略。

`openspec/specs/` 目录下已经有了主规范——它描述了系统__当前的行为__：创建、查询、更新、删除 Todo。

今天的需求是：给 Todo 加一个 `priority` 字段，支持 Low / Medium / High 三级优先级，默认 Medium，还要能按优先级筛选。

### Step 1：用 /opsx:propose 创建增量变更

打开 Claude Code，在 todo-api 项目目录下输入斜杠命令：

```
/opsx:propose add-todo-priority
```

执行后，AI 会弹出 AskUserQuestion 询问你想做什么类型的变更（比如 Priority levels、Priority + deadline 等选项）。你需要用一段需求描述来回答它：

```
给 Todo API 加上任务优先级功能。priority 字段支持 Low / Medium / High 三个值，
默认 Medium。还要能按优先级筛选（GET /todos?priority=High）。
需要修改已有的 Update 接口支持更新 priority。
```

这段描述是整条生成链的输入源头——它直接影响 proposal.md 的生成内容，进而影响后续 Delta Specs、design.md、tasks.md 的生成。__描述越具体，生成的制品质量越高。__

回答后，AI 会自动扫描现有 specs 并生成 4 个制品。它的工作流程是：扫描项目现有的 `openspec/specs/` 目录 → 理解系统当前状态 → 生成 4 个制品。

#### 预期输出

命令执行后，`openspec/changes/` 目录下会新增变更文件夹：

```
openspec/changes/add-todo-priority/
├── proposal.md
├── design.md
├── specs/
│   └── todo-priority/
│       └── spec.md    # Delta Specs
└── tasks.md
```

#### 验证点

✅ 检查 `openspec/changes/add-todo-priority/` 目录已创建

✅ 检查 4 个文件都已生成（proposal.md、design.md、specs/、tasks.md）

#### 关键区别：首次变更 vs 增量变更

上一篇做 CRUD 时，Delta Specs 里全是 __ADDED__——因为项目是空的，所有需求都是新增的。但这次不一样了。来看看 Delta Specs 的内容：

```
## ADDED Requirements

### Requirement: Todo has priority field
Each todo item SHALL have a `priority` field with value
`Low`, `Medium`, or `High`. Default: `Medium`.

#### Scenario: Default priority
- **WHEN** a POST request is sent to `/todos`
  with `{"title": "Buy groceries"}`
- **THEN** the system SHALL create a todo with `priority: "Medium"`

#### Scenario: Explicit priority on create
- **WHEN** a POST request is sent to `/todos`
  with `{"title": "Fix bug", "priority": "High"}`
- **THEN** the system SHALL create a todo with `priority: "High"`

#### Scenario: Invalid priority value
- **WHEN** a POST request is sent to `/todos`
  with `{"title": "Test", "priority": "Critical"}`
- **THEN** the system SHALL return HTTP 400

### Requirement: Filter todos by priority
（筛选需求的场景省略，结构同上）

### Requirement: Update todo priority
（更新优先级需求的场景省略，结构同上）

## MODIFIED Requirements

### Requirement: Update todo
Copy from existing spec:
The system SHALL update a todo via PUT `/todos/{id}`.

#### Scenario: Update title
（已有场景保持不变）

#### Scenario: Update completed status
（已有场景保持不变）

#### Scenario: Update priority  ← 新增场景
- **WHEN** a PUT request is sent to `/todos/{id}`
  with `{"priority": "Low"}`
- **THEN** the system SHALL return HTTP 200
  with `priority: "Low"`

#### Scenario: Update non-existent todo
（已有场景保持不变）

### Requirement: Create todo
Copy from existing spec:
（Successful creation 场景新增了 `priority: "Medium"` 在返回值中）
```

看到了吗？这次 Delta Specs 里出现了两种操作：

- __ADDED__ — 全新的需求（`Todo has priority field`、`Filter todos by priority`、`Update todo priority`）
- __MODIFIED__ — 修改已有需求（`Update todo` 和 `Create todo`，在已有场景基础上扩展了 priority 相关内容）

这就是 Delta Specs 的核心价值：__你不需要通读完整的主规范，只需看增量变化就能理解这次变更的全貌。__官方的说法是 `Review proposals in seconds`。

对比一下首次变更和增量变更的区别：

|  |  |  |
| --- | --- | --- |
| Delta Specs 内容 | 全是 ADDED | ADDED + MODIFIED |
| 审查范围 | 整个系统行为 | 只看变化部分 |
| 主规范状态 | 空 → 有内容 | 有内容 → 更新 |

![Image 2: 首次变更 vs 增量变更 Delta Specs 对比](https://developer.qcloudimg.com/http-save/10642399/081678e859197cb76e3d248b91e3d7b1.png)

首次变更 vs 增量变更 Delta Specs 对比

_图 2：首次变更 vs 增量变更 Delta Specs 对比_

### Step 2：审查 4 个制品

`/opsx:propose` 一次生成了 4 个文件，每个都有明确用途。别急着 apply，先花两分钟扫一遍。

#### proposal.md — 变更提案

回答 4 个问题：__为什么改、改成什么、影响范围、潜在风险__。这是给人类审查用的，AI 编程助手也会读它来理解上下文。

#### specs/ — Delta Specs（上面已解读）

ADDED 描述新增的__任务优先级__和__筛选__功能，MODIFIED 描述对已有__创建接口__和__更新接口__的扩展。

#### design.md — 设计决策

记录关键的技术选择。比如：为什么 priority 用字符串枚举而不是数字？默认值为什么选 Medium 而不是 Low？这些决策在后面维护代码时很有用。

#### tasks.md — 任务清单

类似这样：

```
## 1. Model Changes

- [ ] 1.1 Add priority field to Todo typedef in `src/todo.js`
- [ ] 1.2 Add VALID_PRIORITIES constant and `isValidPriority()` function
- [ ] 1.3 Modify `createTodo()` to accept options object with optional priority

## 2. API Changes - Create

- [ ] 2.1 Modify POST `/todos` handler to accept optional `priority` in body
- [ ] 2.2 Validate priority value against enum, return 400 for invalid values

## 3. API Changes - List

- [ ] 3.1 Modify GET `/todos` handler to check for `priority` query parameter
- [ ] 3.2 Filter results by priority when query param present
- [ ] 3.3 Return 400 for invalid priority filter values

## 4. API Changes - Update

- [ ] 4.1 Modify PUT `/todos/:id` handler to accept `priority` in body
- [ ] 4.2 Validate and apply priority update

## 5. Testing

- [ ] 5.1-5.5 Verify priority scenarios
```

OpenSpec 的 tasks.md 会按功能分组（Model / API / Testing），每组包含细粒度子任务。大任务导致大 diff，拆成可独立验证的步骤才好管理——官方最佳实践里专门提了这点。

实际执行中，AI 可能通过运行现有测试 + curl 手动验证来确认功能，而不一定修改 `test.js` 添加新的测试用例。

#### 审查重点：MODIFIED 需求

增量变更的审查重点是 __MODIFIED__ 部分。这里的 `Update todo` 需求已经在主规范中存在，Delta Specs 只描述了__变化的部分__：从"只能改 title 和 completed"变成"还能改 priority"。`Create todo` 同理，返回值多了 `priority` 字段。

这个设计体现了 OpenSpec 的 __brownfield-first__ 理念——它是为已有项目设计的，不是只适合从零开始的新项目。官方原话：`built for brownfield not just greenfield`。

你在项目中用过类似的增量开发方案吗？欢迎在评论区聊聊你的经验。

### Step 3：用 /opsx:apply 实现代码

审查完制品，确认没问题后，执行 apply：

```
/opsx:apply
```

AI 编程助手会读取变更目录下的所有制品，按 tasks.md 逐条实现。每完成一个任务，就在 tasks.md 里打勾 `[x]`。

#### 预期代码变更

__`src/todo.js` — 新增 priority 字段和验证__

```
const VALID_PRIORITIES = ['Low', 'Medium', 'High'];

function createTodo(title, options = {}) {
  return {
    id: crypto.randomUUID(),
    title,
    completed: false,
    priority: options.priority || 'Medium',
  };
}

function isValidPriority(priority) {
  return VALID_PRIORITIES.includes(priority);
}

// 原有的 isValidTitle 保持不变
function isValidTitle(title) {
  return typeof title === 'string' && title.trim().length > 0;
}

module.exports = {
  createTodo,
  isValidTitle,
  isValidPriority,
  VALID_PRIORITIES,
};
```

__`src/store.js` — 无变化__

store.js 的 `findAll()` 没有变化，筛选逻辑在 app.js 的 GET 路由中处理。

__`src/app.js` — 路由变更（关键片段）__

以下代码使用 Express 风格简化展示。实际项目中使用原生 Node.js http 模块，核心逻辑一致，只是请求解析和响应发送的写法不同。

POST 路由新增 priority 处理：

```
// POST /todos — 创建 todo
app.post('/todos', (req, res) => {
  const { title, priority } = req.body;

  if (!isValidTitle(title)) {
    return res.status(400).json({ error: 'title is required and must be non-empty' });
  }
  if (priority !== undefined && !isValidPriority(priority)) {
    return res.status(400).json({
      error: `priority must be one of: ${VALID_PRIORITIES.join(', ')}`
    });
  }

  const todo = createTodo(title, { priority });
  save(todo);
  res.status(201).json(todo);
});
```

GET 路由新增筛选参数（筛选逻辑在路由中内联处理）：

```
// GET /todos — 查询 todo 列表
app.get('/todos', (req, res) => {
  const { priority } = req.query;

  if (priority !== undefined && !isValidPriority(priority)) {
    return res.status(400).json({
      error: `priority must be one of: ${VALID_PRIORITIES.join(', ')}`
    });
  }

  const todos = findAll();
  if (priority !== undefined) {
    return res.json(todos.filter(t => t.priority === priority));
  }
  res.json(todos);
});
```

PUT 路由支持更新 priority（使用直接赋值方式）：

```
// PUT /todos/:id — 更新 todo
app.put('/todos/:id', (req, res) => {
  if (!exists(req.params.id)) {
    return res.status(404).json({ error: 'Todo not found' });
  }
  const todo = findById(req.params.id);
  const { title, completed, priority } = req.body;

  if (priority !== undefined && !isValidPriority(priority)) {
    return res.status(400).json({
      error: `priority must be one of: ${VALID_PRIORITIES.join(', ')}`
    });
  }

  if (title !== undefined) { todo.title = title; }
  if (completed !== undefined) { todo.completed = Boolean(completed); }
  if (priority !== undefined) { todo.priority = priority; }
  save(todo);
  res.json(todo);
});
```

#### 验证点

实现完成后，可以手动测试关键场景：

```
# 测试 1：创建带优先级的 todo
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Fix bug", "priority": "High"}'

# 预期返回：包含 priority: "High" 的 todo 对象

# 测试 2：不传 priority，默认 Medium
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Buy groceries"}'

# 预期返回：包含 priority: "Medium" 的 todo 对象

# 测试 3：按优先级筛选
curl "http://localhost:3000/todos?priority=High"

# 预期返回：只包含 priority 为 High 的 todo

# 测试 3b：筛选无匹配结果
curl "http://localhost:3000/todos?priority=Low"

# 预期返回：空数组 []

# 测试 4：传入无效优先级
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "priority": "Critical"}'

# 预期返回：400 错误，提示 priority must be one of: Low, Medium, High
```

✅ 如果 4 个测试都符合预期，恭喜你，功能实现正确！

### Step 4：用 /opsx:archive 归档变更

代码写完、测试通过，执行归档：

```
/opsx:archive
```

执行后，AI 会先检查任务完成状态，必要时重新执行未完成的任务，然后再归档。

归档做了一件事：

__将变更目录移入 archive/__

```
openspec/changes/add-todo-priority/
  → openspec/changes/archive/YYYY-MM-DD-add-todo-priority/
```

（日期为归档当天的日期，如 `2026-04-26`）

关于 Delta Specs 合并到主规范——这是 OpenSpec 的设计目标：ADDED 需求追加到主规范，MODIFIED 需求替换主规范中的对应条目。但实际执行中，AI 的合并行为可能因 Delta Specs 的目录结构和命名而异。本次执行中，由于 Delta Specs 在新 capability 目录 `todo-priority/` 下，AI 将其判定为 "new capability only"，未自动合并到已有 `todo-crud-api/` 主规范。后续版本可能会改善这一行为。

#### 验证点

✅ 检查 `openspec/changes/` 下不再有活跃变更（`add-todo-priority` 已移走）

✅ 检查 `openspec/changes/archive/` 下新增了归档目录

✅ 检查 `openspec/specs/` 目录，确认主规范状态（可能已合并，也可能需要手动合并）

![Image 3: 归档前后目录结构对比](https://developer.qcloudimg.com/http-save/10642399/abf18916203e102765efd64d6dcd0e9d.png)

归档前后目录结构对比

_图 4：归档前后目录结构对比_

到这一步，增量变更的完整闭环就跑通了：__propose → apply → archive__。

### 完整流程回顾

整个增量开发流程，其实就是 3 条命令：

```
# 1. 在已有项目上创建增量变更提案
/opsx:propose add-todo-priority

# 2. 审查 4 个制品（proposal、specs、design、tasks）
#    重点看 Delta Specs 的 ADDED 和 MODIFIED

# 3. 按任务清单实现代码
/opsx:apply

# 4. 归档变更，Delta Specs 合并到主规范
/opsx:archive
```

和第一篇的 CRUD 变更相比，命令完全一样。区别在于 Delta Specs 的内容变了——从"全是 ADDED"变成了"ADDED + MODIFIED"。OpenSpec 会自动识别哪些需求是新增的、哪些是对已有的修改。

这就是它和"纯提示"模式的根本区别。纯提示模式下，AI 不知道系统当前是什么状态，每次都得从头解释。而 OpenSpec 的 `specs/` 目录就是项目的__持久化上下文__——AI 读到主规范就知道系统现在是什么样，读到 Delta Specs 就知道你要改成什么样。

![Image 4: 增量开发四步流程](https://developer.qcloudimg.com/http-save/10642399/8b92db8593c3b077317cc9bc3d5df079.png)

增量开发四步流程

_图 3：增量开发四步流程_

### 进阶技巧

#### Delta Specs 三种操作速查

|  |  |  |  |
| --- | --- | --- | --- |
| __ADDED__ | 新增需求 | 追加到主规范 | 加新字段、加新接口、加新功能 |
| __MODIFIED__ | 修改已有需求 | 替换主规范对应条目 | 改验证规则、扩展已有接口 |
| __REMOVED__ | 删除需求 | 从主规范移除 | 废弃功能、删除接口 |

这次实战用到了 ADDED 和 MODIFIED。REMOVED 的场景也很常见——比如项目里有个"记住登录"功能，决定用双因素认证替代它，就可以在 Delta Specs 里标记 `REMOVED: Remember Me`。

#### 增量变更的命名建议

官方推荐用__简短的动作+对象__格式：`add-todo-priority`、`fix-login-timeout`、`remove-legacy-auth`。别用模糊的名称比如 `stuff` 或 `updates`——一个月后翻 archive 目录，你不会记得 `updates` 到底改了什么。

#### 什么时候用 `/opsx:explore`

如果需求不太明确，别急着 propose。先用 `/opsx:explore` 纯思考——不写代码、不改文件，只是和 AI 讨论方案。想清楚后再 propose，生成的制品质量会高很多。官方的说法是：__先对齐再动手__。

#### 长线程的上下文重置

官方最佳实践里有一条容易被忽略的建议：__规划和编码之间重置上下文__。长会话里 AI 会积累很多"噪音"——前面讨论的细节、已经推翻的方案、无关的上下文碎片。propose 完成后，开一个新会话再执行 apply，AI 只读取制品文件，反而效果更好。

这在增量场景里尤其明显。因为 propose 时 AI 需要理解整个系统现状，上下文已经很重了。apply 时只需要关注 tasks.md 里的具体任务，轻装上阵反而更准确。

### 常见问题汇总

#### Q1：apply 过程中某个任务报错了怎么办？

别慌。AI 会暂停并展示选项——你可以选择跳过当前任务、手动修改后继续、或者重新尝试。tasks.md 里已完成的任务会标记 `[x]`，下次 apply 会从第一个未完成的任务开始。

#### Q2：Delta Specs 生成的内容不准确，能手动改吗？

当然可以。OpenSpec 的核心哲学是 __`fluid not rigid`__——所有制品都是普通 Markdown 文件，随时可以手动编辑。改完后 AI 下次读取时就会使用更新后的内容。

#### Q3：增量变更和首次变更的命令有区别吗？

没有。`/opsx:propose`、`/opsx:apply`、`/opsx:archive` 这三条命令在两种场景下完全一样。OpenSpec 会根据项目现有的 `specs/` 内容自动判断哪些是新增、哪些是修改。

#### Q4：每次变更都要归档吗？

建议及时归档。官方最佳实践里强调：保持 `openspec/changes/` 为__活跃工作队列__。完成的变更及时归档，避免变更目录堆积，影响后续 propose 的上下文质量。

#### Q5：多个增量变更有冲突怎么办？

比如你先 propose 了一个 `add-priority` 变更，还没归档，又 propose 了一个 `add-due-date` 变更——两者都要改 `createTodo` 函数。这种情况下，先完成并归档第一个变更，再做第二个。如果已经同时存在了，可以手动审查两个变更的 tasks.md，调整实现顺序避免冲突。

### 总结

两篇教程走下来，你看到的模式是一样的：__3 条命令、4 个制品、1 个闭环__。但从"首次变更"到"增量变更"，OpenSpec 的真正价值才刚刚展现。

Delta Specs 让你__只关注变化__——ADDED 是新做的，MODIFIED 是改过的，REMOVED 是删掉的。审查提案时不用通读几百行规范，几秒钟就能看清变更全貌。归档后自动合并到主规范，项目的 Source of Truth 越来越完整。

增量开发不是从零开始重写，而是在已有基础上精准改动。OpenSpec 的 Delta Specs 就是那把手术刀——精确、可控、可追溯。

说到底，规范驱动开发解决的核心问题不是"怎么写代码"，而是"怎么确保 AI 写的代码是你想要的"。在增量场景里，这个问题更加突出——改错一个已有功能，比没做新功能还严重。OpenSpec 用 Delta Specs 把每次变更的意图、范围、影响都写下来，人和 AI 都有据可查。

下一步你可以试试更复杂的增量场景：比如给 todo-api 加用户认证、加截止日期、或者把内存存储换成数据库。流程不变，还是那 3 条命令。

如果你的同事也在做 AI 辅助开发，转发给他看看这个增量工作流。

__好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！__

来源：https://cloud.tencent.com/developer/article/2663552
