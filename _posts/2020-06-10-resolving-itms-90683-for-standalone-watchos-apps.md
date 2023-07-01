---
layout: post
title: "Resolving ITMS-90683 for standalone watchOS apps"
date: 2020-06-10
categories: watchOS
tags: watchos xcode ITMS-90683
image:
    path: /assets/images/resolving-itms-90683.png
---

Early on during the development of my recent standalone watchOS app while adding HealthKit support I stumbled into a very annoying issue. Upon uploading a TestFlight build the App Store Connect upload API will fail with the following error: 

```text
ITMS-90683: Missing Purpose String in Info.plist
Your app’s code references one or more APIs that access sensitive user data.
The app’s Info.plist file should contain an NSHealthShareUsageDescription key
with a user-facing purpose string explaining clearly and completely why your app needs the data.
```

Naturally, I went back and made sure that I've added the `NSHealthShareUsageDescription` key in my Info.plist for my WatchKit Extension, and sure enough, it's there. It seems for standalone watchOS apps you need to take an additional step when requesting HealthKit permissions. I'm yet to find this being documented anywhere in the Apple Developer Documentation and I'm hoping that the solution provided below will save you some time in the process.

## So, what's the problem here?

Whenever you are requesting permissions like reading the user's HealthKit data you need to provide a reason for doing so. This is done by setting a string value for the `NSHealthShareUsageDescription` key in your Plist.info file.

For iOS apps there is typically one Info.plist file in the root of your Xcode project, you set the `NSHealthShareUsageDescription` providing your reason/purpose for requesting permission and you're off to the races. For watchOS apps it's a little bit different, you have two Info.plist files, one for you WatchKit App and one for your WatchKit Extension. The main Info.plist is the one you find under your WatchKit Extension.

Looking at the error description for `ITMS-90683` it's saying that hey, you haven't set the `NSHealthShareUsageDescription` key in your Info.plist.

## Here's how you resolve the issue

The biggest issue with the error description for `ITMS-90683` is that it doesn't tell you **which** Info.plist file it's referring to making it very misleading and troublesome to debug.

To resolve this, you simply need to manually add another Info.plist under the root level of your Xcode project as seen below:

![image](/assets/images/ITMS-90683-watchos-standalone-app-1.png)

And then in your Info.plist you need to set the `NSHealthShareUsageDescription` key for the reason/purpose for requesting the permission:

![image](/assets/images/ITMS-90683-watchos-standalone-app-2.png)

That's it, that will satisfy the App Store Connect upload API and you should be able to upload your App Store / TestFlight builds.

## Conclusion

It seems just like iOS apps you need to have an Info.plist file containing the HealthKit usages keys like `NSHealthShareUsageDescription` to be present at the project root level.

In the future, I hope Apple will update the Xcode template for standalone watchOS apps to include the additional Info.plist file in the project root or at the very least document this small, yet very important detail somewhere in the documentation.
