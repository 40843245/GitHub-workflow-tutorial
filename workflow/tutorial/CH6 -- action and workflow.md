CH6 -- action and workflow.md
## objectives
You will know 

  + what is an action
  + what is an workflow
  + type of action
  + the basic use of an action and workflow
  + how to define an action and workflow

## CH6-1 -- introduction to an action
### introduction
An action defines a series of step (run).

Analogous to solution in coding,

an action behaves like a utility function, 

you can simply invoke it when use it.

### features

One can extract one core step from full code into a utility function (this technique is called `SOC` (stands for Separation Of Concern)) and making it reusable.

An action has these feature.

+ reusable

## CH6-2 -- introduction of a workflow
A workflow behaves like SOP (標準流程) that once before updating (according to trigger event of the workflow, see CH15 for more details), 

it will be executed (usually to check something).

Image that

You have to brush your teeth, wash your body, and measure your weight before you sleep.

So when you want to sleep, the SOP is executed, you or some robot checks

+ have you brushed your teeth (or is your teeth clean)
+ have you washed your body (through accessing record of using bathtub)
+ have you measured your weight (through accessing record of using weighing scale)
    
Only if all of them are passed, you can go to sleep. 

Otherwise, you can't have a good sleep (for example, the robot hit your leg with a hammer if you lie on the bed) 

In software engineering, this SOP is called a workflow. 

## CH6-3 -- type of an action
An action in GitHub Action can be categorized into

  + composite run steps action (or called composite run action)
  + container action
  + javascript action

### composite run action
In a composite run action, it can execute one or more shell commands (here, you can also call a shell command as a run) and action,

making it more usable by binding one or more runs and actions.

You can think an action as a piece of building blocks.

You can bind them together, making it be a bigger building blocks.

### container action
An action which running environments is encapsulated in a **Docker container**.

This ensures that separation of environment, you can execute it on tools and programming language that is based on Linux.

### javascript action
A javascript action is written in javascript and is executed in Node.js.

Feature:

  + Faster
  + More comapatible: It is cross-platform, it can be executed at Linux, MacOS, Windows runner)

## CH6-4 -- importing an action
In a workflow and a composite run action, you can simply import it by `uses` field. 

### Usage and structure

In a composite run action

```
name: '<descriptive-action-name>'

runs:
  using: "composite" # <-- needed , it defines an action is a composite run action.
  steps:
    - name: '<descriptive-run-name>'
      uses: <action-name-for-importing> # <-- import the action you want here
```

In a workflow,

```
name: <descriptive-workflow-name>

on:
  # ... `on` block omitted

jobs:
  build:
    runs-on: <running-environment>
    steps:
      - name: '<descriptive-run-name>'
        uses: <action-name-for-importing> # <-- import the action you want here
```

### action name

Case 1: If you want to import an action from GitHub Marketplace. 

You need to use absolute path.

Format:

```
<owner>/<repo-name>@<ref>
```

or

```
<organization>/<repo-name>@<ref>
```

where 

`<owner>` is the owner who publishes the action

`<organization>` is the organization who publishes the action

`<repo-name>` is the repo name of the repo that the action places at.

`<ref>` is the reference of the action. It determines which version of release action will be used.

It can be a Git tag, branch branch name, or SHA of the repo.

Case 2:  If you want to import an action from an external repo

Apply the rule of `Case 1: If you want to import an action from GitHub Marketplace.` 

Case 3: If you want to import an action from internal repo.

You MUST use the relative path.

The starting point of the relative path is

| scenario | caller | starting point of the relative path (referenced from) | end point (referenced to) |
| :-- | :-- | :-- | :-- |
| a workflow imports a composite action | workflow | root directory of current repo | directory of the file that defines the composite action |
| an action imports a composite action | a composite action | current working directory (**directory of caller**) (**NOT the file path of callee**) | directory of the file that defines the composite action |

> [!NOTE]
> In this scenario -- a workflow imports an action,
>
> `.` in relative path means the root directory of current repo.
>
> It follows the rule about POSIX file system nagivation.

> [!NOTE]
> In this scenario -- a composite action imports an action,
>
> `..` in relative path means the parent of the directory.
>
> It follows the rule about POSIX file system nagivation.

> [!IMPORTANT]
> For the scenario -- a composite action imports an action,
>
> When Runner executes its steps in the composite action (caller),
>
> it will set current working directory as the directory of the caller.
>
> Therefore, any imports from internal repo, the starting point is the directory of the caller.
>
> Just can reference **from the directory of the caller**,
>
> **NOT** from **root directory of current repo**.

## CH6-5 -- valid path of an action or a workflow
### workflow
For a workflow,

+ the file MUST have extension `.yml`
+ the file MUST be placed under `./.github/workflows/` directory

  where

  again, `.` means the root directory of current repo.

### composite action
For a composite action,

The file name MUST be **`action.yml`** (commonly used) or **`action.json`**.

It is NOT required but it's a convention (that is highly recommend mentioned in offical docs) that

  it is placed under `./github/actions/` directory.

> [!IMPORTANT]
> The reason why a composite action MUST be **`action.yml`** (commonly used) or **`action.json`**.
>
> The `uses` field in `steps` field in `runs` field (`runs`->`steps`->`uses`) can be correctly parsed by the GitHub Action engine.
>
> Only in `action.yml` or `action.json`, the `uses` field can be recognized by GitHub Action engine. 

`action.yml`

```
# 只有在 action.yml 中，這個結構才有效
runs:
  using: "composite"  # 告訴 Runner 這是一個多步驟的 Action
  steps:
    - uses: actions/checkout@v4  # 步驟 1: 引用一個 Action
    - uses: ../another-action    # 步驟 2: 引用另一個 Action
    - run: echo "Done"           # 步驟 3: 執行 Shell 命令
```
 
`action.json`

```
"runs":{
  "using": "composite",
  "steps":[
      "uses": "actions/checkout@v4" 
      "uses": "../another-action"    
      "run": "echo \"Done\""
  ]
}
```

## Examples (A collection of big examples as summary)
### Example 1
Assume that structure of a repo is 

```
repoA/ (root directory of current repo)
├── .github/
│   ├── actions/
│   │   └── trash-tool/
│   │       ├── trash-cleaner/      <-- Caller Action (action.yml)
│   │       │   └── action.yml      
│   │       └── trash-finder/       <-- Called Action (action.yml)
│   │           └── action.yml      
│   └── workflows/
│       └── main.yml                <-- Workflow (Caller)
└── src/
```

Scenario 1: a workflow imports a composite action

In a workflow `main.yml` (caller), you want to call the composite action `trash-cleaner/action.yml` (callee).

You have to uses the relative path `./.github/actions/trash-tool/trash-cleaner`,

since the caller is a workflow, it references from **root directory of current repo**.

structure:

```
# repoA/.github/workflows/main.yml
name: Main CI

on: [push]

jobs:
  run_cleaner:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # 引用路徑：從 repoA 根目錄 (./) 開始計算
      - name: Run Trash Cleaner Tool
        uses: ./.github/actions/trash-tool/trash-cleaner
```

Scenario 2: an action imports a composite action

In an action `trash-cleaner/action.yml` (caller), you want to call the composite action `trash-finder/action.yml` (callee).

You have to uses the relative path `../trash-finder`,

since the caller is an action, it references from **directory of caller** (here is `trash-cleaner/`).

Calculate it from starting point `trash-cleaner/`

=> its parent directory is `trash-tool/`

=> it has a child directory name `trash-finder` which places `trash-finder/action.yml`,

So, the relative path is `../trash-finder`

structure:

```
# repoA/.github/actions/trash-tool/trash-cleaner/action.yml
name: Trash Cleaner

runs:
  using: "composite"
  steps:
    - name: Find Trash First
      # 引用路徑：從 trash-cleaner/ 目錄開始計算 (向上退一層到 trash-tool/，再進入 trash-finder/)
      uses: ../trash-finder
      
    - name: Run Actual Cleaning
      run: echo "Cleaning using the results from trash-finder"
      shell: bash
```

