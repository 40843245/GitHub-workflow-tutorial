# CH14 -- conditional expression
## objectives
You will know how to

  + execute which block by condition

## CH14-1 -- execute which block by condition
### if
On shell (such as `Bash` shell)

If the expresion in `if` is evaluated to true, 

it will execute `then` block.

Otherwise, it will execute `else` block (if exists)

Format:

```
if(<condition>); then
  <executed_when_condition_is_true_statments> # executed iff `<condition>` is evaluated to true
else
  <executed_when_condition_is_false_statments> # executed iff `<condition>` is evaluated to false
fi
```

or

```
if(<condition>); then
  <executed_when_condition_is_true_statments> # executed iff `<condition>` is evaluated to true
fi
```
