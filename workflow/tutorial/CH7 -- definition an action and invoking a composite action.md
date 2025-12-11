# CH7 -- definition an action and invoking a composite action
## objectives
You will know how to

  + define an action
  + invoke a composite action

## CH7-1 -- define an action
It behaves like an utility function.

A composite action mainly consists of 

  Required

  + name (`name` field): a descriptive name that will be displayed (on `Description` section) at GitHub Marketplace. 
  + input parameters (`inputs` block): receives value from other place.
  + output parameters (`outputs` block): returned value.
  + function body (`runs` block): defines how this action behaves.

### input parameters
Input parameters contain a list of input parameter (a list of field name of `inputs` are a list of name of input parameters)

In input parameters, `inputs` has fields that represents a list of input parameter name.

In the block about input parameter name, it consists of

  Required
  
  + description (`description` field of input parameter name): a descriptive name of the input parameter

  Optional

  + required (`required` field of input parameter name): determines whether it is needed to pass the argument corresponding to it when invoking this composite action, or not.

    If it sets to true, it means it is need

    Here, if one doesn't pass the argument when invoking, then exception will be thrown at runtime, thus terminates the job by default.

    Otherwise, it means it is NOT needed (of course, one can pass the argument when invoking)

  + default (`default` field of input parameter name): default value that is used when the one doesn't pass argument corresponding to it when invoking.

### output parameters
Output parameters contain a list of out parameter (a list of field name of `outputs` are a list of name of output parameters)

In output parameter, `outputs` has fields that represents a list of output parameter name.

In the block about input parameter name, it consists of

  Required

  + description (`description` field of output parameter name): a descriptive name of the output parameter

  Optional but **usually used**

  + value (`value` field of output parameter name): returned value of the output parameter

> [!WARNING]
> Though `value` field is NOT required (missing `value` field doesn't cause runtime exception),
>
> **NEVER missing `value` field**,
>
> as if you do so,
>
> then returned value will be NULL or an empty string (usually NULL, but it is determined by GitHub Actions engine).
>
> In this case, if there is no other null safely check (or empty string check),
>
> it will cause exception at runtime at the later use of the output parameter. 
 
### runs block
The `runs` block defines the body of the action.

`runs` has two fields

  Required

  + steps: defines the behavior of steps (a collection of run).
  
  Optional

  + using: `using` field directly which type of action it is.

> [!IMPORTANT]
> In a composite action (that can be invoked),
>
> MUST set `using` field to `'composite'`
>
> **`'using: 'composite'`** is NEEDED

`steps` block defines the behavior of steps (a collection of step (or so-called run)).

Each run is seperator by `-` (and MUST be followed by `name` field) 

Each run has fields

  Required

  + name: a description name of the run

  Optional

  + uses: import other actions (for more details, see CH6)
  + with: passes the argument (corresponding to the action you want to import) with value.
  + shell: determines which CLI of shell will be used on this run.
  + id: defines a unique id so that it can be as a property used internally (for example, in output parameter (see CH9 for more details))
  + 
