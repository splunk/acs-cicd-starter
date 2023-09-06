# acs-pipelines

**Disclaimer**:  When using this repo it will be # authorative for many tasks on your Splunk Cloud environment.  If this repo does not reflect the state of your environment when you wire it up then you may find unintended side effects such as changes to index settings, apps being removed, IP allow lists changing, etc.  

### Using this Repo

- Clone the structure to your own repo
- Enable Branch Protections to prevent direct commits to main
- create a feature branch
- make a change
- commit and push it
- create a pull request

## Secrets

For this to work we need 4 secured secrets populating for our pipelines, these are encrypted environment variables to allow management tasks to be performed.

> TO DO add steps to update these in github

### STACK_NAME

This is the stack name that you wish to manage with your repo

### STACK_TOKEN

This is the auth token that has been created within your Splunk Cloud stack so that you can authenticate and leverage ACS in your pipelines...

> TO DO  add instructions on how to create this.

### SPLUNKBASE_USERNAME

When managing splunkbase apps using ACS we need splunkbase credentials, I would suggest creating a dedicated user for this.

### SPLUNKBASE_PASSWORD

The Splunkbase password to go with the above username.

## Splunk Base App management

When using this repo ACS will be authoratitive for managing Splunkbase apps, so if someone installs a Splunkbase app via the UI and it is not in the `splunkbaseApps.json` file here then it will be uninstalled when the deployment pipeline runs next.

### App Installation

When installing an app the latest version of the app will be used, you need to add the following code snippet to the file `splunkbaseApps.json`

```
    {
        "appName": "SplunkforPaloAltoNetworks",
        "splunkbaseID": "491",
        "license": "http://opensource.org/licenses/ISC"
    }
```

### App Updates

Add the version parameter to your JSON and set to the version you wish to install

```
    {
        "appName": "SplunkforPaloAltoNetworks",
        "version": "7.0.4",
        "splunkbaseID": "491",
        "license": "http://opensource.org/licenses/ISC"
    }
```

**Information**: If the version in your codebase is no longer available from Splunkbase then you may get an error in your pipeline until you update your `splunkbaseApps.json` to represent the present version available in Splunkbase.

### App Removal

Remove the section from the JSON on the app you wish to uninstall, when the pipeline runs it will check the installed Splunkbase apps and if it is not in `splunkbaseApps.json` then it will be removed.

### Unmanaged Splunkbase Apps

There are Splunkbase apps that are installed by default in Splunk Cloud or that you might not want to be managed by this process, these can be added to an ignore list in `workflow_support/splunkbase_exclusion.json`

## Private App Management

`Victoria Only`

### App Installation

Limitations:
- Doesn't currently support local folders, pipeline will fail if one is detected
- Uninstallation of private apps via pipeline is not yet implemented

Add private apps as files to the `privateApps` folder in the repository, treat this folder as you would when working on an app in `$SPLUNK_HOME/etc/apps`, ie uncompressed, this will make it easier to work on individual files within the app.

Only apps that are different in the merge request to what is in main will have actions take to try and deploy.

The Pipeline process is:
- identify apps that have changed and stash them
- append the short git commit ID to version in app.conf
- create tarball of the changed apps
- run each app through appInspect, first failure will kill pipeline
- upload any failure reports as artifacts to the pipeline for review
- If no failures are encountered attempt to install the apps

### Scope for 0.1

#### CI Steps ####

- Validate hec json
- Validate IP Allow JSON
- Validate indexes JSON
- Private Apps (install and update)
- splunkBase apps (install and delete working)

#### CD Steps ####

- no secrets no continue - validate connectivity and auth
- IP Allow Deploy - what's changed
- IP Allow Deploy - install new bits
- IP Allow Deploy - Remove bits in Cloud that aren't in Git
- indexes Deploy - install new
- indexes Deploy - updated changed
- hec Deploy - install new  **what to do with secrets**
- hec Deploy - update changed **what to do with secrets**
- private App - identify changed apps, update version to something relevant to commit/merge in changed apps, package changed apps, push changed apps
- splunkBase apps - get current, workout delta, install new, upgrade existing, remove those not in code

#### Documentation, cause if we don't write it here it won't happen ####

