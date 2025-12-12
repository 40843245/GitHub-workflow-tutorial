# CH10 -- high order function in workflow
## objectives
You will know 

  + commonly used high order functions

## CH10-1 -- always
`always` in current job always returns true, unless current job is cancelled.

purpose:

It forces to executed next one job, unless current job is cancelled.

syntax:

```
always()
```

## CH10-2 -- `success`
`success` returns true iff the previous one executed job executed successfully, which is a default condition.

syntax:

```
success()
```

## CH10-3 -- `cancelled`
`cancelled` returns true iff the previous one executed job is cancelled.

syntax:

```
cancelled()
```

## CH10-4 -- `failure`
`failure` returns true iff the previous one executed job fails to be executed.

```
failure()
```

## CH10-5 -- `contains`
`contains(<seaerched-array>,<target>)` returns true iff `searched-array` array or list contains `<target>.`

syntax:

```
contains(<seaerched-array>,<target>)
```

## CH10-6 -- `startsWith`
`startsWith(<str>,<prefixStr>)` returns true iff the string `<str>` with `<prefixStr>`


```
```
