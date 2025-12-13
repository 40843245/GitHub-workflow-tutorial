# CH13 -- logical operator
## objectives
You will know  

  + logical operator
  + short-circuit feature

## CH13-1 -- logical operator

Like C#,

| logical operator | meaning | description |
| :-- | :-- | :-- |
| `&&` | and | iff all expressions are evaluated to true, it will return true |
| `||` | or | iff one of expressions are evaluated to true, it will return true |
| `!` | negate | iff the expression will be evaluated to false it will return true |
  
## CH13-2 short-circuit feature

Like C#,

In an expression consists of many expressions combined by logical operators,

when the expression can be determined in some expressions, the rest of expression will NOT be executed, saving running time.

The feature is so-called short-circuit.

### Examples
#### Example 1

In 

```
jobs.customized_actions.customized_composite_action.left_hand_side_returned_value && jobs.customized_actions.customized_composite_action.right_hand_side_returned_value
```

if `jobs.customized_actions.customized_composite_action.left_hand_side_returned_value` is evaluated to false,

then `jobs.customized_actions.customized_composite_action.right_hand_side_returned_value` is NOT executed due to it can be determined to false and short-circuit feature.

And it returns `false`.

#### Example 2

In 

```
jobs.customized_actions.customized_composite_action.left_hand_side_returned_value || jobs.customized_actions.customized_composite_action.right_hand_side_returned_value
```

if `jobs.customized_actions.customized_composite_action.left_hand_side_returned_value` is evaluated to true,

then `jobs.customized_actions.customized_composite_action.right_hand_side_returned_value` is NOT executed due to it can be determined to true and short-circuit feature.

And it returns `true`.
