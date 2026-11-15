# React 18 + TypeScript 综合案例：从零搭一个 ToDoList —— 详细学习笔记

> 基于 B 站视频《20分钟学会React TypeScript案例 2023最后一更 让我们相约2024 大学生学习前端全栈必会技能 ts next.js》整理。
> UP 主：吴悠讲编程 ｜ 时长 **26:14** ｜ 发布 2023-09 ｜ 合集《1小时学会React18》第 4 集（共 4 集，**最后一集**）
> 视频地址：https://www.bilibili.com/video/BV1WC4y1B7Uq/
>
> **本文怎么来的**：完整音频（14.4 MB）做语音转写（带时间戳），按视频**原本的动手顺序**重构成文档。转写是同音错字极多的口语稿（"NPXCcreate杠next杠AP"→`npx create-next-app`、"tabs.ts"→`types.ts`、"普渡"→`Todo`、"力"→`li`、"范型"→泛型、"泡普斯"→`props`、"优思"→`useState`、"use clint"→`'use client'`、"土豆MVC"→TodoMVC），已按上下文逐处修正；代码按口述+演示还原补全为**可直接运行**的版本。
>
> **这是增量文档**：已有《React新手学习笔记-95%核心知识.md》、《前端学习路线-从零到React.md》，本系列 `EP01`/`EP02`/`EP03`。重叠处一句带过并注明出处，讲法不同处讲透，差异见[第 17 章](#17-与已有笔记的差异)。
>
> **⚠️ 两个前提**：① 26:14 里有约 2 分钟（3:51–5:50）是 AI 工具广告；② 视频敲的是 `npx create-next-app@latest`，**这本质上是 Next.js（App Router）项目**，直接导致第 11 章那个 `'use client'` 报错——前 3 集用 CRA，根本遇不到。

---

## 目录

- [0. 课程地图：这一集不教知识，教组装](#0-课程地图这一集不教知识教组装)
- [1. 案例长什么样（0:00–1:47）](#1-案例长什么样000147)
- [2. 第 1 步：建项目、清模板（1:47–3:18）](#2-第-1-步建项目清模板147318)
- [3. 插播：AWS CodeWhisperer（3:51–5:50）](#3-插播aws-codewhisperer351550)
- [4. 第 2 步：拆组件、建 4 个空文件（5:50–9:13）](#4-第-2-步拆组件建-4-个空文件550913)
- [5. 第 3 步：组装 page.tsx（9:13–10:18）](#5-第-3-步组装-pagetsx9131018)
- [6. 第 4 步：定数据模型与泛型（10:18–12:47）](#6-第-4-步定数据模型与泛型10181247)
- [7. 第 5 步：四个处理函数（12:47–17:43）](#7-第-5-步四个处理函数12471743)
- [8. 第 6 步：AddTodo（17:43–20:52）](#8-第-6-步addtodo17432052)
- [9. 第 7 步：TodoList 与 TodoItem（20:52–25:16）](#9-第-7-步todolist-与-todoitem20522516)
- [10. 第 8 步：TodoFilter（25:16–26:02）](#10-第-8-步todofilter25162602)
- [11. `'use client'` 到底是怎么回事](#11-use-client-到底是怎么回事)
- [12. 样式怎么处理](#12-样式怎么处理)
- [13. 完整代码：page.tsx 与 types.ts](#13-完整代码pagetsx-与-typests)
- [14. 类型地图](#14-类型地图)
- [15. 速查卡](#15-速查卡)
- [16. 自测题](#16-自测题)
- [17. 📌 与已有笔记的差异](#17-与已有笔记的差异)
- [18. 📌 补充：视频没讲但会踩的坑](#18-补充视频没讲但会踩的坑)
- [19. 📌 四集串联（补充，非视频原意）](#19-四集串联补充非视频原意)
- [20. 复刻清单](#20-复刻清单)
- [21. 一句话总结](#21-一句话总结)
- [附录：本文信息](#附录本文信息)

---

## 0. 课程地图：这一集不教知识，教组装

> "今天我将用几分钟时间带你完成一个 **React + TypeScript 的案例**……它就是**待办事项列表 ToDoList**。"

没有新 Hook、没有新语法，只有**决策**：文件怎么分、类型怎么定、状态放哪、函数怎么传。这些在前三集都是"一句带过"，在这里是全部。

| 时间段 | 内容 |
|---|---|
| 0:00–1:47 | 演示成品：输入 → 添加 → 列表 → 切换/删除 → 筛选 |
| 1:47–3:18 | `create-next-app` 建项目 + 清空模板 |
| 3:51–5:50 | **插播** CodeWhisperer（约 2 分钟，广告） |
| 5:50–9:13 | 划分组件、建 4 个空文件、让 AI 生成骨架 |
| 9:13–12:47 | 组装 `page.tsx`；`types.ts` + `useState<Todo[]>` 泛型（**类型**） |
| 12:47–17:43 | 四个 handler + `filter` 状态（**状态设计**） |
| 17:43–20:52 | AddTodo：form、受控 input、`FormEvent<HTMLFormElement>` |
| 20:52–25:16 | TodoList / TodoItem：`map`、`key`、**漏传 prop 的 bug** |
| 25:16–26:02 | TodoFilter：三个按钮改状态 |
| 26:02–26:14 | 收尾：点赞三连，**没有系列总结** |

| 集数 | 主题 | 这一集用到它吗 |
|---|---|---|
| EP01 | 核心语法、JSX、`useState`、数组状态 | ✅ 全程在用 |
| EP02 | props、子传父回调、Context | ✅ 4 个组件靠 props 串起来 |
| EP03 | 5 个 Hook | ⚠️ **一个都没用**（只在 10:31 提了句"也可以用 `useReducer`"） |

**做个小应用，`useState` + props 就够了**——EP03 那些 Hook 是"需要时才上"的工具，不是"必须用"的流程。

---

## 1. 案例长什么样（0:00–1:47）

| 步骤 | 他做了什么 | 界面变化 | 背后是什么 |
|---|---|---|---|
| 1 | 输入「给女儿买个礼物」 | 文字出现 | 受控组件，输入即状态 |
| 2 | 点「添加事项」 | 列表多一条 | 数组追加一个对象 |
| 3 | 点「切换」 | 文字被**横线划掉** | 该条 `completed` 取反 |
| 4 | 点「删除」 | 那条消失 | 按 id 过滤掉 |
| 5 | 点「未完成」「已完成」「全部」 | 列表按状态筛 | 只改"看哪些"，不改数据 |

功能清单（这就是需求文档）：**F1 新增**（数组 +1 个 `{id, text, completed:false}`）、**F2 切换完成**（该条 `completed` 取反）、**F3 删除**（去掉该 id）、**F4 筛选**（**不改数据**，只改视图）。

**F4 与 F1–F3 有本质区别**，这是全片最重要的一次判断（第 7 章展开）：F1–F3 改**数据**，F4 改**视图**。

> "这块我就采用一个**横线的方式**把它给划掉了，代表它已经完成了。"

这是**唯一一处「完成状态如何被用户看见」**的地方——没有横线，切换等于没生效（第 9 章他真的踩了这个坑）。

**为什么偏偏是 ToDoList**：

> "我们在完成一个**复杂项目**的时候，它无非就是**多个这种类似的功能点对数据和状态的处理**，结合成了一个非常庞大的功能项目而已。"

📌 补充：它在一屏内凑齐五个动作——**读**（列表）、**写**（新增）、**改**（切换）、**删**（移除）、**过滤**（Tab）。真实项目就是这五件事乘以 50，再加权限、分页、请求、错误处理。他明确放弃了两件事：**视觉样式**（"大家可以到 **TodoMVC 的仓库**里面去使用它的样式"，todomvc.com 是"同一应用几十种框架各实现一遍"的对比项目，CSS 可直接扒）、**复杂目录**（"我现在就只有这么一个功能"）。

---

## 2. 第 1 步：建项目、清模板（1:47–3:18）

```bash
npx create-next-app@latest
cd my-app
npm run dev
```

口述里项目名像 "MYapppp"，实际就是 `my-app`（`create-next-app` 不允许大写，会要你改名）。

| 提问 | 他的选择 | 影响 |
|---|---|---|
| 用 **TypeScript**？ | **Yes** | 文件是 `.tsx`，才能写 `interface` |
| 用 **ESLint**？ | **Yes** | 会拦 `any`、未使用变量 |
| 用 **Tailwind**？ | **No**（"我就不做样式处理了"） | 后面全用内联样式 |
| 用 **`src/` 目录**？ | **Yes** | 代码在 `src/` 下 |
| 用 **App Router**？ | **Yes** | 入口是 `src/app/page.tsx`，**并且需要 `'use client'`** |
| 自定义 import 别名？ | **No** | 沿用 `@/*` |

> "我们这块主要看的是 `src` 这个目录……里边呢还有一些像全局的样式文件呀，然后 `layout` 还有配置。"

```
my-app/src/app/
├── layout.tsx      ← 根布局（html/body，全局样式在这引入）
├── page.tsx        ← ★ 首页，主战场
└── globals.css
```

📌 `app/page.tsx` **文件名不能改**——App Router 用文件名当路由约定：`page.tsx` = 该目录的页面，`layout.tsx` = 包住页面的壳。转写稿那句"配置点 TSX"其实是 `page.tsx`（谐音）。和 EP01 的 CRA 结构（`src/App.js`）完全不同。

清空模板，只留标题：

```tsx
// src/app/page.tsx（清空后）
export default function Home() {
  return (
    <div>
      <h1>Todo List</h1>
    </div>
  )
}
```

> "现在页面里边具有很多默认的内容……其他的部分内容呢，我们其实没有什么必要了，把它去掉。"

这一步的意义不是删代码，而是**拿到一个"干净、能跑、有热更新"的基线**——后面每加一块功能，界面立刻能看到变化。

📌 不想用 Next.js，Vite 等价命令是 `npm create vite@latest my-app -- --template react-ts`——区别只有三处：入口 `src/App.tsx`、`@/` 别名要自己配、**不需要 `'use client'`**。本文组件代码原样可用。

---

## 3. 插播：AWS CodeWhisperer（3:51–5:50）

装法：VS Code 扩展商店搜 "AWS"（CodeWhisperer 集成在 **AWS Toolkit** 里）→ 左侧 AWS 图标 → 选 **CodeWhisperer** → **Sign in to get started** → 浏览器弹 `Authorization requested`，确认 → 登录/注册 **AWS Builder ID** → 授权"can now access your data" → 回 VS Code 看到 **Connected to AWS Builder ID** 即可用。他随手写 `function getFourTime()`，AI 自动补出 `new Date()` 的处理——**证明"它会猜你下一行"**。

### 3.1 那句真正有用的话

> "你的角色呢更多就是作为一个**审查者**去看代码，然后做一个判断下决策。"
> "当你的 AI 用熟了以后，你就发现你从一个**运动员**变成了**裁判员**了。"

📌 这个比喻在本片被验证了三次：

| 位置 | AI 给了什么 | 他改成什么 | 谁对 |
|---|---|---|---|
| 7:41 TodoFilter 骨架 | 三个按钮，逻辑对，**缩进乱** | 手动调缩进 | AI 对，但不够"精益求精" |
| 13:00 `addTodo` 参数 | `addTodo(todo: Todo)` 收整对象 | `addTodo(text: string)` 只收文本 | **他对**：输入框只有字符串，对象该由父组件拼 |
| 14:40 `deleteTodo` | 先 `const newTodos` 再 `setTodos`，两句 | 合成一句 | **他对**：没必要多一个中间变量 |

还有一次是 AI **没提醒**、他自己写错的（第 9 章漏传 `todo` prop）——**裁判也会漏判，类型系统才是第二双眼睛。** AI 的参与度：建文件 ❌（"你 AI 不可能帮你把那些文件都给创建了"）、组件骨架 ✅、类型定义 ⚠️（会猜，字段要人定）、handler ✅（`map`/`filter` 写法）、事件类型 ✅（给 `FormEvent`，人加泛型）。**AI 写"怎么写"，人定"写什么"。**

---

## 4. 第 2 步：拆组件、建 4 个空文件（5:50–9:13）

> "就这么几个区域啊，所以现在呢我们就应该去进行一些相应的**组件的划分**。"

| 区域 | 组件 | 职责 |
|---|---|---|
| 上：输入 + 添加 | `AddTodo` | 收集输入，把文本交出去 |
| 中：列表 | `TodoList` | 展示集合，把操作往下传 |
| 中：一行 | `TodoItem` | 显示一条 + 两个按钮 |
| 下：三个按钮 | `TodoFilter` | 改"看哪些" |
| 外层 | `Home`（`page.tsx`） | 持有数据与筛选条件，分发函数 |

> "我们有 `TodoList`……还有像添加的 `AddTodo`……还有 `TodoItem`……最后还有一个我们筛选的功能，`TodoFilter`。"

📌 拆分原则他没明说，但代码泄露了答案——**按「谁会变」拆，不按「谁长什么样」拆**：有自己的局部状态（AddTodo 的输入框）、会被 `map` 出 N 份（TodoItem）、有独立交互（每行两个按钮）→ 拆出去；只是布局容器（外层 `<div>`）→ 不拆。反过来，**若 `TodoItem` 只有一行文字、没有按钮，就不该单独存在**——直接 `li` 写进 `TodoList` 更省事。

```bash
mkdir src/components   # 他说的是 component（单数），惯例是 components
```

四次循环，流程一致：**写 `function Xxx() {` → 等 AI 补全 → 看一眼 → 调缩进 → 补 `export default`。** `TodoFilter` 被补出三个 `<button>`（All / Active / Completed，各占一行），他说"基本结构我觉得是 OK 的"，只是缩进要手动调；`TodoItem` 只留 `li` 骨架，按钮后面加；`TodoList` 直接复制上一个改名（"咱能不能直接复制一下子"）。

> "我这块没有设置那个**保存的时候自动格式化**，我不太喜欢那功能，因为有时候格式化并不是我想要的。"

📌 这是个人偏好。团队项目一般开 format on save（否则 diff 全是空格噪音），开了 Prettier 后 AI 生成的烂缩进会自动修好，他那步"精益求精"就消失了。

---

## 5. 第 3 步：组装 page.tsx（9:13–10:18）

先摆出来、跑通 import，再填内容——**先证明"架子是通的"，再灌逻辑**，出错容易定位。

```tsx
// src/app/page.tsx（组装版，按演示还原）
import AddTodo from '@/components/AddTodo'
import TodoList from '@/components/TodoList'
import TodoFilter from '@/components/TodoFilter'

export default function Home() {
  return (
    <div>
      <h1>Todo List</h1>
      <AddTodo />
      <TodoList />
      <TodoFilter />
    </div>
  )
}
```

注意 **`TodoItem` 没有被 import**：它由 `TodoList` 在 `map` 里使用——这层"谁引用谁"决定了 props 要经过几道手（见 §7.6 的分发图）。

三种 import：**默认导入** `import AddTodo from '@/components/AddTodo'`（组件用 `export default`）、**命名导入** `import { Todo } from '@/types'`、**类型导入** `import type { Todo } from '@/types'`（📌 纯类型这样写更干净，编译后完全消失，团队常用 ESLint 规则 `@typescript-eslint/consistent-type-imports` 强制）。

---

## 6. 第 4 步：定数据模型与泛型（10:18–12:47）

### 6.1 先想数据，再写代码

> "这个事项信息呢，首先它是一个**数组结构**，数组里面呢肯定是一个固定的**对象**，比如它有一些 **ID**，然后有**内容**，**完成状态**。"

```ts
// src/types.ts
export interface Todo {
  id: number
  text: string
  completed: boolean
}

// 📌 补充：视频里筛选类型是内联写在 useState 上的，抽出来更好复用（§7.5）
export type FilterType = 'all' | 'active' | 'completed'
```

| 字段 | 类型 | 为什么 | 谁写入 | 谁读取 |
|---|---|---|---|---|
| `id` | `number` | 唯一标识：`key`、删除、切换定位都靠它 | `addTodo` 的 `Date.now()` | `TodoList` 的 `key`、两个 handler |
| `text` | `string` | 用户输入内容 | 输入框 | `TodoItem` 的 `{todo.text}` |
| `completed` | `boolean` | 只有两态 | 初值 `false`、`toggleTodo` 取反 | 内联样式、筛选 |

### 6.2 为什么单独建 `types.ts`

> "这些信息呢，我们可以在**文档中做一个记录**……我们就直接放在 `src` 目录下……我写一个 `types.ts`。"

关键在于**同一类型被多个文件用**：`page.tsx` 要 `useState<Todo[]>`，`TodoList` 要 `Todo[]`，`TodoItem` 要 `Todo`。**类型也要有单一来源**，否则改一个字段要改三处。所以：跨 3 个以上文件共用的（`Todo`）放 `types.ts`；只被一个组件用的（`AddTodoProps` 等）就写在组件文件里——"因为这个呢，我并不需要去重复用啊，目前并不需要，所以我直接这样写"。

### 6.3 `useState<Todo[]>` —— 尖括号里为什么能写东西

```tsx
const [todos, setTodos] = useState<Todo[]>([])
```

> "很多同学在刚开始使用 TS 的时候可能比较疑惑，就是我这个地方**为什么能写**？它**应该在哪写**？你可以**按住 Ctrl 键**（Mac 是 command），移入到这个函数上……你可以看到 `function useState` 里边是有一个**大 S**，这个大 S 它决定了你的 initial state。"

📌 `Ctrl+点击` 跳到的签名（React 18 的 `@types/react`）大致是：

```ts
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>]
```

| 位置 | 含义 |
|---|---|
| `<S>` | **泛型参数**，一个"待填的类型占位符" |
| `initialState: S \| (() => S)` | 初值要么是 `S`，要么是返回 `S` 的函数（懒初始化） |
| `[S, Dispatch<SetStateAction<S>>]` | 值 + setter，setter 只收 `S` 或 `S => S` |

所以 `useState<Todo[]>([])` 的意思是"**S 我填 `Todo[]`**"，于是 `todos` 是 `Todo[]`，`setTodos` 只接受 `Todo[]`。

> "光写 `Todo` 呢还不行，因为你这个值是一个 `Todo` 组成的**数组**，所以写一个 `Todo[]`。"

新手最容易错的一步：`useState<Todo>` → 单个对象；`useState<Todo[]>` → 对象数组。**差一对方括号，`setTodos([...])` 全红。** 那个 `[]` 是初值；不写泛型时 `useState([])` 会被推断成 `never[]`，之后 `setTodos([{...}])` 直接报错——**这正是"必须显式写泛型"的典型场景**。📌 `useState([] as Todo[])` 也能过，但那是用**类型断言 `as`** 骗编译器；`<Todo[]>` 是**告诉**编译器。写成 `as any` 之后错误就留到运行时了——第 9 章那个 bug 正是如此。

---

## 7. 第 5 步：四个处理函数（12:47–17:43）

### 7.1 先决定：状态有几个，放哪

| 候选 | 是状态？ | 放哪 | 依据 |
|---|---|---|---|
| `todos` | ✅ | `page.tsx` | 多个子组件要读要改，改完要重渲染 |
| `filter` | ✅ | `page.tsx` | 交互会变，且影响展示 |
| 输入框 `text` | ✅ | **`AddTodo` 内部** | 只有它自己用 |
| 筛选后的列表 | ❌ | — | 能由 `todos` + `filter` **算出来** |

> "**而且是交互的时候变，交互的时候变意味着你得有个状态**。"

判断标准就一句：**会随交互变 → 是状态；能从别的状态推出来 → 不是状态。**

📌 状态归属五问（视频没成表）：① 有多个组件要读吗 → 往上提；② 有子组件要改吗 → 往上提并把函数传下去；③ 能算出来吗 → **别建状态**；④ 卸载后还要留吗 → 才考虑 localStorage/后端；⑤ 只是这屏的 UI 细节吗 → 就地放。`text` 就是⑤：只在 `AddTodo` 里活一次，提交完清空。

### 7.2 `addTodo` —— 唯一"造对象"的地方

AI 第一版给 `addTodo(todo: Todo)`，收整对象，他否掉了：

> "我希望的就是你**加一个文本**就行了，因为我们这块是一个**输入框**……然后我在里边 `const` 一个 `newTodo`，然后**我自己拼一个**。"

```tsx
// src/app/page.tsx（片段，按口述还原）
const addTodo = (text: string) => {
  const newTodo: Todo = {
    id: Date.now(),      // ① 时间戳当唯一 id
    text,                // ② 简写属性：text: text
    completed: false,    // ③ 默认没完成
  }
  setTodos([...todos, newTodo])   // ④ 展开旧数组再追加，不 push
}
```

③ 他先说成 true，立刻自我纠正："默认应该是**没完成**对吧"。④ 不能 `push`——`push` 改原数组，React 看不到变化（EP01 §10.2）。📌 为什么参数只收 `text`：拼对象需要知道 id 怎么生成、初值是什么，这些是**应用层知识**，输入框不该知道；它只负责"用户打了几个字"，这也让 `AddTodo` 可复用。

### 7.3 `deleteTodo` —— `filter` 一行

> "那我觉得这 AI 唯一的问题就是呢，它其实**没有必要写成两句**。"

```tsx
const deleteTodo = (id: number) => {
  setTodos(todos.filter((todo) => todo.id !== id))
}
```

`filter` 保留返回 `true` 的项，所以条件写 `!==`——**留下不是要删的那个**。

### 7.4 `toggleTodo` —— `map` + 三元

> "`todos.map` 遍历，然后**找到一样 ID 的，把它那个 completed 设成反向的**。对了，切换是没错的。"

```tsx
const toggleTodo = (id: number) => {
  setTodos(
    todos.map((todo) =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  )
}
```

| 动作 | 代码 | 为什么 |
|---|---|---|
| 遍历全部 | `todos.map` | 返回**新数组**，长度不变 |
| 命中判断 | `todo.id === id` | 用 id 不用下标——下标会因筛选/删除错位 |
| 命中造新对象 | `{ ...todo, completed: !... }` | 直接改字段引用没变，React 认为没变 |
| 未命中原样返回 | `: todo` | 引用不变，配合 `memo` 可跳过渲染 |

### 7.5 `filter` 状态 + `getFilteredTodos`

```tsx
const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all')
```

> "它其实就是个**标记**啊，是个类型，比如你叫做 `all` 全部，或者说是 `completed`，或者 `active` 就激活的这种状态。"

这是**联合类型**：只能取这三个字符串之一。写了之后 `setFilter('done')` 立刻报错，编辑器给自动补全，不需要 `enum`。筛选函数 AI 写了三个分支，他删了 `all` 那个：

> "这个 **O 不要了**（指 `all` 分支）。因为它就我就**直接取默认值**就 OK 了，代码我也不会写别的值。"

```tsx
const getFilteredTodos = () => {
  if (filter === 'completed') return todos.filter((todo) => todo.completed)
  if (filter === 'active') return todos.filter((todo) => !todo.completed)
  return todos          // 剩下的就是 'all'
}
```

> "因为 filter 方法的返回值是 **true 才能取**嘛……默认是不是 false 呀？所以**非 false 的时候就表示是 active**。"

他踩了自己挖的坑并当场爬出来：`addTodo` 里 `completed` 默认 `false`，所以"未完成"判断是 `!todo.completed`。

**为什么不存状态？** 它随时能从 `todos` + `filter` 算出来。存了就要处理"两份数据同步"：删一条，`filteredTodos` 也得跟着变——多一个状态，就多一个忘记同步的地方。📌 这叫**派生状态**，React 文档 "You Might Not Need an Effect" 讲的就是"能算出来的别存"。

### 7.6 分发图

```
page.tsx（持有 todos + filter）
├── addTodo(text)      → AddTodo（直接传）
├── deleteTodo(id)     → TodoList → TodoItem（透传两层）
├── toggleTodo(id)     → TodoList → TodoItem（透传两层）
├── getFilteredTodos() → TodoList（自己调用，结果当 todos 传）
└── setFilter          → TodoFilter（直接传 setter）
```

---

## 8. 第 6 步：AddTodo（17:43–20:52）

> "我这至少得是一个 `form`……里边呢有一个输入框 `input`，有 `input` 呢，它肯定就会有一个**受控组件**的概念，然后有一个 `button`，点击以后是新建事项。"

```tsx
// src/components/AddTodo.tsx（按口述+演示还原补全）
'use client'

import { useState, type FormEvent } from 'react'

interface AddTodoProps {
  addTodo: (text: string) => void
}

export default function AddTodo({ addTodo }: AddTodoProps) {
  const [text, setText] = useState('')

  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    if (!text.trim()) return
    addTodo(text)
    setText('')
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="请输入待办事项"
      />
      <button type="submit">添加事项</button>
    </form>
  )
}
```

### 8.1 受控组件的两个必写项

> "我就把它的 `value` 给绑了……然后你 `onChange` 的时候……**你只要内容一变，我就把 DOM 元素的值取出来，用 `setText` 设置给我们这个状态**。"

只给 `value` 不给 `onChange` → 变只读，控制台警告 *You provided a `value` prop to a form field without an `onChange` handler*；只给 `onChange` 不给 `value` → 变非受控，状态里永远 `''`，界面打的字进不了 React。📌 `e.target.value` 是 `string`，这是**自动推导**——`onChange` 形参被推断为 `ChangeEvent<HTMLInputElement>`，所以不用手写。而 `handleSubmit` 是独立声明的函数，没有上下文可推导，必须手写。

### 8.2 事件类型：为什么要写两层尖括号

> "这给我们写了一个类型对吧？这是我们那个 form 的事件对象。然后这一块你可以**再写一个泛型更细致一些**，比如写一个 `HTMLFormElement`。"

| 写法 | 后果 |
|---|---|
| `(e) => ...` | 隐式 any，`strict` 下报错 |
| `(e: FormEvent) => ...` | 能跑，但 `e.currentTarget` 只是 `Element` |
| `(e: FormEvent<HTMLFormElement>)` | ✅ `currentTarget` 有完整提示 |

📌 泛型实参表示"**事件发生在哪种元素上**"：`FormEvent<HTMLFormElement>`、`ChangeEvent<HTMLInputElement>`、`MouseEvent<HTMLButtonElement>`、`KeyboardEvent<HTMLInputElement>`。注意 `e.target` 是"真正触发的元素"（可能是子元素，类型宽），`e.currentTarget` 是"绑监听器的元素"（就是泛型实参）——**要用元素方法一律走 `currentTarget`。**

### 8.3 `preventDefault` 与那三个判断

> "首先第一个，我要**阻止一下他的默认行为**，我并不希望你真的去做什么提交……然后判断一下，比如你这个 `text` 有没有内容，`trim` 一下。"

| 行 | 作用 | 不写会怎样 |
|---|---|---|
| `e.preventDefault()` | 阻止 form 默认提交 | 页面闪一下刷新，状态全丢 |
| `if (!text.trim()) return` | 空白不提交 | 列表出现空白条目 |
| `addTodo(text)` + `setText('')` | 交出去 + 清空（"为了下次输入更顺畅"） | 没反应 / 文字还留着 |

📌 他提到"也可以直接找 `button` 来一个 `click`"，这是真实取舍：表单提交**回车也能提交**、语义正确，但必须 `preventDefault`；按钮点击简单，但回车不提交，且 `button` 在 form 内**默认就是 submit**、可能意外触发。他选前者是对的——输入框 + 回车提交是待办类应用的肌肉记忆。

### 8.4 props 类型：函数类型的 props 怎么标

> "那我给一个 `interface`……它这个参数就是一个 `text`，是个 `string`，然后**没有什么返回值，那就 `void`**。"

```tsx
interface AddTodoProps {
  addTodo: (text: string) => void
}
```

`void` 表示"调用者不该用返回值"，不是"返回 undefined"。**这个接口是"类型约束"价值最直接的体现**：父组件的 `addTodo` 与子组件声明的必须同形状才能赋值；你若写成 `(todo: Todo) => void`，`<AddTodo addTodo={addTodo} />` 立刻标红。

### 8.5 写到这里报的错：`'use client'`

> "这个是说，**None of its parents are marked with 'use client'**，就是你没有加那个 `'use client'` 啊。我们再顶上这儿……"

```
You're importing a component that needs useState. It only works in a
Client Component but none of its parents are marked with "use client",
so they're Server Components by default.
```

他在文件顶上加了 `'use client'` 解决。**这个错在 EP01–EP03 根本不会出现**（CRA 全是客户端），成因见第 11 章。

---

## 9. 第 7 步：TodoList 与 TodoItem（20:52–25:16）

### 9.1 TodoList：`ul` + `map` + 透传

> "我现在呢应该是有一个 `ul`，它是个列表嘛，列表里边呢是有一些 `li`，当然这个 `li` 到底什么样呢，肯定是需要根据你的事项来给的。"

```tsx
// src/components/TodoList.tsx
import TodoItem from './TodoItem'
import type { Todo } from '@/types'

interface TodoListProps {
  todos: Todo[]
  deleteTodo: (id: number) => void
  toggleTodo: (id: number) => void
}

export default function TodoList({ todos, deleteTodo, toggleTodo }: TodoListProps) {
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          deleteTodo={deleteTodo}
          toggleTodo={toggleTodo}
        />
      ))}
    </ul>
  )
}
```

> "之前我们在这块用的时候呢，是不是给传了**三个值**啊……这个 key 的值呢应该就是你 todo 的 **ID，不重复那个 ID**。"

为什么不在 `li` 里直接写内容："因为我应该是有一个 **`TodoItem` 组件的**"。注意 import 风格：同目录用 `./`，跨目录用 `@/`——社区约定，不是硬规定。📌 `Todo[]` 与 `Array<Todo>` 完全等价，多维 `Todo[][]`，联合 `(Todo | null)[]`（注意括号）。

### 9.2 TodoItem：`li` + 两个按钮 + 内联样式

```tsx
// src/components/TodoItem.tsx
import type { Todo } from '@/types'

interface TodoItemProps {
  todo: Todo
  deleteTodo: (id: number) => void
  toggleTodo: (id: number) => void
}

export default function TodoItem({ todo, deleteTodo, toggleTodo }: TodoItemProps) {
  return (
    <li>
      <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
        {todo.text}
      </span>
      <button onClick={() => toggleTodo(todo.id)}>切换</button>
      <button onClick={() => deleteTodo(todo.id)}>删除</button>
    </li>
  )
}
```

> "`li` 里面肯定会有一些内容，比如就是 `todo.text`，然后里边有两个 `button`……那操作的时候是不是要通过 `onclick` 点击，来触发刚才我们传入的那个处理程序啊？传的都是那个 **ID**。"

| 细节 | 为什么这么写 |
|---|---|
| `onClick={() => toggleTodo(todo.id)}` | **必须包箭头函数**！写 `onClick={toggleTodo(todo.id)}` 会在渲染时立刻执行，并把返回值 `undefined` 当 handler |
| 传 `todo.id` 不传 `todo` | 子组件只需报"哪一条"，不必知道父组件怎么改 |
| 三元给 `textDecoration` | 完成 → `'line-through'`；未完成 → `'none'` |

### 9.3 现场事故：漏传一个 prop

> "这块**报了一个错**，他说**没有 `text`**。为什么会没有 `text` 呢？……**这个地儿我没传**，嗨，没传，`todo`。"

```tsx
// ❌ 漏了 todo={todo}
<TodoItem key={todo.id} deleteTodo={deleteTodo} toggleTodo={toggleTodo} />
// ✅ 补上
<TodoItem key={todo.id} todo={todo} deleteTodo={deleteTodo} toggleTodo={toggleTodo} />
```

> "你看，如果你写的 **TS** 是不是就没这问题了呀？咱们这叫**现身说法**对吧？**好一个懒不能偷啊**。"

📌 因果链要说清，否则会误读成"TS 没用"：他给 `TodoItem` 的 props 写了口述的 **`any`**（"Any 让大家自己写好吧"）→ `any` 允许少传 → 编译期不报错 → 运行时 `todo` 是 `undefined`，读 `todo.text` 就炸。若写全 `todo: Todo`，编译期直接报 `Property 'todo' is missing`。**结论不是"TS 没用"，而是"写了 any 就等于没用 TS"。**

### 9.4 切换点了没反应？因为没写样式

> "你发现**切换怎么没用啊**？没有错啊，切换不是没用，就是**它没有显示**啊。因为我现在没有做任何样式处理，比较纯粹。"

数据其实变了（`completed` 已是 `true`），但界面**没有任何视觉反馈**。

```tsx
<span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
```

📌 他口述"否则给一个 `null`"——**TS 严格模式下会报错**：`Type 'null' is not assignable to type 'TextDecoration | undefined'. ts(2322)`，因为 `React.CSSProperties['textDecoration']` 是 `Property.TextDecoration | undefined`，**不收 `null`**，应写 `'none'`（本文已改）。这是"口述 vs 可运行"的典型差异：JS 能跑，TS 不行。

---

## 10. 第 8 步：TodoFilter（25:16–26:02）

> "这个呢是最简单的啊……你在这块呢，你肯定会接收一个叫做 `setFilter`。"

```tsx
// src/components/TodoFilter.tsx
import type { FilterType } from '@/types'

interface TodoFilterProps {
  setFilter: (filter: FilterType) => void
}

export default function TodoFilter({ setFilter }: TodoFilterProps) {
  return (
    <div>
      <button onClick={() => setFilter('all')}>全部</button>
      <button onClick={() => setFilter('active')}>未完成</button>
      <button onClick={() => setFilter('completed')}>已完成</button>
    </div>
  )
}
```

> "是不是 **3 个按钮**，按钮是不是要点击 `onclick`……我就调用 `setFilter` 去变更状态。这里边的名不一样，一个叫 `all`，一个叫 `active`，这跟**我们外边那个 `getFilteredTodos` 那个地儿是一样的**。"

### 10.1 为什么直接传 setter，而不是包一层 handler

这是全片唯一一处**与 EP02 命名规范相悖**的地方：EP02 §6.3 说 prop 用 `onXxx`、父组件函数用 `handleXxx`；这里 prop 直接叫 `setFilter`，传的就是 setter。📌 两种都对：签名正好合用、没有额外逻辑 → 直接传；需要打日志/埋点/校验/防抖，或想**隐藏"这是 state"** → 包一层。真实项目倾向后者，因为**子组件不该知道父组件用的是 `useState`**——哪天换 Redux，props 名就得改。

### 10.2 类型对得上吗？一个容易忽略的可赋值性

`setFilter` 实际类型是 `Dispatch<SetStateAction<'all' | 'active' | 'completed'>>`，而 props 声明 `(filter: FilterType) => void`。**参数更宽的 setter 可以赋给参数更窄的函数类型**（TS 允许"忽略多余能力"），所以能过。反过来：props 若写成 `Dispatch<SetStateAction<FilterType>>`，子组件就能传函数式更新；写成 `(f: FilterType) => void` 就不能。**想让子组件拥有什么能力，就在类型里给什么能力。**

---

## 11. `'use client'` 到底是怎么回事

App Router 默认**所有组件都是服务端组件（Server Component）**：在 Node 里渲染，没有浏览器环境，所以不能 `useState`、不能绑 `onClick`、不能碰 `window`。一旦某文件（或它的祖先）声明 `'use client'`，它和它 import 的所有模块就变成**客户端组件**。

```tsx
'use client'          // ← 必须是文件第一行（注释可以在上面，import 不行）

import { useState } from 'react'
```

`'use client'` 应该加在**客户端边界**上：加在 `page.tsx` 顶部 → 整棵子树变客户端（本案例做法，且必需，因为它自己就用了 `useState`）；只加 `AddTodo.tsx` → 只救它自己，`page.tsx` 照样报错；加在 `layout.tsx` → 全站客户端，但放弃了 Server Component 的好处。

📌 **客户端边界向下传染，不向上**：`page.tsx` 打了 `'use client'`，它 import 的 `TodoList`/`TodoItem`/`TodoFilter` 自动都是客户端组件，**不需要每个文件都写**；反过来，子组件写了，父组件仍是服务端组件，父组件里照样不能用 Hook。需要写这行的只有 **Next.js App Router**；Pages Router、Vite、CRA 都不需要（Vite 里写了也无害，只是个普通字符串）。

`npm run dev` 起来后打开 `http://localhost:3000`，按第 1 章那 5 个动作走一遍，全通过就说明案例成了。

---

## 12. 样式怎么处理

> "我并没有做很多的样式，大家可以到 TodoMVC 的仓库里面去使用它的样式。"
> "因为我现在没有做任何样式处理，比较纯粹啊……我这块呢就**偷个懒，我直接就用这个内联的写法**去写。"

所以全片只写了一处内联样式（`textDecoration`）。四种方案的取舍：**内联 `style`**（✅ 本片用的）无需额外文件、跟条件表达式天然配合，但写不了 `:hover`、媒体查询，复用差；**Tailwind** 写起来快，但要学类名（建项目时他选了 No）；**CSS Modules** 局部作用域，React 生态最主流；**全局 CSS**（`globals.css`）简单但污染全局。

📌 换成 CSS Modules 的最小改动——加一个 `TodoItem.module.css`（`.item` 布局、`.done` 删除线+灰字、`.del` 红字），组件里把 `style` 换成 `className={styles.xxx}`。

文件名带 `.module` 才被当 CSS Modules 处理；`styles.done` 是编译后的**唯一类名**（如 `TodoItem_done__a1b2c`）；条件不成立时给 `''` 而不是 `undefined`；全局样式放 `src/app/globals.css`，由 `layout.tsx` 自动引入。

---

## 13. 完整代码：page.tsx 与 types.ts

> 其余 4 个组件的完整代码见 §8、§9.1、§9.2、§10。全部**按口述+演示还原补全**，修正了两处会报错的写法（`textDecoration: null` → `'none'`；两处 `any` props → 完整接口），补上 `'use client'` 与 `FilterType`。

```ts
// src/types.ts（最终版）
export interface Todo {
  id: number
  text: string
  completed: boolean
}

export type FilterType = 'all' | 'active' | 'completed'
```

```tsx
// src/app/page.tsx
'use client'

import { useState } from 'react'
import AddTodo from '@/components/AddTodo'
import TodoList from '@/components/TodoList'
import TodoFilter from '@/components/TodoFilter'
import type { Todo, FilterType } from '@/types'

export default function Home() {
  const [todos, setTodos] = useState<Todo[]>([])
  const [filter, setFilter] = useState<FilterType>('all')

  const addTodo = (text: string) => {
    const newTodo: Todo = { id: Date.now(), text, completed: false }
    setTodos([...todos, newTodo])
  }

  const deleteTodo = (id: number) => {
    setTodos(todos.filter((todo) => todo.id !== id))
  }

  const toggleTodo = (id: number) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    )
  }

  const getFilteredTodos = () => {
    if (filter === 'completed') return todos.filter((todo) => todo.completed)
    if (filter === 'active') return todos.filter((todo) => !todo.completed)
    return todos
  }

  return (
    <div style={{ maxWidth: 480, margin: '40px auto', fontFamily: 'sans-serif' }}>
      <h1>Todo List</h1>
      <AddTodo addTodo={addTodo} />
      <TodoList
        todos={getFilteredTodos()}
        deleteTodo={deleteTodo}
        toggleTodo={toggleTodo}
      />
      <TodoFilter setFilter={setFilter} />
    </div>
  )
}
```

```
my-app/src/
├── app/          layout.tsx · page.tsx · globals.css
├── components/   AddTodo · TodoList · TodoItem · TodoFilter
└── types.ts      Todo / FilterType
```

各文件的完整代码：`page.tsx` 见上，`types.ts` 见上，4 个组件分别在 §8、§9.1、§9.2、§10。

---

## 14. 类型地图

| 类型 | 定义 | 出现位置 |
|---|---|---|
| `Todo` | `{ id: number; text: string; completed: boolean }` | `types.ts` |
| `Todo[]` | 事项数组 | `useState<Todo[]>`、`TodoListProps.todos` |
| `FilterType` | `'all' \| 'active' \| 'completed'` | `useState<FilterType>`、`TodoFilterProps` |

| 组件 | props 接口 | 字段 |
|---|---|---|
| `AddTodo` | `AddTodoProps` | `addTodo: (text: string) => void` |
| `TodoList` | `TodoListProps` | `todos: Todo[]`、`deleteTodo`、`toggleTodo` |
| `TodoItem` | `TodoItemProps` | `todo: Todo`、`deleteTodo`、`toggleTodo` |
| `TodoFilter` | `TodoFilterProps` | `setFilter: (f: FilterType) => void` |

| 位置 | 事件类型 | 是否手写 |
|---|---|---|
| `onSubmit={handleSubmit}` | `FormEvent<HTMLFormElement>` | ✅ 手写（独立函数） |
| `onChange={(e) => ...}` | 推导为 `ChangeEvent<HTMLInputElement>` | ❌ |
| `onClick={() => ...}` | 推导为 `MouseEvent<HTMLButtonElement>` | ❌ |

---

## 15. 速查卡

| 场景 | 写法 |
|---|---|
| 对象数组状态 | `useState<Todo[]>([])` |
| 追加 / 删除 / 改字段 | `[...todos, x]` / `todos.filter(t => t.id !== id)` / `todos.map(t => t.id === id ? {...t, completed: !t.completed} : t)` |
| 联合类型状态 | `useState<'all' \| 'active' \| 'completed'>('all')` |
| props 函数类型 | `addTodo: (text: string) => void` |
| 表单提交类型 | `(e: FormEvent<HTMLFormElement>) => void` |
| 受控输入 | `value={text} onChange={e => setText(e.target.value)}` |
| 条件样式 | `style={{ textDecoration: done ? 'line-through' : 'none' }}` |
| Next.js 客户端组件 / 类型复用 | 文件首行 `'use client'` / `import type { Todo } from '@/types'` |

---

## 16. 自测题

| # | 问题 | 答案 |
|---|---|---|
| 1 | `useState<Todo[]>([])` 里尖括号为什么能写？ | `useState` 是泛型函数 `<S>`，尖括号给 `S` 赋值；`Ctrl+点击` 看签名 |
| 2 | 为什么 `todos` 放 `page.tsx` 不放 `TodoList`？`text` 又为什么可以留在 `AddTodo`？ | 前者三个子组件都要用，必须提到共同父级；后者只有它自己用，提交完清空 |
| 3 | `getFilteredTodos` 为什么不写成状态？ | 能从 `todos`+`filter` 算出来，存了就要手动同步（派生状态） |
| 4 | `addTodo` 为什么只收 `text`？ | 生成 id、定初值是应用层知识，输入框不该知道 |
| 5 | `toggleTodo` 为什么必须 `{...todo}`？ | 直接改字段引用没变，React 认为状态没变，不重渲染 |
| 6 | `key` 为什么用 `todo.id` 不用下标？ | 删除后下标错位，React 会复用错组件（EP01 §5.4） |
| 7 | `onClick={toggleTodo(todo.id)}` 错在哪？ | 少了箭头函数，渲染时立刻执行，把 `undefined` 当 handler |
| 8 | `FormEvent<HTMLFormElement>` 的泛型实参决定什么？ | 决定 `e.currentTarget` 的类型 |
| 9 | `'use client'` 加在哪？ | 客户端边界文件（这里 `page.tsx`），向下传染，子组件不用重复 |
| 10 | 漏传 `todo` 为何编译期没报错？`textDecoration: null` 为何报错？ | props 写成 `any`，检查失效 / 该字段类型不含 `null` |

---

## 17. 📌 与已有笔记的差异

### 17.1 重叠：已有笔记讲过的，两句带过

| 本文 | 内容 | 已有出处 |
|---|---|---|
| 6.3 | `useState` 返回 `[值, setter]` | 新手笔记 §4.3、EP01 §8.3（本文只补"泛型实参"） |
| 7.2 / 7.3 | 数组追加不能 `push`、`filter` 删除 | EP01 §10.2 / §10.5 |
| 9.1 | `map` + `key` 用唯一 id | EP01 §5 |
| 8.1 | 受控组件 | 新手笔记 §9.2（本文只补两处遗漏的后果） |
| 8.4 | props 用 `interface` | 新手笔记 §6.2（本文只讲函数类型的 props） |
| 4 | 组件拆分依据 | EP02 §3.3（本文补"按谁会变拆"） |
| 9.3 | 子组件调父组件函数 | EP02 §6（不重复机制，只讲 bug） |

### 17.2 新增：已有笔记完全没有的

| # | 新增点 | 为什么重要 |
|---|---|---|
| 1 | **第一个端到端可运行的完整案例**（4 组件 + 单一数据源 + 派生数据） | 前 3 集都是单点知识，没有"组装"经验 |
| 2 | `useState<Todo[]>` 的**泛型实参怎么"看出能写"** | 新手最常问"尖括号哪来的" |
| 3 | **Next.js App Router 的 `'use client'` 边界** | 前 3 集用 CRA，没这概念；换框架第一个撞墙点 |
| 4 | `FormEvent<HTMLFormElement>` 的**事件泛型实参** | 新手笔记只列事件名，没讲泛型 |
| 5 | **联合类型当状态机** | 比 `enum` 轻，比 `string` 安全 |
| 6 | **派生状态**：`getFilteredTodos` 是函数不是状态 | 决定"状态该有几个"的核心判断 |
| 7 | **props 用 `any` 的现场代价** | 新手笔记只说"interface 是说明书"，没给反例 |
| 8 | `Date.now()` 当 id 的实践与缺陷 | EP01 §10.3 用 `id++` 踩过坑，这里换方案但仍有问题（18.2） |
| 9 | **AI 参与真实编码的工作流**（等补全 → 审查 → 改签名）＋ 从零建 Next.js 项目的目录约定、**TodoMVC** 样式来源 | EP03 的 CodeGeeX 段只是"看它能写"；新手笔记的项目是现成仓库，没讲怎么建 |

### 17.3 说法冲突：四处需要你知道"该信谁"

**① 建项目用哪个脚手架**：EP01 §1.1 教 `npx create-react-app`（并补注"CRA 已停止维护，推荐 Vite"）；EP04 用 `npx create-next-app@latest` + App Router。两集隔了约 1 年，作者跟着社区风向走了，影响就是那个真实报错（`'use client'`）。**只为学 React 语法 → 用 Vite**（`npm create vite@latest -- --template react-ts`），没有这层概念最省心；**想顺便学 Next.js → 照视频走**，但要知道 `page.tsx` 是路由文件、客户端组件要声明。⚠️ 标题说"React + TypeScript 案例"，实际工程是 **Next.js**——不影响 React 知识，但影响目录结构和那行 `'use client'`。

**② `todos` 该用 `useState` 还是 `useReducer`**：EP03 §1.6 的选型表说"改法 5 种以上""开始给 setter 起名字且改的是同一个对象——就该考虑 `useReducer`"；EP04 用 `useState` + 3 个具名函数，只在 10:31 提一句"**当然你也可以使用 `useReducer`**"随即略过。EP03 给的是"信号"，不是"红线"：本案例 3 个改法**彼此独立**（加/删/改一条），没有状态机式流转，按量化标准（5 种以上 + 字段互相牵连）**没到**该上 reducer 的程度。📌 口诀：**改法之间要互相知道对方（如"只有 loading 结束才能 success"）→ reducer；改法互不相干 → `useState`。** 若后面加"编辑中""批量完成""撤销"，改法涨到 6 个以上就该迁。

**③ props 该叫什么名字**：EP02 §6.3 规定 prop 用 `on` + 事件名、处理函数用 `handle` + 事件名（反例 `callback={fn}`）；EP04 直接传 `addTodo`、`deleteTodo`、`toggleTodo`、`setFilter`——**一个 `on` 都没有**。EP02 讲的是**可复用组件的对外契约**，EP04 写的是**页面专用的私有组件**：通用/多处复用 → `onXxx`（描述"发生了什么"）；页面专用 → `动词+名词`（描述"要做什么"）。📌 若哪天 `AddTodo` 要被第二个页面复用，第一件事就是把 prop 改成 `onAdd`——它不再"知道"调用方拿它干嘛，只知道"用户交了一段文本"。

**④ 更新状态用不用函数式写法**：EP01 §9.6 说"依赖旧值时用 `setX(prev => ...)` 更安全"；EP04 全部用闭包外层的 `todos`。单次事件、单次渲染下两者一样，作者图省事。但三种情况会出错：① 一次事件里连调两次 `addTodo`——第二次用的还是旧 `todos`，**只加进去一条**；② `await` 之后再用 `todos`（过期快照，**闭包陷阱**，EP03 §4.6）；③ 子组件在 `useEffect` 里调用。📌 修法见 18.1，**代价为零**——这是本文对视频**唯一一处建议性反驳**。

**⑤（不算冲突）类型严格度**：新手笔记 §6.3 说"传错 props 类型，编辑器立刻标红"；本集两处用 `any`，结果漏传 prop 到运行时才炸（9.3）。作者自己的结论就是答案：**"好一个懒不能偷。"**

---

## 18. 📌 补充：视频没讲但会踩的坑

### 18.1 批量更新会丢数据（最高频）

```tsx
// ❌ 视频写法：第二次调用拿到的还是旧的 todos
setTodos([...todos, newTodo])

// ✅ 函数式更新：始终基于更新队列里的最新值
setTodos((prev) => [...prev, newTodo])
setTodos((prev) => prev.filter((todo) => todo.id !== id))
setTodos((prev) =>
  prev.map((todo) => (todo.id === id ? { ...todo, completed: !todo.completed } : todo))
)
```

| 写法 | 依据 | 安全性 |
|---|---|---|
| `setTodos([...todos, x])` | 渲染时的 `todos` | 单次安全，批量/异步不安全 |
| `setTodos(prev => [...prev, x])` | 更新队列最新值 | ✅ 始终安全 |

📌 另一种常见错法：`setTodos(prev => { prev.push(x); return prev })`——引用没变，可能整片不渲染。

### 18.2 `Date.now()` 当 id 会撞

同一毫秒内连点两次会生成**相同 id**。后果很隐蔽：`key` 重复 → 警告 `Encountered two children with the same key`，删一条会连带删错另一条。可改 `crypto.randomUUID()`（需把 `Todo.id` 改成 `string`），但📌 工程上更常见的答案是**后端给 id**——这也是 EP01 §10.3 结尾作者自己的说法："这个数据正常来说应该是后端给你返回的。"

### 18.3 派生值不要塞进 `useState`

反面教材：`const [filteredTodos, setFilteredTodos] = useState<Todo[]>([])`——每次 `todos` 或 `filter` 变，都要记得同步它。正面就是 §7.5 的 `getFilteredTodos()`：直接算。判断标准：**"它变了，是不是因为别的状态变了？"** 是 → 别存。

### 18.4 性能：什么时候才需要 `useMemo` / `memo`

`getFilteredTodos()` 每次渲染都重算，100 条以内完全无感（`filter` 是 O(n) 极快操作）。上千条卡顿了才上 `useMemo(() => ..., [todos, filter])`（EP03 §5）；某行没变却重渲染才上 `memo(TodoItem)` + `useCallback` 包 handler（EP03 §6）——📌 两者必须配套，否则每次渲染都是新函数引用，`memo` 白加（新手笔记 §11.4"不要过早优化"同理）。

### 18.5 刷新就没了：持久化到 `localStorage`

```tsx
const STORAGE_KEY = 'todo-list'

const [todos, setTodos] = useState<Todo[]>(() => {
  if (typeof window === 'undefined') return []
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY) ?? '[]') as Todo[]
  } catch {
    return []
  }
})

useEffect(() => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
}, [todos])
```

四个细节：`useState(() => ...)` 传函数是**懒初始化**，只在首次执行；`typeof window === 'undefined'` 判断是因为服务端渲染没有 `window`，不判断会报错；`as Todo[]` 只是**断言**（`JSON.parse` 返回 `any`），生产环境该做运行时校验；`try/catch` 兜住坏 JSON，否则抛异常会白屏。

### 18.6 接后端：mock → fetch → loading/error（视频完全没有）

视频里"数据从哪来"的答案是**纯前端内存**。真实项目第一件事就是换成接口：先定 `ApiResponse<T>`（`{ code, data, message }`），再写 `fetchTodos(): Promise<Todo[]>`，然后在页面里接 `useEffect` 补 `loading` / `error`：

```ts
// src/api/todos.ts
import type { Todo } from '@/types'

export interface ApiResponse<T> {
  code: number
  data: T
  message: string
}

const BASE = process.env.NEXT_PUBLIC_API_BASE ?? ''

export async function fetchTodos(): Promise<Todo[]> {
  const res = await fetch(`${BASE}/api/todos`)
  if (!res.ok) throw new Error(`请求失败：${res.status}`)
  const json: ApiResponse<Todo[]> = await res.json()
  return json.data
}
```

```tsx
// src/app/page.tsx（片段）
const [loading, setLoading] = useState(true)
const [error, setError] = useState<string | null>(null)

useEffect(() => {
  let cancelled = false
  fetchTodos()
    .then((data) => { if (!cancelled) setTodos(data) })
    .catch((e: unknown) => {
      if (!cancelled) setError(e instanceof Error ? e.message : '未知错误')
    })
    .finally(() => { if (!cancelled) setLoading(false) })
  return () => { cancelled = true }   // 📌 竞态保护：卸载/依赖变化后不再 setState
}, [])

if (loading) return <p>加载中…</p>
if (error) return <p style={{ color: 'red' }}>出错了：{error}</p>
```

四个坑：`Promise<Todo[]>` 要 `await` 之后才是 `Todo[]`；`catch` 的参数在 TS 里是 **`unknown`**（不是 `any`），得先用 `instanceof Error` 收窄才能读 `.message`；`cancelled` 标志防"旧请求后回来覆盖新数据"（也可用 `AbortController`）；环境变量只有 `NEXT_PUBLIC_` 前缀才会注入浏览器端，不带前缀的前端读到 `undefined`。

📌 这就是"为什么先学纯前端状态、再学请求"：`loading`/`error`/`data` 是三件套状态，一旦引入，`useState` 就开始显得散——正是 EP03 选型表里"用 `useReducer`"的典型场景（`{ loading, data, error }` 互相牵连）。

### 18.7 `button` 在 `form` 里默认是 `submit`

```tsx
<form onSubmit={handleSubmit}>
  <button>添加</button>                                  {/* 默认 type="submit" */}
  <button type="button" onClick={onCancel}>取消</button>  {/* ✅ 必须显式声明 */}
</form>
```

本案例 `TodoItem` 的按钮不在 form 内，所以没踩到；一旦给它加"编辑"输入框（放进 form），不加 `type="button"` 的按钮就会把表单一起提交掉。

---

## 19. 📌 四集串联（补充，非视频原意）

> ⚠️ **视频结尾没有任何系列总结**。26:02 之后作者只说"如果你喜欢我的视频可以给我点赞或者三连……我们下个视频再见"，约 12 秒，没回顾前三集、没"合集完结"陈词、也没下一季预告。下面这条线是**我按四集内容补的**。

| 集 | 你学会了 | 案例里的落点 |
|---|---|---|
| EP01 | 组件、JSX 四条规则、条件渲染、列表 + key、`useState`、不可变更新 | `todos`、`map`、`[...todos, new]`、`filter`、`key` |
| EP02 | props、展开语法、children/插槽、单向数据流、子传父、Context | 4 个组件的 props 契约、`addTodo` 回调、三层透传 |
| EP03 | 5 个 Hook | ⚠️ **一个都没用上**；18.4/18.6 说明它们何时会进来 |
| EP04 | 组装：拆组件、定类型、放状态、发函数 | 全部 |

四句话概括：**EP01 教你「一个组件怎么写」→ EP02「几个组件怎么说话」→ EP03「组件怎么跟外部世界打交道、怎么不变慢」→ EP04「一个功能从需求到可运行，中间要做哪 7 个决定」。**

前 3 集是**零件**，第 4 集是**装配**。而装配阶段做的每个决定（状态放哪、类型怎么定、props 怎么传），恰恰是真实工作里最花时间、也最容易做错的部分。**这一集真正的增量只有三样**：框架边界（`'use client'`）、事件类型（`FormEvent<T>`）、类型建模（`interface` + 联合类型）——正好是"从语法到工程"的三道门槛。下一步练：**加 localStorage（18.5）→ 加"编辑"功能（`useRef` 聚焦，EP03 §3.3）→ 换 mock 接口 + loading/error（18.6）→ 用 Context 干掉三层透传（EP02 §8）**。每加一样，都会撞上一个"当时为什么这么设计"的答案。

---

## 20. 复刻清单

| 步 | 动作 | 检查点 |
|---|---|---|
| 1 | `npx create-next-app@latest`（TS ✅ / Tailwind ❌ / src ✅ / App Router ✅） | `npm run dev` 能开 3000 |
| 2 | 清空 `page.tsx`，只留 `<h1>Todo List</h1>` | 页面只剩标题 |
| 3 | 建 `src/components/`，新建 4 个 `.tsx`，各写一个返回 `div` 的函数 | import 不报错 |
| 4 | `page.tsx` 里 import 并摆上 3 个组件 | 页面出现 3 块区域 |
| 5 | 写 `types.ts` 的 `Todo`；加 `useState<Todo[]>([])` | 悬停 `todos` 显示 `Todo[]` |
| 6 | 写 4 个 handler + `filter` 状态，往下传 | 类型报错消失 |
| 7 | 依次实现 `AddTodo` → `TodoList` → `TodoItem` → `TodoFilter`；再加 `'use client'` 与 `line-through` | 增/删/切/筛全通，切换有视觉反馈 |

**验收**：完成第 1 章那 5 个动作，控制台无红报错、无 key 警告。

📌 提高效率的变体：**第 6 步故意先用 `any` 顶着跑一遍，亲眼看着漏传一个 prop 会怎样；再补全接口，看编辑器怎么提前拦住。** 这个对比实验比看十遍文档都管用——作者在 9.3 已经替你做了一半。

---

## 21. 一句话总结

> **这个案例的全部技术含量，是三个决定：数据放哪（`page.tsx` 的两个 `useState`）、类型怎么定（`Todo` 一个接口喂四个文件）、函数往哪传（三个 handler 往下发、事件往上传）；剩下的 `map`/`filter`/`[...old, new]` 都只是 EP01 的旧账。**

只能记一句的话：**能算出来的别存，要共用的往上提，只收该收的（`addTodo(text)` 而不是 `addTodo(todo)`）——这三条决定了 80% 的组件设计。**

---

## 附录：本文信息

| 项 | 内容 |
|---|---|
| 视频标题 | 20分钟学会React TypeScript案例 2023最后一更 让我们相约2024 大学生学习前端全栈必会技能 ts next.js |
| 合集文案 | 《1小时学会React18》第 4 集（共 4 集），含"React + TypeScript 综合案例" |
| UP 主 / BV 号 | 吴悠讲编程 ｜ BV1WC4y1B7Uq |
| 时长 / 发布 | 26:14（标题写"20 分钟"） ｜ 2023-09 |
| 视频地址 | https://www.bilibili.com/video/BV1WC4y1B7Uq/ |
| 技术栈 | React 18 + TypeScript 5 + Next.js（App Router）+ 内联样式 |
| 代码还原度 | 4 组件 + 1 类型文件 + 主页，全部可直接运行；修正 2 处会报错的写法（`textDecoration: null`、`any` props），补全 `'use client'` 与 `FilterType` |
| 广告段 | 3:51–5:50（AWS CodeWhisperer），保留在第 3 章 |
| 参考 | EP01《React18核心语法》、EP02《组件通信与插槽》、EP03《ReactHooks速成》、《React新手学习笔记-95%核心知识.md》、《前端学习路线-从零到React.md》 |
| 系列状态 | **本集为合集最后一集**；视频结尾无系列总结，第 19 章为本文补充 |
