# CH11 -- property access
## objectives
You will know how to

  + access a property

## CH11-1 -- accessing a property by property name
Like JS, you can access the property through `.` or `["<property-name>"]`

example:

```
jobs.clean_trash.outputs.need_to_clean
```

or

```
jobs.clean_trash.outputs["need_to_clean"]
```

will 

access the property named `need_to_clean` of `jobs.clean_trash.outputs` object.

> [!NOTE]
> However, for special characters (that is NOT a valid identifier in workflows or actions),
>
> one can't access the property through `.`
>
> One MUST access the property through `["<property-name>"]`

## CH11-2 -- accessing a property by index
Like JS, you can access the property through indexing `[<non-negative-integer>]` with given index (zero-based index) for an array.

example:

```
jobs.clean_trash.outputs[0]
```

will

access the first (as it's zero-based) element of `jobs.clean_trash.outputs` object (and it's an array too)
