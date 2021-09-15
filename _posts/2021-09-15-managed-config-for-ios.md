---
layout: post
title: "Managed Config For iOS"
subtitle: "Deploy your iOS apps with custom configuration"
date: 2021-09-15
background: '/img/posts/06.jpg'
---

When your iOS apps are deployed out in the wild, they can be deployed with customer specific configuration.

For the average app store user, this isn't going to be too useful but for Business, School & Enterprise customers, they might wish to easily configure aspects of an app. These customers are likely using MDM solutions such as InTune, Jamf to push apps out to many devices across their organisation. In these circumstances setting configuration points such as email addresses, phone numbers might be vital to the success of the app. In my own experience developing the Mobile client for Senso, the app supports managed config deployment for license information. This allows technicans to automatically setup all iPads to point to the correct place. This article will show you how to read managed configuration on your device.

## Abstracting Configuration

This code will be platform specific and the process on Android is different. We need to define an interface for a service that will read out configuration from the device. For the purposes of this article, we are going to read the configuration from the device as a `Dictionary`:

```csharp
public interface IManagedConfigService
{
    Dictionary<string, object> ReadManagedConfig();
}
```

## Reading The Configuration

The configuration can be found in `NSUserDefaults` under the following key `com.apple.configuration.managed`:

```csharp
public NSDictionary GetManagedConfiguration()
{
    return NSUserDefaults
        .StandardUserDefaults
        .DictionaryForKey("com.apple.configuration.managed");
}
```

This will return an `NSDictionary` object which we will now convert from a native object to something that can be used in our shared code. If the app was not deployed with configuration, the dictionary will be `null`.

The following method will convert an `NSDictionary` to a `Dictionary<string, object>`:

```csharp
public Dictionary<string, object> ConvertToSystemDictionary(NSDictionary nsDictionary)
{
    throw new NotImplementedException();
}
```

Our whole method to extract the config will now look like this:

```csharp
Dictionary<string, object> ReadManagedConfig()
{
    var configDictionary = GetManagedConfiguration();

    if (configDictionary is null)
        return null;
    
    return ConvertToSystemDictionary(configDictionary);
}
```

## Testing Config

Not all of us have access to enterprise grade deployment tools at our fingertips...

Fortunately there is a tool called [TestMDM](https://www.testmdmapp.com/) which will allow you to test deploying your app with some managed configuration. its not free but there is a 7 day free trail available. I have personally used this to test my code, and the beauty of this is that once you've got the `NSDictionary`, everything else can be captured in a more traditional unit test, so you can confidently make updates to your config code without breaking it and needing to pay to retest.