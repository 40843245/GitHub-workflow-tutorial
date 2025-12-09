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

  + GitHub Actions System can't detect that Workflow A will be executed

    when executing the job that is executed in Workflow D (as the consequent Workflow A is a newly created instance and here stacking call of Workflow D is `[C,A,B,D]`)

    => treat NO the circular dependencies
  
  + stacking call of the newly created instance Workflow A will be empty
  
    => treat NO the circular dependencies

bypassing the circular dependencies check.

<img width="258" height="67" alt="image" src="https://github.com/user-attachments/assets/2eaffec8-3335-4aa6-ac20-51cd6eefc72d" />

<img width="308" height="130" alt="image" src="https://github.com/user-attachments/assets/6317a380-60a9-4077-be9a-29f12f72979c" />

See how Google Gemini answers this use case.

I ask:

```
那

Workflow C直接呼叫Workflow A(在不同一個repo)
Workflow A直接呼叫Workflow B(在不同一個repo)
Workflow B直接呼叫Workflow D(在不同一個repo)
Workflow D透過GitHub Restful API呼叫Workflow A(在不同一個repo)
```

Gemini answers:

```
這個呼叫鏈結合了 **原生調用 (`workflow_call`)** 和 **API 觸發 (`workflow_dispatch`)**，會導致一個 **無限循環 (Infinite Loop)**，並最終因為資源耗盡而被 GitHub 系統終止。🚨

這是因為呼叫鏈中的最後一環 **成功地繞過了** GitHub 的原生遞迴檢查。

---

## 🔎 循環機制的分析

### 1. 執行流的啟動與推進

Workflow C 啟動後，執行流會依照原生 (`workflow_call`) 方式推進：

$$C \xrightarrow{\text{Direct}} A \xrightarrow{\text{Direct}} B \xrightarrow{\text{Direct}} D$$

在 D 執行時，原生遞迴檢查是正常的，因為當前堆疊是 $[C, A, B, D]$，沒有重複。

---

### 2. 循環的閉合與失控 (API 繞過)

當 Workflow D 執行到呼叫 API 的步驟時，情況發生變化：

$$\text{Workflow D} \xrightarrow{\text{API Call}} \text{觸發一個全新的 Workflow A 運行實例}$$

* **繞過檢查：** 由於 D 是透過 API 觸發 A，系統將其視為一個獨立的外部事件（例如使用者點擊按鈕）。這完全繞過了 GitHub 的 **`workflow_call` 原生遞迴檢查**。
* **循環閉合：** 這個由 D 觸發的新 Workflow A 實例會再次呼叫 B，B 再呼叫 D，D 再透過 API 觸發**第三個 A 實例**，依此類推。

這個結構將導致 Workflow 運行實例的數量迅速倍增，直到系統介入。

## 🛑 最終結果：系統終止

由於這個循環是透過 API 觸發的，它無法被原生檢查阻止，因此會導致失控的資源消耗。

* **結果：** Workflow 將因為達到 **Actions 分鐘數限制**、**單一 Job 最大執行時間限制**，或被系統偵測到異常資源使用模式而**強制終止 (Terminate)**。

**結論：** 這種混合模式是規避原生遞迴檢查的有效方式，但其最終結果是 Workflow 服務會終止執行以防止資源濫用。
```

