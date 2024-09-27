# Code Analysis

## Sequence: Start Process from main() to executor(ctx)
```mermaid
sequenceDiagram
    autonumber

    main -> cmd.Execute: cmd.Execute(ctx, version)
    Note over cmd: 1. Pharse arguments and initilize "input := new(Input)" <br/> 2. Setup entry point "newRunCommand(ctx, input)"

    cmd.Execute -> newRunCommand: newRunCommand(ctx, input)
    Note over newRunCommand: 1. Return a command running fucntion <br/> "func(*cobra.Command, []string) error {}" <br/>which is the actual entry point.

    newRunCommand -> runner.New(config): r, err := runner.New(config)
    Note over runner.New(config): 1. Create a runner with config <br/> 2. Use the runner to create "excutor chain": r.NewPlanExecutor(plan)  <br/> 3. Run the excutor chain err = executor(ctx)
```

## Runner, runnerImpl, Excutor, RunContext
### Who created the runner and runnerIml?
The r, err := runner.New(config) will create an instance of runnerImpl then setup all the config like

* Workdir: where the github action workflow file is
* EventName: push
* Toke

search the `type Config struct {` for detail


```mermaid
---
title: Runner, runnerImpl, Excutor, RunContext
---
classDiagram
    note "r, err := runner.New(config), here the r initialize 'config' and 'eventJSON'"
    Runner <|-- runnerImpl

    class Runner{
        <<interface>>
        NewPlanExecutor(plan *model.Plan) common.Executor
    }

    class runnerImpl{
        config    *Config
        eventJSON string
        caller    *caller 

        NewPlanExecutor(plan *model.Plan) common.Executor
        configure() (Runner, error)
        newRunContext(ctx context.Context, run *model.Run, matrix map[string]interface) *RunContext
    }
```
### Excutor, RunContext
```golang
// Executor define contract for the steps of a workflow
type Executor func(ctx context.Context) error
```
NewPlanExecutor() will create a list of Executor which actually a list of function, search the code below for detail
```golang
// from runner.go => func (runner *runnerImpl) NewPlanExecutor(plan *model.Plan) common.Executor {
stagePipeline := make([]common.Executor, 0)

// code snippet
for i := range plan.Stages {
    stage := plan.Stages[i]
    stagePipeline = append(stagePipeline, func(ctx context.Context) error {}
```

## What is Plan, Stage, StagePipeline, job etc.
