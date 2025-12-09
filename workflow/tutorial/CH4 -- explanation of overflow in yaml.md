# CH4 -- explanation of overflow in yaml
## objectives
You will know

  + analysis of structure of customized GitHub Actions Workflow

## CH4-1 -- analysis of structure of customized GitHub Actions Workflow
At top level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `name` | a string value you want | A unique name that recognized by GitHub | When you have to use workflow, it recongize by its name | 
| `on` | one or more conditions you want | When one of these conditions is met, it will trigger the Action | | 
| `jobs` | a task that will be run | a task that will be run if the Action is triggered | |

At `on` level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `push` | `branches:` followed by a list of branch name (string type) | When one tries to push it to branches in a list, it will trigger the job | No double quotation and single quotation need for branch name | 
| `pull_request` | `branches:` followed by a list of branch name (string type) | When one tries to commit a pull request to branches in the list, it will trigger the job | | 
| `workflow_dispatch` | none | allow manually run the workflow on GitHub App | |
| `schedule` | `-cron:` followed by a time quoated by single quotation | schedule the job that will be executed at specific time | cron syntax must be used here |

At `jobs` level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `build_and_test` | including a block containing `runs-on:` followed by `steps:` key | ensure that all code snippets of codespace (only is applied for changed file) can be build successfully and pass the test | If NOT all code snippets can be build successfully or pass the test, the job will return failed status. | 
| `check_dependencies` | including a block containing `runs-on:` followed by `steps:` key | ensure that all used dependencies pass the vulnerability test | If NOT, the job will return failed status. |

At `jobs-><key>` level

here, `<key>` is one key name under `jobs` level.

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `runs-on` | one of currently available environment (using enviroment name) | run the task on which environment | No double quotation and single quotation need for environment name | 
| `steps` | including a block containing lots stpes | defines the task will execute which one or more steps | each step is seperatored by `-` (and MUST be followed a whitespace then `name` key).<br>See my table for more details |

At `jobs-><key>->steps` level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `name` | a descriptive name | a descriptive name that describes the step | <li><ul>No double quotation and single quotation need for the descriptive name</ul><ul>MUST be look like this `- name`</ul></li> | 
| `uses` | other action name or workflow name | import and execute other action or workflow | No double quotation and single quotation need here |
| `with` | a key-value pair where its value is the version number | which version will be installed or configured (or both)  | <li><ul>The `with` key MUST be used with `uses` tag.<br>And The `uses` key MUST be used with `with` tag if you need to specify the exact version number for installing and configuration.<br>Usually specifies the version number</ul><ul>the version number in the value **MUST be quotated with single quotation**</ul></li> |
| `run` | a block containing a list of commands on Windows terminal | will run these commands  | <li><ul>No double quotation and single quotation here</ul><ul>seperated by new line</ul></li> | 


### Examples
#### Example 1
```
name: Daily Dependency Security Check

on:
  # 觸發條件：使用 cron 語法，設定在每天 UTC 時間 00:00 執行
  schedule:
    - cron: '0 0 * * *'
  # 允許手動執行
  workflow_dispatch:

jobs:
  check_dependencies:
    runs-on: ubuntu-latest # 安全掃描通常可以在 Linux 環境下執行

    steps:
      # Step 1: 檢出儲存庫代碼
      - name: Checkout Code
        uses: actions/checkout@v4
      
      # Step 2: 設定 .NET SDK 環境
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      
      # Step 3: 使用 .NET 命令檢查 Nuget 套件的漏洞
      # 這是一個示範，您可以整合更進階的第三方安全 Action (如 Snyk 或 Dependabot)
      - name: Run .NET Vulnerability Check
        # 使用 dotnet list package --vulnerable 命令來查找已知漏洞
        run: |
          echo "Scanning for known NuGet vulnerabilities..."
          dotnet list package --vulnerable
          # 如果發現漏洞，可以選擇讓 Workflow 失敗 (例如：使用 exit 1)
```
