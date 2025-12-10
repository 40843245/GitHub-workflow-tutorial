# CH6 -- output parameter of action or job
## objectives
You will know how to 
  
  + return a value in an action or a job
  + receive or access the returned value returned by an action or a job

## CH6-1 -- return a value
### CH6-1-1 return a value in an action
#### In Composite run action
To return a value.

it is need to echo the final result (represented as a key-value pair `<output-parameter-name>=<returned-value>`) to a `GitHub-Provided Special File Path Variable` -- `$GITHUB_OUTPUT`

then fetches the internal final result as returned value in `value` field.

Format:

```
echo "<output-parameter-name>=<returned-value>" >> $GITHUB_OUTPUT
```

where 

`<output-parameter-name>` is the output parameter name. It will be used when accessing the returned value.

and

`<returned-value>` is the returned value of the action.

> [!IMPORTANT]
> Since we have to write the key-value pair to a variable,
>
> we need to quote the key-value piar with double quotations
>
> (even though no need to quote the value of the field).
>
> However, the key-value pair is one part of value of the field.
>
> NOT related to this.  

> [!IMPORTANT]
> Always echo key-value pair to `$GITHUB_OUTPUT`.
>
> Otherwise, when accessing it, you will get `NULL` or empty string, leading unexpected behaviour.

> [!Important]
> The output parameter ONLY can map or forward internal final result (making it external and thus can be used in others)
>
> The output parameter can't simply return `$GITHUB_OUTPUT`.
>
> So, in a composite run action, it is needed to echo the final result (represented as a key-value pair) to `$GITHUB_OUTPUT`,
>
> then fetch the internal final result and return it in `value` field.

##### Examples
###### Example 1

`repoA/.github/actions/code-style-formatter.yml`

```
name: 'Dotnet Format Checker'
description: 'Runs dotnet format and checks for code style issues.'

inputs:
  check-only:
    description: 'If true, only check for formatting issues without applying changes.'
    required: false
    default: 'false'

outputs:
  format-needed: # 輸出參數 1
    description: 'Indicates if any files needed formatting.'
    value: ${{ steps.check_diff.outputs.format_needed }} # 從內部步驟獲取值
  diff-output: # 輸出參數 2
    description: 'The full diff of formatting changes.'
    value: ${{ steps.check_diff.outputs.diff_output }} # 從內部步驟獲取值

runs:
  using: 'composite'
  steps:
    - name: Run dotnet format (Apply or Check)
      continue-on-error: true
      run: |
        if [ "${{ inputs.check-only }}" == "true" ]; then
          dotnet format --verify-no-changes --verbosity normal
        else
          dotnet format
        fi
      shell: bash

    - name: Generate Diff Output
      id: check_diff # 設定步驟 ID 以便在 'outputs' 中引用
      shell: bash
      run: |
        GIT_DIFF=$(git diff --exit-code --name-only || true)
        
        if [ -n "$GIT_DIFF" ]; then
          echo "format_needed=true" >> $GITHUB_OUTPUT # 設定輸出參數的值
          echo "## Formatting Changes Detected" >> $GITHUB_STEP_SUMMARY
          git diff >> $GITHUB_STEP_SUMMARY
          
          # 獲取詳細的 diff 輸出作為另一個輸出參數
          FULL_DIFF=$(git diff)
          echo "diff_output<<EOF" >> $GITHUB_OUTPUT
          echo "$FULL_DIFF" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
          
        else
          echo "format_needed=false" >> $GITHUB_OUTPUT
          echo "Code formatting is clean." >> $GITHUB_STEP_SUMMARY
        fi
```

In above example, you can know that

+ According to `name: 'Dotnet Format Checker'` at top level, the action name is `Dotnet Format Checker`
+ In 

```
# ...
inputs:
  check-only:
    description: 'If true, only check for formatting issues without applying changes.'
    required: false
    default: 'false'
# ...
```

  - there is one input parameter named `check-only`.

  - The parameter with parameter name `check-only` is NOT required, 

    indicating that it is NOT needed to pass the argument named `check-only` when invoking this action.

  - If no argument named `check-only` is passed, it will use the default value `'false'`.

+ In

```
# ...
runs:
  using: 'composite'
  # ...
```

it defines the action is a composite run action.

+ In

```
# ...
runs:
  using: 'composite'
  steps:
    - name: Run dotnet format (Apply or Check)
      continue-on-error: true
      run: |
        if [ "${{ inputs.check-only }}" == "true" ]; then
          dotnet format --verify-no-changes --verbosity normal
        else
          dotnet format
        fi
      shell: bash

    - name: Generate Diff Output
      id: check_diff # 設定步驟 ID 以便在 'outputs' 中引用
      shell: bash
      run: |
        GIT_DIFF=$(git diff --exit-code --name-only || true)
        
        if [ -n "$GIT_DIFF" ]; then
          echo "format_needed=true" >> $GITHUB_OUTPUT # 設定輸出參數的值
          echo "## Formatting Changes Detected" >> $GITHUB_STEP_SUMMARY
          git diff >> $GITHUB_STEP_SUMMARY
          
          # 獲取詳細的 diff 輸出作為另一個輸出參數
          FULL_DIFF=$(git diff)
          echo "diff_output<<EOF" >> $GITHUB_OUTPUT
          echo "$FULL_DIFF" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
          
        else
          echo "format_needed=false" >> $GITHUB_OUTPUT
          echo "Code formatting is clean." >> $GITHUB_STEP_SUMMARY
        fi
```

It defines there are two following tasks needs to run in order.

 - 1th task: With description `Run dotnet format (Apply or Check)`, we can imply that this task is to check the format using dotnet.
 - 2th task: With description `Generate Diff Output`, we can imply that this task is responding to compare the differences and generating them as output.

Let's dive into these tasks one-by-one

+ In 1th task

```
# ...
runs:
  # ...
  steps:
    - name: Run dotnet format (Apply or Check)
      continue-on-error: true
      run: |
        if [ "${{ inputs.check-only }}" == "true" ]; then
          dotnet format --verify-no-changes --verbosity normal
        else
          dotnet format
        fi
      shell: bash
    # ...
```

  - 1th task has a brief and clear description `Run dotnet format (Apply or Check)`
  - it will use `bash` when executing script.
  - it will run these commands at `run` block.
  - these commands means if the input parameter named `check-only` is evaluated to `"true"`,

    then it will only check there are no changes needed using dotnet (`dotnet format --verify-no-changes --verbosity normal`)

    otherwise, it will try to fix (apply changes using dotnet (`dotnet format`)

  - `continue-on-error: true` indicates that when the execution failed (and thus returns exit code 0) during executing 1th task,
  
    the job will NOT be terminated.

> [!NOTE]
> Executing `dotnet format --verify-no-changes --verbosity normal` will
>
> return success (exit code `0`) iff formatting changes or style changes are NOT needed.
>
> Otherwise, return failure (exit code non-zero number).

> [!IMPORTANT]
> Executing `dotnet format` will try to fix changes using dotnet.
>
> When it successfully executed, it returns success (exit code), regradless whether changes are made or not.
     
+ In 2th task 

```
# ...
runs:
  using: 'composite'
  steps:
    # ...
    - name: Generate Diff Output
      id: check_diff # 設定步驟 ID 以便在 'outputs' 中引用
      shell: bash
      run: |
        GIT_DIFF=$(git diff --exit-code --name-only || true)
        
        if [ -n "$GIT_DIFF" ]; then
          echo "format_needed=true" >> $GITHUB_OUTPUT # 設定輸出參數的值
          echo "## Formatting Changes Detected" >> $GITHUB_STEP_SUMMARY
          git diff >> $GITHUB_STEP_SUMMARY
          
          # 獲取詳細的 diff 輸出作為另一個輸出參數
          FULL_DIFF=$(git diff)
          echo "diff_output<<EOF" >> $GITHUB_OUTPUT
          echo "$FULL_DIFF" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
          
        else
          echo "format_needed=false" >> $GITHUB_OUTPUT
          echo "Code formatting is clean." >> $GITHUB_STEP_SUMMARY
        fi
```

  - 2th task has a brief and clear description `Generate Diff Output`
  - and its unique step ID is `check_diff`, you can use it in `outputs` block
  - it will use `bash` when executing script.
  - it will run these commands at `run` block.
  - In 

  ```
  git diff --exit-code --name-only
  ```

  Description:
  
  Git will check there is any uncommitted files changes (through `git diff` command)

  The `--name-only` will ONLY list of name of file changes rather than list of file changes.
  
  The `--exit-code` option handles STO (Standard Output) returned by `git diff --name-only`, returning EC (Exit Code)

  > [!NOTE]
  > relationship of STO and EC
  >    
  > | STO | EC |
  > | :-- | :-- |
  > | empty string | 0 |
  > | non-empty string | 1 |
  
  Let's analyze some situation that happens in real world.
  
  Case 1: If there are no uncommitted file changes
  
  It happens when

    * Case A.1: `check-only` is evaluated to false and thus no execution of `dotnet format`.

    * Case A.2: `check-only` is evaluated to true, but during execution of `dotnet format` and before changes file, an exception is thrown, so not to continue execute this command.

  Then Git will detect no uncommitted file changes and 
  
  `git diff --exit-code --name-only` command will return an empty string (when this command successfully executed)

  Case 2: If there are uncommitted file changes

    It happens when

    * Case B.1: `check-only` is evaluated to true and executes `dotnet format` successfully. Additionally, this commands changes code and successfully save changes. 
    
    In this case, it will return exit code 0.

    * Case B.2: `check-only` is evaluated to true. The execution of `dotnet format` failed, but this commands changes code and successfully save changes.

    In this case, it will return exit code 1.

  Then Git will detect some uncommitted file changes and 
  
  `git diff --exit-code --name-only` command will return a list of name of uncommitted files changes (when this command successfully executed)

  Case 3:
  
  
    
  
