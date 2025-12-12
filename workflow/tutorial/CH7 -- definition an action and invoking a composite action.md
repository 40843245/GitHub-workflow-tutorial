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
  + run: commands that will be executed on the shell.

> [!TIP]
> You can think that assignning `id` field to a unique id value defines a property named the unique id, used for reflection.

+ mapping tables for matching

| field | needed matching word (with full sentence) | needed matching word | description | notes |
| :-- | :-- | :-- | :-- | :-- |
| `uses` | In some case, `use` fields is needed to use `with` field together | `with` | import the action (specified by `uses` field) with passing these argument (specified by `with` field)| According to the definition of the action you want to import |
| `shell` | `shell` field ALWAYS is needed to use with `run` together | `run` | execute the script (these commands specified in `run` fields) in which shell | | 

<details>
<summary>`shell` field</summary>
  
mapping table of available value

| field | available value | description | available platform | notes |
| :-- | :-- | :-- | :-- | :-- |
| `shell` | `Bash` | execute the script on `Bash` terminal | <li><ul>`Linux`</ul><ul>macOS</ul><ul>Windows</ul></li> | use `Bash` syntax |
| `shell` | `sh` | execute the script on shell (which shell be used according to OS version and platform)  | <li><ul>`Linux`</ul><ul>`macOS`</ul><ul>`Windows<`/ul></li> | |
| `shell` | `pwsh` | execute the script on `Powershell Core` (or called `Powershell 7+`)  | all platform that supports GitHub runner, including<li><ul>`Linux`</ul><ul>`macOS`</ul><ul>`Windows`</ul></li> | |
| `shell` | `powershell` | execute the script on `Powershell` | <li><ul>`Windows`</ul></li> | as ONLY Windows platform has `Powershell` |
| `shell` | `cmd` | execute the script on `Windows Command Prompt` | <li><ul>`Windows`</ul></li> | as ONLY Windows platform has `Command Prompt` |
| `shell` | `python` | execute the script on `python` interpreter | platforms that installs `python` interpreter | If you installs many `python` interpreters, typically, it will use the interpreter that is pointed by `python` system environment variable. |

Default value of `shell` field

According to Google Gemini's response, I take it as a table that

is ordered by default value of `shell` field desc (i.e. from highest priority to lowest priority)

| runner on which platform | default value of `shell` field |
| `Linux` runner | <li><ul>`bash` (if available)</ul><ul>`sh`</ul></li> | 
| `macOS` runner | <li><ul>`bash` (if available)</ul><ul>`sh`</ul></li> | 
| `Windows` runner | <li><ul>`pwsh`</ul></li> | 

Customized shell as template

Format (represented by Regex):

```
{template} := command {option} {path-of-script} ({more-options})*
{option} := {running-interpreter} {value-for-running-interpreter}*
{running-interpreter} := which interpreter which interprets and execute the script
{flag-for-running-interpreter} := a value for {running-interpreter}
{path-of-script} := the path will be executed.
{more-options} := one or more options (In each option, option name and its value)
```

</details>

<details>
<summary>`using` field</summary>

| available value of `using` field | action type | running enviroment | needed matching word | available scanerio | notes |
| :-- | :-- | :-- | :-- | :-- | :-- |
| `node20` | JavaScript action | `Node.js` 20 | `main` | <li><ul>For better performance</ul></li>| Javascript script is lightweight and has better performance | 
| `docker` | Docker container action | Docker containetr | `image` | using Docker | Dockerfile buils a Docker container and a Docker image can run the container |
| node20 | JavaScript action | `Node.js` 20 | `main` | <li><ul>For better performance</ul></li>| Javascript script is lightweight and has better performance |
</details>
