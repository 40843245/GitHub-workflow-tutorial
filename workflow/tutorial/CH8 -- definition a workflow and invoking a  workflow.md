# CH8 -- definition a workflow and invoking a workflow
## objectives
You will know how to

  + define a workflow
  + invoke a workflow

## CH8-1 -- define a workflow

It behaves like an utility project in a solution.

A workflow mainly consists of

  Required
  
  - name: a descriptive name that will be displayed (on `Description` section) at GitHub Marketplace and will be recognized by GitHub Action. It will be text on the option when one selects a workflow.
  - on: defines which events will trigger this workflow.
  - jobs: defines a collection of job (or called task)

### jobs
#### In a workflow

In a callable or main workflow, jobs contain a list of job key

`jobs` has fields a list of job key

Each job key is used to recognize the job.

Each job key has fields

    Required

    + runs-on: determines the workflow runs on environment
    + steps: defines the behavior of a collection of job.

`steps` block defines the behavior of job.

Each job is seperator by `-` (and MUST be followed by `name` field)
    
Each job has fields
    
    Required
    
    -   name: a description name of the job
    
    Optional
    
    -   uses: invoke an action or a workflow (for more details, see CH6)
    -   with: passes the argument (corresponding to the action you want to invoke) with value.
    - needs (`needs` field of job name): depends on one or more specified jobs (by a list of job key or a job key), ensuring this job executes after these jobs are complete.
    -   id: defines a unique id so that it can be as a property used internally in this workflow (NEED when it invoke an action or a workflow because you have to access the returned value using the unique id.)
    -   shell: which shell will be used for executing these commands
    -   run: commands that will be executed on the shell.

In the block about job name, it consists of

  Required

  - name (`name` field of job name): a descriptive name of the job

  Optional

  - uses (`uses` field of job name): import an action or workflow (see CH6 to know how to import them)  
  - with (`with` field of job name): pass arguments to the action or workflow that you want to import.
  - needs (`needs` field of job name): depends on one or more specified jobs, ensure this job executes after these jobs are complete.

### on 
`on` field defines which events will trigger this workflow

<details>
<summary>
Event handlers
</summary>

`on` field can have this fields (represents event handlers)

  Repository Events (that are automatically triggered)
  
  + `push`: once one pushes commits to specific branch or Git Tag, it will trigger.
  + `pull_request`: once one CUD (create, update, and delete) for PR (pull request), it will trigger.
  + `pull_request_target`: similar to `pull_request`, but you can set the event handler will be applied to which targeting branch.
  + `tag`: once new tag is released, it will trigger.
  + `create`: once one creates branch or Git Tag, it will trigger.
  + `delete`: once one deletes branch or Git Tag, it will trigger.

  Scheduled Events (that are automatically triggered by schedule)

  + `schedule`: schedules the event. Once the schedule is up, it will trigger.

  Manual Events (that are trigger manually or through API)

  + `workflow_dispatch`: allow users to manually trigger it on GitHub GUI also allow it is triggered by GitHub API.

  Workflow Communication Events (that allow invocation between workflows)

  + `workflow_call`: mark the workflow as callable.
  + `workflow_run`: add a queue of workflow stacking trace. Once after specified workflow is completed, it will execute the workflow.

  Interaction Events (that are triggered when specific interaction on Git is met)

  + `issue`: Once the specified issue is CUDed (created, upadated and deleted) or the tag is updated, it will trigger.
  + `issue_comment`: Once one pull requests on the issue or comments on the issue, it will trigger.
  + `release`: Once a released is CUDed (created, upadated and deleted), it will trigger.

  External Events (that allow it is triggered externally)

  + `repository_dispatch`: allows it is triggered by external API.

</details>

Additionally, in most event handler, you can set the conditions (i.e. when the conditions are met, it will trigger) by filtering (using subfields of `on` (i.e. a field of event handler))

<details>
<summary>
Filters  
</summary>    

<details>
<summary>
`push` event handler
</summary>

| id | event handler | filter | acceptable value of filter | description | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | `push` | `branches` | a list containing branch name | **ONLY** applied to specific branches | it uses Regex. |
| 1 | `push` | `branches-ignore` | a list containing branch name | **NOT** applied to specific branches (opposite of `branches`) | it uses Regex. |
| 1 | `push` | `tags` | a list containing tag name | **ONLY** applied to specific tags | it uses Regex. |
| 1 | `push` | `tags-ignore` | a list containing tag name | **NOT** applied to specific branches (opposite of `tags`) | it uses Regex. |
| 1 | `push` | `paths` | a list containing pathes | **ONLY** applied to specific pathes | it uses Regex. |
| 1 | `push` | `paths-ignore` | a list containing pathes | **NOT** applied to specific pathes (opposite of `paths`) | it uses Regex. |

</details>

<details>
<summary>
`pull_request` event handler
</summary>

| id | event handler | filter | acceptable value of filter | description | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 2 | `pull_request` | `branches` | a list containing branch name | **ONLY** applied to specific branches | it uses Regex. |
| 2 | `pull_request` | `branches-ignore` | a list containing branch name | **NOT** applied to specific branches (opposite of `branches`) | it uses Regex. |
| 2 | `pull_request` | `tags` | a list containing tag name | **ONLY** applied to specific tags | it uses Regex. |
| 2 | `pull_request` | `tags-ignore` | a list containing tag name | **NOT** applied to specific branches (opposite of `tags`) | it uses Regex. |
| 2 | `pull_request` | `paths` | a list containing pathes | **ONLY** applied to specific pathes | it uses Regex. |
| 2 | `pull_request` | `paths-ignore` | a list containing pathes | **NOT** applied to specific pathes (opposite of `paths`) | it uses Regex. |
| 2 | `pull_request` | `types` | a list containing type (of search condition to filter pull requests on GitHub) | **ONLY** applied when specific types of operations are met  | it don't use Regex. |

</details>

<details>
<summary>
`issue` event handler
</summary>

| id | event handler | filter | acceptable value of filter | description | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 3 | `issue` | `types` | a list containing type (of search condition to filter issues on GitHub) | **ONLY** applied when specific types of operations are met | it don't use Regex. |

</details>

<details>
<summary>
`release` event handler
</summary>

| id | event handler | filter | acceptable value of filter | description | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 4 | `release` | `types` | a list containing type (of search condition to filter releases on GitHub) | **ONLY** applied when specific types of operations are met | it don't use Regex. |

</details>

<details>
<summary>
`workflow_call` event handler
</summary>

| id | event handler | filter | acceptable value of filter | description | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 5 | `workflow_call` | `inputs` | input parameters | usage is similar to `inputs` in a composite action | see CH6 for more details |
| 5 | `workflow_call` | `outputs` | output parameters | usage is similar to `outputs` in a composite action | see CH6 for more details |

</details>

<details>
<summary>
`workflow_run` event handler
</summary>

| id | event handler | filter | acceptable value of filter | description | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 6 | `workflow_run` | `workflows` | a list of workflow name | when one of these specified workflows is completed, it will trigger. | |
| 6 | `workflow_run` | `types` | a list of type | ONLY when one of these specified workflows (`workflows` fields) are in the specified status (`workflow_run` field), it will trigger. | |
| 6 | `workflow_run` | `branches` | a list of branch name | restricts the `workflow_run` event handler listens on these branches | |

</details>

<details>
<summary>
`workflow_dispatch` event handler
</summary>

| id | event handler | filter | acceptable value of filter | description | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 7 | `workflow_dispatch` | `inputs` | input parameters | user has to fill in these inputs when one wants to fill in |  |

At `on->workflow_dispatch->inputs-><input-parameter-name>` level

| key | value (or content) | description | notes |
| :-- | :-- | :-- | :-- |
| `description` | description of the input parameter | see following section | see following section |
| `required` | boolean value | A boolean value that determines that it is needed to pass the value to this argument when invoking the action.<br><ul><li>True: needed</li><li>False: Not needed</li></ul> | defaults to false | 
| `default` | default value | default value used, when this argument is NOT passed when invoking the action | |
| `type` | one of following <ul><li>choice</li><li>string</li><li>boolean</li><li>environment</li></ul> | type of the input | |
| `options` | a list of strings as available option (separated by `-`) | availables options when user choose on the input | <ul><li>It is needed iff `type` is set to `choice`<br>It means</li><li>If the `type` is set to `choice`, the `options` is required.</li><li>If the type is NOT set to `choice`, you can't set this field</li></li>|

</details>

<details>
<summary>
`schedule` event handler
</summary>

uses cron syntax to filter.

</details>

</details>

> [!IMPORTANT]
> In `schedule` event handler, you MUST use cron syntax to schedule a schedule at specific time.
