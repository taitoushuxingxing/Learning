# React 18 Hooks 速成（useReducer / useRef / useEffect / useMemo / useCallback）—— 详细学习笔记

> 基于 B 站视频《20分钟学会React Hooks 前端开发必看 AI编程工具 CodeGeeX 体验》整理。
> UP 主：吴悠讲编程 ｜ 时长 22:09 ｜ 发布 2023-09 ｜ 合集《1小时学会React18》第 3 集（共 4 集）
> 视频地址：https://www.bilibili.com/video/BV1kc411D7F9/
>
> **本文怎么来的**：完整音频（13.1 MB）做语音转写（带时间戳），按视频**原本的讲解顺序**重构成文档，没有按我自己的知识体系重排。转写稿是同音错字极多的口语稿（"userereducer"→`useReducer`、"优思瑞福"→`useRef`、"useinfect"→`useEffect`、"usemam"→`useMemo`、"use call back"→`useCallback`、"胡克"→Hook、"副组件"→父组件、"carry 属性"→`.current`、"记忆组件"→memo 组件），已逐处按上下文修正；代码按口述还原并补全为**可直接运行**的 React 18 函数组件。
>
> **这是一篇增量文档**。已有《React新手学习笔记-95%核心知识.md》（957 行）、《前端学习路线-从零到React.md》（231 行）、本系列 `EP01`、`EP02`。重叠处一句话带过并注明出处，差异汇总在[第 11 章](#11-与已有笔记的差异)。
>
> **⚠️ 时间账要算清**：22 分 09 秒里**有 2 分 26 秒（4:00–6:26）是 CodeGeeX 广告**，真正的 Hooks 内容约 15 分钟，5 个 Hook 平均一个 3 分钟。所以这集的定位是**「知道有这个东西、知道它长什么样」**，不是「讲透」。本文保留广告段（第 2 章，忠实还原顺序），真正的深度在「📌 补充」里——**依赖数组、闭包陷阱、StrictMode 双执行、memo 三件套的取舍**这四块，视频只演示了现象，没讲机制，而它们恰恰是上手后必踩的坑。
>
> **读法提醒**：本集**不讲** `useState` 与 `useContext`（作者说前几集讲过）——见 `EP01` §8–§10、`EP02` §8；也**不讲**自定义 Hook、受控组件、数据请求——见已有笔记 §9、§11。

---

## 目录

- [0. 课程地图：五个 Hook，一条主线](#0-课程地图五个-hook一条主线)
- [1. 第一章 useReducer：把「怎么改」收进一个函数](#1-第一章-usereducer把怎么改写进一个函数)
- [2. 插播：CodeGeeX（4:00–6:26）](#2-插播codegeex400626)
- [3. 第二章 useRef：一个盒子，两个用途](#3-第二章-useref一个盒子两个用途)
- [4. 第三章 useEffect：组件和外部世界打交道的窗口](#4-第三章-useeffect组件和外部世界打交道的窗口)
- [5. 第四章 useMemo：缓存「算出来的值」](#5-第四章-usememo缓存算出来的值)
- [6. 第五章 useCallback：缓存「函数本身」](#6-第五章-usecallback缓存函数本身)
- [7. 全局复盘：五个 Hook 各站哪个位置](#7-全局复盘五个-hook-各站哪个位置)
- [8. 速查卡](#8-速查卡)
- [9. 动手路线](#9-动手路线)
- [10. 自测题（含答案）](#10-自测题含答案)
- [11. 📌 与已有笔记的差异](#11-与已有笔记的差异)
- [12. 📌 补充：视频没讲但会踩的坑](#12-补充视频没讲但会踩的坑)
- [13. 一句话总结](#13-一句话总结)
- [附录：本文信息](#附录本文信息)

---

## 0. 课程地图：五个 Hook，一条主线

作者开场先接前情：「在之前的视频中，我们已经讲过了两个非常常用的 React Hooks，例如说创建状态的 `useState`，以及用来传递状态的 `useContext`。那今天我们要讲的是**另外 5 个**非常常用的 React Hooks。」

讲解顺序**不要打乱**——后一个总要用前一个的概念（讲 `useMemo` 时说"跟刚才 `useEffect` 比较相似"，讲 `useCallback` 又说"跟 `useMemo` 比较相似"）：

| 顺序 | Hook | 视频里的一句话定位 | 解决的那类问题 |
|---|---|---|---|
| 1 | `useReducer` | "以一种更加规范、更加统一的方式对一个状态的操作方式做管理" | 一个状态有**很多种改法**，散落各处 |
| 2 | `useRef` | "主要有两个功能"——存值、取 DOM/组件 | 有些数据变了**不该触发渲染**；有些东西要**直接摸到** |
| 3 | `useEffect` | "`effect` 代表的是副作用函数" | 渲染之外的活儿：请求、定时器、订阅 |
| 4 | `useMemo` | "一种用来进行数据缓存的钩子" | 每次渲染都**重算一遍很贵的值** |
| 5 | `useCallback` | "用来缓存的是函数" | 每次渲染都**造一个新函数**，害得子组件白渲染 |

一条主线：React 的循环只有「状态变了 → 重跑组件函数 → 更新 DOM」，五个 Hook 都在给这个循环打补丁。

```
                 ┌──────────── React 的核心循环 ────────────┐
                 │  状态变了 → 重新执行组件函数 → 更新 DOM   │
                 └──────────────────────────────────────────┘
                                    │
        ┌───────────────┬───────────┴──────────┬─────────────────┐
   状态怎么改？      什么不该进这个循环？    循环之外的活儿      循环太贵怎么办？
   ┌─────────┐     ┌──────────────┐     ┌────────────┐    ┌──────────────────┐
   │useReducer│     │    useRef     │     │ useEffect  │    │ useMemo/useCallback│
   │ 改法收口 │     │ 存了也不渲染  │     │ 提交后再做 │    │  省计算 / 省渲染   │
   └─────────┘     └──────────────┘     └────────────┘    └──────────────────┘
```

面试问"什么时候用 `useRef`"，正确回答不是"存 DOM 的时候"，而是**"有个值我想让它跨渲染活着、但不想让它触发渲染的时候"**。

**时间轴**：0:00 开场 → 0:29 `useReducer` → 4:00 广告 → 6:28 `useRef` → 13:16 `useEffect` → 15:05 `useMemo` → 18:12 `useCallback` → 22:03 收尾。

---

## 1. 第一章 useReducer：把「怎么改」收进一个函数

### 1.1 起点：一个加减计数器

作者先给了一个用 `useState` 写的计数器，还自嘲"加减我好像写反了，不过没有关系"——`const [count, setCount] = useState(0)`，两个按钮分别 `setCount(count - 1)` 和 `setCount(count + 1)`，下面挂一个 `<span>{count}</span>`。能跑，没毛病。那为什么要换？

### 1.2 痛点：一个状态有 N 种改法，散落在 N 个地方

作者的观察（1:08–1:22）：「对一个数据的更改方式可能是多种多样的，有可能是加，有可能是减，也有可能是一些别的修改方式。那在我们的组件操作中，就可能有**多个位置**都包含了对某个状态的修改操作……它并不能很好地做一个**统一管理**。」

翻译成代码就是这种烂摊子——同一个 `count`，三种改法散在四处，想知道"它到底能变成什么"只能全文搜 `setCount`：

```jsx
const [count, setCount] = useState(0);

<button onClick={() => setCount(count + 1)}>加</button>
<button onClick={() => setCount(count * 2)}>翻倍</button>
<button onClick={() => setCount(0)}>清零</button>
<DeepChild onReset={() => setCount(count - 10)} />
```

`useReducer` 只干一件事：**把这些改法全部收进一个函数，组件只负责"发请求"。**

### 1.3 reducer 是什么：一个普通的纯函数

「这个函数就是一个普通的函数，我们在函数中需要接收两个值，一个叫做 `state`，一个叫做 `action`。第一个参数 `state` 表示当前 reducer 函数要管理的状态是哪一个，第二个 `action` 表示你要对这个状态进行哪些操作。」（1:40–1:54）

「我们通过 `switch` 方式来判断 `action` 里面的一个属性叫做 `type`，也就是它的操作类型……你可以使用这种**抛异常**的方式，或者增加一些额外操作都是可以的。」（1:54–2:12）

```jsx
function countReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return state + 1;
    case 'decrement':
      return state - 1;
    case 'reset':
      return 0;
    default:
      // 作者说的「抛异常」：写错 action.type 时立刻暴露，而不是悄悄返回 undefined
      throw new Error('未知的 action.type: ' + action.type);
  }
}
```

| 要点 | 说明 |
|---|---|
| **返回新值，不改旧值** | `return state + 1`，绝不写 `state++`。和 `setState` 的不可变更新是同一套规矩 |
| **`action` 是约定，不是语法** | 可以是 `{ type, payload }`，也可以直接是字符串；**只有 `type` 是社区惯例**，React 不检查 |
| **`default` 别省** | 返回 `undefined` 会让状态变 `undefined`，界面白屏且报错位置离现场很远 |

### 1.4 接到组件上：两个参数、两个返回值

「参数 1 就是你刚才设置的 reducer 函数，也就代表了对状态的一些操作方式。参数 2 表示状态的默认值。`useReducer` 的返回值，其实跟之前我们使用的 `useState` 是比较相似的，它的返回值也是一个**数组**，第一个元素表示状态的当前值，第二个值是用来进行状态修改的一个**触发器**，它是一个函数值。」（2:12–2:40）

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
//      ↑        ↑                    ↑         ↑
//   当前状态   派发器              改法集合    初始值
```

关键的一步——不再调 `setCount`，改成 `dispatch` 一个"操作描述"：「我们就不需要你再使用比如像 `setCount` 的这种操作方式了，而是使用你在操作中指代好的一些处理方式，调用 `dispatch` 来进行相关的处理操作。里面还有一个参数是对象，这个参数就是 `action`。」（2:40–3:24）

```jsx
dispatch({ type: 'increment' });        // 加
dispatch({ type: 'reset' });            // 清零
dispatch({ type: 'set', payload: 42 }); // 带参数的改法
```

### 1.5 完整可运行版本

```jsx
import { useReducer } from 'react';

function countReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return state + 1;
    case 'decrement':
      return state - 1;
    case 'reset':
      return 0;
    case 'set':
      return action.payload;
    default:
      throw new Error('未知的 action.type: ' + action.type);
  }
}

export default function Counter() {
  const [count, dispatch] = useReducer(countReducer, 0);
  return (
    <div>
      <span>{count}</span>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'reset' })}>reset</button>
    </div>
  );
}
```

作者的结论（3:40–3:55）：「你会发现它实现的功能跟我们之前使用 `useState` 实现的操作是**完全一致**的，只不过我们通过 reducer 可以更好地管理对一个状态的多种复杂操作。所以如果单单从**实现功能角度**来讲，你到底使用的是 `useState` 还是 `useReducer` 都是没有问题的。」

这句"都没问题"很重要——**`useReducer` 不是 `useState` 的升级版，它是另一种组织方式**。判断标准不在功能，而在"改法会不会变多"。

### 1.6 选型表

| 判断维度 | 用 `useState` | 用 `useReducer` |
|---|---|---|
| 状态个数 | 1 个独立的值 | 多个字段**互相牵连**（如 `{loading, data, error}`） |
| 改法数量 | 1–2 种（开关、输入） | 5 种以上，或还会继续加 |
| 改法位置 | 都在同一个组件里 | 散在多个子组件、多个事件里 |
| 是否"状态机"式流转 | 不需要 | 需要（idle → loading → success） |
| 想单独测改法逻辑 | 难（逻辑粘在组件里） | 容易（reducer 是纯函数，可直接 import 测） |
| 代码量 | 少 | 多一个函数 + 一个 switch |

一句话判断法：**当你开始给 setter 起名字（`handleAddTag`、`handleRemoveTag`、`handleClearTags`）且它们改的是同一个对象——就该考虑 `useReducer` 了。**

### 1.7 📌 补充：视频没说的四条纪律

**① `dispatch` 引用稳定**，`useEffect`/`useCallback` 的依赖数组里**不用写它**（React 保证同一实例的 `dispatch` 永远同一个函数）。这比传 `setState` 省心，也是"把 dispatch 通过 Context 传下去"能成立的原因。

**② reducer 必须是纯函数。** 不能发请求、写 `localStorage`、更不能 `state.push()`。📌 因为 **StrictMode 下 reducer 会被调用两次**（见 §4.3），React 也可能在并发渲染时丢弃重算——不纯的 reducer 会执行两遍。

**③ 第三个参数是惰性初始化**，从 `localStorage` 读初始值时就该用它：

```jsx
const [state, dispatch] = useReducer(reducer, initFromStorage());   // ❌ init 每次都执行
const [state, dispatch] = useReducer(reducer, null, initFromStorage); // ✅ 只在首次挂载执行
```

**④ `useReducer` 天然免疫"连续 set 拿到旧值"**：一个事件里连发三个 `dispatch`，每个 reducer 收到的都是**上一次 reducer 返回的最新 state**；而 `setCount(count + 1)` 连写三次只会 +1。视频没点出来，但这是最实在的隐性好处。

---

## 2. 插播：CodeGeeX（4:00–6:26）

这段和 Hooks 无关，是作者演示自己装的 AI 编程插件。**保留它是因为它占了全片 11% 的时长，也解释了你在这集里看到的所有"代码自己跳出来"的画面**——不是剪辑，是插件在补全。

| 项目 | 视频里的说法 |
|---|---|
| 名字 / 安装 | 转写稿记成 "codeges / codegx / codegeeks"，标准拼写是 **CodeGeeX**（智谱）；VS Code 扩展市场安装，左侧栏多一个图标 |
| 用法一 | 侧边栏对话框，像 ChatGPT 一样提问（现场问了"如何使用 useState"） |
| 用法二 | 行内补全：写到 `<button` 时自动联想出上下文相关的 JSX |
| 用法三 | 选中代码 → 底部输入 `/` 调指令 |

| 指令 | 作用 | 作者评价 |
|---|---|---|
| `/explain` | 解释选中代码 | 现场演示，能说清"这段在根据 action 判断如何改 state" |
| `/comment` | 自动加注释 | 原话："你同事不写注释，你就用这个工具给他写好了发过去，他就会发现你这个人非常好" |
| `/fix bug` / `/test` | 修 bug / 生成测试 | 一句带过 |

> 📌 **补充（2026 年视角）**：这段演示今天已经过时——Copilot / Cursor / Trae 的能力边界早就不是"补全一行 JSX"。但它引出的判断依然成立：AI 能帮你写 `useEffect`，但**依赖数组写没写对、cleanup 有没有漏、`memo` 是不是白加了，只有你自己看得懂才判断得了**。这正是第 4–6 章的价值。

---

## 3. 第二章 useRef：一个盒子，两个用途

### 3.1 它到底是什么

作者的定义（7:00–7:29）：「我们通过 `useRef` 来创建一个用来进行数据存储的**容器**……它的值是可以你手动做修改的，但是它**并不是一个响应式的状态**，而且它**不会随着你组件的更新而进行重新的更新**。」

最后半句转写稿有点绕，准确说法是两条：

| 特性 | 含义 |
|---|---|
| **同一个盒子** | `useRef(x)` 返回的对象，在**每一次渲染里都是同一个对象**（React 把它存在 Fiber 上） |
| **改了不渲染** | `ref.current = 新值` **不会**触发重渲染。界面不动，直到**别的原因**导致重渲染你才看到新值 |

还有一条他特别强调（7:29）——**赋值赋给 `.current`，不是赋给 ref 本身**：「它并不是给这个值本身做设置，而是通过调用 `useRef` 返回给你一个包含了 `current` 属性的对象。」

```jsx
const box = useRef(0);
box = 1;          // ❌ 报错：box 是 const
box.current = 1;  // ✅ 改盒子里的东西，盒子本身没换
```

📌 为什么绕一层 `.current`？因为**"盒子"的引用必须永远不变**（React 靠它认出"还是那个 ref"），变的只能是盒子里的东西。这也正是 `useRef` 与 `useState` 的分工：`useState` 换盒子（换引用 → 触发渲染），`useRef` 只换内容（不触发渲染）。

### 3.2 用途一：记住「上一次的值」

作者的示例（6:28–8:13）：计数器除了显示当前值，还要显示"上一次是多少"。难点在于 `count` 一变组件重跑，`let prev = count` 这种普通变量**每次都被重置**，永远等于当前值——所以需要一个"跨渲染还活着、但不参与渲染"的地方，正是 `useRef`。

```jsx
import { useState, useRef } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef(null);   // 一个盒子，初始是 null

  const handleClick = () => {
    prevCountRef.current = count;      // ① 先把「这一轮的值」塞进盒子
    setCount(count + 1);               // ② 再改状态 → 触发重渲染
  };

  return (
    <div>
      <p>当前：{count}</p>
      <p>上一次：{prevCountRef.current ?? '—'}</p>
      <button onClick={handleClick}>+1</button>
    </div>
  );
}
```

跑出来的效果（7:57–8:13）：「默认它是空的。这块我们点击增大 count，它就会记录上一次的这个值，而不会随着你组件变化而出现更新的情况。」

**为什么显示的是"上一次"？** 因为赋值发生在 `setCount` 之前，界面刷新发生在 `setCount` 之后：

```
初始渲染        count = 0     box.current = null     界面：当前 0 / 上一次 —
   │
点击（第 1 次）
   ├─ box.current = 0          ← 手动赋值，此刻界面不动（ref 不触发渲染）
   ├─ setCount(0 + 1) → 重新渲染  count = 1
   └─ 读 box.current           → 0    ✅ 界面上「上一次」是 0
   │
点击（第 2 次）
   ├─ box.current = 1
   ├─ setCount(1 + 1) → 重新渲染  count = 2
   └─ 读 box.current           → 1    ✅ 界面上「上一次」是 1
```

> 📌 **补充：封装成 `usePrevious`（社区标准写法）**

```jsx
import { useEffect, useRef } from 'react';

export function usePrevious(value) {
  const ref = useRef();
  // 渲染时 ref.current 还是「上一次」的值；等这一轮提交完才写入新值，留给下一轮用
  useEffect(() => { ref.current = value; }, [value]);
  return ref.current;
}
```

它和视频版本的差别：**视频在事件处理函数里赋值，`usePrevious` 在 `useEffect` 里赋值**。后者不依赖"谁先谁后"——定时器、请求回调、父组件传值引起的变化都能被正确记录。

> 📌 **补充：三条禁令**
>
> 1. **不要在渲染期读写 `ref.current`**（初始化除外）。渲染可能被中断、丢弃、重跑（StrictMode 会跑两次），渲染期写入会写出脏数据。
> 2. **不要拿 `ref.current` 驱动界面**。它变了界面不更新，会得到"数据对了但页面不动"的幽灵 bug。
> 3. **不要在 JSX 里写 `{ref.current}` 还指望它更新**——上面的例子能成立，是因为 `setCount` 顺带触发了渲染，不是因为 ref 有响应性。

### 3.3 用途二：拿到 DOM（让 input 自动聚焦）

```jsx
import { useRef } from 'react';

export default function FocusDemo() {
  const inputRef = useRef(null);
  return (
    <div>
      <input ref={inputRef} placeholder="点右边的按钮让我聚焦" />
      <button onClick={() => inputRef.current?.focus()}>聚焦</button>
    </div>
  );
}
```

作者的讲法（8:52–9:24）：「这个 `inputRef` 就等于已经与这个 `input` 的标签形成关联了，你可以通过它来**直接操作 input 标签的 DOM 功能**……通过 `inputRef.current.focus()`，`current` 就获取到了 input 的 DOM 对象，然后通过 DOM 的方法 `focus` 来触发获取焦点事件。」

| 关键点 | 说明 |
|---|---|
| `ref={inputRef}` 绑到原生标签 | React 把**真实 DOM 节点**塞进 `inputRef.current` |
| 首次渲染时 `current` 是 `null` | DOM 还没创建，渲染期读会拿到 `null` |
| 卸载后 `current` 变回 `null` | 别再访问，否则 `Cannot read properties of null` |
| 为什么用 `?.` | 防御性写法，避免"点太快、DOM 还没挂"的边界 |

📌 补充：`ref` 还能做这些"React 不帮你管"的事——`scrollIntoView()`、`videoRef.current.play()`、`canvas.getContext('2d')`、`getBoundingClientRect()` 量尺寸。判断标准：**凡是"命令式"的操作（做一下、动一下），声明式 props 表达不了，就走 ref。**

### 3.4 用途三：拿到子组件（forwardRef + useImperativeHandle）

这是本章最长、最容易卡住的一段（9:39–13:15）。目标：**父组件点按钮，执行子组件内部定义的一个方法。**

#### 第一步：直接挂 ref，会失败

```jsx
function Child() { return <div>子组件</div>; }

const childRef = useRef(null);
<Child ref={childRef} />       // ❌ 报错
```

作者的描述（10:10–10:47）：「它会给告诉你这个操作是**有问题**的……因为默认来说，我们子组件是**不对外开放它内部的功能**的。」真实报错：

```
Warning: Function components cannot be given refs.
Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

而且 `childRef.current` 是 `null`，调方法直接 `TypeError`。📌 **这个设计是故意的**：函数组件没有实例，React 不希望你绕过 props 去摸它的内部。

#### 第二步：forwardRef 让子组件"愿意接收 ref"

「我们的函数组件需要通过一个 React 的方法来做处理，方法叫做 `forwardRef`。我们把整个函数包在这个 `forwardRef` 里面……设置完毕以后，子组件还需要去**接收一下外部传入的 ref**……我们的参数 2 就是我们要使用的 ref。」（10:47–11:27）

```jsx
import { forwardRef } from 'react';

const Child = forwardRef(function Child(props, ref) {
  //                    ↑ 第一个参数还是 props   ↑ 第二个就是父组件传进来的 ref
  return <div ref={ref}>子组件</div>;
});
```

> 📌 **纠正一个细节**：作者说"不能直接定义为一个普通的函数，而是要通过 `const` 这种函数表达式"。**真正的要求是"必须用 `forwardRef` 包一层"，不是"必须用 const"**。`function ChildInner(props, ref) {...}; export default forwardRef(ChildInner);` 同样合法。他用 `const Child = forwardRef(...)` 是因为这样 DevTools 里名字好看（匿名会显示成 `ForwardRef`）。

#### 第三步：useImperativeHandle 决定"开放哪几个方法"

到这一步父组件只能拿到子组件的 **DOM**，**还拿不到内部函数**。作者踩了这个坑（11:37–11:54）：「你如果直接在子组件内部封装函数的话，像这种普通的 `handleClick` 是**不可能被外界直接使用**的，因为它是一个局部作用域内部的函数。」

解法（12:03–12:09）：「需要使用 `useImperativeHandle` 这个方法……参数 1 传入的是你外部设置的这个 ref……参数 2 是一个函数，这个函数的**返回值是一个对象**，所以我们需要在这个对象外面包裹一个小括号。」

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

// 子组件：只开放 focusInput 和 sayHi，别的一概不给
const Child = forwardRef(function Child(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focusInput() { inputRef.current?.focus(); },
    sayHi() { console.log('子组件的 sayHi 被父组件调用了'); },
  }), []);   // 第三个参数是依赖数组，和 useMemo 一个规矩

  return <input ref={inputRef} placeholder="子组件里的输入框" />;
});

export default function App() {
  const childRef = useRef(null);
  return (
    <div>
      <Child ref={childRef} />
      <button onClick={() => childRef.current?.sayHi()}>调用子组件方法</button>
      <button onClick={() => childRef.current?.focusInput()}>让子组件输入框聚焦</button>
    </div>
  );
}
```

结果（13:04–13:15）：「现在我们去点击按钮，你就发现呢，在控制台中就输出了子组件 `sayHi` 方法的一个执行的效果。」链路是 `childRef.current` → 子组件 `useImperativeHandle` 返回的那个对象 → 只有 `focusInput` / `sayHi` 两个方法，**不是整个 DOM，也不是子组件的所有内部函数**。

> 📌 **补充：什么时候真的该用它**
>
> 这是"父调子"的**唯一正规通道**，但**能不用就不用**。多数需求（打开弹窗、聚焦、滚动、播放）可以改成"状态提升 + props 控制"：
>
> ```jsx
> // ❌ 命令式：父组件必须知道子组件有 open 这个方法
> <Modal ref={modalRef} />
> <button onClick={() => modalRef.current.open()}>打开</button>
>
> // ✅ 声明式：父组件只说「现在该开」，子组件自己决定怎么开
> <Modal open={isOpen} onClose={() => setIsOpen(false)} />
> ```
>
> 真正适合它的是**"动作"而非"状态"**：`focus()`、`scrollTo()`、`play()`、`reset()`。`EP02` §14⑤ 有弹窗版对照（那里只有 8 行，本集是完整三步）。**React 19 起 `ref` 已是普通 prop**，`forwardRef` 变成可选。

### 3.5 useRef vs useState

| | `useState` | `useRef` |
|---|---|---|
| 存什么 | 要显示在界面上的数据 | 不需要显示、但要跨渲染记住的数据 |
| 改了会重渲染吗 | ✅ 会 | ❌ 不会 |
| 跨渲染存活吗 | ✅ 会 | ✅ 会 |
| 读到的值 | 这一轮渲染的快照 | **永远是最新写入的**（`ref.current`） |
| 典型用途 | 表单值、开关、列表 | 定时器 id、上一次的值、DOM 节点、是否首次渲染 |
| 渲染期读写安全吗 | 读安全（是快照） | **读写都不安全**（会破坏纯度） |

一句话记法：**要"看见"就用 `useState`；要"记住但不看见"就用 `useRef`。**

---

## 4. 第三章 useEffect：组件和外部世界打交道的窗口

### 4.1 前提：React 要求函数组件是"纯"的

作者的切入角度很准（13:16–13:31）：「React 要求我们所有的函数式组件都是**纯函数**，意味着**同样的输入应该有同样的输出**。如果你希望设置副作用的话，那我们就可以使用像事件这种功能。」

同一个 `props` + 同一个 `state`，渲染出来的东西必须一模一样。下面这些都**不纯**，因为它们碰到了组件之外的世界：

| 副作用 | 为什么不能在渲染期做 |
|---|---|
| `fetch('/api')` | 每次渲染都发一遍请求，StrictMode 下还发两遍 |
| `setInterval(...)` | 渲染可能被中断重跑，定时器越积越多 |
| `document.title = ...` | 改的是组件之外的东西 |
| `localStorage.setItem(...)` | 同上 |
| `console.log` | 严格说也是，但无害，所以大家都用它调试 |

`useEffect` 的作用就是：**把这些活儿挪到"渲染完成、DOM 已更新"之后再执行。**

### 4.2 最小用法：不传依赖数组

```jsx
import { useState, useEffect } from 'react';

export default function Demo() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('副作用执行了，当前 count =', count);
    document.title = `点了 ${count} 次`;
  });

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

作者的描述（13:57–14:11）：「它呢默认是在我们组件**渲染的时候重新执行**。我们保存，这时我们看到呢，它**默认执行了两次**，这个是因为我的 React 开启了**严格模式**，它默认会对每个函数组件进行两次执行，以检查是否出现**函数不纯**的情况。只要我去点击按钮状态变更组件重新渲染，就会导致 `useEffect` 的重新执行。」

这里同时发生了两件事，很容易混：**"跑了两次"是 StrictMode，"又跑一次"是状态变更。**

### 4.3 为什么 StrictMode 下 useEffect 会跑两次

作者说严格模式"对每个函数组件执行两次，检查是否函数不纯"。📌 补充完整机制——React 18 的 `StrictMode` 在**开发环境**下做的是**两件**事：

1. **渲染阶段双调用**：组件函数连续执行两次，**只采用第二次的结果**。目的是抓"渲染期有副作用"（比如在函数体里改外部变量）。
2. **挂载时 effect 的 setup → cleanup → setup**：副作用会**执行、立刻清理、再执行**。目的是抓"cleanup 写漏了"。这一条才是"控制台打印两次"的主因。

```
开发环境 + <StrictMode>，首次挂载的真实顺序：

  render #1  ──► 结果被丢弃（但你的 console.log 已经打出来了）
  render #2  ──► 采用
     │
     ├─ 提交 DOM（commit）
     ├─ effect 的 setup    ← 第 1 次执行（你看到的第一条日志）
     ├─ effect 的 cleanup  ← 立刻清理（没写 return 就什么也不发生）
     └─ effect 的 setup    ← 第 2 次执行（你看到的第二条日志）
```

| | 开发 + StrictMode | 开发（无 StrictMode） | 生产构建 |
|---|---|---|---|
| 组件函数执行次数 | **2 次** | 1 次 | 1 次 |
| 挂载时 effect 执行次数 | **2 次**（中间夹一次 cleanup） | 1 次 | 1 次 |
| 状态更新后的 effect | 1 次 | 1 次 | 1 次 |
| 目的 | **提前暴露**：漏 cleanup、渲染期副作用、不纯的 reducer | 无检查 | 性能优先 |

**怎么用这个"双执行"自检？** 它是探针，不是噪音：

```jsx
// ❌ 漏了 cleanup：StrictMode 下留下两个定时器，控制台每秒打两条
useEffect(() => { setInterval(() => setCount(c => c + 1), 1000); }, []);

// ✅ 有 cleanup：不管跑几次，最后只活一个
useEffect(() => {
  const timer = setInterval(() => setCount(c => c + 1), 1000);
  return () => clearInterval(timer);
}, []);
```

> 判断口诀：**"跑两次"本身不用修；"跑两次之后留下了两份东西"必须修。** 如果两遍执行后资源只剩一份（cleanup 生效了），代码就是对的。

### 4.4 依赖数组：三种写法

作者的讲法（14:12–15:05）：「如果我们仅仅希望在组件渲染的时候**执行一次**以后就不再变更的话，我们可以在 `useEffect` 的参数 2 传入一个**数组**，这个数组代表是一个**依赖数组**，也就是哪些状态变化会导致副作用函数重新执行。由于我们这里传的是一个**空数组**，所以意味着任何状态变更都不会导致副作用重新执行。」

```jsx
useEffect(fn)           // 不传   → 每次渲染提交后都跑
useEffect(fn, [])       // 空数组 → 只在挂载后跑一次
useEffect(fn, [count])  // 有依赖 → 挂载后 + count 变化后
```

| 写法 | 视频里的现象 | 准确含义 |
|---|---|---|
| 不传 | 「只要我去点击按钮状态变更，就会导致 `useEffect` 重新执行」 | 每次渲染提交后都执行 |
| `[]` | 「无论你如何操作，底部都不会出现 `useEffect` 的重复触发」 | 挂载后一次；**卸载时**执行 cleanup 一次 |
| `[count]` | 「每次 `count` 的变化都会导致 `useEffect` 的一个重复执行」 | 挂载后一次 + `count` 每次变化后一次 |

执行时序（`count` 从 0 变成 1，依赖是 `[count]`）：

```
用户点击
  │
  ├─ 1. render 阶段：重新执行组件函数，算出新 JSX（还没碰 DOM）
  │        └─ effect 里的代码这一轮【不执行】
  ├─ 2. commit 阶段：React 把差异写进真实 DOM
  ├─ 3. 浏览器绘制（paint）——用户已经看到新界面了
  ├─ 4. 执行【上一轮】effect 的 cleanup（如果写了 return）
  └─ 5. 执行【这一轮】effect
```

记住第 4 步在第 5 步**之前**：**先收尾上一次，再开始这一次**。这就是为什么 `clearInterval` 永远能清掉上一次那个。

### 4.5 📌 依赖数组到底写什么：四条判断规则

视频只说了"哪些状态变化会导致重新执行"，没说"该往里写哪些"。这是实战里 90% 的 `useEffect` bug 来源。

**规则 1：effect 里读到的、会变的、来自组件作用域的东西，全写。**

```jsx
// ✅ 读了 count 和 step，两个都写
useEffect(() => { document.title = `${count} / step ${step}`; }, [count, step]);
```

**规则 2：这些不用写**（写了反而找麻烦）——`setCount` / `dispatch`（引用永远不变）、`useRef` 返回的 ref 对象（对象本身不变）、组件外定义的函数与常量、`import` 进来的模块。

**规则 3：不要为了"只跑一次"而撒谎。**

```jsx
// ❌ 撒谎：读了 count，却声称"没有依赖" → 永远打印 0
useEffect(() => { console.log(count); }, []);
```

**规则 4：拿不准就别手写，让 ESLint 算。**

```bash
npm i -D eslint-plugin-react-hooks
```

```json
{ "extends": ["plugin:react-hooks/recommended"] }
```

`exhaustive-deps` 规则会把漏写的依赖标红并给出一键修复。📌 这是**唯一一个不用理解原理就该先装上的规则**。

| 场景 | ❌ 反例 | ✅ 正例 | 写错的后果 |
|---|---|---|---|
| 用了 state 却写空数组 | `useEffect(() => { fetch('/u/' + id) }, [])` | `}, [id])` | 切到别的用户，页面不刷新 |
| 用了 props 却没写 | `useEffect(() => { setLocal(value) }, [])` | `}, [value])` | 父组件传新值不生效 |
| 依赖对象字面量 | `useEffect(fn, [{ a: 1 }])` | 提到组件外或用 `[a]` | 每次都是新对象 → **无限循环** |
| 依赖内联函数 | `useEffect(fn, [() => {}])` | `useCallback` 包一层 | 同上 |
| 依赖整个对象 | `useEffect(fn, [user])` | `useEffect(fn, [user.id])` | 父组件每次新建对象 → 白跑 |

### 4.6 📌 闭包陷阱（stale closure）：视频没讲，但这是 useEffect 第一大坑

视频那句话埋着这个雷："这个数组代表的是**哪些状态变化**会导致副作用函数重新执行"——言下之意是"没写进去的状态，effect 里就看不到它的新值"。为什么？因为 **effect 里的函数是一个闭包，捕获的是"创建它的那一轮渲染"里的变量**。

```jsx
import { useState, useEffect } from 'react';

export default function BadTimer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log('闭包里看到的 count =', count);  // 永远打印 0
      setCount(count + 1);                        // 永远算 0 + 1 = 1
    }, 1000);
    return () => clearInterval(timer);
  }, []);   // ← 空数组：effect 只跑这一次，闭包被"冻"在这里

  return <p>{count}</p>;   // 结果：1、1、1、1……
}
```

```
挂载时：count = 0
  ├─ render 执行 → 创建 effect 闭包 A
  │                  A 里面 count = 0  ← 被"拍了张照"，之后再也不变
  ├─ effect 执行 → setInterval(回调A)

1 秒后：回调A 执行
  ├─ 读 count → 从闭包 A 里读 → 0（不是现在的 0，是"拍照时的 0"）
  ├─ setCount(0 + 1) → 1 → 触发重渲染
  │        └─ 组件函数重新执行，count 变成 1，界面显示 1
  │        └─ 但依赖是 []，effect 不重跑
  │              └─ 定时器里【还是回调A】，回调A 里 count【还是 0】

2 秒后：回调A 又执行 → setCount(0 + 1) → 还是 1 → 状态没变 → 不重渲染
```

**病因一句话：`setInterval` 里的回调生在"count = 0 的那一轮渲染"，它这辈子只能看见 0。** 这和 `useRef` 正好相反——`ref.current` 每次读都是最新的，闭包变量永远是创建时那一刻的。

| 解法 | 代码 | 适合 | 代价 |
|---|---|---|---|
| **A. 函数式更新** | `setCount(c => c + 1)`，deps 保持 `[]` | 只是"基于旧值算新值" | 只能用在 `setState` 上，读不到别的值 |
| **B. 补齐依赖** | `}, [count])` | 需要在 effect 里读最新值做别的事 | 每次变化都重建定时器（本例每秒 clear + set） |
| **C. 用 ref 存最新值** | `countRef.current = count` | 想在回调里读最新值、又不想重建订阅 | 多一个 ref，代码稍绕 |

```jsx
// A：最省，也最常用
useEffect(() => {
  const timer = setInterval(() => setCount(c => c + 1), 1000);
  return () => clearInterval(timer);
}, []);

// C：适合「只在回调里读、不想重订阅」
const countRef = useRef(count);
useEffect(() => { countRef.current = count; }, [count]);
useEffect(() => {
  const timer = setInterval(() => console.log('最新值：', countRef.current), 1000);
  return () => clearInterval(timer);
}, []);
```

> 📌 **怎么快速判断自己踩了？** 三个症状：① `console.log` 打出来的值和界面显示的对不上；② 界面"卡"在某个值，怎么点都不动；③ 把依赖补全就正常了（说明是 deps 的问题，不是逻辑的问题）。
>
> **根治办法不是背这三种解法，而是让 eslint 把 deps 补全**，然后接受"effect 会重跑"。真正该优化的是"重跑会不会出问题"（有没有 cleanup），不是"怎么让它别跑"。

### 4.7 清理函数与依赖数组的联动

作者本集**没讲 cleanup**（已有笔记 §5.5 讲得完整），但它和 deps 的关系必须补一句：**依赖数组里每多一个依赖，cleanup 的重要性就多一分。** `[]` 的 effect 漏了顶多泄漏一次；`[count]` 的 effect 漏了，就是**每变一次多漏一个定时器**——`setState` 越跑越快，关掉页面还在后台跑。

```jsx
useEffect(() => {
  const timer = setInterval(tick, 1000);
  return () => clearInterval(timer);   // ← 没有这一行，[count] 每变一次就多起一个定时器
}, [count]);
```

---

## 5. 第四章 useMemo：缓存「算出来的值」

### 5.1 场景：一个"很贵"的子组件

作者的示例（15:05–16:38）：父组件 `App` 里一个输入框 + 一个子组件 `DoSomeMath`，子组件收到输入值后做复杂计算。他先承认"这个本身是没有什么问题的"（16:05）：「因为你的每次计算结果是一定与你输入内容要有直接关联的，输入数据变更计算结果一定要改。」

问题出在**另一个状态**上——他在父组件里加了一个跟计算毫无关系的 `count`：「由于我们现在的 `DoSomeMath` 是一个子组件，而子组件会随着父组件状态变化而变化……我们的父组件出现了状态的变更，导致重新渲染，这时呢我们内部子组件也同样会重新执行一次计算操作。」

```jsx
import { useState } from 'react';

// 假装很贵（真实项目里是排序、过滤、统计、格式化大列表）
function expensiveSum(n) {
  const big = Array.from({ length: 2_000_000 }, (_, i) => i % 100);
  return big.reduce((a, b) => a + b, 0) + n * 2;
}

function DoSomeMath({ value }) {
  const result = expensiveSum(value);   // ← 每次父组件渲染，这里都重算
  return <p>计算结果：{result}</p>;
}

export default function App() {
  const [value, setValue] = useState(1);
  const [count, setCount] = useState(0);   // 和计算完全无关的状态
  return (
    <div>
      <input value={value} onChange={e => setValue(Number(e.target.value))} />
      <button onClick={() => setCount(count + 1)}>无关按钮（{count}）</button>
      <DoSomeMath value={value} />
    </div>
  );
}
```

现象（16:38–17:17）：「比如我们去点击 `count` 的变更，你发现我们这个 `DoSomeMath` **同样会计算**，但是**内部的数据并没有变更**。如果这时它是一个非常复杂计算的话，你外部的变更就会导致内部计算重新执行是**没有任何意义**的。」

### 5.2 useMemo 改造

「你直接通过 `useMemo` 来创建一个可以进行缓存的操作空间。这时我们写一个函数，将你可能需要进行缓存的复杂操作放入，然后在我们外部去接收一个结果……它就会将这个计算的结果作为返回值返回。同时呢，你还需要在 `useMemo` 的参数 2 中去输入一些关联的数据，这一点跟刚才我们使用的 `useEffect` 是比较相似的，它是一个**依赖数组**。」（17:17–17:46）

```jsx
import { useMemo } from 'react';

function DoSomeMath({ value }) {
  // 只有 value 变了才重算；别的渲染直接拿上一轮的缓存
  const result = useMemo(() => expensiveSum(value), [value]);
  return <p>计算结果：{result}</p>;
}
```

验证（17:48–18:12）：「我们现在如果直接变更输入框，它还是会正常执行的……但如果现在我们去点击外部父组件的状态变更，导致父组件重新渲染的时候，你发现状态变了，渲染也重新执行了，但是**没有导致子组件中指定的这个区域重新执行**。」

### 5.3 它到底省了什么：省计算，不省渲染

这是本节最需要说清的一点——作者说的"区域"指的是**那段计算**，不是组件：

```
父组件 setCount(1)
   │
   ├─ App 函数重新执行                        ← 省不掉
   ├─ <DoSomeMath value={1} /> props 没变，但没 memo
   │     └─ DoSomeMath 函数重新执行           ← 省不掉（"渲染也重新执行了"）
   │           ├─ useMemo(() => expensiveSum(value), [value])
   │           │     └─ value 没变 → 返回上次的结果  ✅ 省掉了（函数根本没被调用）
   │           └─ return <p>…</p>             ← 重建 JSX，但 diff 后不写 DOM
   └─ 提交 DOM（无变化，不写）
```

| 想省的东西 | 该用的工具 | 说明 |
|---|---|---|
| 省「重复计算」 | `useMemo` | 组件照样重跑，只是跳过那段贵逻辑 |
| 省「组件重渲染」 | `React.memo` | 见第 6 章 |
| 省「函数重建」 | `useCallback` | 见第 6 章 |

### 5.4 什么时候值得用

| 场景 | 值得吗 | 理由 |
|---|---|---|
| 遍历/排序/过滤上万条数据 | ✅ | 省的是真实的几十毫秒 |
| 结果要作为别的 Hook 的依赖 | ✅ | 保持引用稳定，避免下游 effect 反复触发 |
| 深度递归、复杂格式化、正则大文本 | ✅ | 同上 |
| 父组件有高频无关更新（输入、滚动、拖拽） | ✅ | 每次按键都重算的痛感很明显 |
| `count * 2`、`a + b`、`arr.length` | ❌ | **`useMemo` 自己的比较开销就超过计算本身** |
| 子组件本来就在重渲染，且计算依赖它的 props | ❌ | 依赖变了照样重算，缓存等于没写 |
| 没测过，只是"感觉会慢" | ❌ | 先跑起来，用 Profiler 看火焰图再决定 |

```jsx
const double = useMemo(() => count * 2, [count]);   // ❌ 比不加还慢
const double = count * 2;                           // ✅ 直接算
```

📌 **判断口诀：先问"这段代码重跑一次要多久"。超过 1ms 才值得考虑；低于 1ms，`useMemo` 是负优化。**

---

## 6. 第五章 useCallback：缓存「函数本身」

### 6.1 场景：memo 了子组件，它却还在渲染

作者的示例（18:12–19:21）：父组件 `App` 有一个 `count`，还有一个子组件 `Button`，点击事件由父组件通过 props 传下去。

```jsx
import { useState } from 'react';

function Button({ onClick }) {
  console.log('🔁 Button 重新渲染了');
  return <button onClick={onClick}>子组件按钮</button>;
}

export default function App() {
  const [count, setCount] = useState(0);
  const handleClick = () => { console.log('按钮被点了'); };

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>父组件 count: {count}</button>
      <Button onClick={handleClick} />
    </div>
  );
}
```

观察（19:09–19:33）：「当父组件操作时会导致子组件的重新渲染，这时呢可能不是复杂操作，而是**子组件整体重新渲染**的问题。」

### 6.2 第一步：把子组件变成「记忆组件」

「我们需要将这个组件本身设置为一个**记忆组件**……使用 `memo` 方法来进行处理。这个 `memo` 并**不是**刚才我们那个 `useMemo`，而是 React 本身的一个 `memo` 方法。它可以将我们的组件变更为一个记忆组件，这时变更完毕以后，如果你给这个组件传入的 **prop 没有出现任何变化**的话，它就**不会受到外部父组件更新的影响**。」（19:33–19:53）

```jsx
import { memo } from 'react';

const Button = memo(function Button({ onClick }) {
  console.log('🔁 Button 重新渲染了');
  return <button onClick={onClick}>子组件按钮</button>;
});
```

> 注意作者特意强调的区别：`memo`（从 `react` 直接导入、包组件、浅比较 props）≠ `useMemo`（Hook、包一段计算、返回一个值）。名字像，作用层级完全不同。

### 6.3 可是加了 memo 还是渲染了

作者现场遇到了（20:07–20:12）：「保存以后呢，现在你去点击这个按钮，你会发现**哎，好像并没有实现我所说的效果**。」

原因（20:15–20:30）：「问题其实在于这个 `onClick`，因为我们的父组件重新渲染以后，你的 `handleClick` 是不是变成了一个**新的函数**呀？我们知道函数跟函数如果并不是**相同引用**的话，JS 会认为它并不是同一个函数。」

```js
const f1 = () => {};
const f2 = () => {};
f1 === f2;   // false —— 内容一样，但引用不同
```

```
父组件每次渲染：
  const handleClick = () => {...}   ← 内存里新建了一个函数对象
        │
        └─► <Button onClick={handleClick} />
                              └─► memo 浅比较：上次的 onClick === 这次的 onClick ?
                                        └─ false（新函数） → 重新渲染 ❌
```

总结（20:30–21:07）：「我们如果希望子组件不会重新渲染的话，就从「组件本身是否会重新渲染」，变更为了「你传入的这个方法能不能是同一个」的问题。如果我们能够在 `App` 进行重新渲染的时候**保持 `handleClick` 不被重新创建**……子组件发现「你给我传入的所有的 prop 都是完全相同的」，这时呢，我就不会重新渲染了。」

### 6.4 第二步：useCallback 缓存这个函数

「参数 1 是你希望缓存的**函数**，而参数 2 同样是一个**依赖数组**。如果我们现在没有任何依赖的话，那我就什么也不写。」（21:07–21:25）

```jsx
import { useState, memo, useCallback } from 'react';

const Button = memo(function Button({ onClick }) {
  console.log('🔁 Button 重新渲染了');
  return <button onClick={onClick}>子组件按钮</button>;
});

export default function App() {
  const [count, setCount] = useState(0);

  // 依赖数组为空 → 永远是同一个函数引用
  const handleClick = useCallback(() => { console.log('按钮被点了'); }, []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>父组件 count: {count}</button>
      <Button onClick={handleClick} />
    </div>
  );
}
```

验证（21:26–22:03）：「我去点击按钮，你发现它并不会出现任何的子组件的重新渲染操作。因为我们的子组件本身已经通过 `memo` 设置为了一个记忆组件，而且父组件传递给我们的所有的 prop 当前只有一个就是 `onClick`，它跟之前的函数也是同一函数。」

### 6.5 三件套的关系：memo 负责"拦"，useMemo/useCallback 负责"让拦得住"

| 工具 | 缓存的对象 | 用在谁身上 | 一句话 |
|---|---|---|---|
| `React.memo(Component)` | **组件**的渲染结果 | 子组件 | "props 没变就别重跑我" |
| `useMemo(fn, deps)` | `fn` 的**返回值** | 当前组件内部 | "deps 没变就别重算这个值" |
| `useCallback(fn, deps)` | `fn` **本身** | 当前组件内部 | "deps 没变就别重造这个函数" |

三者是一条链，**缺一环就全废**：

```
useCallback（保住函数引用）
      │
      ▼
memo（比对 props 引用，发现全都一样）
      │
      ▼
子组件不渲染（这才省到了）
```

反过来：**子组件没用 `memo`，`useCallback` 基本没用**（父组件一渲染，子组件无条件重跑，函数引用稳不稳定没人看）。

> 恒等式：`useCallback(fn, deps)` ≡ `useMemo(() => fn, deps)`。`useCallback` 只是语法糖，让你少写一层 `() =>`。

### 6.6 什么时候值得用

| 场景 | 值得吗 | 说明 |
|---|---|---|
| 传给 `memo` 子组件的函数 prop | ✅ | 不加就等于白 memo |
| 函数要放进 `useEffect` 的依赖数组 | ✅ | 否则 effect 每次渲染都重跑 |
| 函数要传给 Context 的 `value` | ✅ | 否则所有消费者重渲染 |
| 普通子组件（没 memo）的回调 | ❌ | 子组件反正要重渲染 |
| 只在 JSX 里内联用一次的函数 | ❌ | 引用了也没人比较 |
| 为了"看起来专业"全量包一层 | ❌ | 依赖数组写错反而引入闭包陷阱 |

```jsx
// ❌ 给不 memo 的子组件包 useCallback：白包，还多一个依赖数组要维护
const handleChange = useCallback((e) => setValue(e.target.value), []);
<NormalInput onChange={handleChange} />   // NormalInput 没 memo，照样重渲染
```

> 📌 **补充：`memo` 最容易失效的三种情况**
>
> 1. **props 里传了新对象/新数组**：`<Child style={{color:'red'}} />` 每次都是新对象。解法：提到组件外做常量，或 `useMemo` 包起来。
> 2. **props 里传了内联 JSX**：`<Child icon={<Icon />} />` 每次都是新元素。
> 3. **props 里有 `children`**：`<Child><span>hi</span></Child>` 里的 `<span>` 每次都是新对象（纯文本 `"hi"` 是字符串，同值相等，不受影响）。
>
> 所以"加了 memo 还在渲染"时，**先 `console.log(props)` 看是哪个 prop 每次都变**，别急着怀疑 `memo` 坏了。

---

## 7. 全局复盘：五个 Hook 各站哪个位置

### 7.1 一次渲染里，它们分别在干什么

```
┌──────────────────────── 组件函数执行（render 阶段）────────────────────────┐
│  const [s, dispatch] = useReducer(...)      ← 状态：改法收口（第 1 章）      │
│  const box = useRef(null)                   ← 盒子：变了不渲染（第 3 章）    │
│  const result = useMemo(() => 贵计算(x), [x]) ← 省计算（第 5 章）            │
│  const fn = useCallback(() => {}, [])         ← 省函数重建（第 6 章）        │
│  return <div ref={box}>{result}</div>       ← 产出 JSX（必须是纯的）        │
└──────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼  commit：写真实 DOM
                                    ▼  paint：用户看到界面
┌─────────────────────────── useEffect（第 4 章）─────────────────────────────┐
│  cleanup（上一轮） → effect（这一轮）                                         │
│  请求 / 定时器 / 订阅 / 手动改 DOM —— 所有"渲染之外"的活儿                    │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 四句口诀

| 你遇到的问题 | 用它 | 判断依据 |
|---|---|---|
| "这个状态有五六种改法，散得到处都是" | `useReducer` | 改法数量 > 状态数量 |
| "这个值我要跨渲染记住，但不想让它触发渲染" | `useRef` | 变了**不需要**出现在界面上 |
| "渲染完之后我还得干点事" | `useEffect` | 有"之后"这个时间概念 |
| "这次渲染和上次一模一样，能不能别干了" | `useMemo` / `useCallback` + `memo` | 有实测的性能痛点 |

判断「要不要加优化」的顺序：**没测过就别加 → 测过发现慢在「计算」→ `useMemo`（且单次耗时 < 1ms 也别加）→ 慢在「重渲染」→ `React.memo`，同时 props 里的函数必须 `useCallback`、对象/数组/JSX 必须先稳定下来，否则 memo 白 memo。**

---

## 8. 速查卡

| Hook | 签名 | 返回 | 会触发渲染吗 | 依赖数组 | 典型误用 |
|---|---|---|---|---|---|
| `useReducer` | `useReducer(reducer, init, initFn?)` | `[state, dispatch]` | `dispatch` 后 ✅ | 无 | reducer 里写副作用；`default` 返回 `undefined` |
| `useRef` | `useRef(initialValue)` | `{ current }` | ❌ 永远不会 | 无 | 渲染期读 `.current`；指望它驱动界面 |
| `useEffect` | `useEffect(fn, deps?)` | `undefined`（可 return cleanup） | 内部 `setState` 会 | `[]` / `[a,b]` / 不传 | 漏依赖 → 闭包陷阱；漏 cleanup → 资源叠加 |
| `useMemo` | `useMemo(fn, deps)` | `fn` 的返回值 | ❌ | 必填 | 给廉价计算加缓存；把 `useMemo` 当 `memo` 用 |
| `useCallback` | `useCallback(fn, deps)` | 同一个 `fn` 引用 | ❌ | 必填 | 子组件没 `memo` 却包 `useCallback` |

| 配套 API | 作用 | 和谁搭配 |
|---|---|---|
| `React.memo(Component)` | 浅比较 props，相同就跳过渲染 | `useCallback` / `useMemo` |
| `forwardRef(fn)` | 让函数组件能接收 `ref` | `useImperativeHandle` / DOM ref |
| `useImperativeHandle(ref, fn, deps?)` | 限定对外暴露的方法 | `forwardRef` |

---

## 9. 动手路线

每步都要**先制造 bug，再修好**——只看正确代码学不会。

- [ ] **1. 计数器改 `useReducer`**：加 `'double'` 和 `'setTo'`（带 `payload`），体会"所有改法在一个 switch 里"。
- [ ] **2. `useRef` 做「上一次的值」**：故意把 `prevCountRef.current = count` 写在 `setCount` **之后**，看显示的是什么（这就是顺序的重要性）。
- [ ] **3. 3 秒后自动聚焦**：`useEffect(() => { setTimeout(() => inputRef.current?.focus(), 3000) }, [])`，然后去掉 `?.` 再卸载组件，看报什么错。
- [ ] **4. 抄一遍 `forwardRef` + `useImperativeHandle`**：先故意不包 `forwardRef`，把那条 warning 完整读一遍；再故意让第二个参数返回函数而不是对象，看报错。
- [ ] **5. 亲手制造闭包陷阱**：抄 §4.6 的 `BadTimer`，改成每秒 +1；再用 A/B/C 三种解法各修一次，比较控制台输出。
- [ ] **6. 做一次「加不加 memo」对照实验**：在 §6.4 的 `Button` 里加 `console.log`，分别在「不 memo / memo 但不 useCallback / memo + useCallback」三种状态下点父组件按钮，数日志条数。**这一步做完，你就真的懂 `useCallback` 了。**

---

## 10. 自测题（含答案）

| # | 问题 | 答案 |
|---|---|---|
| 1 | reducer 的返回值是什么？可以直接改 `state` 吗？ | 返回**新的** state；不能改旧的（必须纯，StrictMode 会调两次） |
| 2 | `dispatch` 要写进 `useEffect` 的依赖数组吗？ | 不用，React 保证它的引用永远不变 |
| 3 | `useRef` 改了 `.current`，界面会更新吗？ | 不会。要等**别的原因**触发重渲染才看得到 |
| 4 | 为什么 `useRef` 要包一层 `.current`？ | 保证"盒子"的引用永远不变，变的只能是内容 |
| 5 | 函数组件直接挂 `ref` 会怎样？ | 报 `Function components cannot be given refs`，`ref.current` 是 `null` |
| 6 | `forwardRef` 的回调接收几个参数？ | `(props, ref)`，第二个就是父组件传来的 ref |
| 7 | `useImperativeHandle` 第二个参数返回什么？ | **一个对象**，键就是暴露给父组件的方法名 |
| 8 | StrictMode 下 effect 跑两次，要不要修？ | "跑两次"不用修；"跑两次留下两份资源"必须修（补 cleanup） |
| 9 | `useEffect(fn, [])` 里读了 `count` 会怎样？ | 永远读到首次渲染的 `count`（闭包陷阱） |
| 10 | 依赖数组里放 `{ a: 1 }` 会怎样？ | 每次渲染都是新对象 → effect 每次都跑 → 可能无限循环 |
| 11 | `useMemo` 能阻止子组件重渲染吗？ | 不能。只跳过计算；要阻止重渲染用 `React.memo` |
| 12 | `useMemo(() => count * 2, [count])` 有问题吗？ | 有。计算太廉价，比较开销更大，直接算 |
| 13 | 子组件加了 `memo` 还在渲染，最常见原因？ | 传进去的函数/对象/JSX 每次都是新引用 |
| 14 | `useCallback(fn, [])` 等价于什么？ | `useMemo(() => fn, [])` |
| 15 | 子组件没 `memo`，父组件用 `useCallback` 有用吗？ | 基本没用，父渲染子必渲染，没人比较函数引用 |

---

## 11. 📌 与已有笔记的差异

### 11.1 重叠：一句话带过，别重复看

| 主题 | 本集怎么讲 | 去哪儿看更完整的 |
|---|---|---|
| `useState` 基础 | 只在计数器里当工具用，没讲原理 | 已有笔记 §4、`EP01` §8–§10 |
| `useContext` | 开头一句"之前讲过了" | `EP02` §8 |
| `useEffect` 三种依赖写法 | 演示了现象，没讲 cleanup | 已有笔记 §5.4–§5.5（更完整） |
| `useMemo` / `useCallback` 概念 | 各用一个例子演示"能缓存" | 已有笔记 §11.2–§11.3 |
| 「别过早优化」 | **完全没提**（甚至反着来，见冲突①） | 已有笔记 §11.4（这是它的核心观点） |
| 受控组件 / 表单 / 自定义 Hook | 没讲 | 已有笔记 §9.2、§11.1 |

### 11.2 新增：已有笔记完全没有的

| 主题 | 增量点 | 为什么重要 |
|---|---|---|
| **`useReducer`** | 全套：reducer 签名、`switch` + `default` 抛错、`dispatch` 用法、与 `useState` 选型表 | 两份已有资料里**一次都没出现**，但它是中大项目的标配（Redux 的心智模型就是它） |
| **`useRef` 用途一：记住上一次的值** | 完整时序图 + `usePrevious` 封装 | 已有笔记只在 §9.2 提了一句"非受控组件用 ref 取值"，会让人以为 ref 只是 DOM 工具 |
| **`forwardRef` + `useImperativeHandle` 三步** | 从"直接挂 ref 失败"到"暴露白名单方法"的完整过程 | `EP02` §14⑤ 只有 8 行代码补充，没有"为什么失败"这一步 |
| **StrictMode 与 effect 的关系** | 渲染双调用 + setup→cleanup→setup 完整时序 | `EP01` §3 讲了 StrictMode 双执行，没讲它**和 cleanup 的关系** |
| **`React.memo` 的完整用法** | 只做浅比较、什么时候失效 | 已有笔记 §9.5 只在讲 useCallback 时带了一句 |
| **memo 三件套链路图** | "缺一环全废"的因果链 | 已有笔记讲了区别，没讲"为什么必须三个一起用" |

### 11.3 说法冲突：四处需要你知道"该信谁"

**冲突 ①：`useMemo`/`useCallback` 的使用门槛**

| | 已有笔记 §11.4 | 本集（视频） |
|---|---|---|
| 立场 | "**只有真正遇到性能问题时才用**……过早地到处套这些，只会让代码更难读" | 直接当常规手段教："你希望缓存，就可以用" |
| 演示场景 | `NotesWidget` 里真实的大列表统计 | **人为加一个无关的 `count`**，制造"子组件白算一次" |
| 判断顺序 | 先写简单代码 → Profiler 定位 → 再优化 | 没提 |

**结论**：视频的演示场景是**为了让你「看见」缓存生效**（不然控制台没东西可看），不是为了说明"这种场景该优化"。真实项目按已有笔记的顺序走。**但有一个例外**：如果那个函数是传给 **`memo` 子组件**的，且子组件在长列表里渲染很多次，那么**一开始就加 `useCallback` 是合理的**——这不是过早优化，而是"让 `memo` 生效的必要条件"（见 §6.5）。

**冲突 ②：StrictMode 为什么要跑两次**

| | 视频 | `EP01` §3 |
|---|---|---|
| 说法 | "对每个函数组件进行两次执行，以检查是否出现**函数不纯**的情况" | 渲染双调用 + 挂载时 **setup → cleanup → setup**，专查"副作用没清理" |
| 覆盖面 | 只提到"函数不纯" | 两种检查都提到 |

**结论**：视频说对了一半，漏了更要紧的那一半。React 18 对 **effect** 的检查方式是"故意多跑一次 setup/cleanup"——**如果你的 effect 没写 cleanup，第二次 setup 就会叠一份资源**，这正是它想让你发现的（§4.3 有完整时序图）。别把"跑两次"当噪音忽略。

**冲突 ③：`ref` 到底该不该用来"存数据"**

| | 已有笔记 §9.2 | 本集 |
|---|---|---|
| 说法 | "非受控组件靠 DOM 自己存值，要用 `ref` 去取——**更麻烦**，新手先掌握受控组件即可" | 用 `ref` 存"上一次的值"是**标准用法**，专门演示了一整节 |

**结论**：不冲突，但容易被误读成"ref 不推荐用"。准确分工是：**表单值 → 优先受控组件**（要校验、要联动、要能读）；**"变了不该渲染"的值 → 只能用 ref**（上一次的值、定时器 id、是否首次渲染、滚动位置）。前者是"有更好的选择"，后者是"没有别的选择"。

**冲突 ④：`setCount(count + 1)` 该不该一律改成函数式**

| | 已有笔记 §4.4 | 本集 |
|---|---|---|
| 说法 | "凡是新值依赖旧值，**一律**用 `setCount(prev => ...)`" | 计数器全程 `setCount(count + 1)`；`useReducer` 直接 `dispatch({type:'increment'})` |

**结论**：视频的 `useReducer` 写法**天然免疫**这个问题——reducer 收到的 state 是 React 保证的**最新值**，你不需要自己写 `prev`。这正是 `useReducer` 的隐性优点（视频没点出来）。而 `useState` 那边，已有笔记的规则更安全，尤其在**一次事件里连发多次 set**、或在 `setTimeout`/`useEffect` 回调里 set 时。

> 一句话：**单次点击 → 两种都行；连续更新/异步更新 → `prev =>` 或 `useReducer`。**

---

## 12. 📌 补充：视频没讲但会踩的坑

**① `ref.current` 别在渲染期读写**

```jsx
// ❌ 渲染期写入：StrictMode 下会写两次，并发渲染下可能写到被丢弃的那一版
function Bad() {
  const ref = useRef(0);
  ref.current = ref.current + 1;   // 每次渲染都变，且不触发渲染
  return <p>{ref.current}</p>;
}
```

只在**事件处理函数**、`useEffect`、`useLayoutEffect` 里读写；初始化用 `useRef(初始值)` 就够。

**② `useEffect` 里 `setState` 要小心"依赖了自己"**

```jsx
useEffect(() => { setCount(count + 1); }, [count]);   // ❌ 无限循环
useEffect(() => { setCount(c => c + 1); }, []);        // ✅ 挂载后跑一次
```

**③ `useMemo` 缓存的是"这一次的返回值"，不是"这个函数"**

```jsx
const fn = useMemo(() => () => doSomething(value), [value]);
// 缓存的是内层那个函数（value 变才新建）—— 这其实就是 useCallback 在做的事
```

**④ `memo` 只做浅比较；依赖数组里放对象/数组 = 等于没写**（同一个病根：**每次渲染都新建了一个引用**）

```jsx
<Child user={{ name: 'Tom' }} />                                  // ❌ 新对象 → memo 永远失效
useEffect(() => { fetchData(options); }, [options]);              // ❌ 新数组 → 每次都跑
useEffect(() => { fetchData({ page, size }); }, [page, size]);    // ✅ 依赖真正会变的标量
```

**⑤ `useReducer` 的 reducer 里不能发请求**

```jsx
// ❌ reducer 必须纯：StrictMode 会调两次，你会发两遍
function reducer(state, action) {
  if (action.type === 'load') fetch('/api').then(...);
  return state;
}

// ✅ 请求放 effect，reducer 只负责改数据
useEffect(() => {
  fetch('/api').then(r => r.json()).then(data => dispatch({ type: 'loaded', payload: data }));
}, []);
```

**⑥ `forwardRef` 在 React 19 里不再是必需的**：`ref` 已是普通 prop，会直接出现在 `props.ref`。`forwardRef` 仍兼容，但新项目可以省掉。**本集是 React 18 的内容，按 `forwardRef` 写没错。**

**⑦ Hook 规则：只能在顶层调用**——不能写在 `if`、`for`、`while`、嵌套函数里。因为 React 靠"调用顺序"把每个 Hook 和它的状态对应起来：

```
第 1 次渲染：useState → useRef → useEffect      （按顺序存 3 个槽位）
第 2 次渲染：useState → useRef → useEffect      （按顺序取 3 个槽位）
              ↑ 某次渲染少了一个，后面全部错位 → "Rendered fewer hooks than expected"
```

📌 所以 `eslint-plugin-react-hooks` 的 `rules-of-hooks` 和 `exhaustive-deps` 要一起装（`plugin:react-hooks/recommended` 已包含）。

**⑧ 自定义 Hook 就是"把这五个装进一个 `use` 开头的函数"**

```jsx
// 把「存最新值」抽出来，任何组件都能用
function useLatest(value) {
  const ref = useRef(value);
  useEffect(() => { ref.current = value; }, [value]);
  return ref;
}
```

写法已有笔记 §11.1 讲过，这里只提醒：**自定义 Hook 里可以调别的 Hook，但名字必须以 `use` 开头**，否则 lint 不认、React 也不做规则检查。

---

## 13. 一句话总结

> **React 的五个 Hook 其实是三种分工：`useReducer` 管"状态怎么改"（把散落的 setter 收进一个纯函数），`useRef` 管"什么不该触发渲染"（一个跨渲染存活、改了也不吭声的盒子），`useEffect` 管"渲染之外的活儿"（提交之后才执行，依赖数组决定它何时重跑、cleanup 决定它会不会漏），而 `useMemo`/`useCallback` 管"重复的活儿能不能别干"（前者缓存值、后者缓存函数，且必须配上 `React.memo` 才真的省到渲染）—— 前两个解决"代码怎么组织"，后三个解决"时机和成本"，而所有坑的根源只有一句话：闭包捕获的是它出生那一轮的快照，不是现在的值。**

---

## 附录：本文信息

| 项目 | 内容 |
|---|---|
| 视频标题 | 20分钟学会React Hooks 前端开发必看 AI编程工具 CodeGeeX 体验（合集文案作「React Hooks 速成」） |
| UP 主 / 时长 / 发布 | 吴悠讲编程 ｜ 22 分 09 秒 ｜ 2023-09 |
| 视频地址 | https://www.bilibili.com/video/BV1kc411D7F9/ |
| 所属合集 | 《1小时学会React18》第 3 集（共 4 集） |
| 视频 ID / 转写源 | BV1kc411D7F9；`.tmp_audio/ep03.m4a`（13.1 MB），本地 ASR 全量转写并带时间戳 |
| 本文规模 | 1174 行 / 16 个二级章节 + 目录 + 附录；35 段可运行代码、186 行表格 |
| 视频覆盖 | `useReducer`（计数器改造）、CodeGeeX 广告（2 分 26 秒）、`useRef` 三种用途（存上一次值 / 取 DOM / 取子组件方法）、`useEffect`（StrictMode 双执行 + 依赖数组三种写法）、`useMemo`、`useCallback` + `React.memo` |
| 视频未覆盖 | `useState` 与 `useContext`（作者说前几集讲过）、闭包陷阱的成因、cleanup 写法、自定义 Hook、`useLayoutEffect`、`memo` 浅比较细节、数据请求模式、TS 类型 |
| 本文重点增量 | §4.3 StrictMode 完整时序、§4.5 依赖数组四条规则、§4.6 闭包陷阱（视频完全没讲，用整节 + 时序图补齐）、§5.4 / §6.6 两张"要不要优化"判定表、§6.5 memo 三件套链路图、§11.3 四处说法冲突 |
| 配套已有资料 | `../React新手学习笔记-95%核心知识.md`（957 行）、`../前端学习路线-从零到React.md`（231 行）、`./EP01-React18核心语法.md`（1000 行）、`./EP02-组件通信与插槽.md`（999 行） |
| 排版参照 | `../software-architect/EP01-Redis-详细学习笔记.md`（课程地图 → 分章 → 全局复盘 → 速查卡 → 动手路线 → 自测 → 差异 → 补充 → 一句话总结） |
| 下一集 | 合集共 4 集，第 4 集为收尾（本集结尾只说"我们下个视频再见"） |

**「📌」标记共 24 处**，集中在：`dispatch` 引用稳定性、reducer 纯度四纪律、`useRef` 三条禁令、`usePrevious` 封装、`forwardRef` 的 const/函数声明之辨、`useImperativeHandle` 的取舍、StrictMode 真实机制、依赖数组四条规则、闭包陷阱三种解法、`memo` 失效三因、React 19 的 ref 变化、Hook 规则与 lint 配置等位置。**未标记部分均为视频原意。**
