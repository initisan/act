```mermaid
sequenceDiagram
    autonumber

    main ->> cmd.Execute: cmd.Execute(ctx, version)
    Note over cmd.Execute: 解析参数并初始化输入

    cmd.Execute ->> newRunCommand: newRunCommand(ctx, input)
    Note over newRunCommand: 返回实际运行的命令函数

    newRunCommand ->> runner.New: r, err := runner.New(config)
    Note over runner.New: 创建runner实例并配置环境

    runner.New ->> runner.NewPlanExecutor: 创建执行计划
    Note over runner.NewPlanExecutor: 创建执行器链

    runner.NewPlanExecutor ->> executor(ctx): 执行计划
    Note over executor(ctx): 运行工作流中的各个步骤

    executor(ctx) ->> common.NewParallelExecutor: 并行执行任务
    Note over common.NewParallelExecutor: 使用并行执行器来高效运行任务
```



好的，以下是两个图表来说明GitHub Actions YAML文件中的字段如何映射到执行器链中的字段，以及执行器链的结构。

### 图1：GitHub Actions YAML字段到执行器链字段的映射

```mermaid
graph TD
    A[GitHub Actions YAML] -->|解析| B[Workflow 结构体]
    B -->|转换| C[Stage 结构体]
    C -->|转换| D[Step 结构体]
    D -->|映射| E[Executor 结构体]

    subgraph YAML字段
        A1[jobs]
        A2[steps]
        A3[runs-on]
        A4[name]
        A5[uses]
        A6[run]
    end

    subgraph Workflow结构体
        B1[Jobs]
        B2[Steps]
        B3[RunsOn]
        B4[Name]
    end

    subgraph Stage结构体
        C1[Steps]
        C2[Name]
    end

    subgraph Step结构体
        D1[Name]
        D2[Uses]
        D3[Run]
    end

    subgraph Executor结构体
        E1[Name]
        E2[Action]
        E3[Script]
    end

    A1 --> B1
    A2 --> B2
    A3 --> B3
    A4 --> B4
    B2 --> C1
    B4 --> C2
    C1 --> D1
    C1 --> D2
    C1 --> D3
    D1 --> E1
    D2 --> E2
    D3 --> E3
```

### 图2：执行器链结构

```mermaid
graph TD
    subgraph 执行器链
        A[Head Executor]
        B[Executor Node 1]
        C[Executor Node 2]
        D[Executor Node 3]
        E[Tail Executor]
    end

    A --> B
    B --> C
    C --> D
    D --> E

    A -->|执行| B
    B -->|执行| C
    C -->|执行| D
    D -->|执行| E

    subgraph Executor Node
        F[Name]
        G[Action/Script]
        H[Next Executor]
    end

    B --> F
    B --> G
    B --> H
```

### 解释

1. **GitHub Actions YAML字段到执行器链字段的映射**：
   - YAML文件中的字段（如`jobs`、`steps`、`runs-on`等）首先被解析为`Workflow`结构体。
   - `Workflow`结构体中的字段（如`Jobs`、`Steps`等）被转换为`Stage`结构体。
   - `Stage`结构体中的字段（如`Steps`、`Name`等）被转换为`Step`结构体。
   - `Step`结构体中的字段（如`Name`、`Uses`、`Run`等）被映射到`Executor`结构体中的字段（如`Name`、`Action`、`Script`等）。

2. **执行器链结构**：
   - 执行器链是一个链表结构，每个节点都是一个`Executor`。
   - 链的头部是`Head Executor`，尾部是`Tail Executor`。
   - 每个`Executor`节点包含`Name`、`Action/Script`和指向下一个`Executor`的指针`Next Executor`。
   - 执行器链按顺序执行每个`Executor`节点，直到链的尾部。

通过这两个图表，可以更清晰地理解GitHub Actions YAML文件中的字段是如何映射到执行器链中的字段，以及执行器链的结构和执行流程。





Q: 这个项目中，特别是对于runner有下面的概念 stagePipeline pipeline stageExecutor

这三个概念在项目中的关系可以通过一个图表来解释。以下是它们的关系和执行流程：

1. **stagePipeline**：表示一个阶段的执行器链，每个阶段包含多个并行执行的任务。
2. **pipeline**：表示整个工作流的执行器链，包含多个阶段，每个阶段可以并行执行。
3. **stageExecutor**：表示一个阶段的具体执行器。

### 图表说明

```mermaid
graph TD
    A[Pipeline] -->|包含多个| B[StagePipeline]
    B -->|包含多个| C[StageExecutor]

    subgraph Pipeline
        B1[StagePipeline 1]
        B2[StagePipeline 2]
        B3[StagePipeline 3]
    end

    subgraph StagePipeline
        C1[StageExecutor 1]
        C2[StageExecutor 2]
        C3[StageExecutor 3]
    end

    A --> B1
    A --> B2
    A --> B3

    B1 --> C1
    B1 --> C2
    B1 --> C3

    B2 --> C1
    B2 --> C2
    B2 --> C3

    B3 --> C1
    B3 --> C2
    B3 --> C3
```

### 解释

1. **Pipeline**：整个工作流的执行器链，由多个`StagePipeline`组成。
2. **StagePipeline**：表示一个阶段的执行器链，每个阶段包含多个并行执行的任务。
3. **StageExecutor**：表示一个阶段的具体执行器，负责执行具体的任务。

在代码中，`stagePipeline`是一个包含多个

stageExecutor

的列表，而

pipeline

是一个包含多个`stagePipeline`的列表。每个

stageExecutor

负责执行具体的任务，并且可以并行执行。
