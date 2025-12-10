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

At `outputs-><output-parameter-name>` level

| key | value (or content) | description | notice |
| :-- | :-- | :-- | :-- |
| `description` | description of the output parameter | see following section | see following section |
| `value` | a variable or an object, or an expression consist of them that can be evaluated to a value | the returned value | see following section |

> [!IMPORTANT]
> The output parameter ONLY can map or forward internal final result (making it external and thus can be used in others)
>
> The output parameter can't simply return `$GITHUB_OUTPUT`.
>
> So, in a composite run action, it is needed to echo the final result (represented as a key-value pair) to `$GITHUB_OUTPUT`,
>
> then fetch the internal final result and return it in `value` field.
>
> See CH6 for more details.

### usage and importance of description for input parameter and output parameter
It is very important and a good practice to give a descriptive name to a description. Because

+ it makes it be more readable (and thus be more maintainable)
+ (The most important) If published to GitHub Action Marketplace, then

information (such as description) will be displayed on the installation page.

A brief and clear description can make it be easilier to understand for searchers and 

even affect the number of downloads.


  
