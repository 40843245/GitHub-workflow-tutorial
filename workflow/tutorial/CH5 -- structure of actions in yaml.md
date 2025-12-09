# CH5 -- structure of actions in yaml
## objectives
You will know

   + analysis of structure of customized Action

## CH5-1 -- introduction of Action
You can think an Action as a single sync function.

Can define input parameter and output parameter on it.

## CH5-2 -- analysis of structure of customized Action
At top level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `name` | a descriptive name | name of the action | Github Actions will recognize it by its `name` | 
| `description` | description of the action | see following section | see following section | 
| `inputs` | a block defines input parameters | see following table | |
| `outputs` | a block defines outputs parameters | see following table | |

At `inputs-><input-parameter-name>` level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `description` | description of the input parameter | see following section | see following section |
| `required` | boolean value | A boolean value that determines that it is needed to pass the value to this argument when invoking the action.<br><li><ul>True: needed</ul><ul>False: Not needed</ul></li> | defaults to false | 
| `default` | default value | default value used, when this argument is NOT passed when invoking the action | |
| `deprecationMessage` | warning message | warning message when this parameter is depreciated | When one pass the value to this argument, a warning message will be written into Action Log |

At `outputs-><input-parameter-name>` level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `description` | description of the input parameter | see following section | see following section |

### usage and importance of description for input parameter
