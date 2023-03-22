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

# Xamarin in 2023

It's 2023 and Xamarin is still kicking. Maui is technically GA but any team with a few Xamarin apps under their belt would tell you that migration is not easy and Xamarin still has some life in it. The best thing about Xamarin at this point is that it is very stable, it has been stable for a few years but you seldom run into issues with the framework and if you do, the ecosystem is very mature and theres most likely a nuget package to solve your problem.

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
TODO
```

### .NET 5 Issues

I had alot of issues when targetting .NET 5, the `dotnet test` step would always fail with minimal information, even in debug mode!

The azure pipeline would display the following message:
TODO: IMAGE HERE

```
The error message here
```

Googling this error message didn't really pull back any results that helped me, it wasn't until I looked at how XCT was setting up their test pipelines that I updated to .NET 6 and the issues vanished.

## Mac Support

Xamarin.CommunityToolkit contains a rouge reference to a windows library that affects the test runner when performing tests on visual studio for mac.

The error is:

To fix this add the following lines to your csproj:
```xml
<PropertyGroup>
    <TargetFrameworks>netcoreapp3.1</TargetFrameworks>
    <TODO>true</TODO>
</PropertyGroup>
```

## Conclusion

TODO