---
layout: post
title: "iOS animations using Lottie"
date: 2019-03-16
categories: ios
tags: ios swift lottie animations
image:
    path: /assets/images/ios-animations-with-lottie.png
---

[Lottie](https://github.com/airbnb/lottie-ios) is a mobile library for iOS and Android that parses Adobe After Effects animations exported as JSON with [bodymovin](https://github.com/bodymovin/bodymovin) and renders the vector animations natively on mobile and through React Native!

Animations, especially complex animations can take a long time to implement by hand when using Core Animation or even just simple UIView animations. Using Lottie can save you a lot of time and manual effort. Using Lottie is easy, so let's get started!

## Setting up Lottie

The easiest way to install and use Lottie is by using [CocoaPods](https://cocoapods.org), but there are [other options](https://github.com/airbnb/lottie-ios#installing-lottie) too. We start by creating a Podfile in the same directory as our Xcode project and add the lottie-ios pod:

```shell
target 'LottieAnimations' do
  use_frameworks!
  pod 'lottie-ios'
end
```

With the `lottie-ios` pod added we simply need to do install the pod:

```shell
pod install
```

CocoaPods will generate a new Xcode workspace adding both your existing project and a new project called Pods. Remember to open the Xcode workspace and not the project.

## Working with LOTAnimationView

Animations in Lottie is exported to JSON from Adobe After Effect and can be loaded from your app's bundle or a URL. In the example below we are loading a file from the bundle called "loading.json":

```swift
let loadingAnimation = LOTAnimationView(name: "loading")
```

With Lottie animations, you have full control over when to start, stop, or whether you'd like to play the animation in a loop. In it's simplest form you can play the animation once:

```swift
@objc private func animateOnce() {
  loadingAnimation.play { (finished) in
    print("Animation completed!")
    }
  }
```

In some cases, you might want to do a little bit more than just playing the animation once and this is where Lottie shines, giving you full control over the animation. You can easily loop, start, stop, or just play a portion of the animation:

```swift
@objc private func loopAnimation() {
  loadingAnimation.loopAnimation = true

  loadingAnimation.play(fromProgress: 0,
                        toProgress: 0.76,
                        withCompletion: nil)
}
```

```swift
@objc private func stopAnimation() {
  loadingAnimation.stop()
}
```

The example Xcode project can be found on my GitHub [repository](https://github.com/rynaardb/ios-animations-using-lottie).

## Lottie Files

For inspiration check out all the animations shared by very talented motion designers on [LottieFiles](https://lottiefiles.com) the [animation](https://lottiefiles.com/4693-loading) used in my example was also sourced from LottieFiles.
