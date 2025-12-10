# CH6 -- output parameter of action or job
## objectives
You will know how to 
  
  + return a value in an action or a job
  + receive or access the returned value returned by an action or a job

## CH6-1 -- return a value
### CH6-1-1 return a value in an action
#### In Docker container Workflow (commonly used)
Echo the output parameter name and its value (represented as a key-value pair `<output-parameter-name>=<returned-value>`) to a `GitHub Actions `$GITHUB_OUTPUT`

Format:

```
echo "<output-parameter-name>=<returned-value>" >> $GITHUB_OUTPUT
```

where 

`<output-parameter-name>` is the output parameter name. It will be used when accessing the returned value.

and

`<returned-value>` is the returned value of the action.

> [!IMPORTANT]
> Since we have to write the key-value pair to a variable,
>
> we need to quote the key-value piar with double quotations
>
> (even though no need to quote the value of the field).
>
> However, the key-value pair is one part of value of the field.
>
> NOT related to this.  
