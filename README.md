All the contexts: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs
Printing context to logs: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs#about-contexts

``` bash
# all of it
echo <output> | base64 -d | rev | jq

# only envvars
echo <output> | base64 -d | rev | jq -r '.[0].envvars' | base64 -d
```
