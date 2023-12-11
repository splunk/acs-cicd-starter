# acs-cicd-starter

This is an example repository for using [ACS API](https://docs.splunk.com/Documentation/SplunkCloud/latest/Config/ACSIntro) in Github Actions to manage your Splunk Cloud environment. In addition to ACS API, Splunk also provides [ACS CLI](https://docs.splunk.com/Documentation/SplunkCloud/latest/Config/ACSCLI) and [ACS Terraform](https://registry.terraform.io/providers/splunk/scp/latest/docs) tools to manage your Splunk Cloud environment.

**Disclaimer**:  When using this repo it will be # authorative for many tasks on your Splunk Cloud environment.  If this repo does not reflect the state of your environment when you wire it up then you may find unintended side effects such as changes to index settings, apps being removed, IP allow lists changing, etc.

## Using this Repo

- Clone the structure to your own repo or fork it
- Enable Branch Protections to prevent direct commits to `main`
- Create a feature branch
- Make a change
- Commit and push it
- Create a pull request

## Secrets

For the existing Github Action to work, we need 4 secured secrets in our pipelines. These are encrypted environment variables to allow management tasks to be performed.

To update/populate them in Github, see [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)


### STACK_NAME

This is the stack name that you wish to manage with your repo

For example, if you access your stack using `https://my-awesome-stack.splunkcloud.com`, the stack name will be `my-awesome-stack`.

### STACK_TOKEN

This is the auth token that has been created within your Splunk Cloud stack so that you can authenticate and leverage ACS in your pipelines.

To create a new token, see [Create an authentication token guide](https://docs.splunk.com/Documentation/SplunkCloud/9.0.2209/Config/ACSusage#Create_an_authentication_token).

### SPLUNKBASE_USERNAME

When managing apps using ACS, we need Splunkbase credentials to generate a Splunkbase session ID. This is required for retrieving Splunkbase app information and inspecting private apps. We suggest creating a dedicated user for this.

### SPLUNKBASE_PASSWORD

The Splunkbase password to go with the above username.

## Splunkbase App management

When using this repo, ACS will be authoratitive for managing Splunkbase apps. So if someone installs a Splunkbase app via the UI and it is not in the [splunkbaseApps.json](./splunkbaseApps.json) file here then it will be uninstalled when the deployment pipeline runs next.

### App Installation

When installing an app the latest version of the app will be used, you need to add the following code snippet to the file [splunkbaseApps.json](./splunkbaseApps.json)

```json
    {
        "appName": "SplunkforPaloAltoNetworks",
        "splunkbaseID": "491",
        "license": "http://opensource.org/licenses/ISC"
    }
```

**Note:**
- Information about the app including the required license can be found at [splunkbase.splunk.com](https://splunkbase.splunk.com/).
- If the `version` is not specified, ACS will install the latest available version for the app.

### App Updates

Add the version parameter to your JSON and set to the version you wish to install

```json
    {
        "appName": "SplunkforPaloAltoNetworks",
        "version": "7.0.4",
        "splunkbaseID": "491",
        "license": "http://opensource.org/licenses/ISC"
    }
```

**Note:**
- If the version in your codebase is no longer available from Splunkbase then you may get an error in your pipeline until you update your [splunkbaseApps.json](./splunkbaseApps.json) to represent the present version available in Splunkbase.

### App Removal

Remove the section from the JSON on the app you wish to uninstall. When the pipeline runs it will check the installed Splunkbase apps and if it is not in [splunkbaseApps.json](./splunkbaseApps.json) then it will be removed.

### Unmanaged Splunkbase Apps

There are Splunkbase apps that are installed by default in Splunk Cloud or that you might not want to be managed by this process, these can be added to an ignore list in [workflow_support/splunkbase_exclusion.json](./workflow_support/splunkbase_exclusion.json)

## Private App Management

**Limitations:**
- This repo only supports private app installation on `Victoria` experience. However, see [instructions here](https://docs.splunk.com/Documentation/SplunkCloud/latest/Config/ManageApps#Manage_private_apps_using_the_ACS_API_on_Classic_Experience) for managing private apps using ACS API on `Classic` experience.
- It doesn't currently support local folders, pipeline will fail if one is detected
- Uninstallation of private apps via pipeline is not yet implemented

### App Installation

Add private apps as files to the [privateApps](./privateApps/) folder in the repository, treat this folder as you would when working on an app in `$SPLUNK_HOME/etc/apps`, i.e. uncompressed. This will make it easier to work on individual files within the app.

Only apps that are different in the merge request to what is in main will have actions take to try and deploy.

The Pipeline process is:
1. Identify apps that have changed and stash them
2. Append the short git commit ID to version in app.conf
3. Create tarball of the changed apps
4. Run each app through appInspect, first failure will kill pipeline
5. Upload any failure reports as artifacts to the pipeline for review
6. If no failures are encountered attempt to install the apps

## Scope for 0.1

#### CI Steps ####

- Validate HEC json
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

