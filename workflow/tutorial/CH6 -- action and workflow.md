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

| scenario | caller | starting point of the relative path |
| :-- | :-- | :-- |
| a workflow imports an action | workflow | root directory of current repo | 
| a composite action imports an action | a composite action | current working directory (directory of caller) | 

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


## CH6-5 

### Examples
#### Example 1
