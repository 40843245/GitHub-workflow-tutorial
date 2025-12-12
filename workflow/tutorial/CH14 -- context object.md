# CH14 -- context object 
## objectives
You will have a basic knowledge of 

    + commonly used, available context objects in a workflow or an action

## CH14-1 -- `job`
`job` context object contains information about the currently running job.

## CH14-2 -- `jobs`
For reusable workflows only, `job` context object contains information of current job (for example, internal final result (that is redirected out into `$GITHUB_OUTPUT` variable) of jobs from the reusable workflow. 

## CH14-3 -- `steps`
`steps` context object contains the information of steps that are executed (in `steps` block)

## CH14-4 -- `runner`
`runner` context object contains information about the runner that is running the current job. 

## CH14-5 -- `needs`
`needs` context object contains the outputs of all jobs that are defined as a dependency (in `needs` block) of the current job. 

## CH14-6 -- `inputs`
`inputs` context object contains the inputs of a reusable or manually triggered workflow.

## ref
See [a list of context objects in official docs](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts)
