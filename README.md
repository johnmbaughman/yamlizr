# Yamlizr - Azure DevOps Designer-to-YAML Pipeline Conversion Tool

[cascap.yamlizr-badge]: https://img.shields.io/nuget/v/yamlizr?color=blue
[cascap.yamlizr-url]: https://nuget.org/packages/yamlizr

![CI](https://github.com/f2calv/yamlizr/actions/workflows/ci.yml/badge.svg) [![Coverage Status](https://coveralls.io/repos/github/f2calv/yamlizr/badge.svg?branch=main)](https://coveralls.io/github/f2calv/yamlizr?branch=main) [![SonarCloud Coverage](https://sonarcloud.io/api/project_badges/measure?project=f2calv_yamlizr&metric=code_smells)](https://sonarcloud.io/component_measures/metric/code_smells/list?id=f2calv_yamlizr) [![Nuget][cascap.yamlizr-badge]][cascap.yamlizr-url]

> This tool was created when there was no means of exporting a designer/classic build/release definition to YAML. As of November 2020 there is a new [Export to YAML](https://devblogs.microsoft.com/devops/replacing-view-yaml/) feature which allows you to export a _Build_ pipeline to YAML with a single click. This official function covers more edge cases than this CLI for Build pipelines. Where this CLI still has benefits is that it also converts Release definitions to YAML, which the official tool does not. It also allows the conversion of every single Build/Release definition en-masse, so much less clicking! Then you can then cut/copy/paste/manipulate the generated YAML steps as required to fit into a build and/or deployment pipeline unique to your own requirements.

**yamlizr** is a [.NET Global Tool](https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools) which converts Azure DevOps Classic Designer Build/Release Definitions and any referenced Task Groups en-masse into their [YAML Pipeline](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema) or [GitHub Action](https://github.com/features/actions) equivalent.

The tool itself uses the [Azure DevOps .NET Client Libraries](https://docs.microsoft.com/en-us/azure/devops/integrate/concepts/dotnet-client-libraries?view=azure-devops) to pre-cache relevant data from your Azure DevOps organisation/account. This data includes build/release definitions, task groups, tasks/extensions data and variable groups. This Azure DevOps data is converted into Azure DevOps Pipeline objects (stages/jobs/steps/variables) which are then persisted to YAML using the [YamlDotNet](https://github.com/aaubry/YamlDotNet) library.

This _is not_ a delicate tool to create perfectly constructed YAML pipelines. Instead consider it to be a hammer which will spawn as many YAML files as possible and from this YAML you can pick/choose and copy/paste the relevant stages/jobs/steps/variables into your own preferred YAML CI/CD deployment architecture.

Also... there is an optional switch whereby the tool can pass the Azure DevOps Pipeline objects into the [AzurePipelinesToGitHubActionsConverter library](https://github.com/samsmithnz/AzurePipelinesToGitHubActionsConverter) (by [@samsmithnz](https://github.com/samsmithnz)) and export your pipelines as GitHub Actions YAML.

**Disclaimer**: _Do not consider any of the YAML generated by this tool to be 'production ready'. Do your own testing/research and [post any issues](https://github.com/f2calv/yamlizr/issues) and/or make a PR!_

If you find this tool of use then please give it a thumbs-up by giving this repository a :star: ... :wink:

## Installation/Set-up

- [Create a Personal Access Token (PAT)](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page) with the following scopes/permissions;
  | Scope             | Permission    |
  | ----------------- | ------------- |
  | Build             | Read          |
  | Deployment Groups | Read & Manage |
  | Release           | Read          |
  | Task Groups       | Read          |
  | Variable Groups   | Read          |
- Download and install either [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1) or [.NET 6.0 SDK](https://dotnet.microsoft.com/download/dotnet/6.0).
- From a command line shell install the tool;
  `dotnet tool update --global yamlizr`

### CLI Operation

To generate YAML files in the `c:/temp/myoutputfolder` output folder execute the following command;

```pwsh
yamlizr generate -pat <your PAT here> -org <your AzDO organisation> -proj <your AzDO project> -out c:/temp/myoutputfolder
```

For context-sensitive help execute;

```pwsh
yamlizr --help
```

Optional arguments;

- `--filter <some string here>` filter build/release definitions (if you want to use a more granular approach).
- `--phasetype <phase type here>` filter deployment jobs by Deploy Phase Type the default is `AgentBasedDeployment` DeployPhaseTypes(tested), other (un-tested) options are `RunOnServer`, `MachineGroupBasedDeployment` & `DeploymentGates`.

Optional switches;

- `--inline` merge the tasks from task groups into the steps of the calling job instead of creating additional template files.
- `--githubactions` generate GitHub Actions workflows via [AzurePipelinesToGitHubActionsConverter](https://github.com/samsmithnz/AzurePipelinesToGitHubActionsConverter).

To generate both Azure Pipelines and GitHub Actions YAML for a build definition called 'wibble-CI' and a release definition called 'wibble-CD';

```pwsh
yamlizr generate -pat <your PAT here> -org <your AzDO organisation> -proj <your AzDO project> -out c:/temp/myoutputfolder --filter wibble --githubactions
```

All YAML files generated are output into sub-folders of a project folder, i.e. using the above example of `-o c:/temp/myoutputfolder` the following folders are created;

- `c:/temp/myoutputfolder/<your AzDO project>/AzureDevOpsBuilds/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/AzureDevOpsReleases/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/AzureDevOpsTaskGroups/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/GitHubBuilds/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/GitHubReleases/*.yml`

### Core Dependencies

- [Azure DevOps .NET Client Libraries](https://docs.microsoft.com/en-us/azure/devops/integrate/concepts/dotnet-client-libraries?view=azure-devops)
- [AzurePipelinesToGitHubActionsConverter](https://github.com/samsmithnz/AzurePipelinesToGitHubActionsConverter)
- [YamlDotNet](https://github.com/aaubry/YamlDotNet)
- [CommandLineUtils](https://github.com/natemcmaster/CommandLineUtils)
- [ShellProgressBar](https://github.com/Mpdreamz/shellprogressbar)

### Misc Tips

- The [NuGet package](https://www.nuget.org/packages/yamlizr/) includes [SourceLink](https://github.com/dotnet/sourcelink) which enables you to jump inside the library and debug the API yourself. By default Visual Studio 2017/2019 does not allow this and will pop up an message "You are debugging a Release build of...", to disable this message go into the Visual Studio debugging options and un-check the 'Just My Code' option (menu path, Tools > Options > Debugging).

### Known Issues

- ShellProgressBar gives random formatting problems.
- `--parallelism` command line option for faster processing is a bit buggy so disabled.
- Various YAML structures are 'missing', PR's welcome.

### Feedback/Issues

Please post any issues or feedback [here](https://github.com/f2calv/yamlizr/issues).
