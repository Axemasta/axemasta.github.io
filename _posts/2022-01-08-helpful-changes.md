---
layout: post
title: "Xamarin iOS Api Differences"
subtitle: "Some useful code snippets comparing api's in swift and c#"
date: 2022-01-08
background: '/img/posts/06.jpg'
background-alt: "Preview Image For This Post"
category: "Xamarin"
tags: [ios,c#,swift]
---

# Useful Api Differences In Xamarin iOS

When working with the iOS api in Xamarin iOS you will sometimes encounter swift api's that have been renamed or moved in Xamarin. Whilst I was recently converting a swift app to c# I encountered some interesting changes that I thought might be useful to share.

## CGAffineTransform

//Covered in this blog

## CGRect Origin

// See this

```swift
let frame = CGRect(100, 340, 100, 100)

let originX = frame.origin.x
```



```csharp
var originX = frame.Location.X;
```

## [CALayer layerRectConverted](https://developer.apple.com/documentation/avfoundation/avcapturevideopreviewlayer/1623498-layerrectconverted)

Converts a rectangle in the coordinate system used for metadata outputs to one in the preview layer’s coordinate system.


Swift

```swift
let myLayer = CALayer()
let metadataRect = CGRect(x: 0, y: 0, width: 1, height: 1)

let mappedRect = myLayer.layerRectConverted(fromMetadataOutputRect: metadataRect)
```

C#

In Xamarin this api is called `MapToLayerCoordinates`:

```csharp
CALayer myLayer = new CALayer();
CGRect metadataRect new CGRect(0, 0, 1, 1);;

CGRect mappedRect = myLayer.MapToLayerCoordinates(metadataRect);
```

