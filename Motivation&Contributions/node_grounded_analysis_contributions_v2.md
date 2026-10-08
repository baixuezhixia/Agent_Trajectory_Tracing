# 从 Node Inspection 到 Node-grounded Analysis：研究定位与 Contributions

## 1. 核心研究定位

现有 Agent trajectory visualization（以 Graph of Trace, GoT 为代表）已经能够：

```text
Agent Execution
      ↓
Execution Trace
      ↓
Task / Subtask Graph
      ↓
Real-time Visualization
```

用户可以查看 graph、点击节点并检查节点已有的信息。

这种交互本质上主要是：

> **Node Inspection**

即：

```text
看到节点
   ↓
点击节点
   ↓
查看系统已经展示的信息
```

我们的目标是在此基础上进一步支持：

> **Node-grounded Question-driven Analysis**

即用户可以围绕 visualization 中的节点继续提出新的问题并进行分析。

例如：

```text
Node:
完成计算机作业

        ↓

Question:
这个作业的 ddl 是什么时候？

        ↓

系统根据 query
定位真正相关的 node / nodes
        ↓
取出相关节点维护的 context
        ↓
回答问题
```

因此，我们希望将 visualization 从：

```text
Inspection Interface
```

扩展为：

```text
Question-driven Analysis Interface
```

---

# 2. 与 Graph of Trace 的区别

## 2.1 GoT 已经做到的部分

GoT 已经能够：

- 将 Agent execution trace 转换为 graph；
- 使用 task / atomic subtask 表示执行过程；
- 使用 dependency edge 表示任务依赖；
- 实时展示 graph；
- 点击节点并查看已有节点信息；
- 展示 description、intermediate output、artifact、dependency 等内容。

因此以下内容不能作为我们的核心创新：

```text
× Real-time graph visualization
× Execution trace → graph
× Dependency DAG
× Click node
× Inspect node information
```

可以概括为：

> **GoT makes agent execution visible and inspectable.**

---

## 2.2 我们进一步解决的问题

我们关注的是：

```text
Visualization
      ↓
User asks a question
      ↓
Locate relevant node(s)
      ↓
Retrieve query-specific context
      ↓
Analyze
      ↓
Answer + Evidence
```

核心区别：

```text
GoT
Node → Inspect existing information

Our Work
Node / Graph → Ask questions
             → Locate relevant node(s)
             → Retrieve relevant context
             → Analyze
             → Answer
```

因此我们的系统不只是 interactive visualization，而是：

> **Node-grounded interactive trajectory analysis**

---

# 3. Motivation

现有 trajectory visualization 已经能够帮助用户理解：

```text
What happened?
```

例如：

- Agent 执行了哪些任务；
- workflow 如何展开；
- 节点之间有什么 dependency；
- 某个节点已有的执行信息是什么。

但在实际复杂场景中，用户往往会在观察 graph 的过程中产生新的问题：

```text
这个任务的 deadline 是什么时候？
这个结果为什么会这样？
这个节点为什么失败？
失败之前发生了什么？
Agent 是怎么恢复的？
这个结果来自哪些 earlier tasks？
这个节点和另一个节点有什么关系？
```

这些问题不是简单点击节点就能够回答的。

同时，如果系统只依赖用户当前选中的节点，也可能出现问题：

```text
用户选中了错误节点
        ↓
问题实际对应另一个节点
        ↓
错误地使用当前节点 context
        ↓
回答不准确
```

因此系统不仅需要支持对 trajectory 提问，还需要：

1. 根据 query 自动定位真正相关的节点；
2. 针对 query 构造精确、紧凑的回答上下文。

---

# 4. Contribution 1：Node-grounded Question-driven Analysis Framework

第一个 contribution 是：

> **提出一个支持围绕可视化节点进行提问和进一步分析的 Agent trajectory visualization framework。**

传统交互：

```text
View → Inspect
```

我们扩展为：

```text
View → Select / Ask → Analyze
```

Graph 不再只是一个展示结果的界面，而成为：

> **用户进行 trajectory question answering 和 further analysis 的交互入口。**

完整交互过程：

```text
Visualize
   ↓
Ask Question
   ↓
Locate Relevant Node(s)
   ↓
Retrieve Relevant Context
   ↓
Analyze
   ↓
Answer
   ↓
Highlight Supporting Nodes / Events
```

---

# 5. Contribution 2：Query-aware Context Management

为了支持 Contribution 1，仅仅加入一个聊天框是不够的。

真正的问题是：

> 当用户提出一个问题后，应该给 Agent 哪些 context？

一个长 Agent trajectory 中可能包含：

```text
大量 Task
大量 Subtask
大量 Tool Calls
大量 Observations
大量 Errors / Retries
大量 Intermediate Results
```

如果将完整 trajectory 直接提供给 Agent，可能产生：

- context 过长；
- irrelevant information 过多；
- 重要证据被淹没；
- token cost 增加；
- 回答准确性下降。

因此我们提出：

> **Query-aware Context Management**

它不是通用 memory management，而是：

> **专门服务于 trajectory question answering 的上下文管理机制。**

## 5.1 Context 的组织方式

Context 按照任务语义组织：

```text
Global Context
      ↓
Task Context
      ↓
Subtask Context
      ↓
Raw Execution Evidence
```

每个节点可以维护：

```text
Goal
Input Context
Execution Context
Important Observation
Failure Context
Recovery Context
Result Context
Artifacts
Raw Evidence
```

同时，每个节点还维护一份较短的：

```text
Summary Context
```

用于后续 node localization。

## 5.2 Query-aware Context Selection

真正回答问题时，并不是把节点全部 context 都交给 Agent，而是：

```text
Detailed Node Context
        ↓
Query-aware Selection
        ↓
Question-specific Context
```

例如：

```text
Query:
Why did Train Model fail?

Need:
Failure Context
Error Event
Relevant Observation
Retry Context
Dependency Context
```

Contribution 2 主要解决：

> **找到相关节点以后，应该给 Agent 什么 context？**

---

# 6. Contribution 3：Query-oriented Node Localization Mechanism

第三个 contribution 解决另一个关键问题：

> **用户的问题到底对应哪个节点？**

如果系统只依赖用户选中的节点：

```text
Selected Node
      ↓
Retrieve Context
      ↓
Answer
```

一旦用户选错节点，回答就可能不准确。

因此我们提出：

> **Query-oriented Node Localization Mechanism**

系统根据用户 query，搜索各节点维护的 Summary Context，自动定位最相关的 node / nodes。

## 6.1 Node Summary Index

每个节点维护：

```text
Node
├── Summary Context
└── Detailed Context
```

其中：

```text
Summary Context
→ 用于搜索和定位节点

Detailed Context
→ 用于真正回答问题
```

例如：

```text
Node A
Summary: Preprocess the dataset and handle missing values.

Node B
Summary: Train a prediction model on the cleaned dataset.

Node C
Summary: Generate the final report.
```

用户问题：

```text
为什么数据清洗第一次失败？
```

系统搜索：

```text
Query
  ↓
Search Node Summaries
  ↓
Node A: 0.94
Node B: 0.35
Node C: 0.12
  ↓
Anchor Node = Node A
```

随后再读取 Node A 的 detailed context。

## 6.2 User-selected Node 不是唯一约束

用户当前选中的节点可以作为：

> **Soft Hint / Prior**

但不能成为硬性的 retrieval scope。

例如：

```text
Selected Node:
Train Model

Query:
为什么数据清洗失败？
```

系统仍然应该根据 query 判断：

```text
Train Model           0.32
Preprocess Dataset    0.94
Load Dataset          0.51
```

最终定位：

```text
Preprocess Dataset
```

因此：

> **Query determines the relevant node; user selection only provides an optional interaction hint.**

## 6.3 Node Localization Pipeline

```text
User Query
     ↓
Search Node Summary Context
     ↓
Candidate Nodes
     ↓
Node Ranking
     ↓
Relevant Node(s)
     ↓
Retrieve Detailed Context
```

Contribution 3 主要解决：

> **应该从哪个节点取 context？**

---

# 7. 三个 Contribution 的关系

三个 contribution 分别解决三个不同问题：

```text
Contribution 1
Node-grounded Question-driven Analysis Framework
              │
              ↓
        用户可以提问
              │
              ↓
Contribution 3
Query-oriented Node Localization
              │
              ↓
       找到正确 Node
              │
              ↓
Contribution 2
Query-aware Context Management
              │
              ↓
构造回答所需的精确 Context
              │
              ↓
        Accurate Answer
```

可以简化成：

> **C1：怎么问？**

> **C3：这个问题对应哪个节点？**

> **C2：找到节点以后，用什么上下文回答？**

---

# 8. 整体 System Pipeline

```text
Agent Execution
      ↓
Trajectory Graph
      ↓
Node Summary Context
+ Detailed Node Context
      ↓
Visualization
      ↓
User Query
      ↓
Query-oriented Node Localization
      ↓
Relevant Node(s)
      ↓
Query-aware Context Management
      ↓
Relevant Context
      ↓
LLM / Agent
      ↓
Answer
      +
Graph Highlight
      +
Supporting Evidence
```

---

# 9. 整体 Research Story

```text
Existing Work
GoT-style trajectory visualization
        ↓
Users can inspect nodes
        ↓
But inspection alone cannot support
follow-up question-driven analysis
        ↓
Contribution 1
Node-grounded Question-driven
Analysis Framework
        ↓
New Challenge 1
The user's current node selection
may not be the correct context scope
        ↓
Contribution 3
Query-oriented Node Localization
        ↓
Locate relevant node(s)
from Node Summary Context
        ↓
New Challenge 2
How to provide the right context
for the current query?
        ↓
Contribution 2
Query-aware Context Management
        ↓
Retrieve + Expand + Compress
query-relevant trajectory context
        ↓
Accurate Question Answering
```

---

# 10. 与 GoT 的核心区别

## Graph of Trace

```text
Execution Trace
      ↓
Task DAG
      ↓
Real-time Visualization
      ↓
Node Inspection
```

重点：

> **Make execution visible and inspectable.**

## Our Work

```text
Execution Trace
      ↓
Trajectory Graph
      ↓
Question-driven Interaction
      ↓
Query-oriented Node Localization
      ↓
Query-aware Context Management
      ↓
Answer + Evidence + Graph Feedback
```

重点：

> **Make trajectory visualization queryable and analyzable.**

---

# 11. 最终一句话定位

> **We extend agent trajectory visualization from node inspection to question-driven analysis by introducing a query-oriented node localization mechanism and a query-aware context management mechanism for accurate trajectory question answering.**

更简洁地说：

> **GoT lets users inspect nodes; our system lets users ask questions about the trajectory, automatically locates the relevant nodes, and constructs the right context for answering them.**
