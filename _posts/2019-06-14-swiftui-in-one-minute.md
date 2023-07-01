---
layout: post
title: "SwiftUI in one minute"
date: 2019-06-14
categories: swiftui
tags: swiftui
image:
    path: /assets/images/swiftui.png
---

[SwiftUI](https://developer.apple.com/xcode/swiftui/), a declarative UI programming framework introduced by Apple during WWDC 2019 aims to provide a shorter path to building great user interfaces across Apple platforms.

With SwiftUI you learn the concepts once and apply it everywhere across all platforms i.e. iOS, iPadOS, watchOS, macOS.

To start playing around with SwiftUI you will need to have Xcode 11 installed alongside macOS 10.15.

## The Basics

Views are defined declaratively, in other words, you tell SwiftUI what your UI should look like without having to worry about the "how it's done". Before SwiftUI's introduction we were using UIKit's imperative paradigm, the complete opposite of declarative where you need to tell the framework both what and how it should be done. 

Views are typically composed of several small **single-purpose** views that are extremely light-weight. What is also really great about SwiftUI is that there is little to no performance overhead having all these single-purpose views as the framework takes care of performance optimization etc.

## Creating Views

A view is nothing more than a struct conforming to the `View` protocol. The contents of your view go inside the `body` property:

```swift
struct AlbumView: View {
    var body: some View {
        Text("U2 - Joshua Tree")
    }
}
```

Let's break this down to understand what is going on here.

1. As mentioned before, `AlbumView` is a struct (view) conforming to the `View` protocol. Every view in SwiftUI **must** conform to the `View` protocol.
2. The return type of `body` property is `some View` (a new feature in Swift 5.1 called [opaque return types](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html)) meaning that `body` will return some type of View. This is helpful since SwiftUI doesn't know or care about the exact type of view it will return. 
3. The body of a view must always return only one child view.
4. `Text("U2 - Joshua Tree")` is another view containing a text label.

That in a nutshell is all there is to SwiftUI. There is obviously much more you can do with SwiftUI given all the primitive views and platform adaptivity to build rich user interfaces. SwiftUI also works great with the new [Combine](https://developer.apple.com/documentation/combine) and [Previews](https://developer.apple.com/documentation/swiftui/previews) frameworks, giving you an unparalleled app building experience. 

## Additional Resources

* [204 - Introducing SwiftUI: Building Your First App](https://developer.apple.com/videos/play/wwdc2019/204/)
* [Apple Developer Documentation](https://developer.apple.com/documentation?changes=latest_minor)
