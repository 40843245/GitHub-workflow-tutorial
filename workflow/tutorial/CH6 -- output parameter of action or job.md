# CH6 -- output parameter of action or job
## objectives
You will know how to 
  
  + return a value in an action or a job
  + receive or access the returned value returned by an action or a job

## CH6-1 -- return a value
### CH6-1-1 return a value in an action
#### In Composite run action
To return a value.

it is need to echo the final result (represented as a key-value pair `<output-parameter-name>=<returned-value>`) to a GitHub-Provided Special File Path Variable -- `$GITHUB_OUTPUT`

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
> The output parameter ONLY can map or forward internal final result (making it external and thus can be used in others)
>
> The output parameter can't simply return `$GITHUB_OUTPUT`.
>
> So, in a composite run action, it is needed to echo the final result (represented as a key-value pair) to `$GITHUB_OUTPUT`,
>
> then fetch the internal final result and return it in `value` field.

> [!IMPORTANT]
> Always redirect key-value pair out into `$GITHUB_OUTPUT`.
>
> Otherwise, when accessing it, you will get `NULL` or empty string, leading unexpected behaviour.

> [!IMPORTANT]
> To redirect key-value pair out into `$GITHUB_OUTPUT`,
>
> The seperator of key-value pair MUST be one of `=` and `<<`
>
> `=`: just redirects ones (in one command) to the key (output parameter name) 
>
> `<<`: tells that redirects multiple line (in many commands) to the key (output parameter name)
>
> After executing one command about redirecting, it will automatically add a new break line after this line, before executing the next command.
>
> In this case, one MUST append a name as seperator after `<<`, marking the starting point of the redirect
>
> and MUST just simply redirect the name at the end, marking the end point of the redirect

> [!TIP]
> To understand why we need to do such thing when redirecting using `<<` as separator.
>
> You can image a situation.
>
> After open an uncorrupted PDF with NotePad++.
>
> You will see `%PDF-<version>` at the beginning of PDF content and `%%EOF` at the end of PDF content.
>
> The main purpose is that PDF reader recognizes it is an uncorrepted PDF.
>
> See following example for more details.  

> [!IMPORTANT]
> The start delimiter MUST be same as end delimiter for redirecting message out into buffer of a GitHub-Provided Special File Path Variable -- `$GITHUB_OUTPUT`.

> [!NOTE]
> relationship of STO and EC
>
> | STO | EC |
> | :-- | :-- |
> | empty string | 0 |
> | non-empty string | 1 |

> [!TIP]
> In `Bash`, `[ xxx ]` checks whether xxx is an empty string, or not.
>
> If it is, then returns boolean true.
>
> Otherwise, returns boolean false.   

> [!TIP]
> In `Bash`, `-n` in `[]` means negate.
>
> So, `[ -n xxx ]` checks whether xxx is NOT an empty string, or not.
>
> If it is NOT an empty string, returns boolean true.
>
> Otherwise, returns boolean false.

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

<details>
<summary>Analysis of `run` block of 2th task</summary>
  
<details>
<summary>Analysis of `git diff --exit-code --name-only`</summary>
Description:
  
Git will check there is any uncommitted files changes (through `git diff` command)
  
The `--name-only` will ONLY list of name of file changes rather than list of file changes.

The `--exit-code` option handles STO (Standard Output) returned by `git diff --name-only`, returning EC (Exit Code)
  
Let's analyze some situation that happens in real world.

Case 1: If there are no uncommitted file changes
  
=> Git will detect no uncommitted file changes 

=> `git diff --exit-code --name-only` command will return an empty string (when this command successfully executed)

  It happens when
  
  * Case A.1: `check-only` is evaluated to false and thus no execution of `dotnet format`.
  
  * Case A.2: `check-only` is evaluated to true, but during execution of `dotnet format` and before changes file, an exception is thrown, so not to continue execute this command.
    
Case 2: If there are uncommitted file changes
  
=> Git will detect some uncommitted file changes 

=> `git diff --exit-code --name-only` command will return a list of name of uncommitted files changes when this command successfully executed

  It happens when
  
  * Case B.1: `check-only` is evaluated to true and executes `dotnet format` successfully. Additionally, this commands changes code and successfully save changes. 
  
  In this case, it will return exit code 0.
  
  * Case B.2: `check-only` is evaluated to true. The execution of `dotnet format` failed, but this commands changes code and successfully save changes.
  
  In this case, it will return exit code 1.
  
Case 3: This command executes failed.
  
=> `git diff --exit-code --name-only` will return exit code 1 (due to exception during execution), and terminate the task.

</details>

<details>
<summary>Analysis of `GIT_DIFF=$(git diff --exit-code --name-only || true)`</summary>

Case 1: If there are no uncommitted file changes
  
=> Git will detect no uncommitted file changes 

=> `git diff --exit-code --name-only` command will return an empty string (when this command successfully executed)

=> So the right-hand side of expression `git diff --exit-code --name-only || true` will NOT be executed, returning the left-hand of the expression

=> `GIT_DIFF` variable stores an empty string.

Case 2: If there are uncommitted file changes
  
=> Git will detect some uncommitted file changes 

=> `git diff --exit-code --name-only` command will return a list of name of uncommitted files changes when this command successfully executed

=> So the right-hand side of expression `git diff --exit-code --name-only || true` will NOT be executed, returning the left-hand of the expression

=> `GIT_DIFF` variable stores a list of name of uncommitted files changes.
  
Case 3: This command executes failed.
  
=> `git diff --exit-code --name-only` will return exit code 1 (due to exception during execution)

=> However, due to the short-circuit feature, the right-hand sid of expression `git diff --exit-code --name-only || true` will be executed, returning the right-hand of the expression, which is 0 (meaning the boolean true).

=> `GIT_DIFF` variable stores the number 0.

</details>

<details>
<summary>Analysis of `if` condition</summary>  

```
if [ -n "$GIT_DIFF" ];
```

It will check there is the variable `$GIT_DIFF` is NOT an empty string,

If it is, it will execute `then` block.

Otherwise, it will execute the `else` block.

</details>

<details>
<summary>Analysis of `then` block</summary>

```
     # ...
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
      # ...
    fi
```

Let's break them into two parts.

<details>
<summary>Part 1-- Analysis of `formatted_need`</summary>

```
# ...
     if [ -n "$GIT_DIFF" ]; then
          echo "format_needed=true" >> $GITHUB_OUTPUT # 設定輸出參數的值
          echo "## Formatting Changes Detected" >> $GITHUB_STEP_SUMMARY
          git diff >> $GITHUB_STEP_SUMMARY
          # ...
    else
      # ...
    fi
```

+ It redirects the key-value pair `format_needed=true` out into the buffer of Github-provided special path variable `$GITHUB_OUTPUT`, 

(which used in output parameter).

Here, `format_need` is the one output parameter name of output parameters and it sets to string `true`. 

+ Then it redirects `## Formatting Changes Detected` out into the buffer of Github-provided step summary file.

+ Then it redirects uncommitted file changes (`git diff`) out into the buffer of Github-provided step summary file.

Overall, the output parameter `formatted_need` is set to true (stored in buffer of Github-provided special path variable `$GITHUB_OUTPUT`), for returning.

Additionally, it writes the uncommitted file changes into the buffer of Github-provided step summary file, which displayed on top of Action Log, for logging.

</details>

<details>
<summary>Part 2-- Analysis of `diff_output`</summary>

```
     # ...
     if [ -n "$GIT_DIFF" ]; then
          # ...
          
          # 獲取詳細的 diff 輸出作為另一個輸出參數
          FULL_DIFF=$(git diff)
          echo "diff_output<<EOF" >> $GITHUB_OUTPUT
          echo "$FULL_DIFF" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
    else
      # ...
    fi
```

+ it redirects `diff_output<<EOF` out into the buffer of Github-provided special path variable `$GITHUB_OUTPUT`.

It tells multiple line will be redirected.

Here, `diff_output` is the one output parameter name of output parameters and its starting delimiter is `EOF`.

+ Then it redirects the uncommitted file changes (stored in variable `$FULL_DIFF`) out into the buffer of Github-provided special path variable `$GITHUB_OUTPUT`.

+ After that, it redirects the end delimiter `EOF` out into the buffer of Github-provided special path variable `$GITHUB_OUTPUT`, finishing writing data to the diff_output parameter.

Overall, it writes the uncommitted file changes as log message into `diff_output` output parameter.
</details>

</details>

<details>
<summary>Analysis of `else` block</summary>

```
        # ...
        if [ -n "$GIT_DIFF" ]; then
          # ...          
        else
          echo "format_needed=false" >> $GITHUB_OUTPUT
          echo "Code formatting is clean." >> $GITHUB_STEP_SUMMARY
        fi  
```

+ It redirects the key-value pair `format_needed=false` out into the buffer of Github-provided special path variable `$GITHUB_OUTPUT`, 

Here, `format_need` is the one output parameter name of output parameters and it sets to string `false`. 

+ Then It redirects message `Code formatting is clean.` out into the buffer of Github-provided special step summary variable `$GITHUB_STEP_SUMMARY`,

displaying message `Code formatting is clean.` at the top of Action Log.
 
</details>

Overall,

Case 1: If uncommitted file changes are detected by Git and it executes successfully

Assuming `git diff` returns 

```
Index: fileA.cs\n--- line 1\n+++ line 2\n
```

=> It will execute the `then` block.

=> Thus, it will sets the `format_need` to string `true` 

and `diff_output` to

```
EOF
Index: fileA.cs
--- line 1
+++ line 2
EOF
```

that will be used on other action or job later.

=> Lastly, it writes the message 

```
## Formatting Changes Detected
Index: fileA.cs
--- line 1
+++ line 2
```

to step summary file that will be displayed at the top of Action Log.

Case 2: If no uncommitted file changes are detected by Git and it executes successfully (i.e. `git diff` returns an empty string)

=> It will execute the `else` block.

=> Thus, it will sets the `format_need` to string `false` 

that will be used on other action or job later.

=> Lastly, simply writes the message

```
Code formatting is clean.
```

to step summary file that will be displayed at the top of Action Log.

Case 3: If executing `git diff` command failed, 

=> due to `|| true` on `git diff --exit-code --name-only || true` command, the `$GIT_DIFF` stores true.

=> Thus, executing `else` block (since boolean true is NOT considered as an empty string)

=> ... do samething in Case 1 (it has been analyzed, so skip it)

</details>
