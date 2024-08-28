## Background

All the contexts: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs
Printing context to logs: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs#about-contexts

## Usage

``` bash
# all of it
echo <output> | base64 -d | rev | jq

# only envvars
echo <output> | base64 -d | rev | jq -r '.[0].envvars' | base64 -d
```

## Next steps

1. Add target host to send data to
2. Exfil to Azure Storage Account because of firewall exception for https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#communication-between-self-hosted-runners-and-github
