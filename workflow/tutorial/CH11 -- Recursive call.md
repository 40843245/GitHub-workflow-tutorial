# CH11 -- Recursive call
## objectives
You will know what happen if recursive call happens using different methods for

    + workflow -> workflow
    + action -> action
    + etc

## CH11-1 -- Recursively call about actions
Case 1: Recursively call about actions

Case 1.1: Directly call on same action 

Case 1.1.1: Directly call on same action using action call

Scenario example 1:

Assume Action A and Action C are BOTH composite actions.

```
In Action C, directly calls Action A.
In Action A, directly calls Action A.
```

Result:

**Prohibited**, **crashed** at `Action A -> Action A`.

Case 1.1.2: Directly call on same action using GitHub Restful API

(NEVER HAPPEN) Scenario example 1:

```
In Action A, a request about GitHub Restful API is sent, and then executing Action A
```

Result:

**NEVER happen**

since it is impossible to execute ONLY one Action without executing workflow.

Case 1.2: Indirectly call on different actions

Case 1.2.1: Indirectly call on different actions by directly call other actions

Scenario example 1:

Assume Action A and Action B are BOTH composite actions.

```
In Action A, directly calls Action B.
In Action B, directly call Action A.
```

Result:

NO error, can run, BUT it will be forced terminated by GitHub Actions, if there is NO termination condition to exit the recursively call chain.

The reason why it is one of following

    + after a long while, the resource is exhausted since executing one action will consume some resources.
    + it will take more than maximum executing time limit of the job (that executes Action A) since, 
    
    it will take the execution time of job into account, even if Action B is executing (as at that moment, Action A is still executing (executing Action B and wait for it until Action B is completed) 

Scenario example 2:

Assume Action A, Action B, and Action C are BOTH composite actions.

```
In Action A, directly calls Action B.
In Action B, directly call Action C.
In Action C, directly call Action A.
```

Result:

Same as Scenario example 1.

Case 1.3: Indirectly call on same action through calling workflow

Scenario example 1:

Assume Action A is a composite action and Workflow A is callable.

```
In Action A, send a request to GitHub Restful API to execute Workflow A.
In Workflow A, directly executes Action A.
```

Result:

NO error, can run, BUT it will be forced terminated by GitHub Actions, if there is NO termination condition to exit the recursively call chain.

Due to one of following reasons:

    + resource is exhausted since executing one action will consume some resources.
    + the job execution reaches the maximum execution time limit of the job (that executes Action A) since, 
    
    it will take the execution time of job into account, even if Action B is executing (as at that moment, Action A is still executing (executing Action B and wait for it until Action B is completed) 

    + quotas of total execution time of actions is exhausted (if you use free plan) 
    + etc.

## CH11-2 -- Recursively call about workflows
Case 2: Recursively call about workflows

Case 2.1: Directly call on same workflows

Scenario example 1:

Assume Workflow A and Workflow C are BOTH callable.

```
In Workflow C, directly calls Workflow A in a job that is executed by Workflow C.
In Workflow A, directly calls Workflow A in a job that is executed by Workflow A.
```

Result:

**throw error** (error message: **Circular dependency detected**) and **terminated** when executing job in Workflow A.

The reason why it is that when executing a job in a workflow, Github Actions will detect the executed workflows in the job will cause circular dependencies. 

Case 2.2: call on different workflows but there are circular dependencies

Case 2.2.1: Directly call on different workflows but there are circular dependencies

Scenario example 1:

Assume Workflow A, Workflow B, and Workflow C are BOTH callable.

```
In Workflow C, directly calls Workflow A in a job that is executed by Workflow C.
In Workflow A, directly calls Workflow B in a job that is executed by Workflow A.
In Workflow B, directly calls Workflow A in a job that is executed by Workflow B.
```

Result:

**throw error** (error message: **Circular dependency detected**) and **terminated** when executing job in Workflow B.

The reason why it is that when executing a job in a workflow, Github Actions will detect the executed workflows in the job will cause circular dependencies. 

Case 2.2.2: call on different workflows through Github Restful API but there are recursive calls.

Scenario example 1:

Assume Workflow A, Workflow B, Workflow C, and Workflow D are BOTH callable.

```
In Workflow C, directly calls Workflow A in a job that is executed by Workflow C.
In Workflow A, directly calls Workflow B in a job that is executed by Workflow A.
In Workflow B, directly calls Workflow D in a job that is executed by Workflow B.
In Workflow D, sending a request to Github Restful API to call Workflow A in a job that is executed by Workflow D.
```

Result:

No error, BUT it will be **forced terminated**, if there is NO termination condition to exit the recursively call chain.

Due to one of following reason:

    + resource insufficient
    + a execution of job reaches maximum execution time limit of the job
    + quotas of total execution time of actions is exhausted (if you use free plan) 
    + etc.

The reason is same as `Scenario example 1` in `Case 1.3` section.

For more details, see `Scenario example 1` in `Case 1.3` section.

Q&A:

Someone may get confused. 

Why does it NOT force terminate when executing the job that is executed in Workflow D?

Since the job that is executed in Workflow D sends a request to call Github Restful API, it will trigger an event (treated as an external event) that establish a new instance Workflow A, 

GitHub Actions System will treat it as an external event 

and thus 

  + stacking call of Workflow D will be `[C,A,B,D]`.
    
    GitHub Actions System can't detect that Workflow A will be executed

    when executing the job that is executed in Workflow D (as the consequent Workflow A is a newly created instance)

    => treat NO the circular dependencies
  
  + stacking call of the newly created instance Workflow A will be empty
  
    => treat NO the circular dependencies

bypassing the circular dependencies check.


