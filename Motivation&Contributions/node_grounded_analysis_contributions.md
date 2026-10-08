# 从 Node Inspection 到 Node-grounded Analysis

## 1. 研究定位

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

例如：

```text
Node:
完成计算机作业
```

用户点击之后，可以看到：

```text
description
parent / dependency
intermediate output
artifact
execution information
```

但这种交互本质上仍然主要是：

> **Node Inspection**

我们的目标是在此基础上进一步支持：

> **Node-grounded Question Answering / Analysis**

即用户可以直接围绕某个可视化节点提出新的问题。

例如：

```text
Node:
完成计算机作业

        ↓ click / select

Question:
这个作业的 ddl 是什么时候？

        ↓

系统基于该节点及相关 trajectory context
检索相关信息并回答问题
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


可以概括为：

> **GoT makes agent execution visible and inspectable.**

---

## 2.2 我们进一步解决的问题

我们关注的是：

```text
Selected Node
     ↓
User Question
     ↓
Context Retrieval
     ↓
Node-grounded Analysis
     ↓
Answer + Evidence
```

即：

> 用户不仅能够“查看节点”，还能够“针对节点继续提问”。

核心区别：

```text
GoT
Node → Inspect existing information

Our Work
Node → Ask new questions → Retrieve relevant context → Analyze → Answer
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

因此核心问题是：

> **现有 visualization 支持 node inspection，但缺少围绕 selected node 的 question-driven analysis。**

我们的目标是：

> **让 visualization 本身成为用户进行 trajectory question answering 和进一步分析的交互入口。**

---

# 4. Contribution 1：Node-grounded Question-driven Analysis Framework

第一个 contribution 是：

> **提出一个支持围绕可视化节点直接提问和分析的 Agent trajectory visualization framework。**

用户首先在 graph 中选择一个节点：

```text
Task A
  ↓
Task B   ← selected
  ↓
Task C
```

然后直接提出问题：

```text
Why did Task B fail?
```

此时：

```text
Selected Node
      +
User Query
      ↓
Analysis Request
```

系统不再只是打开一个静态 detail panel，而是以该节点作为：

> **Query Anchor / Context Anchor**

进行后续分析。

完整交互：

```text
Visualize
   ↓
Select Node
   ↓
Ask Question
   ↓
Retrieve Relevant Context
   ↓
Analyze
   ↓
Answer
   ↓
Highlight Supporting Nodes / Events
```

因此 visualization 从：

```text
View → Inspect
```

扩展为：

```text
View → Select → Ask → Analyze
```

---

# 5. Contribution 2：Query-aware Context Management

为了支持 Contribution 1，仅仅提供一个聊天框是不够的。

真正的问题是：

> 当用户针对一个节点提出问题时，应该给 Agent 哪些 context？

一个长 Agent trajectory 中可能包含：

```text
大量 Task
大量 Subtask
大量 Tool Calls
大量 Observations
大量 Errors / Retries
大量 Intermediate Results
```

如果将所有 trajectory 直接提供给 Agent：

```text
Full Trajectory
      ↓
LLM
```

会产生：

- context 过长；
- irrelevant information 过多；
- 重要证据容易被淹没；
- 回答不够准确；
- token cost 增加。

因此我们提出：

> **Query-aware Context Management**

它不是通用 memory management，而是：

> **专门服务于 node-grounded question answering 的上下文管理机制。**

---

## 5.1 Query-aware Context Retrieval

输入：

```text
Selected Node
      +
User Query
```

系统动态确定：

```text
Which context is actually needed?
```

例如：

```text
Node:
Train Model

Query:
Why did this task fail?
```

需要的 context 可能是：

```text
Failure Context
Retry Context
Relevant Observation
Predecessor Task
Raw Error Evidence
```

而不是：

```text
所有历史 trajectory
```

---

## 5.2 Context Management Pipeline

整体过程：

```text
Selected Node
      +
User Query
      ↓
Node-grounded Retrieval
      ↓
Query-relevant Context Selection
      ↓
Graph-aware Context Expansion
      ↓
Context Compression / Organization
      ↓
Agent
      ↓
Answer
```

可以概括为：

> **Index / Node 用于定位，Graph 用于扩展，Context Manager 用于构造最终回答所需上下文。**

---

## 5.3 Context 的语义层级

Context 仍按照任务语义组织：

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

但是实际回答时并不是全部使用，而是：

```text
Full Node Context
       ↓
Query-aware Selection
       ↓
Question-specific Context
```

因此同一个 Node 面对不同 Query，会产生不同 Context。

---

## 5.4 示例

### Query 1

```text
Why did Train Model fail?
```

Context：

```text
Train Model
├── Failure Context
├── Error Event
├── Previous Observation
├── Retry Context
└── Relevant Dependency
```

### Query 2

```text
What result did Train Model produce?
```

Context：

```text
Train Model
├── Result Context
├── Artifact
├── Final Observation
└── Relevant Raw Evidence
```

### Query 3

```text
Where did the input data for Train Model come from?
```

Context：

```text
Preprocess Data
       ↓
Dependency Edge
       ↓
Train Model
```

因此：

> **Context is constructed specifically for the current question.**

---

# 7. Contribution 3：Future Contribution


目前有两个可能方向。

## Option A：Benchmark / Evaluation Protocol

构建一个面向 Agent trajectory question answering 的 benchmark，用于评估：

```text
Node Localization
Context Retrieval
Question Answering
Failure Diagnosis
Provenance Tracing
Cross-node Reasoning
```

例如 benchmark question：

```text
Why did this node fail?

Which previous task caused this result?

How did the agent recover from the error?

Which artifact was used by this node?

How did the execution plan change?
```

评估：

```text
Retrieval Accuracy
Answer Accuracy
Evidence Recall
Context Length
Token Cost
Latency
```

---

## Option B：Technical Contribution

根据实际实现中发现的核心技术问题，进一步提出一个 technical method，例如：

```text
Query-aware Graph Traversal

Node-grounded Context Retrieval

Adaptive Context Compression

Automatic Task Boundary Detection

Dynamic Graph Restructuring
```

第三个 contribution 目前不必强行确定。

---

# 8. 三个 Contribution 的当前结构

```text
Contribution 1
Node-grounded Question-driven
Trajectory Analysis Framework
              ↓
解决：
Visualization 只能 inspect，
不能围绕 node 继续分析


Contribution 2
Query-aware Context Management
              ↓
解决：
如何针对 selected node + query
构造精确回答所需的 context


Contribution 3
Benchmark or Technical Method
              ↓
后期根据实验和系统瓶颈确定
```

---

# 9. 整体 Research Story

整个研究故事可以写成：

```text
Existing Work
GoT-style real-time trajectory visualization
        ↓
Users can inspect nodes
        ↓
But cannot naturally ask questions
grounded in a selected node
        ↓
Production analysis often requires
follow-up questions and deeper reasoning
        ↓
Our Goal
Turn node inspection into
node-grounded interactive analysis
        ↓
Contribution 1
Node-grounded Question-driven
Analysis Framework
        ↓
New Challenge
How to provide the agent with the right
context for each node-specific query?
        ↓
Contribution 2
Query-aware Context Management
        ↓
Retrieve + Expand + Compress
query-relevant trajectory context
        ↓
Accurate Question Answering
        ↓
Contribution 3
Benchmark / Technical Method
```

---

# 10. 最核心的一句话

GoT：

> **GoT makes agent execution visible and inspectable.**

我们的工作：

> **We make agent trajectory nodes directly queryable and analyzable.**

更完整的研究定位：

> **We extend agent trajectory visualization from node inspection to node-grounded question-driven analysis, supported by a query-aware context management mechanism that dynamically constructs relevant trajectory context for accurate answering.**
