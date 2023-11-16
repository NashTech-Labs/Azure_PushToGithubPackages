# Azure_PushToGithubPackages
Use this task to restore, pack, or push NuGet packages, or run a NuGet command. This task supports NuGet.org and authenticated feeds like Azure Artifacts. This task also uses NuGet.exe and works with .NET Framework apps.

## Pipeline Requirements

The dotNet Module pipeline requires the following parameters to be defined:

Parameters:

| Name  | Displayname | type | Default | Values | Opional/Required | Comments |
| ------------- | ------------- | :-------------: | :-------------: | :-------------: | :-------------: | ------------- |
| useServiceConnectionForPush | Push to GitHub using Service Connection | Boolean | true | true / false | Required | Use when 1)useServiceConnectionForPush = true && nuGetFeedType = external then Push packages to Github Nuget feed using Service Connection   2) useServiceConnectionForPush = true && nuGetFeedType = internal then Push packages to Target feed |
| publishFeedCredentials | NuGet server | String | |  | Required | Required when command = push && nuGetFeedType = external Specifies the NuGet service connection that contains the external NuGet server’s credentials |
| publishVstsFeed | Target feed | String | | | Required | Required when command = push && nuGetFeedType = internal. Specifies a feed hosted in this account. You must have Azure Artifacts installed and licensed to select a feed here. |
| packagesToPush | Path to NuGet package(s) to publish | String | '*.nupkg' |  | Required | Specifies the pattern to match or path to nupkg files to be uploaded. Multiple patterns can be separated by a semicolon |
| nuGetFeedType | Target feed location | String | internal |  | Required | Allowed values: internal (This organization/collection), external (External NuGet server (including other accounts/collections)). Specifies whether the target feed is an internal feed/collection or an external NuGet server |
| allowPackageConflicts | Allow duplicates to be skipped | Boolean | false | true / false | Optional | Use when command = push && nuGetFeedType = internal. Reports task success even if some of your packages are rejected with 409 Conflict errors. |
| publishPackageMetadata | Publish pipeline metadata | Boolean | true | true / false | Optional | Use when command = push && nuGetFeedType = internal && command = push. Changes the version number of the subset of changed packages within a set of continually published packages. |
| verbosityPush | Verbosity | String | 'Detailed' | Quiet, Normal, Detailed | Optional | Specifies the amount of detail displayed in the output |

These parameters provide multiple use case options for the dotnet templates pipeline, enable/disable flags for the utilization of different templates as per the requirements.


## Use Cases

You can directly call a particular template as per the requirement. for example: 

  ```yaml
  # azure-pipeline.yml
  resources:
  repositories:
    - repository: Template
      type: github
      name: your_username/Azure_PushToGithubPackages
      ref: <respective branch name>
      endpoint: 'githubServiceConnectioNname'

  steps:
  # passing the parameters
  - ${{ if eq( parameters.useServiceConnectionForPush, true) }}:
    - ${{ if eq( parameters.nuGetFeedType, 'external') }}:
      - template: PushToGithubPackages.yaml
        parameters:
          packagesToPush: '${{ parameters.packagesToPush }}'
          nuGetFeedType: '${{ parameters.nuGetFeedType }}'
          publishFeedCredentials: '${{ parameters.publishFeedCredentials }}'
          verbosityPush: '${{ parameters.verbosityPush }}'
          useServiceConnectionForPush: '${{ parameters.useServiceConnectionForPush }}'
    - ${{ elseif eq( parameters.nuGetFeedType, 'internal') }}:
      - template: PushToGithubPackages.yaml
        parameters:
          packagesToPush: '${{ parameters.packagesToPush }}'
          nuGetFeedType: '${{ parameters.nuGetFeedType }}'
          publishVstsFeed: '${{ parameters.publishVstsFeed }}'
          publishPackageMetadata: '${{ parameters.publishPackageMetadata }}'
          allowPackageConflicts: '${{ parameters.allowPackageConflicts }}'
          verbosityPush: '${{ parameters.verbosityPush }}' 
          useServiceConnectionForPush: '${{ parameters.useServiceConnectionForPush }}'
  - ${{ else }}:
    - template: PushToGithubPackages.yaml
      parameters:
        packagesToPush: '${{ parameters.packagesToPush }}'
        useServiceConnectionForPush: '${{ parameters.useServiceConnectionForPush }}'                  

  ```
Make sure to adjust the repository name, branch name, and parameter values according to your project's requirements.
