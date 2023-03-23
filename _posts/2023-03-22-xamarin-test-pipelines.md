---
layout: post
title: "Unit Testing Xamarin in 2023"
subtitle: "Stay up to date with dotnet tooling on your xamarin azure pipelines"
date: 2023-03-22
background: '/img/posts/06.jpg'
background-alt: "Preview Image For This Post"
category: "Pipelines"
tags: [xamarin,azure,pipelines]
---

# Xamarin in 2023?

Thats right Xamarin is still kicking. Maui is technically GA but any team with a few Xamarin apps under their belt would tell you that migration is not easy and Xamarin still has some life in it. The best thing about Xamarin at this point is that it is very stable, it has been stable for a few years and you seldom run into issues with the framework.If you do the ecosystem is very mature and theres most likely a nuget package to solve your problem.

This post will focus on running unit tests in Azure Pipelines, making sure that builds work properly with some of the caviats that we need to account for with Xamarin being incompatible with newer dotnet versions.

## .NET 5+ Compatibility

Xamarin projects do not work well with .NET 5 and above, the moment you increase the unit test csproj version to target `net5.0`, you are in for a world of problems. Looking at popular library [Xamarin.CommunityToolkit](https://github.com/xamarin/XamarinCommunityToolkit/blob/da30dde36b7be118c1a4f1c2e282b0b32f8b3d8c/src/CommunityToolkit/Xamarin.CommunityToolkit.UnitTests/Xamarin.CommunityToolkit.UnitTests.csproj#L4), their test project is still targetting `.netcoreapp3.1`. 

At this point newer dotnet versions (5+) won't support your xamarin projects so we have to make do with dotnet core 3.1.

## Pipelines

You might be using global.json to pin down a .NET version. If you are using a .NET 5 version, it is time to upgrade to .NET 6. .NET 5 and .NET Core 3.1 are incompatible and the dotnet cli cannot run tests correctly when using these 2 versions, this is not an issue when using .NET 6 and .NET Core 3.1.

The steps are simple:
- Install Nuget
- Restore projects
- Run Tests

The following YAML should work to build and run your tests:
```yaml
- task: NuGetToolInstaller@1
  displayName: Use latest NuGet
  inputs:
    checkLatest: true

- task: UseDotNet@2
  displayName: 'Install .NET SDK'
  inputs:
    version: '6.0.x'

- task: UseDotNet@2
  displayName: 'Install .NET 3.1 Test SDK'
  inputs:
    version: '3.1.x'

- task: UseDotNet@2
  displayName: 'Install .NET 2.1 Test SDK'
  inputs:
    version: '2.1.x'

- task: DotNetCoreCLI@2
  displayName: 'dotnet restore'
  inputs:
    command: restore
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'dotnet test'
  inputs:
    command: test
    projects: '**/*.Test*.csproj'
    publishTestResults: false
```

### .NET 5 Issues

I had alot of issues when targetting .NET 5, the `dotnet test` step would always fail with minimal information, even in debug mode!

The azure pipeline would display the following message:
```bash
##[error]Error: The process 'C:\hostedtoolcache\windows\dotnet\dotnet.exe' failed with exit code 1
##[warning].NET 5 has some compatibility issues with older Nuget versions(<=5.7), so if you are using an older Nuget version(and not dotnet cli) to restore, then the dotnet cli commands (e.g. dotnet build) which rely on such restored packages might fail. To mitigate such error, you can either: (1) - Use dotnet cli to restore, (2) - Use Nuget version 5.8 to restore, (3) - Use global.json using an older sdk version(<=3) to build
Info: Azure Pipelines hosted agents have been updated and now contain .Net 5.x SDK/Runtime along with the older .Net Core version which are currently lts. Unless you have locked down a SDK version for your project(s), 5.x SDK might be picked up which might have breaking behavior as compared to previous versions. You can learn more about the breaking changes here: https://docs.microsoft.com/en-us/dotnet/core/tools/ and https://docs.microsoft.com/en-us/dotnet/core/compatibility/ . To learn about more such changes and troubleshoot, refer here: https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/dotnet-core-cli?view=azure-devops#troubleshooting
##[error]Dotnet command failed with non-zero exit code on the following projects : [ 'D:\\a\\1\\s\\AcmeProject.Tests\\AcmeProject.Tests.csproj' ]
Finishing: dotnet test
```

This error is specific to .NET 5, to fix the issue upgrade your SDK target to 6:
```yaml
- task: UseDotNet@2
  displayName: 'Install .NET SDK'
  inputs:
    version: '6.0.x'
```

## Mac Support

Xamarin.CommunityToolkit contains a rouge reference to a windows library that affects the test runner when performing tests on visual studio for mac.

You will find that the unit test project builds but the test runner UI picks up no tests and when executing the runner, it will hang and not execute any tests. Regardless of how many tests are in your project, the tests will not get picked up:

![Tests not getting picked up in the runner on vsmac](/img/posts/xamarin-test-pipelines/vsmac_missing_tests.png)

The following issues over on their github all are caused by this rogue reference:
- [![#985 - Unit Test projects now reference Xamarin.Forms.Platform.WPF](https://img.shields.io/github/issues/detail/state/xamarin/XamarinCommunityToolkit/985?style=plastic)](https://github.com/xamarin/XamarinCommunityToolkit/issues/985)
- [![#1167 - Unit Test Compile Error When Targeting .NET 5.0](https://img.shields.io/github/issues/detail/state/xamarin/XamarinCommunityToolkit/1167?style=plastic)](https://github.com/xamarin/XamarinCommunityToolkit/issues/1167)
- [![#1482 - Unable to run Unit Test after adding the XamarinCommunityToolkit package to a project](https://img.shields.io/github/issues/detail/state/xamarin/XamarinCommunityToolkit/1482?style=plastic)](https://github.com/xamarin/XamarinCommunityToolkit/issues/1482)

To fix this add the following lines to your csproj:
```xml
<PropertyGroup>
    <TargetFrameworks>netcoreapp3.1</TargetFrameworks>
    <GenerateErrorForMissingTargetingPacks>false</GenerateErrorForMissingTargetingPacks>
</PropertyGroup>
```

You can also optionally ignore the nuget warnings generated by the same issue with XCT:
```xml
<NoWarn>NU1701</NoWarn>
```

Your tests should now work in Visual Studio for Mac again! 😄

![Tests now getting picked up in the runner on vsmac](/img/posts/xamarin-test-pipelines/vsmac_showing_tests.png)