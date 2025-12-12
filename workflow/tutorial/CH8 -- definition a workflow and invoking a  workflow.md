# CH8 -- definition a workflow and invoking a workflow
## objectives
You will know how to

  + define a workflow
  + invoke a workflow

## CH8-1 -- define a workflow

It behaves like an utility project in a solution.

A workflow mainly consists of

  Required
  
  - name (`name` field): a descriptive name that will be displayed (on `Description` section) at GitHub Marketplace and will be recognized by GitHub Action. It will be   text on the option when one selects a workflow.
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

# on 
`on` field defines which events will trigger this workflow

`on` field can have this fields (all are optional)

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
