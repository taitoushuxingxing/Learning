# 用 Langfuse 搭建 AI Coding 评测与改进闭环 —— 详细学习笔记

> 基于 B 站视频《Agent 系统设计 05：用 Langfuse 搭建 AI Coding 评测与改进闭环》整理。
> UP 主：桃子可冰乐 ｜ 时长 17:41 ｜ 发布 2026-09-30
> 视频地址：https://www.bilibili.com/video/BV1B8a96PEf6/
> 所属合集：《Agent系统设计》第 5 集（共 6 集）｜ 简介中的文稿地址：https://my.feishu.cn/wiki/Ba9mwre86iflkCkXHlec9klHn4b
>
> **本文是怎么来的**：先对完整音频做语音转写（带时间戳），再按视频原本的讲解顺序（背景 → 数据模型 → 界面导览 → 评测分工 → 评测三问 → 指标 → 团队接入 → 意义）重构成文档。
> 转写稿口语化且同音错误极多（Langfuse → 「兰福斯 / 男朋S / nonfuse / longfuse」、Session → 「C型 / 设置ID」、Observation → 「obsessing / obsidian」、Rubric → 「如布雷克」），文中已按上下文全部修正，对照表见附录 A.3。
> 标「📌 补充」的是我在原讲解之外补的工程细节（部署、埋点、评分 Prompt、门禁配置），其余均为视频原意。**视频本身以「界面演示 + 方法论」为主，几乎没有贴代码；所有可复制代码块均为补充，请按你团队实际环境调整。**
>
> **在两套系列中的位置**：本篇属于 UP 主「桃子可冰乐」的《Agent系统设计》6 集系列，是**第 5 集**。系列顺序为：01 意图识别 → 02 单意图评测 → 03 多意图评测 → 04 Agent 架构选型 → **05 用 Langfuse 搭建评测与改进闭环（本篇）** → 06 LLM Context Engineering。
> 同时，在本目录的 4 篇 Agent 专题里，本篇是**第 4 篇**，对应《从零到一设计 Agent》那张 8 层架构总图的 **L7 工程层（Trace → Eval → 灰度）**——也是四篇里唯一一篇「讲怎么动手搭」的。
>
> **一句话读法**：先记住 Langfuse 只有「Session / Trace / Observation」三层（需求 ID 是你自己加出去的第四层），再看评测的三种分工（程序 / 大模型 / 人）与三问（做完了吗 / 质量好吗 / 划算吗），最后照着第 6、11、14 章把管道接起来。作者反复强调的一句话是：**「我感觉提效了」不算数，得有白纸黑字的数字。**

---

## 目录

- [0. 坐标系：这是 8 层总图里的 L7](#0-坐标系这是-8-层总图里的-l7)
- [1. 背景：为什么「我感觉提效了」不算数](#1-背景为什么我感觉提效了不算数)
- [2. Langfuse 的三层数据模型：一本故事书](#2-langfuse-的三层数据模型一本故事书)
- [3. 第四层：需求 ID 怎么塞进只有两层的 Langfuse](#3-第四层需求-id-怎么塞进只有两层的-langfuse)
- [4. 一条 Trace 的 Span 树长什么样](#4-一条-trace-的-span-树长什么样)
- [5. 界面导览：作者现场点过的 8 个页面](#5-界面导览作者现场点过的-8-个页面)
- [6. 部署与埋点：把数据灌进 Langfuse](#6-部署与埋点把数据灌进-langfuse)
- [7. 评测分工：程序 / 大模型 / 人，各管一段](#7-评测分工程序--大模型--人各管一段)
- [8. 评测流程与三问：做完了吗 / 质量好吗 / 划算吗](#8-评测流程与三问做完了吗--质量好吗--划算吗)
- [9. 评分维度设计：按需求类型选 Rubric](#9-评分维度设计按需求类型选-rubric)
- [10. 三类评分器：规则 / LLM-as-a-Judge / 人工](#10-三类评分器规则--llm-as-a-judge--人工)
- [11. 阈值、回归门禁与 CI/CD](#11-阈值回归门禁与-cicd)
- [12. 看板与核心指标：把「感觉」换成数字](#12-看板与核心指标把感觉换成数字)
- [13. 评测与改进闭环全景](#13-评测与改进闭环全景)
- [14. 团队接入：一个统一评测网关](#14-团队接入一个统一评测网关)
- [15. 开源 self-host vs 云版本](#15-开源-self-host-vs-云版本)
- [16. 速查卡：概念 / 页面 / 命令一页查完](#16-速查卡概念--页面--命令一页查完)
- [17. 📌 补充：视频没讲但一定会踩的坑](#17-补充视频没讲但一定会踩的坑)
- [18. 🔗 与本目录其他篇的关系](#18-与本目录其他篇的关系)
- [19. ✅ 落地检查清单](#19-落地检查清单)
- [20. 一句话总结](#20-一句话总结)
- [附录：本文信息](#附录本文信息)

---

## 0. 坐标系：这是 8 层总图里的 L7

《从零到一设计 Agent》给过一张 8 层架构总图，其中 **L7 是工程层**，三个动作是：

```
L7 工程层
  Trace（看见）  →  Eval（判断）  →  灰度（放量）
     ↑                                  │
     └──────────── 反馈回流 ─────────────┘
```

这一篇就是把这三个动作落成一个能跑的系统。三者关系一句话说清：

| 动作 | 回答的问题 | 本篇对应章节 |
|---|---|---|
| **Trace** | AI 到底干了什么？每一步花了多久、多少钱？ | 第 2～6 章 |
| **Eval** | 干得好不好？做完了吗？质量行吗？划算吗？ | 第 7～12 章 |
| **灰度** | 改完的版本敢不敢放给全组用？ | 第 11 章（回归门禁） |

作者在开场把这件事的动机讲得很直白：

> 「现在目前 AI 时代，大家已经不缺怎么用 Skill、怎么用 MCP 了，更缺的是——**我怎么来衡量它的效果，以及我下一步怎么把它做得越来越好**。这就是我们为什么要去做评测。」

📌 **补充**：L7 之所以排在最后，是因为它**依赖前面所有层的产出**。L3 记忆写得对不对、L4 上下文塞得准不准、L5 工具接得全不全，最终都只能通过 L7 的 Trace 看出来。所以这一层不是「锦上添花」，而是**前面所有层唯一的度量衡**。

---

## 1. 背景：为什么「我感觉提效了」不算数

### 1.1 作者描述的现状

> 「每个团队都会有一套自己的 AI Coding 方式，大家都会说自己的方式是最好的，AI 提效的效果基本上处于**每个人都说自己感觉提效了多少**，根本没有过程的记录和评测。」

这段话点出了三个具体病症：

| 病症 | 表现 | 后果 |
|---|---|---|
| **没有过程记录** | 只有「最后代码长什么样」，没有「AI 中间做了什么」 | 出了问题无法定位：是知识库没给准？Skill 没写好？还是某个 MCP 没接？ |
| **没有统一口径** | 各人说各人的体感 | 无法横向比较：A 说提效 50%，B 说提效 30%，谁的模型对？ |
| **没有回归能力** | 换了模型 / 改了 Prompt / 升了 Codex 版本，效果变差也不知道 | 悄悄退化，等到线上出问题才发现 |

### 1.2 评测的三个目的（作者在结尾才点透，这里提前说）

1. **用数字替代体感**——「人的感觉有的时候是反直觉的，你得用数字去落地。」
2. **定位问题在哪一层**——知识库？向量库组织方式？Skill？MCP 工具？执行轨迹本身？
3. **找到下一步的优化空间**——「你希望它完成得又快、花的钱又少，你的下一步优化空间从哪里来？你是不是得看这些具体的过程？」

### 1.3 一个必须先建立的认知：Langfuse 只是个「后台」

> 「Langfuse 本质上是一个后台，你只要按照它的接口要求，上传你的 Agent 的一个调用过程就可以了。」

所以它不绑定工具。作者点名了这些都能接：

| 类型 | 举例（作者点名） |
|---|---|
| 命令行 Coding Agent | Codex、Claude Code、Qoder、Cursor |
| 大厂内部 Agent | 字节 Trae、阿里 Coder、腾讯 CodeBuddy |
| 自研 Agent | 任何能发 HTTP / OTEL 的程序 |

📌 **补充**：这句话是整篇最重要的「解耦点」。**评测系统不应该长在你的 Agent 里面**，否则你换个 Agent 就得重写一遍评测。正确做法是 Agent 只管吐 Trace，评测系统只管收 Trace 和打分，两边通过一个稳定的协议（OpenTelemetry 或 Langfuse SDK）连接。

---

## 2. Langfuse 的三层数据模型：一本故事书

作者用了一个很好记的比喻：**把 Langfuse 想象成一本故事书。**

### 2.1 三个概念，三层粒度

| 概念 | 比喻 | 对应到 AI Coding | 举例（作者原话） |
|---|---|---|---|
| **Session** | 一章 | 一段连续对话 | 「查找网页设计美化技能」这个任务从头到尾的对话 |
| **Trace** | 一章里的一节 | 你问的**一次**问题 | 你在那个对话里问了 10 次 → 就有 10 个 Trace |
| **Observation** | 一节里的一句话 | Agent 执行的**一步** | 调浏览器、执行命令、思考、调用工具 |

作者的原话拆解：

> 「这个 Session，可以理解为这里的这个『查找网页设计美化技能』，这整个里面的一次对话，就是一个 Session。
> 那这个 Trace 呢，就是比如说你在这个对话里面，你可能曾经问过 AI 好几次问题，你可能问了 10 次，那它这里会有 10 个 Trace。
> 那这个 Trace 里面呢，你问了他一次问题，这个 Agent、这个 Codex，它本身会去开始干活，它可能会调浏览器、可能会执行命令，那么具体执行了哪些命令、思考了、或者用了什么工具，它都叫一个 Observation。」

### 2.2 关键事实：Langfuse 原生只有两层

> 「你可以看到在 Langfuse 里面它**只有两层**，一层就是我们说的 Session，一层就是 Trace，Trace 其实是包含在 Session 里面的。」

这一句是理解后面所有设计的前提，务必记住这张图：

```
Langfuse 原生层级
─────────────────────────────────────────────
Session  （一段对话，有 sessionId）
   └── Trace  （一次提问 = 一次 Agent 任务，有 traceId）
          └── Observation  （一步动作：span / generation / event）
─────────────────────────────────────────────
```

📌 **补充**：在 Langfuse 的数据模型里，Observation 其实是三类对象的统称：

| Observation 子类型 | 含义 | 典型字段 |
|---|---|---|
| `SPAN` | 一段有开始/结束的工作（工具调用、检索、子 Agent） | `name`、`startTime`、`endTime`、`input`、`output` |
| `GENERATION` | 一次 LLM 调用 | `model`、`modelParameters`、`promptTokens`、`completionTokens`、`cost` |
| `EVENT` | 时间点事件（打点、报错） | `name`、`level`、`statusMessage` |

注意 `GENERATION` 是 `SPAN` 的特化——因为它多记了 **Token 用量**和**成本**，所以**只有把 LLM 调用标成 generation，成本看板才有数**。这是新手最常犯的错：全部打成 span，然后发现 Dashboard 里花费永远是 0。

### 2.3 为什么这个粒度刚好够用

| 你想问的问题 | 该看哪一层 |
|---|---|
| 这个需求一共折腾了多少轮？ | Session |
| 这一轮 AI 到底做了哪些动作？超时超在哪一步？ | Trace + Observation |
| 这次调用用了哪个模型、烧了多少 Token、多少钱？ | Generation |
| 整个需求跨了几个 Session、总共多久？ | **Langfuse 答不了 → 需要第 3 章的需求 ID** |

最后一行就是下一章要解决的问题。

---

## 3. 第四层：需求 ID 怎么塞进只有两层的 Langfuse

### 3.1 业务真实形状是三层，工具只给两层

作者的工作方式是「**需求维度**」：

> 「我们会在 Codex 旁边创建一个工作区，把我们要这个需求相关的代码都放在一个工作区里面。然后我们会开一些不同的会话，每一个会话都有对 Codex 来说就是 Session ID，做一个需求可能会开很多个 Session ID 去做。」

所以真实形状是：

```
真实业务形状                        Langfuse 形状
─────────────────────              ─────────────────
需求 (Requirement)                  ✗ 没有这一层
  ├── Session A  ───────────────→   Session
  │     ├── Trace 1  ───────────→     Trace
  │     └── Trace 2
  └── Session B  ───────────────→   Session
        └── Trace 3
```

**一个需求 = 多个 Session；一个 Session = 多个 Trace。** 差的那一层，作者的做法是：

> 「你的需求 ID 其实是最上面这一层，是多出来的，是第三层（相对 Session/Trace 而言的更高层），那你都是通过这里的——你要把你的需求 ID 生成出来之后，携带在这个 **Attribute / Metadata** 里面去。」

### 3.2 光带 ID 不够：还得告诉它「什么时候开始、什么时候结束」

这是全片最容易被忽略、但工程上最关键的细节：

> 「Langfuse 也不知道你需求什么时候开始、什么时候结束，所以说你得**主动地告诉它**，你得写一个相当于**触发的钩子函数**，告诉它『我需求开始了』，那这个时候它就会形成一个需求 ID；你需求结束了，你也得告诉它，它就会把这个需求 ID 不会记录到别的 Session 和 Trace 上面去了。」

用图表示这个「开关」语义：

```
时间 ──────────────────────────────────────────────────────→

         ┌── requirement.start() ──┐
         │                         │
Session: │   S1        S2        S3│   S4（新需求）
Trace:   │  t1 t2 t3   t4 t5      │  t6
         │                         │
         └── requirement.end() ────┘
              ↑ 只有这个窗口内的 Trace 才打上 REQ-1024

若忘记 end(): S1 S2 S3 S4 全被打上 REQ-1024 → 数据污染
若忘记 start(): 整个需求没有 ID → 无法聚合
```

📌 **补充**：Langfuse 没有原生的「需求」实体，所以这个第三层本质是**约定 + 元数据**，落地时有三种做法，作者采用的是第一种：

| 做法 | 实现 | 优点 | 缺点 |
|---|---|---|---|
| **① metadata 打标**（作者用的） | 每条 trace 的 `metadata.requirement_id` 写入 | 零改造、SDK 原生支持 | 只能按 metadata 过滤/聚合，没有「需求详情页」 |
| ② tag 打标 | `tags: ["REQ-1024"]` | 界面上直接可筛，一眼可见 | tag 数量膨胀后筛选器很难用 |
| ③ 用 `userId` 复用 | 把 `userId` 塞成需求 ID | 能直接过滤 | 语义污染，多用户场景立刻崩 |

如果既要「需求详情页」又要少改造，📌 补充一种折中：额外往 Langfuse 发一条 `EVENT` 类型的 Observation，名字叫 `requirement_start` / `requirement_end`，把开始结束时间变成可查的数据点，再用第 14 章的网关做聚合。

### 3.3 Version 过滤器：为什么它是用得最多的一个

作者在讲 Filter 时说：

> 「它这里提到了 5 个概念，那我们用得最多的就是这个 **Version**。为什么 Version 很重要呢？就是你的 Codex 平时会更新、会升级，你的 **Prompt、Skill 和 MCP** 都会不停地更改。那你动了这些东西之后，你编程的相关的一些实现方式可能就会效果比以前好或比以前差。那这种情况下，我们会因为它变动了，所以它的**版本号也会加 1**，这样来区分不同的版本。」

Langfuse 的 Trace 过滤器大致是这五类（转写只念出了 User / Release / Version，其余按界面惯例补全，**字段名按演示推测**）：

| 过滤器 | 作用 | 作者用法 |
|---|---|---|
| **Tags** | 自由标签 | 按需求类型打 `web` / `backend` |
| **User** | 按用户过滤 | C 端 Agent 才用得上；内部评测只有一个人 |
| **Release** | 按发布批次过滤 | 对应一次上线 |
| **Version** | 按版本号过滤 | **最常用**：模型 / Prompt / Skill / MCP 一改就 +1 |
| Session / Metadata | 按会话、元数据 | 用来筛 `requirement_id` |

**版本号该在什么时候 +1？** 作者给了判据：**凡是会影响「AI 怎么干活」的东西变了，就 +1。**

| 变更类型 | 是否 +1 | 理由 |
|---|---|---|
| Codex / Claude Code 本体升级 | ✅ | 底层 Agent 行为可能整体变化 |
| System Prompt 改动 | ✅ | 直接改变行为 |
| Skill / 子 Agent 定义改动 | ✅ | 工具能力边界变了 |
| MCP 工具增删、Schema 改动 | ✅ | 可用动作集合变了 |
| 换了模型（如 Sonnet → 某个更便宜的） | ✅ | 成本与质量同时变 |
| 只改了业务代码、没改 Agent 配置 | ❌ | 不属于「Agent 版本」 |

📌 **补充**：版本号一定要**可追溯**。建议格式 `v{major}.{date}`，例如 `v7.20260928`，并在你的 Git 仓库里打同名 tag。否则三个月后看到「v7 比 v6 好」，没人记得 v7 到底改了什么。

---

## 4. 一条 Trace 的 Span 树长什么样

作者演示时点开的那条 Trace，是一个「帮我优化网站视觉」的需求。按他描述的 Observation 粒度还原成树：

```
Trace  trace_id = 7f3a...c21    session_id = ses_...469    version = v7.20260928
metadata: { requirement_id: "REQ-1024", repo: "my-site", model: "..." }
│
├─ SPAN  user_turn                     3.2s   input: "我想给我自己做的一个网站优化一下视觉"
│  │
│  ├─ GENERATION  planning             1.4s   model=gpt-5  in=3.2k out=420  $0.021
│  │     output: "先读 index.html 和 styles.css，再改配色与间距"
│  │
│  ├─ SPAN  tool:read_file            0.2s   input: src/styles.css   output: (1.2k chars)
│  ├─ SPAN  tool:read_file            0.1s   input: src/App.tsx
│  ├─ SPAN  tool:grep                 0.3s   input: "primary-color"
│  │
│  ├─ GENERATION  edit_styles          6.8s   model=gpt-5  in=8.9k out=1.1k  $0.087
│  │     ⚠ 本步耗时占整条 Trace 的 32%  ← 时间线里最显眼的那根柱子
│  │
│  ├─ SPAN  tool:apply_patch           0.4s   files: 2   +38 −12
│  ├─ SPAN  tool:run_tests             12.6s  ← 卡住的地方：CI 慢，不是模型慢
│  │     └─ EVENT  test_failed         level=ERROR  "styles.spec.ts: 3 failed"
│  │
│  ├─ GENERATION  fix_after_failure    4.1s   model=gpt-5  in=9.4k out=680
│  ├─ SPAN  tool:apply_patch           0.3s
│  ├─ SPAN  tool:run_tests             11.9s  └─ EVENT test_passed
│  │
│  └─ GENERATION  summarize             0.9s   output: "已调整主色、字号阶梯与卡片间距"
│
└─ SCORE  user_feedback = 1（人工，标注在 Trace 上）
   SCORE  task_done     = 1（规则：测试全绿 + 200）
   SCORE  cost_usd      = 0.163（程序回填）
```

### 4.1 这张树能回答什么问题

| 现象 | 从树上怎么看 | 结论方向 |
|---|---|---|
| 「怎么这么慢」 | 看每段 `startTime/endTime`，作者原话：「这个时间线它到底耗时耗到哪一步，为什么这么慢，你都可以在这里看到」 | 常常是 `run_tests` 慢，不是模型慢 |
| 「怎么这么贵」 | 看每个 GENERATION 的 `promptTokens` 与 `cost` | 上下文太长 → 回到 L4 上下文管理 |
| 「为什么没做完」 | 看有没有 `test_failed` 之后没修复 | 定位到具体失败用例 |
| 「它到底用了哪个模型」 | 看 `metadata.model` 与 GENERATION 的 `model` | 确认没被网关偷偷降级 |
| 「知识库到底有没有被检索」 | 看有没有 `retrieval` 类 SPAN、命中了几条 | 没有 → RAG 那层没接上 |

### 4.2 作者特别强调 Metadata 的重要性

> 「还可以看到它的一些元数据，就是你在上报你的监控日志到 Langfuse 过程当中，你可以给他打一些原日志、一些元数据，这个就非常重要了。因为我们其实是**通过这个来知道我们到底调什么模型**，以及它的一些其他的基本信息。」

📌 **补充**：建议在 metadata 里固定塞这 8 个字段，后面所有看板都靠它们分组：

| 字段 | 示例 | 用途 |
|---|---|---|
| `requirement_id` | `REQ-1024` | 需求维度聚合（第三层） |
| `requirement_type` | `web` / `backend` / `data` | 选不同 Rubric |
| `repo` | `my-site` | 按仓库看效果 |
| `model` | `gpt-5` | 模型对比 |
| `agent` | `codex` / `claude-code` | 跨工具对比 |
| `skill_version` | `v3` | Skill 迭代对比 |
| `mcp_tools` | `["browser","fs","git"]` | 工具覆盖面排查 |
| `ai_seconds` / `human_wait_seconds` | `812` / `3600` | 拆分耗时（见 12.3） |

---

## 5. 界面导览：作者现场点过的 8 个页面

作者明确说了「我不是一个把 Langfuse 所有功能全部讲完的介绍」，所以这一章只还原他真正点开的页面和他当时说的话。

### 5.1 Home：看板

> 「这个 Home 页面呢，它其实是一个面板，可以看到你今天到底进行了几次对话？花了多少钱？你的分数到底怎么样？然后这里一些线状图，这相当于是一个看板。然后 **P99 耗时**，你执行一个任务到底花了多久时间。然后这里有 **5 个 dashboard**，这个是 Langfuse 可以帮你初始化的。后面是什么作用呢？在这里有描述啊，但我们平时一般用得比较少。」

要点提炼：

| 卡片 | 指标 | 作者用法 |
|---|---|---|
| 对话数 | Trace / Session 数量 | 看今天干了多少活 |
| 花费 | 总 cost（USD） | 看钱烧得快不快 |
| 分数 | Score 均值 | 看质量趋势 |
| 折线图 | 时间序列 | 看趋势，不看单点 |
| **P99 耗时** | 尾延迟 | **重点**：平均值会骗人，P99 才暴露长尾 |

**为什么盯 P99 而不是平均？** 因为 Coding Agent 的耗时分部极度长尾——大部分任务 3 分钟搞定，偶尔有一个卡 40 分钟。平均值看起来「还行」，但你的体感由那 1% 的烂任务决定。

### 5.2 Traces：一次提问一条

> 「你问一次问题就会形成一个 Trace。按照我们正常使用情况下来看的话，一个需求里面有多个 Session，一个 Session 里面一般会有多个 Trace。所以我们看这个 Session 具体每一步执行得怎么样，我们会先去看你的这个 Session 具体的执行号是多少。」

作者的实操顺序（**照这个顺序点**）：

```
1. 先从 Session 列表找到 session id（他举例「结尾是 469 的这个 Session」）
2. 记下这个 session 里共有多少条 Trace（他举例「总共有 14 条」）
3. 点进去，默认按时间倒序
4. 看到第一条问题的原始 prompt
5. 展开看每一步在干什么、花了多少时间
```

作者当时看到的第一条问题是：「我想给我自己做的一个网站去优化一下它的视觉」。

> 「这里可以看到我具体问他什么问题，然后整个执行过程每一步他在干嘛、花了多少时间。」
> 「这里还可以把它添加到所谓的问题集里面，但目前来说我们还不用这个东西。」

> **注意这句「我们还不用这个东西」**——作者对 Dataset 的态度是「知道有、暂时不用」，原因在第 8 章：他们的评测集是按「需求类型」组织的 Rubric，而不是把单条 trace 攒成数据集。这点和很多教程不一样，别被带偏。

### 5.3 Trace 详情：时间线 + 打分 + Metadata

三个动作都在这一页：

> 「这个时间线它到底耗时耗到哪一步，为什么这么慢，你都可以在这里看到。」
> 「这里应该还有一个打分的，就是可以针对这个单个 Trace 的这样过程进行一个分数评估的。哦对，在这里。」
> 「还可以看到它的一些元数据……这个就非常重要了。」

| 区域 | 作用 | 备注 |
|---|---|---|
| 时间线（Timeline / Waterfall） | 每个 Observation 的起止与耗时 | 找「卡在哪」的唯一入口 |
| 输入输出 | 原始 prompt、工具调用参数与返回 | 排查幻觉的第一现场 |
| **Scores** | 给这条 Trace 打分 | 支持多个分数并存（规则分 + 人工分） |
| Metadata | 自打的 key-value | 需求 ID 挂这里 |

### 5.4 Users：C 端才需要，但要配告警

> 「有一些 User，那我这个是我自己的一个内部评测，我写代码的 Codex 效果到底怎么样，所以它只有一个人。但是这个 Langfuse 不仅仅是可以给你的 B 端那些提效的 AI 编程评测，你可能自己也做了一些面向用户的 C 端 Agent，那你这里可能就会有一些用户进来问问题了，那这个用户数据量就会很多，**还是得告警**。你可以让它 Langfuse 后台发现有问题之后，自动地去调一个你们自己的监控后台的钩子函数。」

这段给了一个很实用的分层：

| 场景 | 用户数 | 是否需要告警 |
|---|---|---|
| 内部 AI Coding 评测 | 1（就是你自己） | 不需要，人工看就行 |
| 团队共用 | 十几人 | 弱告警（成本异常时） |
| C 端 Agent | 成千上万 | **必须告警**：错误率、负反馈、成本突增 |

### 5.5 Prompt Management：管理 + 实验

> 「这里有 Prompt 管理，包括你管理完成之后，你可以在这里做实验。」

📌 **补充**：Prompt Management 的价值不在于「存文本」，而在于**版本可追溯 + 可回滚 + 能和 Trace 关联**。当你在 Trace 上看到 `promptVersion=12`，就能精确知道这条 trace 跑的是哪一版 Prompt——这正是第 3 章 Version 过滤器的数据来源。

### 5.6 Scores：分数是你自己定义的

> 「这里有分数，这个分数呢就是你自己定义的嘛，就是你做的这个事情效果怎么样，到底花了多少 Token 啊这些东西。你可能需要在这里相当于列一个分数列表。」

作者的三个例子：**效果好不好的主观分、花了多少 Token、花了多少钱**。注意后两个其实是**数值型指标**，在 Langfuse 里也能当 Score 存（`value` 是数字），这样就能和主观分画在同一张图上对比。

### 5.7 Annotation（人工标注）与 Datasets（评测集）

> 「还有这个是人工。因为你的执行效果怎么样，除了有大模型的一些评测标准可以放在这里、规则可以放在这里之外，你的人工的评测标准也可以放好。然后这里的它是放你的一些评测集。」

这句话信息密度很高，它把「评分标准」分成了三个抽屉：

| 抽屉 | 放什么 | 谁来执行 |
|---|---|---|
| 规则标准 | 测试通过、HTTP 200、数据一致 | 程序 |
| 大模型标准 | Rubric（如「用户是否在要求返工」） | LLM-as-a-Judge |
| **人工标准** | 验收清单、上线判断 | 人（Annotation 队列） |

### 5.8 Settings：接第三方模型作为 Judge

> 「Settings 里面啊，你可以去接你自己的一些比如说 **DeepSeek** 一些第三方大模型来作为其他的 LLM 评测模型，还有你的这些打分的标准可以放到这里。」

| 配置项 | 作用 |
|---|---|
| LLM Connections | 接 OpenAI / Anthropic / DeepSeek 等，作为 judge 模型 |
| Scores / Score Configs | 定义分数名、类型（numeric / categorical / boolean）、取值范围 |
| Evaluators | 配置自动评分器（LLM-as-a-Judge），指定用哪个模型、哪套 Rubric |
| Webhooks | 出问题时回调你们自己的监控系统（对应 5.4 的告警） |

📌 **补充**：**judge 模型和被评模型必须是不同的**，最好是不同厂商。用 GPT-5 评 GPT-5 会有明显的自我偏好（self-preference bias）。作者的组合是「Codex 干活 + DeepSeek 当裁判」，这个搭配在成本上也很划算——裁判只需要读文本，不需要写代码。

---

## 6. 部署与埋点：把数据灌进 Langfuse

> ⚠️ 本章除「为什么自建」这段是视频原意，其余**全部为 📌 补充**。视频只说了「把 Langfuse 部署到自己团队的一个机器上面去」，没有演示部署命令。

### 6.1 为什么必须 self-host

> 「最终可能需要你们把这个 Langfuse 部署到自己团队的一个机器上面去，然后大家用的时候只需要去接入一下你们的一个通用的网关，通用的一个评测网关，然后由它来统一地去上报到 Langfuse 里面去。」

三个理由：

| 理由 | 说明 |
|---|---|
| **代码不能出门** | Trace 里全是你的源码片段、需求文档、内部 URL |
| **成本可控** | 云版本按事件数计费，Coding Agent 一天能产生几万条 |
| **可定制** | 要接内部 SSO、要走内网、要接自己的告警 |

### 6.2 Docker Compose 部署（v3 架构）

Langfuse v3 把存储拆成了多个组件，这是新手最容易翻车的地方——**不是只有一个 Postgres**。

```
                    ┌──────────────┐
   浏览器 / SDK ───→ │  langfuse-web │  (Next.js, :3000)
                    └──────┬───────┘
                           │
              ┌────────────┼─────────────┬──────────────┐
              ▼            ▼             ▼              ▼
        ┌──────────┐ ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ Postgres │ │  Redis   │  │ClickHouse│  │  MinIO   │
        │ 元数据    │ │ 队列/缓存 │  │ Trace 存储│  │ 大对象    │
        └──────────┘ └──────────┘  └──────────┘  └──────────┘
                           ▲
                    ┌──────┴───────┐
                    │ langfuse-    │  (异步消费、批量写入)
                    │ worker       │
                    └──────────────┘
```

| 组件 | 存什么 | 挂了会怎样 |
|---|---|---|
| **Postgres** | 用户、项目、Prompt、Dataset、Score 定义 | 全站不可用 |
| **ClickHouse** | Trace / Observation 明细（OLAP） | 列表查不出来、看板空白 |
| **Redis** | 队列、缓存、限流 | 写入积压，worker 不消费 |
| **MinIO（S3）** | 大文本、附件、多媒体 | 详情页 input/output 打不开 |

```yaml
# docker-compose.yml（📌 补充：精简版，仅用于本地/小团队验证）
version: "3.9"

services:
  langfuse-web:
    image: langfuse/langfuse:3
    depends_on: [postgres, redis, clickhouse, minio]
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/langfuse
      SALT: "change-me-salt"
      ENCRYPTION_KEY: "0000000000000000000000000000000000000000000000000000000000000000"
      NEXTAUTH_URL: http://localhost:3000
      NEXTAUTH_SECRET: "change-me-nextauth"
      CLICKHOUSE_URL: http://clickhouse:8123
      CLICKHOUSE_USER: clickhouse
      CLICKHOUSE_PASSWORD: clickhouse
      REDIS_HOST: redis
      LANGFUSE_S3_EVENT_UPLOAD_BUCKET: langfuse
      LANGFUSE_S3_EVENT_UPLOAD_ENDPOINT: http://minio:9000
      LANGFUSE_S3_EVENT_UPLOAD_ACCESS_KEY_ID: minio
      LANGFUSE_S3_EVENT_UPLOAD_SECRET_ACCESS_KEY: miniosecret
      LANGFUSE_S3_EVENT_UPLOAD_FORCE_PATH_STYLE: "true"

  langfuse-worker:
    image: langfuse/langfuse-worker:3
    depends_on: [postgres, redis, clickhouse, minio]
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/langfuse
      SALT: "change-me-salt"
      ENCRYPTION_KEY: "0000000000000000000000000000000000000000000000000000000000000000"
      CLICKHOUSE_URL: http://clickhouse:8123
      CLICKHOUSE_USER: clickhouse
      CLICKHOUSE_PASSWORD: clickhouse
      REDIS_HOST: redis
      LANGFUSE_S3_EVENT_UPLOAD_BUCKET: langfuse
      LANGFUSE_S3_EVENT_UPLOAD_ENDPOINT: http://minio:9000
      LANGFUSE_S3_EVENT_UPLOAD_ACCESS_KEY_ID: minio
      LANGFUSE_S3_EVENT_UPLOAD_SECRET_ACCESS_KEY: miniosecret
      LANGFUSE_S3_EVENT_UPLOAD_FORCE_PATH_STYLE: "true"

  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: langfuse
    volumes: ["pg_data:/var/lib/postgresql/data"]

  clickhouse:
    image: clickhouse/clickhouse-server:24
    environment:
      CLICKHOUSE_USER: clickhouse
      CLICKHOUSE_PASSWORD: clickhouse
    volumes: ["ch_data:/var/lib/clickhouse"]
    ulimits:
      nofile: { soft: 262144, hard: 262144 }

  redis:
    image: redis:7
    volumes: ["redis_data:/data"]

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: miniosecret
    volumes: ["minio_data:/data"]

volumes:
  pg_data:
  ch_data:
  redis_data:
  minio_data:
```

```bash
# 起服务
docker compose up -d

# 建 MinIO bucket（首次必做，否则 worker 报 NoSuchBucket）
docker compose exec minio mc alias set local http://localhost:9000 minio miniosecret
docker compose exec minio mc mb local/langfuse

# 看日志确认 worker 已连上 ClickHouse
docker compose logs -f langfuse-worker | grep -i clickhouse

# 打开 http://localhost:3000 → 注册第一个账号 → 建 Project → 拿 API Key
```

**踩坑清单（📌 补充）**：

| 现象 | 原因 | 解决 |
|---|---|---|
| 看板一直空白 | worker 没起来 / ClickHouse 没建表 | 看 worker 日志；首次启动会自动 migrate，耐心等 |
| 写入 401 | Public Key / Secret Key 写反了 | `pk-lf-` 开头是 public，`sk-lf-` 开头是 secret |
| 详情页 input 打不开 | MinIO bucket 不存在 | 执行上面的 `mc mb` |
| 数据写进去但列表延迟很久 | worker 是批量消费 | 正常，v3 有秒级到分钟级延迟，别用「写完立刻查」做测试 |
| 中文乱码 | 数据库编码 | Postgres 用 UTF8 初始化 |

### 6.3 埋点方式一：SDK（Python）

📌 补充。这是最可控的方式，适合自研 Agent。

```bash
pip install langfuse
```

```bash
# .env
LANGFUSE_PUBLIC_KEY=pk-lf-xxxxxxxx
LANGFUSE_SECRET_KEY=sk-lf-xxxxxxxx
LANGFUSE_HOST=http://localhost:3000
```

```python
# 📌 补充：一个最小可用埋点骨架
import os
from langfuse import Langfuse
from langfuse.decorators import observe, langfuse_context

langfuse = Langfuse(
    public_key=os.environ["LANGFUSE_PUBLIC_KEY"],
    secret_key=os.environ["LANGFUSE_SECRET_KEY"],
    host=os.environ["LANGFUSE_HOST"],
)

# ── 需求开关：对应第 3 章的 hook ──────────────────────────
def requirement_start(requirement_id: str, req_type: str = "web"):
    """在需求开始时调用一次"""
    langfuse_context.update_current_trace(
        session_id=f"req-{requirement_id}",   # 让一个需求的所有 trace 归到同一 session
        metadata={
            "requirement_id": requirement_id,  # ← 第四层就靠这个字段
            "requirement_type": req_type,      # ← 决定用哪套 Rubric
            "phase": "start",
        },
        tags=[req_type],
    )

def requirement_end(requirement_id: str):
    """在需求结束时调用一次：发一条 EVENT，避免 ID 泄漏到下一个需求"""
    langfuse.create_event(
        name="requirement_end",
        metadata={"requirement_id": requirement_id},
    )

# ── 一次提问 = 一条 trace ────────────────────────────────
@observe(name="user_turn")
def handle_turn(user_input: str, requirement_id: str, version: str = "v7.20260928"):
    langfuse_context.update_current_trace(
        name="coding-turn",
        session_id=f"req-{requirement_id}",
        version=version,                     # ← 第 3 章的 Version 过滤器
        user_id="dev-01",
        metadata={"requirement_id": requirement_id, "agent": "codex", "repo": "my-site"},
        input=user_input,
    )
    plan = plan_step(user_input)
    patch = edit_step(plan)
    test_result = run_tests()
    return summarize(patch, test_result)

# ── 一步动作 = 一个 span ─────────────────────────────────
@observe(name="tool:read_file", as_type="span")
def read_file(path: str) -> str:
    langfuse_context.update_current_observation(input={"path": path})
    content = open(path, encoding="utf-8").read()
    langfuse_context.update_current_observation(output={"chars": len(content)})
    return content

# ── 一次 LLM 调用 = 一个 generation（有 token 和 cost）───
@observe(name="planning", as_type="generation")
def plan_step(user_input: str) -> str:
    langfuse_context.update_current_observation(model="gpt-5")
    resp = call_llm(user_input)          # 你的真实调用
    langfuse_context.update_current_observation(
        model_parameters={"temperature": 0.2},
        usage={"input": resp.usage.input_tokens, "output": resp.usage.output_tokens},
        output=resp.text,
    )
    return resp.text

# ── 程序打分：回填到当前 trace ───────────────────────────
def score_current(trace_id: str, name: str, value, comment: str = ""):
    langfuse.score(
        trace_id=trace_id,
        name=name,          # 例：task_done / test_pass_rate / cost_usd
        value=value,
        comment=comment,
    )
```

### 6.4 埋点方式二：OpenTelemetry（接第三方 Agent）

📌 补充。当你的 Agent 是 Codex / Claude Code 这类你改不了源码的工具时，只能走 OTEL。Langfuse 有原生 OTLP 端点：

```bash
# 任何支持 OTEL 的程序，只需三个环境变量
export OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:3000/api/public/otel"
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Basic $(echo -n 'pk-lf-xxx:sk-lf-xxx' | base64)"
export OTEL_SERVICE_NAME="codex-team-a"
```

| 方式 | 适用 | 能拿到什么 | 拿不到什么 |
|---|---|---|---|
| Langfuse SDK | 自研 Agent | 全量：trace/span/generation/score | — |
| OTEL（OTLP） | 第三方 CLI Agent | span 树、耗时、部分 token | 业务语义（需求 ID 需自己加 attribute） |
| 包装脚本 / 代理 | 任何 CLI | 可自定义 metadata | 需要自己解析 stdout |
| 网关代理 | 多工具混用 | 模型调用、token、成本 | 工具调用细节 |

### 6.5 最小验证：三行自检

```bash
# 1. 健康检查
curl -s http://localhost:3000/api/public/health

# 2. 手动发一条 trace（不依赖 SDK，验证鉴权）
curl -X POST http://localhost:3000/api/public/ingestion \
  -H "Content-Type: application/json" \
  -u "pk-lf-xxx:sk-lf-xxx" \
  -d '{"batch":[{"type":"trace-create","body":{"id":"smoke-001","name":"smoke","metadata":{"requirement_id":"REQ-0001"}}}]}'

# 3. 去 UI 的 Traces 页面搜 smoke-001，能看到就通了
```

---

## 7. 评测分工：程序 / 大模型 / 人，各管一段

这是全片方法论的核心之一。作者的原话：

> 「你如何去评测呢？那评测其实对于我们来说，评测分工分为三个。」

### 7.1 三者的边界

| 角色 | 评什么（作者原话） | 成本 | 适合的事 |
|---|---|---|---|
| **程序** | 「你的测试是否通过，页面是否返回 200，数据量是否一致，是否有变化，回滚」；「你的花费额多少」；「用户是否明确说你是通过」 | 极低 | **有明显规则和阈值可以界定**的一切 |
| **大模型** | 「用户是不是在要求返工，是不是在询问有没有移动的场景」 | 中 | 需要读懂语义、但不需要最终决策的判断 |
| **人** | 「这个页面是不是真的 OK 了，是不是已经可以上线了」 | **最高** | 最终验收、审美判断、责任归属 |

作者对「为什么不能让大模型全包」的解释很实在：

> 「大模型就是它有可能会犯错，其实人也可能会犯错，只是有一些工作我们觉得人做起来太繁琐了，**人的精力是最高的，我们需要把人这个事情放在最应该发挥作用的地方**。」

### 7.2 为什么程序必须排第一

因为程序评分是**确定性**的。测试通过就是通过，HTTP 200 就是 200。它不该消耗任何模型 Token，也不该占用人的注意力。作者的排序隐含了一条流水线：

```
程序（毫秒级、零成本）
   │ 不通过 → 直接判「没做完」，后面全不用跑
   ▼ 通过
大模型（秒级、低成本）
   │ 初筛：找出「看起来做完了其实没做完」的情况
   ▼ 通过
人（分钟级、高成本）
   └ 终审：能不能上线
```

> 作者原话印证：「他先初筛一轮，其实就是为了省你人的精力嘛，然后再交给人家去做。」

### 7.3 冲突处理：程序说通过、大模型说不行，听谁的

作者提了这个问题（「程序和大模型的一些结论有冲突的时候怎么处理」），但没展开。📌 补充一套可执行的仲裁规则：

| 冲突组合 | 处理策略 | 理由 |
|---|---|---|
| 程序 ❌ / 模型 ✅ | **信程序** | 程序是确定性的，模型看不懂编译错误 |
| 程序 ✅ / 模型 ❌（说没做完） | **人工抽检 20%** | 模型可能幻觉，也可能真发现了漏项 |
| 程序 ✅ / 模型 ❌（说质量差） | 记为「质量告警」，不阻断 | 质量分不阻断上线，但进周报 |
| 人 ❌ / 程序 ✅ | **人的结论优先，且必须回写规则** | 说明你的规则漏了一条，把它补进测试 |
| 三方全 ✅ 但线上出问题 | 记入「漏检案例库」 | 这是最有价值的样本，下次加进黄金集 |

最后一行是关键：**每一次「评测说 OK 但线上翻车」，都必须变成一条新规则或一个新测试用例**，否则你的评测体系永远停在原地。

### 7.4 结构化输出：让大模型的结论能被程序消费

> 「大模型评测的话，还要反馈一个**规范化的结构化文件**。」

为什么必须结构化？因为非结构化的结论无法聚合、无法做趋势、无法进门禁。📌 补充一个 schema：

```json
{
  "trace_id": "7f3a...c21",
  "requirement_id": "REQ-1024",
  "rubric_version": "web-v3",
  "judge_model": "deepseek-chat",
  "scores": {
    "task_done":        { "value": 1, "reason": "三个需求点均有对应改动与测试" },
    "requirement_cover":{ "value": 0.67, "reason": "10 个需求点中完成 6.7 个，缺『移动端适配』" },
    "rework_signal":    { "value": 1, "reason": "用户第 3 轮说『我不是说了要移动端吗』" },
    "regression_safe":  { "value": 0, "reason": "未运行全量回归，仅跑了新增用例" }
  },
  "evidence": [
    { "type": "test", "path": "reports/junit.xml", "passed": 42, "failed": 0 },
    { "type": "screenshot", "path": "shots/home-1440.png" }
  ],
  "verdict": "needs_human_review",
  "confidence": 0.72
}
```

| 字段 | 为什么必须有 |
|---|---|
| `rubric_version` | 换了评分标准后，历史分数才可比 |
| `judge_model` | 换裁判模型会导致分数漂移，必须记录 |
| `reason` | 没有理由的分数无法定位问题 |
| `evidence` | 「证据是否齐全」是评测项之一 |
| `verdict` | 三态：`pass` / `needs_human_review` / `fail`，直接驱动流水线 |
| `confidence` | 低置信度自动转人工，这是省钱的关键 |

---

## 8. 评测流程与三问：做完了吗 / 质量好吗 / 划算吗

### 8.1 前置：评分表要按需求类型分版本

> 「这个固定的评测框架呢，是你需要**按照需求类型去选择不同版本的一个评分表**。因为你做的需求有很多，可能它是一个做网站的需求，也可能是一个纯后端需求，不同的需求有不同的评分标准。那么你对于你们自己团队来说，你得建立一套你们自己团队对于不同项目类型的一个评分表，然后当你做这种类型的时候，你按照你们这个团队自己的评分表去进行上传。」

这条很容易被跳过，但它是**评测能不能复用**的分水岭：

| 需求类型 | 评分重点 | 不适用的维度 |
|---|---|---|
| 前端 / 网站 | 视觉还原、响应式、交互可用性、浏览器控制台无报错 | 接口幂等性 |
| 纯后端 | 接口契约、错误码、幂等、并发安全、单测覆盖 | 视觉还原 |
| 数据 / 脚本 | 数据一致性、可重跑、结果可对账 | UI 相关全部 |
| 重构 | 行为不变（回归全绿）、圈复杂度下降 | 新功能覆盖度 |

### 8.2 第一问：做完了吗（硬门槛）

> 「第一个是有没有做完。因为自动评测我们不会去关注这种……如果你这个需求都没有做完，你其实**人和大模型都不需要再介入了**，这个是非常重要的。」

怎么判「做完了」？作者给的全是**可程序化**的判据：

> 「你这个需求做之前可能要写单元测试。你的页面能不能打开？有没有报错？你如果程序都报错了，你的单元测试没通过，那其实就是没做完的。而且还要看你的**需求清单**里面，每一个需求的那种单元测试是不是都通过了，**证据是不是齐全的，是不是符合预期的**。」

> 「有没有做完这个事情，除了程序可以评判之外，那么大模型也可以进行一个初筛，人工进行一个终审。」

| 判据 | 执行者 | 通过标准 |
|---|---|---|
| 单元测试全绿 | 程序 | failed = 0 |
| 页面可打开、无报错 | 程序 | HTTP 200 + console error = 0 |
| 需求清单逐项有对应单测 | 程序 | 覆盖率 = 100%（按条目，不按行） |
| 证据齐全 | 程序 + 模型 | 截图 / 日志 / 测试报告三件套 |
| 是否符合预期 | 模型初筛 → 人终审 | — |

作者描述的完整流水线：

```
AI 完成开发
   ↓
程序：接口 / 测试全通过
   ↓
大模型：初筛（找「假装做完了」）
   ↓
用户：验收（自己看一眼说 OK）
   ↓
真正完成
```

> 「用户用户自己也验收一下说 OK 了，然后那真正完成了。」

### 8.3 第二问：质量好不好

> 「第二个需要关注的点是他这个作业的质量到底好不好。比如说你做这个事情，你写了一个需求，他很快给你把任务完成了，什么单元测试、验证层都通过了，但是他写的到底质量好吗？是不是达到了所谓的你的耗时控制在一个很低的范围内、你的成本很低、你的功能是不是正确的，并且它们有没有影响别的功能。」

拆成四个子问题：

| 子问题 | 怎么测 | 谁测 |
|---|---|---|
| 功能是否正确 | 需求点逐条对照 + E2E | 程序 + 模型 |
| **有没有影响别的功能** | **全量回归** | 程序 |
| **需求覆盖度** | 需求清单逐条勾 | 模型（关键！） |
| 耗时 / 成本是否合理 | Trace 统计 | 程序 |

**需求覆盖度为什么要模型来判？** 作者的例子非常传神：

> 「有可能你只做了两三个需求，总共需求点有 10 个，你只做了两三个，然后大模型可能会出现幻觉，告诉你他都做完了，那其实剩下的其他（需求点）没做完。那这种情况下其实是需要另外一个（模型）去判断的。」

也就是说：**干活的是模型 A，判断「有没有干完」的必须换成模型 B。** 自己评自己必然说干完了。

### 8.4 第三问：上线稳定性

> 「最后呢上线稳定性，就是这个东西上线了，它到底有没有触发告警。那这个东西其实也是由你们之前的自己系统的运维的那些告警程序来进行负责的。还有你的证据是不是完整。」

| 检查项 | 数据来源 |
|---|---|
| 是否触发告警 | 监控系统（错误率、5xx、P99） |
| 灰度期是否有异常 | APM / 日志 |
| 证据是否完整 | 截图、测试报告、变更单 |
| 能否回滚 | 程序（作者强调「回滚」是程序该测的） |

### 8.5 硬指标不能跳过

> 「所以说你做这个事情呢，**硬指标是不能跳过去的**。你首先要保证你的核心测试必须通过，功能并不能被破坏，生产环境可以访问，用户必须能验收。」

这四条是**一票否决**，不参与加权：

1. 核心测试通过
2. 功能没被破坏
3. 生产环境可访问
4. 用户能验收

📌 **补充**：工程上要把「一票否决」和「加权评分」分开实现。否决项用 `gate`（布尔），质量项用 `score`（0~1）。混在一起会导致「质量分很高所以勉强上线」这种荒唐决策。

### 8.6 第四问：划算吗（作者单列的一大段）

> 「最后一个就是你做这个事情真的划算吗？因为大家搞现在搞 AI 搞到最后……不是说我一定要用 AI 做什么事情，而是我们大家现在有一种 AI 热潮，就是做什么事情我都用 AI，但其实真的是最好的吗？其实回到这个事情的本质上，应该是**我们用 AI 是不是真的让我们的工作变得更好了、变得更快了**。」

三个对比维度：

| 维度 | 怎么比 | 作者的例子 |
|---|---|---|
| **钱** | AI 成本 vs 人力成本 | 「如果这个事情你做一个需求就要花个几百美刀，那你肯定扛不住」 |
| **时间** | AI 耗时 vs 预期耗时 | 「以前可能一个很简单的需求，人可能就花个几秒钟他就看完了，但你非要用 AI 做，那可能他花很长的时间」 |
| **返工次数** | 用户要求返工的轮数 | 「他每次告诉你我做完了，老大你过来看一下，然后你检查这个过程中，你发现他其实没做对，你就一而再、再而三地告诉他」 |

**返工这段是整篇最扎心的部分**，作者把条件都设到最理想了还是翻车：

> 「你把需求文档写得非常棒了，但是呢，你的这些 **SDD、Skill、MCP** 这些验证机制做得很好，**Loop、Harness** 都做得非常棒。但是你做完之后，他每次告诉你我做完了，你检查发现他其实没做对。」

这说明：**返工不是 Prompt 写得不够好，而是一个必须被度量、被优化的独立指标。** 如果返工率一直很高，那「AI 提效」就是负的——因为你的 review 时间被吃掉了。

### 8.7 三个问题的顺序不能颠倒

```
① 做完了吗？  ── 没做完 ──→ 结束，不谈后面（省下模型和人的成本）
     │ 做完了
     ▼
② 质量好吗？  ── 有硬伤 ──→ 打回重做
     │ 质量合格
     ▼
③ 划算吗？    ── 不划算 ──→ 换方案（换模型 / 换工具 / 干脆人做）
     │ 划算
     ▼
   上线
```

作者原话总结：**「当你的质量，第一个就是他到底有没有做完嘛，如果都没做完，不谈后面的；做完之后再看他做的质量怎么样。」**

---

## 9. 评分维度设计：按需求类型选 Rubric

> ⚠️ 本章的维度细化与权重为 📌 补充。视频只给了「三问」和「三种执行者」，没有给具体评分表；作者说完整文档放在评论区（飞书链接见开篇信息块）。

### 9.1 通用 Rubric 骨架

一个可用的 Rubric 至少要包含四列：**维度 / 判据 / 分值 / 谁判**。

| # | 维度 | 判据 | 分值 | 执行者 | 是否否决项 |
|---|---|---|---|---|---|
| 1 | 任务完成度 | 需求清单逐项有对应实现 | 0/1 | 程序 | ✅ |
| 2 | 测试通过率 | 单测 + 集成测试全绿 | 0~1 | 程序 | ✅ |
| 3 | 功能正确性 | E2E 主流程走通 | 0~1 | 程序 | ✅ |
| 4 | 回归安全 | 全量回归无新增失败 | 0/1 | 程序 | ✅ |
| 5 | 需求覆盖度 | 完成条目 / 总条目 | 0~1 | 模型 | ❌ |
| 6 | 代码质量 | 可读性、重复度、命名、分层 | 1~5 | 模型 | ❌ |
| 7 | 视觉还原 | 与设计稿一致度（前端类） | 1~5 | 模型 + 人 | ❌ |
| 8 | 返工信号 | 用户是否要求返工、返工轮数 | 0~n | 模型 | ❌ |
| 9 | 成本合理性 | 实际 cost / 预算 cost | 比值 | 程序 | ❌ |
| 10 | 耗时合理性 | 实际耗时 / 预期耗时 | 比值 | 程序 | ❌ |
| 11 | 上线稳定性 | 灰度期告警数、错误率 | 0/1 | 程序 | ✅ |
| 12 | 最终验收 | 人看一眼「能不能上线」 | 0/1 | 人 | ✅ |

### 9.2 三种需求类型的权重差异

| 维度 | 前端 / 网站 | 纯后端 | 重构 |
|---|---|---|---|
| 任务完成度 | 20% | 20% | 10% |
| 测试通过率 | 15% | 25% | 30% |
| 功能正确性 | 15% | 20% | 10% |
| 回归安全 | 15% | 20% | **40%** |
| 需求覆盖度 | 10% | 5% | 5% |
| 代码质量 | 5% | 5% | 5% |
| 视觉还原 | **20%** | — | — |

注意「回归安全」在重构类里占 40%——因为重构的唯一目的就是**行为不变**，其他都是次要的。

### 9.3 Rubric 怎么写才不像废话

| ❌ 坏判据 | ✅ 好判据 |
|---|---|
| 「代码质量好」 | 「函数不超过 50 行；无重复代码块超过 10 行；新增文件有对应测试」 |
| 「视觉还原度高」 | 「主色、字号阶梯、圆角、间距四项与设计稿一致；1440 与 375 两个宽度下无横向滚动」 |
| 「需求做完了」 | 「需求清单 10 条，逐条给出对应文件与行号，缺失则该项计 0」 |
| 「性能可以」 | 「首屏 LCP < 2.5s；接口 P99 < 300ms」 |

**原则：判据必须能被第二个人独立复现。** 如果两个人看同一条 trace 会给出不同的分，那这条判据就还没写好。

### 9.4 阈值设定

> ⚠️ 📌 补充。作者只说了「有明显规则和阈值可以界定」，没给具体数字。下表是经验起点，**请用你们自己的历史数据校准**。

| 指标 | 绿灯 | 黄灯 | 红灯 | 动作 |
|---|---|---|---|---|
| 任务完成率 | ≥ 90% | 75% ~ 90% | < 75% | 红灯阻断发版 |
| 测试通过率 | 100% | 95% ~ 99% | < 95% | 红灯阻断 |
| 需求覆盖率 | ≥ 95% | 80% ~ 95% | < 80% | 黄灯转人工 |
| 回归新增失败 | 0 | 1~2 | ≥ 3 | 红灯阻断 |
| 单任务成本 | ≤ $0.5 | $0.5 ~ $2 | > $2 | 红灯需审批 |
| 单任务耗时（AI 侧） | ≤ 15 min | 15 ~ 40 min | > 40 min | 黄灯排查 |
| 返工轮数 | 0 | 1 | ≥ 2 | 黄灯，进周报 |
| 人工验收通过率 | ≥ 90% | 70% ~ 90% | < 70% | 红灯，回看 Rubric |

**阈值怎么定才不拍脑袋？** 三步：

1. **先测基线**：拿最近 30 个已完成需求跑一遍，得到分布（P50 / P90）。
2. **取 P90 当黄灯线**：因为 90% 的历史任务都在这个值以内，超了就是异常。
3. **每季度重校准一次**：模型在升级，阈值不跟着动就会失真。

---

## 10. 三类评分器：规则 / LLM-as-a-Judge / 人工

### 10.1 对比表

| 维度 | 规则评分器（Code-based） | LLM-as-a-Judge | 人工标注（Annotation） |
|---|---|---|---|
| **实现方式** | 断言、脚本、测试框架 | Prompt + Rubric + 结构化输出 | 标注队列 + 评分表单 |
| **成本** | ≈ 0 | 低（一次几分钱） | **高**（分钟级人力） |
| **速度** | 毫秒 | 秒 | 分钟~小时 |
| **一致性** | 100%（确定性） | 较高（受 temperature 影响） | 低（人对同一件事判断会漂） |
| **能判语义** | ❌ | ✅ | ✅ |
| **能判审美** | ❌ | 部分 | ✅ |
| **可解释性** | 强（失败就报哪条断言） | 中（有 reason，但可能是编的） | 强（人会说为什么） |
| **适合场景** | 测试、HTTP、数据一致、成本、耗时 | 需求覆盖、返工信号、代码质量初筛 | 最终验收、上线判断、争议仲裁 |
| **作者定位** | 「有详细规则和阈值可以界定的」 | 「人做起来太繁琐的」 | 「最应该发挥作用的地方」 |

### 10.2 规则评分器：写成测试，不要写成脚本

📌 补充。规则分最容易腐化的方式是「一个巨大的 if-else 脚本」，半年后没人敢改。正确做法是把它做成**可独立运行的检查项集合**：

```python
# 📌 补充：规则评分器骨架
from dataclasses import dataclass
from typing import Callable

@dataclass
class Rule:
    name: str
    weight: float
    gate: bool          # True = 否决项
    fn: Callable[[dict], tuple[bool, str]]

def rule_tests_pass(ctx) -> tuple[bool, str]:
    r = ctx["test_report"]
    return r["failed"] == 0, f"failed={r['failed']}, passed={r['passed']}"

def rule_http_200(ctx) -> tuple[bool, str]:
    codes = ctx["http_codes"]
    bad = [c for c in codes if c >= 400]
    return not bad, f"bad_codes={bad}"

def rule_no_console_error(ctx) -> tuple[bool, str]:
    errs = ctx["console_errors"]
    return len(errs) == 0, f"errors={len(errs)}"

def rule_regression_clean(ctx) -> tuple[bool, str]:
    new_fail = ctx["regression"]["new_failures"]
    return len(new_fail) == 0, f"new_failures={new_fail}"

def rule_cost_budget(ctx) -> tuple[bool, str]:
    cost, budget = ctx["cost_usd"], ctx["budget_usd"]
    return cost <= budget, f"{cost:.3f}/{budget:.3f} USD"

def rule_latency_budget(ctx) -> tuple[bool, str]:
    sec, budget = ctx["ai_seconds"], ctx["budget_seconds"]
    return sec <= budget, f"{sec}s/{budget}s"

RULES = [
    Rule("tests_pass",       1.0, True,  rule_tests_pass),
    Rule("http_200",         1.0, True,  rule_http_200),
    Rule("no_console_error", 0.5, True,  rule_no_console_error),
    Rule("regression_clean", 1.0, True,  rule_regression_clean),
    Rule("cost_budget",      0.3, False, rule_cost_budget),
    Rule("latency_budget",   0.3, False, rule_latency_budget),
]

def run_rules(ctx) -> dict:
    results, gate_failed = {}, []
    for r in RULES:
        ok, detail = r.fn(ctx)
        results[r.name] = {"pass": ok, "detail": detail}
        if r.gate and not ok:
            gate_failed.append(r.name)
    return {
        "rules": results,
        "gate_passed": not gate_failed,
        "gate_failed": gate_failed,
        "score": sum(r.weight for r in RULES if results[r.name]["pass"])
                 / sum(r.weight for r in RULES),
    }
```

### 10.3 LLM-as-a-Judge：Prompt 写法

📌 补充。下面是作者描述的三个判断点（返工信号、需求覆盖、是否可上线）的可执行版本。**关键设计：强制结构化输出 + 强制引用证据 + 允许说「不确定」。**

```text
# System
你是一个严格的 AI Coding 任务评审员。你不负责鼓励，只负责判断。
你的结论会被用于决定该任务是否可以进入人工验收。

# 规则
1. 只依据给定的需求清单、执行轨迹（trace）与证据文件做判断，不得推测未给出的内容。
2. 每一条判断必须引用具体证据：需求条目编号、工具调用序号、或文件路径。
3. 如果证据不足以判断，必须输出 "unknown" 并把 confidence 设为 <= 0.5，禁止编造。
4. 不要因为「代码看起来很多」就认为任务完成。逐条对照需求清单。

# 输入
[需求清单]
{{requirement_checklist}}     # 例：1. 改主色 2. 字号阶梯 3. 卡片间距 4. 移动端适配

[执行轨迹摘要]
{{trace_summary}}             # 每步：序号 / 动作 / 文件 / 结果

[证据]
{{evidence_list}}             # junit.xml / 截图路径 / 测试报告

# 输出（严格 JSON，不要 markdown 代码块）
{
  "requirement_cover": <0~1 的浮点数>,
  "covered_items": [<需求条目编号>],
  "missing_items": [<需求条目编号>],
  "rework_signal": <0 或 1>,
  "rework_evidence": "<引用用户原话或轮次>",
  "regression_risk": <0~1>,
  "code_quality": <1~5 的整数>,
  "verdict": "pass" | "needs_human_review" | "fail",
  "confidence": <0~1>,
  "reason": "<不超过 80 字>"
}
```

作者点名的两个判断点，落到具体判据：

| 判断点 | 判据（写进 Prompt） | 输出 |
|---|---|---|
| **是否要求返工** | 用户后续消息中出现「我不是说了」「又」「还是不对」「重新」等否定/重复语义 | `rework_signal` = 1 |
| **是否问移动端** | 用户提到「手机」「移动端」「小屏」「适配」且需求清单无此项 | 记为「需求遗漏」而非「返工」 |
| 需求覆盖 | 逐条比对 `requirement_checklist` 与轨迹中出现的文件改动 | `requirement_cover` |

### 10.4 人工标注：把人的时间花在刀刃上

> 「人的精力是最高的，我们需要把人这个事情放在我最应该发挥作用的地方。」

📌 补充一张「该不该转人工」的决策表：

| 情况 | 是否转人工 | 理由 |
|---|---|---|
| 规则全绿 + 模型 `pass` + confidence > 0.8 | ❌ 不转 | 省钱，直接放行 |
| 模型 `needs_human_review` | ✅ 转 | 它自己都不确定 |
| 模型 `fail` 但规则全绿 | ✅ 转（抽检） | 可能是模型幻觉 |
| 涉及视觉/审美 | ✅ 转 | 模型判不准 |
| 首次做某类需求 | ✅ 转 | 没有历史基线 |
| 成本超预算 3 倍 | ✅ 转 | 需要人决定值不值 |

**人工标注表单长什么样**（📌 补充，字段直接对应第 9 章 Rubric）：

```text
Task: REQ-1024 / trace 7f3a...c21 / version v7.20260928
────────────────────────────────────────────────
[ ] 功能验收：三个需求点是否都能看到效果？      1 2 3 4 5
[ ] 视觉验收：主色 / 字号 / 间距 / 圆角          1 2 3 4 5
[ ] 回归验收：原功能是否正常？                   通过 / 不通过
[ ] 可上线：敢不敢今天就发？                     是 / 否
备注（必填，若任一项 <= 3 或选「否」）：
________________________________________________
```

最后一条「备注必填」很重要：**没有理由的低分等于没有信息**，下周你还是不知道怎么改。

---

## 11. 阈值、回归门禁与 CI/CD

> ⚠️ 本章为 📌 补充。视频只提到「版本号 +1」「硬指标不能跳过」「上线稳定性」，没有演示流水线。但它正是「闭环」能不能自动转起来的关键，所以补全。

### 11.1 门禁的三道闸

```
PR 提交
  │
  ├─ 闸门 1：规则闸（秒级）      测试 / 200 / 回归 / 预算
  │     失败 → 直接 block，不烧 Token
  │
  ├─ 闸门 2：模型闸（十秒级）    覆盖率 / 质量 / 返工信号
  │     失败 → needs_human_review，进标注队列
  │
  └─ 闸门 3：人工闸（分钟级）    验收 + 上线判断
        通过 → 允许合并 / 发布
```

### 11.2 回归门禁规则表

| 规则 | 条件 | 严重级别 | 动作 |
|---|---|---|---|
| 核心用例失败 | `core_tests.failed > 0` | **Blocker** | 阻断合并 |
| 新增回归失败 | `regression.new_failures > 0` | **Blocker** | 阻断合并 |
| 通过率下跌 | `pass_rate < baseline - 5%` | **Blocker** | 阻断合并 |
| 成本超基线 | `cost > baseline_cost × 2` | Major | 需审批 |
| 耗时超基线 | `p90_latency > baseline × 1.5` | Major | 需审批 |
| 需求覆盖不足 | `cover < 0.8` | Major | 转人工 |
| 质量分下跌 | `quality < baseline - 1`（5 分制） | Minor | 记周报 |
| 连续两次返工 | `rework_rounds >= 2` | Minor | 记周报 + 复盘 |

**注意 Blocker 只有前三条。** 门禁设计最常见的错误是「什么都卡」，结果大家开始绕过门禁——一旦绕过成为习惯，门禁就死了。

### 11.3 Golden Set / Baseline：回归的锚点

| 概念 | 是什么 | 怎么建 |
|---|---|---|
| **Golden Set**（黄金集） | 一组**已知正确答案**的任务，用来防止退化 | 挑 20~50 个历史需求，人工标注「什么样的产出算合格」 |
| **Baseline** | 当前版本的指标快照 | 每次发版前跑一遍，存下 `pass_rate / cost / latency / quality` |
| **Run** | 一次评测执行 | 同一个 Golden Set 跑一遍 = 一个 Run |
| **A/B** | 两版配置对比 | 同 Golden Set 跑 v6 与 v7，比分数分布 |

```
Golden Set（固定 30 个需求）
        │
        ├── Run A：v6.20260901  →  pass 24/30  cost $0.42  p90 18min
        └── Run B：v7.20260928  →  pass 27/30  cost $0.51  p90 22min
                                    ↑ 质量 +3      ↑ 成本 +21%
                                    └─ 决策：值得，因为省下的人工 review 时间更多
```

**没有 Golden Set 的评测是假的。** 你今天看到「通过率 90%」，明天看到「85%」，但你不知道是变差了还是今天的任务变难了。只有固定集合，数字才可比。

### 11.4 CI 配置示例

```yaml
# .github/workflows/agent-eval.yml（📌 补充）
name: agent-eval

on:
  pull_request:
    paths: ["prompts/**", "skills/**", "mcp/**", "agent/**"]

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 拉起 Langfuse（复用 self-host compose）
        run: docker compose -f infra/langfuse/docker-compose.yml up -d

      - name: 等待健康
        run: |
          for i in $(seq 1 30); do
            curl -sf http://localhost:3000/api/public/health && break
            sleep 2
          done

      - name: 跑 Golden Set
        env:
          LANGFUSE_PUBLIC_KEY: ${{ secrets.LF_PK }}
          LANGFUSE_SECRET_KEY: ${{ secrets.LF_SK }}
          LANGFUSE_HOST: http://localhost:3000
          AGENT_VERSION: ${{ github.sha }}
        run: python eval/run_golden_set.py --set golden-v1 --out reports/run.json

      - name: 规则闸
        run: python eval/gate.py --report reports/run.json --mode rules

      - name: 模型闸
        env:
          JUDGE_API_KEY: ${{ secrets.JUDGE_KEY }}
        run: python eval/gate.py --report reports/run.json --mode judge

      - name: 对比基线（回归门禁）
        run: |
          python eval/compare.py \
            --baseline eval/baselines/v6.json \
            --current reports/run.json \
            --max-cost-ratio 2.0 \
            --max-latency-ratio 1.5 \
            --min-pass-delta -0.05

      - name: 上传报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: eval-report
          path: reports/
```

```python
# eval/compare.py 的核心判断（📌 补充，节选）
def compare(baseline, current):
    checks = []

    d_pass = current["pass_rate"] - baseline["pass_rate"]
    checks.append(("pass_rate", d_pass >= args.min_pass_delta,
                   f"{baseline['pass_rate']:.1%} → {current['pass_rate']:.1%}"))

    r_cost = current["avg_cost"] / baseline["avg_cost"]
    checks.append(("cost", r_cost <= args.max_cost_ratio,
                   f"×{r_cost:.2f}"))

    r_lat = current["p90_latency"] / baseline["p90_latency"]
    checks.append(("latency", r_lat <= args.max_latency_ratio,
                   f"×{r_lat:.2f}"))

    failed = [name for name, ok, _ in checks if not ok]
    for name, ok, detail in checks:
        print(f"{'PASS' if ok else 'FAIL'}  {name:12s} {detail}")
    sys.exit(1 if failed else 0)
```

### 11.5 版本发布与评测的关系

```
改 Prompt / Skill / MCP / 升级 Codex
        │
        ▼
   version + 1（例：v7 → v8）   ← 第 3 章那条规则
        │
        ▼
   跑 Golden Set（Run）
        │
        ├─ 通过 → 灰度 10% 用户 → 观察 24h → 全量
        └─ 不通过 → 回滚配置，version 再 +1（v9 = v7 的配置）
```

📌 注意最后一行：**回滚也要有新版本号**。否则你会得到两条 `version=v7` 但行为不同的 trace，历史数据彻底失去可比性。

---

## 12. 看板与核心指标：把「感觉」换成数字

### 12.1 作者点到的业内指标

> 「在业内呢，它还有一些其他的指标，什么**任务成功率、验收通过率、单个任务的成功成本**这些东西，大家可以看我这个文档……这就是业内大家去评判你这个效果做得好不好。」

> 「还有一些核心指标，比如说有多少任务真正做成功了，然后完成一个被验收的任务需要花多少钱，多久才能拿到验收结果，多少任务第一次……」

最后一句转写被截断，按上下文应为「多少任务**第一次就通过**」（first-pass rate）。📌 按语义补全，标注为推测。

| 指标 | 定义 | 为什么重要 |
|---|---|---|
| **任务成功率** | 成功任务 / 总任务 | 最粗的健康度 |
| **验收通过率** | 人工验收通过 / 提交验收 | 区分「自以为完成」与「真完成」 |
| **首次通过率** | 一次通过 / 总任务 | 衡量返工成本（📌 推测补全） |
| **单任务成功成本** | 总成本 / **成功**任务数 | 关键：分母是成功数，不是总数 |
| **验收周期** | 从提交到拿到验收结果的时间 | 衡量流程效率 |
| **返工轮数** | 平均往返次数 | 衡量需求表达与 Agent 能力 |

**「单任务成功成本」为什么比分母用总数更狠？** 举例：跑 100 个任务花 $100，成功 50 个 → 单任务成本是 $1，但**单成功任务成本是 $2**。失败的任务不仅浪费钱，还浪费人的 review 时间。这个指标一算出来，很多「AI 提效」的故事就讲不下去了。

### 12.2 建议自建的 6 块看板

📌 补充。Langfuse 自带的 5 个 dashboard 作者说「用得比较少」，因为它们是通用的。真正有用的是按你的业务问题切的看板：

| 看板 | 回答的问题 | 图表 |
|---|---|---|
| **① 总量与成本** | 今天花了多少？趋势如何？ | 折线（cost/day）+ 柱状（traces/day） |
| **② 质量趋势** | 分数在涨还是在跌？ | 折线（score 均值，按 version 分组） |
| **③ 版本对比** | v7 真的比 v6 好吗？ | 箱线图（同指标跨 version） |
| **④ 耗时分解** | 卡在哪一步？ | 堆叠柱（planning / edit / test / wait） |
| **⑤ 失败归因** | 为什么没做完？ | 饼图（测试失败 / 覆盖率不足 / 超时 / 预算） |
| **⑥ 返工分析** | 哪类需求最容易返工？ | 热力图（需求类型 × 返工轮数） |

### 12.3 一个必须拆开的指标：Session 执行时间

这是作者讲得最细的一个坑：

> 「在这个 Session 里面，我们可以看到它每一个 Session 它的执行时间，这个 4 个小时、2 个小时，就是我自己在跑的一些任务。这个执行时间呢，它是包含了**你大模型在里面执行**和**人的等待时间**。有可能你问他一个问题，你自己喝水去了或者吃饭去了，你回来之后这个时间也是在算的。你后面又问了他新的问题，他都会被统计进来。所以说呢，我们可能还得有两个指标得去看一下，就是说：**AI 工作了多久**，以及**人等了多久**。」

用图说明这个「虚高」：

```
Session 墙钟时间 = 4h 00m
├──────────────────────────────────────────────────────────┤
│ AI 干活  ██ 12m    ████ 25m      ██ 8m                   │  ≈ 45m（19%）
│ 人等待        ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │  ≈ 3h 15m（81%）
└──────────────────────────────────────────────────────────┘
        ↑ 如果只看墙钟时间，你会得出「AI 太慢了」的错误结论
```

| 指标 | 定义 | 数据来源 |
|---|---|---|
| **墙钟时间**（wall clock） | Session 首尾时间差 | Langfuse Session 起止 |
| **AI 有效时间** | 所有 GENERATION + 工具 SPAN 的耗时之和 | Trace 内 span 累加 |
| **人的等待时间** | 墙钟 − AI 有效时间 | 计算得出 |
| **人的介入次数** | 人发消息的轮数 | Trace 条数 |

**为什么必须拆？** 因为两者的优化手段完全不同：

| 如果瓶颈是 | 优化方向 |
|---|---|
| AI 有效时间长 | 换模型、缩上下文（回 L4）、减少工具往返、并行化 |
| 人的等待时间长 | 不是技术问题——是**流程问题**：人没盯着、或者任务粒度太大 |

📌 补充一个推论：**「AI 提效」的正确度量是「AI 有效时间 vs 人做同样事的时间」，而不是「墙钟时间」。** 很多团队的「AI 让开发从 8 小时变 4 小时」其实是把人喝咖啡的时间算进去了。

### 12.4 成本看板的三个坑

| 坑 | 现象 | 解法 |
|---|---|---|
| 只统计成功的任务 | 看起来成本很低 | 分母改成总任务，另算「单成功成本」 |
| 忘记缓存命中 | 成本虚高 3~10 倍 | 记录 `cached_tokens`，单独算 |
| 不同模型混在一起平均 | 看不出谁在烧钱 | 按 `metadata.model` 分组 |

---

## 13. 评测与改进闭环全景

把所有碎片拼成一张图。这张图是本篇最重要的产出——**它是一个环，不是一条线**。

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        AI Coding 评测与改进闭环                                │
└──────────────────────────────────────────────────────────────────────────────┘

   ① 生产使用                     ② 采集                    ③ 取样/成集
 ┌──────────────┐            ┌──────────────┐          ┌──────────────┐
 │ 开发者用      │  埋点上报   │  Langfuse    │  筛选    │  Golden Set  │
 │ Codex / CC / │ ─────────→ │  Session     │ ───────→ │  + Dataset   │
 │ Cursor / 自研 │  (OTEL/SDK)│   └ Trace    │ 按类型/   │  (按需求类型) │
 └──────────────┘            │      └ Obs   │ 按失败    └──────┬───────┘
        ▲                    │  + metadata  │                   │
        │                    │  (需求ID/版本)│                   │
        │                    └──────────────┘                   │
        │                                                       ▼
        │                                            ┌──────────────────┐
        │                                            │  ④ 评分（三层）   │
        │                                            │ 程序 → 模型 → 人  │
        │                                            │ 规则  Judge 标注  │
        │                                            └────────┬─────────┘
        │                                                     │
        │                                                     ▼
        │                                            ┌──────────────────┐
        │            ⑦ 上线 / 灰度                    │  ⑤ 定位归因       │
        │        ┌──────────────────────┐            │ 哪一步慢？哪一步贵？│
        │        │ version +1，灰度 10% │            │ 哪个需求点漏了？   │
        │        └──────────▲───────────┘            └────────┬─────────┘
        │                   │                                 │
        │                   │                                 ▼
        │        ┌──────────┴───────────┐            ┌──────────────────┐
        │        │  ⑥ 回归门禁（CI）     │◄───────────│  改 Prompt /     │
        │        │ Golden Set + Baseline│   改完重跑  │  Skill / MCP /   │
        │        │ 规则闸→模型闸→人工闸  │            │  知识库 / 上下文  │
        └────────┤ 不通过则阻断          │            └──────────────────┘
      下一个需求  └──────────────────────┘
```

**七步各自的最小可行动作：**

| 步 | 动作 | 产出 | 对应章节 |
|---|---|---|---|
| ① 生产 | 接 SDK / OTEL，打 metadata | 有 Trace | 6.3 / 6.4 |
| ② 采集 | 起 Langfuse，确认数据进来 | Trace 列表有数据 | 6.2 |
| ③ 取样 | 建 Golden Set，按类型分组 | 30 条固定任务 | 11.3 |
| ④ 评分 | 规则脚本 + Judge Prompt + 标注表单 | 结构化分数 | 10.2 / 10.3 / 10.4 |
| ⑤ 定位 | 看时间线、看失败 span、看覆盖率 | 一条明确的改进项 | 4.1 / 8.3 |
| ⑥ 回归 | CI 三道闸 + baseline 对比 | 通过/阻断 | 11.1 / 11.4 |
| ⑦ 上线 | version+1，灰度观察 | 新基线 | 11.5 |

**环上最容易断的三个点**（📌 补充，来自实践）：

| 断点 | 表现 | 后果 |
|---|---|---|
| ② → ③ 断了 | 只采集不整理，Trace 堆成山没人看 | 有数据没结论 |
| ⑤ → ⑥ 断了 | 定位到问题，改完不跑回归 | 改 A 坏 B，反复横跳 |
| ⑥ → ① 断了 | 门禁跑在 CI 里，但线上版本没打 version | 数据混在一起，无法归因 |

---

## 14. 团队接入：一个统一评测网关

### 14.1 作者的方案

> 「你们可能整个团队大家用的工具五花八门，有用 Codex 的，有用 Claude Code 的，有用 Qoder 的，有用 Cursor 的。但是最终可能需要你们把这个 Langfuse 部署到自己团队的一个机器上面去，然后大家用的时候只需要去接入一下你们的一个**通用的网关，通用的一个评测网关**，然后由它来统一地去上报到 Langfuse 里面去。你们可以去看自己执行的一个效果，**但是这个东西得你们自己去写了**。」

注意最后半句：**作者明确说这个网关是要自己写的**，没有现成方案。

### 14.2 为什么要网关，而不是让每个人直连

| 直连的问题 | 网关怎么解 |
|---|---|
| 每个人配 Key、配环境变量，配错就丢数据 | 统一配置，客户端零配置 |
| 不同工具字段不一致，看板没法聚合 | 统一 schema（特别是 `requirement_id`、`version`） |
| 有人忘记 `requirement_end`，ID 泄漏 | 网关做超时自动关闭 |
| 敏感信息（内网 URL、密钥）被打进 Trace | 网关做脱敏 |
| 采样率无法统一控制 | 网关做 Sampling |
| Langfuse 挂了，业务跟着挂 | 网关做异步缓冲 + 降级 |

### 14.3 网关架构

```
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Codex   │ │ Claude  │ │ Qoder   │ │ 自研    │
│         │ │ Code    │ │         │ │ Agent   │
└────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘
     │           │           │           │
     └───────────┴─────┬─────┴───────────┘
                       ▼
        ┌──────────────────────────────┐
        │      统一评测网关             │
        │  ┌────────────────────────┐  │
        │  │ ① 鉴权（SSO / Token）   │  │
        │  │ ② 归一化 schema         │  │
        │  │ ③ 脱敏（密钥/内网 IP）   │  │
        │  │ ④ 补全（req_id/version） │  │
        │  │ ⑤ 采样（Sampling）       │  │
        │  │ ⑥ 缓冲队列（异步）        │  │
        │  └────────────────────────┘  │
        └──────────────┬───────────────┘
                       ▼
              ┌────────────────┐
              │   Langfuse     │
              │ (self-host)    │
              └────────────────┘
```

### 14.4 网关最小实现（FastAPI）

```python
# 📌 补充：极简版网关，仅演示六个职责的落点
import re, time, asyncio
from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel
from langfuse import Langfuse

app = FastAPI()
lf = Langfuse()  # 读环境变量

SECRET_PATTERNS = [
    (re.compile(r"sk-[A-Za-z0-9]{20,}"), "sk-***"),
    (re.compile(r"AKIA[0-9A-Z]{16}"), "AKIA***"),
    (re.compile(r"\b10\.\d+\.\d+\.\d+\b"), "10.x.x.x"),
]

def sanitize(text: str) -> str:
    for pat, repl in SECRET_PATTERNS:
        text = pat.sub(repl, text)
    return text

class IngestBody(BaseModel):
    agent: str                 # codex / claude-code / qoder / cursor
    session_id: str
    trace_id: str
    name: str
    version: str
    requirement_id: str | None = None
    requirement_type: str = "web"
    model: str | None = None
    input: str | None = None
    output: str | None = None
    ai_seconds: float | None = None

@app.post("/v1/ingest")
async def ingest(body: IngestBody, x_gateway_token: str = Header(...)):
    if x_gateway_token != "team-token":
        raise HTTPException(401, "bad token")

    # ② 归一化 + ③ 脱敏 + ④ 补全
    lf.trace(
        id=body.trace_id,
        name=body.name,
        session_id=body.session_id,
        version=body.version,
        tags=[body.requirement_type, body.agent],
        input=sanitize(body.input or ""),
        output=sanitize(body.output or ""),
        metadata={
            "agent": body.agent,
            "model": body.model,
            "requirement_id": body.requirement_id,
            "requirement_type": body.requirement_type,
            "ai_seconds": body.ai_seconds,
            "gateway_ts": time.time(),
        },
    )
    lf.flush()
    return {"ok": True}
```

### 14.5 采样策略

全量上报在团队规模上去之后会撑爆 ClickHouse。📌 补充推荐策略：

| 数据 | 采样率 | 理由 |
|---|---|---|
| 失败的 Trace | **100%** | 失败样本最贵，不能丢 |
| 人工标注过的 Trace | **100%** | 是黄金集的来源 |
| 成本超阈值的 Trace | **100%** | 要查为什么贵 |
| 普通成功 Trace | 10% ~ 20% | 用于统计趋势足够 |
| 工具调用明细（span 级） | 成功任务可只留 30% | 体积最大，价值最低 |

一条经验：**采样一定要「按 Trace 整条采」，不要按 span 随机采**。半条 trace 的 span 树是没法看时间线的。

---

## 15. 开源 self-host vs 云版本

📌 补充。视频里作者只说「用开源工具节省时间」「部署到自己团队机器上」，没有做版本对比。下表供选型参考，**具体配额与价格请以官方为准**。

| 维度 | 开源 self-host | 云版本（Cloud） |
|---|---|---|
| **部署** | Docker Compose / K8s，4 个组件 | 注册即用 |
| **运维成本** | 高（备份、升级、扩容、ClickHouse 调优） | 零 |
| **数据主权** | 完全自有 | 在第三方 |
| **合规** | 可满足内网/等保要求 | 需评估 |
| **成本模型** | 服务器 + 人力 | 按事件数 / 席位 |
| **上手速度** | 半天到两天 | 十分钟 |
| **功能完整度** | 基本一致（核心功能开源） | 略超前（新功能先上云） |
| **多租户 / SSO** | 需自行接 | 内置 |
| **告警 / Webhook** | 支持 | 支持 |
| **适合** | 有代码保密要求、量大、有运维 | 小团队、快速验证、无运维 |

**决策建议（📌 补充）**：

```
先 Cloud 跑通方法论（1 周）
      │  确认「评测确实有用、指标体系确实能落地」
      ▼
量上来 / 合规要求出现
      │
      ▼
迁 self-host（用同一套 SDK，只改 LANGFUSE_HOST）
```

关键点：**SDK 是同一套**。所以先用云版本验证方法论、再迁自建，代码几乎不用改——这也是作者说「Langfuse 只是个后台」的另一个好处。

---

## 16. 速查卡：概念 / 页面 / 命令一页查完

### 16.1 概念对照

| 概念 | 一句话 | 在你的系统里对应什么 |
|---|---|---|
| **Session** | 一段连续对话 | 一个需求的多个会话 |
| **Trace** | 一次提问 / 一次任务 | 开发者问 AI 一次 |
| **Observation** | 一步动作 | 读文件、跑测试、调模型 |
| **Span** | 有起止的工作段 | 工具调用、检索、子 Agent |
| **Generation** | 一次 LLM 调用 | 有 token 和 cost |
| **Event** | 时间点事件 | 报错、打点、需求开始/结束 |
| **Score** | 一个分数 | 规则分 / 模型分 / 人工分 |
| **Dataset / Dataset Item** | 评测集 / 单条样本 | Golden Set |
| **Run** | 一次评测执行 | 跑一遍 Golden Set |
| **Annotation** | 人工标注队列 | 验收表单 |
| **Prompt Management** | Prompt 版本管理 | System Prompt 迭代 |
| **Playground** | 在线试 Prompt | 调 judge prompt 时用 |
| **Version** | 版本号 | Agent 配置的版本 |

### 16.2 页面速查

| 页面 | 什么时候去 |
|---|---|
| **Home / Dashboard** | 每天上班第一眼：花了多少、P99 多长、分数趋势 |
| **Sessions** | 想知道「这个需求一共折腾了几轮」 |
| **Traces** | 想知道「这一轮到底干了什么、卡在哪」 |
| **Trace 详情** | 定位单点问题：时间线、span、metadata、打分 |
| **Users** | C 端 Agent 排查某个用户 |
| **Prompt Management** | 改 Prompt、对比版本 |
| **Scores / Annotation** | 人工验收、看分数分布 |
| **Datasets** | 维护 Golden Set |
| **Settings → LLM Connections** | 接 judge 模型（如 DeepSeek） |
| **Settings → Evaluators** | 配自动评分器 |
| **Settings → Webhooks** | 接自己的告警系统 |

### 16.3 埋点五件套（缺一不可）

| 字段 | 不填会怎样 |
|---|---|
| `session_id` | 无法把多轮对话串起来 |
| `version` | 无法做版本对比 |
| `metadata.requirement_id` | 无法按需求聚合（第三层丢失） |
| `metadata.model` | 无法做模型对比，也无法解释成本 |
| `usage`（input/output tokens） | 成本看板永远是 0 |

### 16.4 命令速查

```bash
# 起服务
docker compose up -d

# 健康检查
curl -s http://localhost:3000/api/public/health

# 看 worker 是否在消费
docker compose logs -f langfuse-worker | tail -50

# 进 ClickHouse 查 trace 数量（排查「写了但看不到」）
docker compose exec clickhouse clickhouse-client \
  --user clickhouse --password clickhouse \
  -q "SELECT count() FROM traces"

# 备份 Postgres（元数据，必须备）
docker compose exec postgres pg_dump -U postgres langfuse > backup_$(date +%F).sql
```

---

## 17. 📌 补充：视频没讲但一定会踩的坑

### 17.1 需求 ID 泄漏

**现象**：某天发现 `REQ-1024` 下面挂着 300 条不相关的 trace。
**原因**：`requirement_end` 没被调用（异常退出、进程被杀、开发者忘了）。
**解法**：网关侧加**租约（lease）**——`requirement_start` 写入带 TTL 的键（比如 8 小时），过期自动视为结束；同时在新 `requirement_start` 到来时强制关闭上一个。

### 17.2 把 Session 当成需求

作者的模型是「一个需求多个 Session」，但很多人图省事直接用 `session_id = requirement_id`。**短期没问题，长期会崩**：因为 Session 在 Langfuse 里是「对话」语义，UI 上的展示、统计口径都按对话设计；当你把一个跨 3 天、含 5 段对话的需求塞进一个 Session，时间线会变得没法读。

### 17.3 成本永远是 0

三个原因，按概率排序：

1. 把 LLM 调用打成了 `span` 而不是 `generation`
2. 没传 `usage`（尤其是流式响应，最后一个 chunk 才有 usage）
3. 用的模型不在 Langfuse 的价格表里 → 需要手动配 `modelParameters` 或自定义价格

### 17.4 Judge 模型的三个偏差

| 偏差 | 表现 | 缓解 |
|---|---|---|
| **自我偏好** | 用 A 评 A 的产出，分数偏高 | judge 必须换厂商 |
| **长度偏好** | 越长越觉得好 | Prompt 里明说「不因篇幅长而加分」 |
| **位置偏好** | A/B 对比时偏爱第一个 | 交换顺序跑两次，取平均 |

📌 尤其第一条：作者用「Codex 干活 + DeepSeek 当裁判」，天然规避了自我偏好，这个组合值得抄。

### 17.5 「通过率」造假的三条路径

这不是恶意，而是无意识的自欺：

| 路径 | 怎么发生 | 怎么防 |
|---|---|---|
| 分母缩水 | 只统计「跑完了的」任务，超时的不算 | 分母必须是「发起的任务数」 |
| 标准漂移 | 分数低了就放宽 Rubric | Rubric 必须版本化，改版要重跑基线 |
| 样本挑选 | Golden Set 里全是简单任务 | 按难度分层抽样，并公开分布 |

### 17.6 全量回归太慢

Coding Agent 的全量回归可能要 30 分钟以上，直接拖垮评测流水线。📌 三级策略：

| 级别 | 跑什么 | 何时 |
|---|---|---|
| L1 冒烟 | 核心 10 个用例 | 每次提交，< 2 min |
| L2 影响面 | 按改动文件的依赖图选测 | 每次 PR，< 10 min |
| L3 全量 | 全部用例 | 合并前 / 每晚 |

### 17.7 别把 Langfuse 当监控用

Langfuse 是**评测后台**，不是 APM。它的写入是批量异步的（v3 走 worker + ClickHouse），**不适合做实时告警**。实时告警还是得走你们的监控系统——这也正是作者说「Langfuse 发现有问题后去调你们监控后台的钩子」的原因：**判断在 Langfuse，告警在监控**。

### 17.8 评测本身的成本

一个容易忽略的账：judge 模型也要钱。如果每个 trace 都跑一次 GPT-5 级别的 judge，评测成本可能超过 Agent 本身。

| 策略 | 效果 |
|---|---|
| judge 用便宜模型（DeepSeek / 小模型） | 成本降 10 倍，质量损失可接受 |
| 只对「规则通过」的 trace 跑 judge | 省掉 30%~50% |
| 采样：成功 trace 只跑 20% | 再省一半 |
| 缓存相同输入的 judge 结果 | 重复任务直接命中 |

### 17.9 一个反直觉的结论

作者在结尾说了一句值得单独拎出来的话：

> 「大家现在都疯狂地在学习怎么用 AI、怎么用 AI 做出一些 demo 或者用一些效果，但是真正把它用好、用细了，**评测是你始终绕不开的一个最重要的东西**。」

📌 换句话说：**会用 Skill 和 MCP 已经是标配，会度量才是分水岭。** 这也解释了为什么这个系列把「评测」放在第 5 集——前四集讲「怎么让它能干」，第 5 集讲「怎么知道它干得好不好」，第 6 集才讲「怎么让它干得更好」。

---

## 18. 🔗 与本目录其他篇的关系

本目录 4 篇是一个整体，本篇是最后一篇，也是最「落地」的一篇。

**第一篇《从零到一设计 Agent》** 给了 8 层架构总图（L1 模型 → L2 工具 → L3 记忆 → L4 上下文 → L5 RAG → L6 安全 → L7 工程 → L8 规模化），其中 L7 就是「Trace → Eval → 灰度」。本篇是把 L7 那三个词展开成一整套可运行的系统。读本篇前如果对 8 层没有印象，建议先回去看那张总图。

**第二篇《Agent 上下文分层管理》** 展开的是 L4。它和本篇的关系最直接：本篇第 4 章那张 Span 树里，`planning` 和 `edit_styles` 两个 Generation 的 `promptTokens` 之所以一个 3.2k、一个 8.9k，就是 L4 分层策略的直接结果。**上下文管得好不好，最终会体现在 L7 的成本和耗时上**——这是两篇之间唯一的、也是最硬的连接点。

**第三篇《Agent Memory 记忆管理》** 展开的是 L3。它管的「过去留下什么」，在本篇里的观测点是：Span 树里有没有 `retrieval` 类调用、命中了多少条、有没有把旧记忆错误地塞进新任务。作者在第 17 章提到的「我是用向量库组织的，还是用 LLM Wiki 组织的」，正是 L3 的选型问题——而**判断选型对错的唯一方法，就是看 L7 的评测数据**。

四篇合起来是一条完整的因果链：

```
L3 记忆 / L4 上下文 / L5 工具   →  决定了 Agent 的能力上限
              ↓
        L7 Trace 看见过程
              ↓
        L7 Eval 判断好坏
              ↓
        反馈回 L3 / L4 / L5 去改
              ↓
        L7 回归验证有没有变好
```

**所以本篇不是「第四篇」，而是前三篇的「验收器」。** 前三篇讲的所有设计决策，在这里都要接受数字的审判。

---

## 19. ✅ 落地检查清单

按顺序做，做完一项勾一项。前 5 项是「能看见」，中间 5 项是「能判断」，后 5 项是「能自动转」。

**阶段一：能看见（Trace）**

- [ ] **1. 部署 Langfuse self-host**：Postgres + ClickHouse + Redis + MinIO 四个组件都起来，`/api/public/health` 返回正常，MinIO bucket 已创建。
- [ ] **2. 打通一条最小链路**：用 SDK 或 curl 发一条 trace，能在 UI 的 Traces 页面搜到（先别管字段对不对）。
- [ ] **3. 埋点五件套齐全**：每条 trace 都有 `session_id`、`version`、`metadata.requirement_id`、`metadata.model`、`usage`。抽查 10 条，缺一不可。
- [ ] **4. LLM 调用标成 generation**：确认 Dashboard 的成本不再是 0，且单个 generation 的 token 数与真实调用对得上。
- [ ] **5. 需求开关 hook 上线**：`requirement_start` / `requirement_end` 成对出现，并加了 TTL 兜底，验证「新需求不会污染旧需求 ID」。

**阶段二：能判断（Eval）**

- [ ] **6. 定义版本号规则**：写清「哪些变更要 +1」，并在 Git 里打同名 tag；回滚也 +1。
- [ ] **7. 建 Golden Set**：挑 30 个历史需求，覆盖前端 / 后端 / 数据三类，人工标注「什么算合格」。
- [ ] **8. 规则评分器落地**：至少 6 条规则（测试通过、HTTP 200、无 console error、回归无新增失败、成本预算、耗时预算），且区分 gate 与 score。
- [ ] **9. Judge Prompt 写好**：强制 JSON 输出、强制引用证据、允许输出 `unknown`；judge 模型与被评模型**不同厂商**。
- [ ] **10. 人工标注表单可用**：4~5 个必填项，低分必填理由，能直接在 Langfuse 的 Annotation 里打分。

**阶段三：能自动转（闭环）**

- [ ] **11. 三道闸接进 CI**：规则闸（秒级）→ 模型闸（十秒级）→ 人工闸（分钟级），Blocker 只保留 3 条。
- [ ] **12. Baseline 快照机制**：每次发版前存一份 `pass_rate / cost / p90_latency / quality`，下次对比用。
- [ ] **13. 六个看板建好**：总量成本、质量趋势、版本对比、耗时分解、失败归因、返工分析。
- [ ] **14. 耗时拆成两段**：`ai_seconds` 与 `human_wait_seconds` 分开统计，别再拿墙钟时间说事。
- [ ] **15. 统一网关 + 采样**：多人多工具接同一个网关；失败 / 标注过 / 超预算的 trace 100% 采，成功 trace 10%~20%，**按整条 trace 采**。

**收尾自检（三个问题）**

- [ ] 我能用一句话说清「这个月 AI 帮我省了多少时间、花了多少钱」吗？说不清 → 回第 12 章。
- [ ] 我能指出「上周哪个需求没做完、卡在哪一步」吗？指不出 → 回第 4 章。
- [ ] 我改了一版 Prompt，能证明它比上一版好吗？证明不了 → 回第 11 章。

---

## 20. 一句话总结

> **Langfuse 给你的是「看见」的能力——Session / Trace / Observation 三层把 AI 干活的每一步摊开；而评测闭环给你的是「变好」的能力——用程序、大模型、人三种分工，按「做完了吗 / 质量好吗 / 划算吗」三问打分，再用版本号和回归门禁把每一次改动都关进可比较的笼子里。会用 Skill 和 MCP 只是入场券，能量化、能回归、能持续变好，才是分水岭。**

---

## 附录：本文信息

### A.1 视频信息

| 项 | 内容 |
|---|---|
| 标题 | Agent 系统设计 05：用 Langfuse 搭建 AI Coding 评测与改进闭环 |
| UP 主 | 桃子可冰乐 |
| 时长 | 17 分 41 秒 |
| 发布日期 | 2026-09-30 |
| 地址 | https://www.bilibili.com/video/BV1B8a96PEf6/ |
| 所属合集 | 《Agent系统设计》第 5 集（共 6 集） |
| 文稿地址 | https://my.feishu.cn/wiki/Ba9mwre86iflkCkXHlec9klHn4b |
| 本文定位 | 本目录 Agent 专题第 4 篇（共 4 篇），8 层总图中的 **L7 工程层** |

**6 集系列全景**（本篇为第 5 集，不涉及其他集内容）：

| 集数 | 主题 | 与本篇的关系 |
|---|---|---|
| 01 | 意图识别 | 上游：识别出「这是什么需求」，本篇才能按类型选 Rubric |
| 02 | 单意图评测 | 同族：评测的细粒度版本，本篇是工程化落地 |
| 03 | 多意图评测 | 同族：多意图场景的评测设计 |
| 04 | Agent 架构选型 | 上游：选了什么架构，决定了 Trace 长什么样 |
| **05** | **用 Langfuse 搭建评测与改进闭环** | **本篇** |
| 06 | LLM Context Engineering | 下游：本篇定位出的「上下文太长导致贵/慢」，在 06 里解决 |

### A.2 视频章节与时间戳

| 时间 | 内容 | 对应本文 |
|---|---|---|
| 00:00 | 开场：为什么评测比「会用 Skill/MCP」更稀缺 | 第 1 章 |
| 00:31 | 现状：每个团队都说自己提效了，但没有过程记录 | 1.1 |
| 00:40 | 需求维度的工作方式：工作区 + 多个 Session | 3.1 |
| 01:01 | Langfuse 三层：Session / Trace / Observation（故事书比喻） | 第 2 章 |
| 01:58 | 弊端：真实业务是「需求 → 多 Session」，工具只有两层 | 3.1 |
| 02:20 | Filter 的五个概念；Version 为什么要 +1 | 3.3 |
| 03:12 | 为什么选 Langfuse：开源、成熟、省时间 | 1.3 |
| 03:55 | Langfuse 是后台，任何 AI Coding Agent 都能接 | 1.3 |
| 04:09 | Home 看板：对话数、花费、分数、P99、5 个 dashboard | 5.1 |
| 04:39 | Trace 列表：一个 Session 14 条 Trace，按时间倒序 | 5.2 |
| 05:34 | 时间线：耗时耗在哪一步 | 4.1 |
| 05:41 | 单条 Trace 打分 | 5.3 |
| 05:48 | Metadata：知道调了什么模型；需求 ID 挂这里 | 4.2 / 3.1 |
| 06:26 | 需求开始/结束的 hook：不告诉它就会污染 | 3.2 |
| 06:56 | Users 页；C 端 Agent 必须告警；Webhook 回调 | 5.4 |
| 07:50 | Prompt 管理与实验；Scores 列表由自己定义 | 5.5 / 5.6 |
| 08:06 | 人工标准 / 规则标准 / 评测集三个抽屉 | 5.7 |
| 08:07 | Settings 里接 DeepSeek 当 judge 模型 | 5.8 |
| 08:07 | 评测分工三种：程序 / 大模型 / 人 | 第 7 章 |
| 09:30 | 评测流程：按需求类型选评分表 | 8.1 |
| 10:04 | 第一问：做完了吗（程序 → 模型初筛 → 人终审） | 8.2 |
| 10:43 | 全流程：AI 开发 → 程序验 → 模型初筛 → 用户验收 | 8.2 |
| 11:18 | 第二问：质量好不好（回归、覆盖度、幻觉） | 8.3 |
| 11:58 | 上线稳定性、证据完整性 | 8.4 |
| 12:20 | 硬指标不能跳过：核心测试、不破坏功能、可访问、能验收 | 8.5 |
| 12:25 | 第三问：划算吗（钱 / 时间 / 返工） | 8.6 |
| 13:48 | 三方分别统计什么；业内指标 | 8.6 / 12.1 |
| 14:31 | Session 执行时间包含人的等待；需拆两个指标 | 12.3 |
| 15:07 | 团队接入：self-host + 统一评测网关 | 第 14 章 |
| 15:27 | 多工具（Codex/Claude Code/Qoder/Cursor）统一上报 | 14.1 |
| 16:00 | 评测的两个目的：数字落地 + 定位优化空间 | 1.2 |
| 17:16 | 总结：评测是绕不开的一环 | 第 20 章 |

### A.3 转写同音错误修正对照

| 转写原文 | 正确术语 | 出现场景 |
|---|---|---|
| 兰福斯 / 男朋S / nonfuse / longfuse / non field / longfeuse / non费 / 那费 / nonfi / 男朋S | Langfuse | 全片高频 |
| 设置ID / 设置型 / C型 / since / 系 | Session（ID） | 第 2、3 章 |
| obsessing / obsidian | Observation | 1:01 |
| liquid ID | 需求 ID（requirement ID） | 1:01、6:23 |
| Tuserrelease 还有 version | Tags / User / Release / Version | 2:20 |
| Pro scale / Pro 管理 | Prompt / Skill、Prompt Management | 2:57、7:50 |
| C 文件里面 | Settings 里面 | 8:07 |
| Grace 为什么这么慢 | Trace 为什么这么慢 | 5:39 |
| 问题集 | Dataset（评测集） | 5:31 |
| 字节的 trace | 字节 Trae | 3:55 |
| Q拉code / q code / 客用 Cursor | Claude Code / Qoder / Cursor | 15:27 |
| 路普啊哈里斯 | Loop、Harness | 13:48 |
| SDD、Sskill | SDD、Skill | 13:48 |
| LMvki | LLM Wiki（按语音推测） | 16:26 |
| 欧特 / 克里克豪斯 / 如布雷克 | OTEL / ClickHouse / Rubric | （本篇补充章节涉及的术语） |
| 用户必须明天预收验收 | 用户必须能验收 | 12:20 |
| 多少任务第一次……（截断） | 首次通过率 | 14:30 |

### A.4 整理说明

- **结构**：按视频讲解顺序重排为 20 章 + 附录。原视频的实际顺序是「背景 → 数据模型 → 界面漫游 → 评测分工 → 评测流程 → 指标 → 团队接入 → 意义」，本文把散落在界面漫游里的「需求 ID / Version」抽成了独立的第 3 章，把「三问」合并成第 8 章，其余顺序未变。
- **补充边界**：视频是「界面演示 + 方法论」，几乎未贴代码。第 6、9、10、11、14、15、17 章的具体实现（docker-compose、SDK 埋点、OTEL 配置、Rule 评分器、Judge Prompt、CI 门禁、网关、坑位）**全部为 📌 补充**，已在各章开头标注。第 6 章开头有一句总声明：「除『为什么自建』是视频原意，其余全部为补充」。
- **字段名处理**：Langfuse 界面上的字段名（Session / Trace / Observation / Score / Version / Metadata / Annotation / Dataset / Settings → LLM Connections）按作者口述还原；作者未念出的部分（如 Filter 的完整五项、Evaluators / Webhooks 的具体菜单路径）标注为「按演示推测」。
- **两张 ASCII 图**：第 4 章的 Span 树是按作者描述的粒度与真实 Coding Agent 的典型轨迹重建的（非视频截图）；第 13 章的闭环全景是按作者「记录 → 评测 → 定位 → 改进 → 回归」的叙述重建的。
- **数字与阈值**：第 9.4 节的阈值表、第 11.2 节的门禁规则均为经验起点，**必须用你们自己的历史数据校准**，不要直接照搬。作者本人的完整评分表在简介给出的飞书文档里，本篇未引用其具体数值。

### A.5 引用作者原话速查

| 主题 | 原话 |
|---|---|
| 为什么评测 | 「大家已经不缺怎么用 Skill、用 MCP 了，更缺的是我怎么来衡量它的效果」 |
| 三层模型 | 「它总共就三个概念，一个是 Session，还有 Trace、还有 Observation」 |
| 需求 ID | 「你得主动地告诉它……写一个相当于触发的钩子函数」 |
| 为什么用 Version | 「你动了这些东西之后，效果可能比以前好或比以前差，所以版本号也会加 1」 |
| Langfuse 定位 | 「Langfuse 本质上是一个后台，你只要按照它的接口要求上传你的 Agent 的调用过程」 |
| 三方分工 | 「人的精力是最高的，我们需要把人这个事情放在最应该发挥作用的地方」 |
| 三问顺序 | 「如果都没做完，不谈后面的；做完之后再看他做的质量怎么样」 |
| 硬指标 | 「硬指标是不能跳过去的」 |
| 划算吗 | 「你做一个需求就要花个几百美刀，那你肯定扛不住」 |
| 耗时陷阱 | 「你自己喝水去了或者吃饭去了，你回来之后这个时间也是在算的」 |
| 网关 | 「这个东西得你们自己去写了」 |
| 意义 | 「真正把它用好、用细了，评测是你始终绕不开的一个最重要的东西」 |
